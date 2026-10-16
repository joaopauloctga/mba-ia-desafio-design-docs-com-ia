# ADR-001: Outbox transacional no MySQL

- **Data:** 2026-10-15
- **Decisores:** Larissa (Tech Lead), Diego (Engenheiro Sênior, Plataforma), Bruno (Engenheiro Pleno, Pedidos)
- **Relacionados:** [RFC](../RFC.md) · [FDD](../FDD.md) (fluxo de criação do evento e modelo de dados) · [PRD](../PRD.md) · [ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md) · [ADR-006](./ADR-006-reuso-dos-padroes-existentes-do-projeto.md) · [ADR-007](./ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md)

## Status

Aceito. Fechado em [09:08] Larissa ("Tá decidido então: outbox em MySQL") e confirmado no resumo de [09:48] Larissa ("Padrão outbox no MySQL, transação atômica com mudança de status").

## Contexto

- Três clientes B2B pediram para ser notificados quando o status dos pedidos muda. Hoje eles consultam `GET /orders` de tempos em tempos, o que deixa a integração "lenta e cara" ([09:00] Marcos). Para eles, menos de 10s já é "tempo real" ([09:02] Marcos).
- A notificação nasce na mudança de status, feita em `OrderService.changeStatus` (`src/modules/orders/order.service.ts:126-179`). O método já roda uma transação (`this.prisma.$transaction`, L131) que lê o pedido, debita ou repõe estoque quando a transição exige (L151-156), atualiza `orders` (L158) e grava o histórico em `order_status_history` (L159-167). O Bruno disse que essa transação "já é pesada" ([09:04] Bruno).
- O código não tem mecanismo de eventos, filas ou webhooks: os módulos de `src/modules/` são apenas `auth`, `customers`, `orders`, `products` e `users`.
- A pergunta que abriu o tema foi se o disparo seria síncrono no service de orders ou por "algum tipo de fila/outbox" ([09:03] Larissa).
- Duas forças limitam a escolha:
  - a mudança de status não pode ficar mais lenta nem falhar por causa de um cliente lento ou fora do ar ([09:04] Bruno);
  - o time é pequeno e não quer operar infraestrutura nova ([09:07] Diego).

## Decisão

**Registrar cada evento de mudança de status numa tabela `webhook_outbox` do MySQL existente, dentro da mesma transação do `changeStatus`. Se a inserção na outbox falhar, a transação inteira sofre rollback e o status não muda.**

- Padrão outbox, e não "fila": o evento é inserido na mesma transação SQL que atualiza `orders` e `order_status_history`, e um worker separado lê a tabela e faz as chamadas HTTP ([09:06] Diego). O worker é tratado no [ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md).
- Garantia buscada: se a transação commitou, o evento foi registrado; se deu rollback, o evento some junto ([09:06] Diego).
- Banco: o MySQL que a aplicação já usa ([09:07] Diego; `docker-compose.yml:3`, imagem `mysql:8.0`).
- Inserção dentro da transação, com rollback se ela falhar: "Não pode ter caso de status mudar e evento não sair" ([09:40] Bruno). "Se ficar fora da transação, perde a garantia toda" ([09:41] Diego).
- A tabela é indexada por status (pendente, processando, falhou, entregue) e por data de criação, e o worker lê só os pendentes, em lote pequeno ([09:08] Diego). Colunas, tipos e índices ficam no [FDD](../FDD.md).
- O ponto de chamada dentro do `changeStatus` (função que recebe o client da transação) está no [ADR-006](./ADR-006-reuso-dos-padroes-existentes-do-projeto.md). O conteúdo do evento e o filtro por endpoint estão no [ADR-007](./ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md).
- O arquivamento das linhas entregues ("depois de 30 dias ou assim") fica fora do escopo desta feature ([09:08] Diego).

## Alternativas Consideradas

### Alternativa 1: disparo HTTP síncrono dentro do `changeStatus` (ALT-01, discutida)

- **Descrição:** chamar o endpoint do cliente no meio da transação de mudança de status.
- **Prós:** nenhuma tabela nova nem processo extra; a notificação sairia no mesmo instante.
- **Contras:** a transação já é pesada, e "qualquer cliente lento vai travar mudança de status pra outros pedidos" ([09:04] Bruno). Com o cliente fora do ar, não há resposta aceitável: "dá rollback na mudança de status? Não dá" ([09:04] Bruno).
- **Motivo do descarte:** "Síncrono está fora de questão" ([09:06] Diego), com a concordância da Larissa ([09:04] Larissa).

### Alternativa 2: Redis Streams ou outra fila externa (ALT-02, discutida)

- **Descrição:** publicar o evento num broker dedicado em vez de uma tabela.
- **Prós:** mecanismo feito para filas, com consumo reativo.
- **Contras:** "a gente acabaria precisando subir mais infra" ([09:07] Larissa).
- **Motivo do descarte:** "a gente é um time pequeno. Subir Redis Cluster pra isso é overengineering. Outbox no MySQL existente resolve" ([09:07] Diego).

### Alternativa 3: gravar o evento depois do commit, fora da transação (plausível, não discutida)

- **Descrição:** concluir o `changeStatus` e só então registrar ou disparar o evento (por exemplo, numa chamada assíncrona em memória após o `$transaction`).
- **Prós:** a transação crítica não ganha nenhuma escrita a mais.
- **Contras:** um crash entre o commit e o registro perde o evento sem rastro.
- **Motivo do descarte:** ninguém propôs esse caminho, mas a reunião o descarta de antemão: "Se ficar fora da transação, perde a garantia toda" ([09:41] Diego), e a garantia exigida é que status commitado implique evento registrado ([09:06] Diego).

## Consequências

### Positivas

- Atomicidade entre a mudança de status e o registro do evento. Não existe status alterado sem evento, nem evento de uma mudança que sofreu rollback ([09:06] Diego).
- Nenhuma infraestrutura nova: a outbox mora no MySQL 8.0 que já roda (`docker-compose.yml:3`).
- O `changeStatus` não faz chamada HTTP. Um cliente lento ou fora do ar não atrasa nem bloqueia mudanças de status ([09:04] Bruno).
- O contrato de `PATCH /orders/:id/status` não muda (`src/modules/orders/order.routes.ts:19-23`); só a transação interna ganha uma escrita.

### Negativas

- A transação de mudança de status, que já é pesada ([09:04] Bruno), ganha mais uma escrita. Uma falha nessa inserção derruba a mudança de status por decisão explícita ([09:40] Bruno).
- A outbox cresce a cada mudança de status com inscritos. O Bruno levantou o risco de o worker ficar lento com o acúmulo ([09:07] Bruno); os índices mitigam, mas o arquivamento ficou fora desta feature ([09:08] Diego). Sem uma política de retenção, a tabela só cresce (questão em aberto no [RFC](../RFC.md)).
- A latência passa a depender de quem lê a tabela. Com polling, há uma espera adicional de até 2s ([ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md)).
- Surge um consumidor obrigatório (o worker). Se ele parar, os eventos ficam acumulados na outbox, sem perda, mas sem entrega.

### Trade-off explícito

> **CONS-1:** Aceitamos acoplar a mudança de status ao sucesso da inserção na outbox, e conviver com uma tabela que cresce sem arquivamento nesta feature, em troca de consistência garantida entre status e evento sem subir nenhuma infraestrutura nova.

## Referências

- Transcrição: [09:00] Marcos, [09:02] Marcos, [09:03] Larissa, [09:04] Bruno, [09:04] Larissa, [09:06] Diego, [09:07] Larissa, [09:07] Diego, [09:07] Bruno, [09:08] Diego, [09:08] Larissa, [09:40] Bruno, [09:41] Diego, [09:48] Larissa.
- Código: `src/modules/orders/order.service.ts` (L126-179, `changeStatus`; L131, `$transaction`; L151-156, estoque; L158-167, update e histórico), `src/modules/orders/order.routes.ts` (L19-23), `docker-compose.yml` (L3).
