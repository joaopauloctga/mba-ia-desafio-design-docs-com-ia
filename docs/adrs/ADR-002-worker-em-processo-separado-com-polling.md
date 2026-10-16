# ADR-002: Worker em processo separado com polling de 2s

- **Data:** 2026-10-15
- **Decisores:** Larissa (Tech Lead), Diego (Engenheiro Sênior, Plataforma), Bruno (Engenheiro Pleno, Pedidos), Marcos (Product Manager)
- **Relacionados:** [RFC](../RFC.md) · [FDD](../FDD.md) (fluxo de processamento pelo worker, resiliência) · [PRD](../PRD.md) · [ADR-001](./ADR-001-outbox-transacional-no-mysql.md) · [ADR-003](./ADR-003-retry-com-backoff-exponencial-e-dlq.md) · [ADR-005](./ADR-005-entrega-at-least-once-com-x-event-id.md) · [ADR-006](./ADR-006-reuso-dos-padroes-existentes-do-projeto.md)

## Status

Aceito. O polling foi registrado como decisão em [09:10] Larissa ("Vamos registrar isso como uma decisão. Worker em polling, 2s"), o processo separado com entry-point nova foi fechado em [09:12] Larissa ("Anotado") e a limitação de ordering, em [09:13] Larissa. Tudo foi confirmado no resumo de [09:48] Larissa ("Worker separado em polling de 2 segundos").

## Contexto

- A outbox do [ADR-001](./ADR-001-outbox-transacional-no-mysql.md) precisa de um consumidor: "Um worker separado fica lendo essa tabela e disparando as chamadas HTTP" ([09:06] Diego).
- A meta de produto é notificar em menos de 10s ([09:02] Marcos).
- O MySQL não avisa processos externos: "MySQL não tem listener nativo tipo o NOTIFY/LISTEN do Postgres. Trigger no banco a gente até tem, mas ela não notifica processo externo" ([09:09] Diego).
- A aplicação tem hoje um único entry-point, `src/server.ts` (L6-27: `bootstrap`, `app.listen` e graceful shutdown em SIGINT/SIGTERM nas L13-21), iniciado pelos scripts `dev` e `start` (`package.json:11` e `package.json:13`).
- Se o worker rodasse dentro da API, um reinício da API levaria o worker junto ([09:11] Diego).
- A Larissa levantou a ordem de entrega: se um pedido passa por PAID, PROCESSING e SHIPPED em sequência rápida, o cliente recebe na ordem certa? ([09:12] Larissa). Os clientes nunca pediram ordem global, "eles só querem saber se cada pedido deles mudou" ([09:14] Marcos).

## Decisão

**Consumir a outbox com um processo Node separado da API, com entry-point própria, que a cada 2s busca os eventos pendentes mais antigos e os processa, rodando como instância única (single-worker).**

- Entry-point `src/worker.ts` **(novo, a criar)** e script `npm run worker`, no molde do `src/server.ts` ([09:11] Larissa; "Anotado" em [09:12] Larissa).
- A lógica de processamento fica dentro do módulo, em `src/modules/webhooks/webhook.worker.ts` **(novo, a criar)** ([09:28] Bruno, aceito em [09:28] Diego). O Bruno citou `src/modules/webhooks/webhook.processor.ts` **(novo, a criar)** como nome alternativo; o nome final fica no [FDD](../FDD.md).
- Polling em loop: "A cada 2 segundos, busca os eventos pendentes mais antigos, processa, marca" ([09:09] Diego). Registrado como decisão em [09:10] Larissa, com o aceite do PM: "2 segundos serve, perfeito" ([09:10] Marcos).
- Leitura só dos pendentes, em lote pequeno ([09:08] Diego). O tamanho do lote é parâmetro de implementação do FDD.
- Mesmo banco e mesma stack da API, mas "só não pode ser o mesmo processo" ([09:11] Diego). A conexão própria com o banco está no [ADR-006](./ADR-006-reuso-dos-padroes-existentes-do-projeto.md).
- Instância única, processando em ordem de criação na outbox. Com isso, o cliente recebe os eventos de cada pedido em ordem ([09:12] Diego). Fica registrado como limitação conhecida: "Não é garantia de ordering global, só por order_id e enquanto for single-worker" ([09:13] Larissa).
- Sobre a frase "A latência mínima vai ser 2 segundos no pior caso. Aceitamos." ([09:10] Larissa): "mínima" e "pior caso" se contradizem. A leitura adotada é que o polling acrescenta **até 2s** de espera no pior caso, sem mudar a meta de menos de 10s ([09:02] Marcos).

## Alternativas Consideradas

### Alternativa 1: trigger de banco ou notificação reativa (ALT-03, discutida)

- **Descrição:** usar uma trigger do MySQL para acordar o worker assim que um evento entra na outbox ([09:09] Bruno: "pra ser mais reativo").
- **Prós:** eliminaria a espera do polling.
- **Contras:** a trigger "só executa SQL"; para avisar o worker seria preciso "improvisar algo tipo escrever em arquivo ou bater num endpoint, fica esquisito" ([09:09] Diego).
- **Motivo do descarte:** o MySQL não tem LISTEN/NOTIFY, e "Polling de 2 segundos atende o requisito de 'abaixo de 10 segundos' tranquilo" ([09:09] Diego).

### Alternativa 2: worker dentro do processo da API (ALT-04, discutida)

- **Descrição:** rodar o loop de polling na mesma instância Node da API.
- **Prós:** um processo a menos para implantar e operar.
- **Contras:** os dois ciclos de vida ficam presos um ao outro.
- **Motivo do descarte:** "o worker tem que rodar como processo separado, não dentro da mesma instância da API. Senão se a API reinicia, perde o worker" ([09:11] Diego).

### Alternativa 3: vários workers em paralelo (discutida, adiada)

- **Descrição:** escalar horizontalmente com mais de uma instância do worker.
- **Prós:** mais vazão de envio.
- **Contras:** "Se a gente escala pra múltiplos workers em paralelo no futuro, perde a garantia" de ordem ([09:12] Diego).
- **Motivo do descarte:** adiada. As saídas citadas (particionar por `order_id` ou usar lock pessimista) são "problema do futuro, não agora" ([09:13] Diego). Fica como questão em aberto no [RFC](../RFC.md).

## Consequências

### Positivas

- A API pode reiniciar ou ser reimplantada sem derrubar o worker ([09:11] Diego).
- Polling de 2s atende com folga a meta de menos de 10s ([09:09] Diego, [09:10] Marcos).
- Nenhuma infraestrutura nova nem stack diferente: mesmo banco, mesma linguagem, mesmo projeto ([09:11] Diego).
- Com um único worker lendo em ordem de criação, os eventos de um mesmo pedido saem na ordem em que aconteceram ([09:12] Diego).

### Negativas

- Espera adicional de até 2s no pior caso, aceita explicitamente ([09:10] Larissa).
- O worker consulta o banco a cada 2s, mesmo quando não há nada pendente.
- Não escala horizontalmente sem redesenho: um segundo worker quebra a ordem por pedido ([09:12] Diego), e a solução ficou para o futuro ([09:13] Diego).
- Mesmo com um único worker, a ordem por pedido vale para o primeiro envio. Se um evento entra em backoff ([ADR-003](./ADR-003-retry-com-backoff-exponencial-e-dlq.md)), o evento seguinte do mesmo pedido pode chegar antes. É um ponto para os revisores confirmarem no [RFC](../RFC.md); como os clientes só querem saber se cada pedido mudou ([09:14] Marcos), podem reconciliar consultando `GET /orders/:id` ([09:43] Diego).
- Mais um processo para implantar, operar e monitorar. Se ele parar, os eventos se acumulam na outbox sem entrega.
- Se o worker cair no meio de um envio, o evento pode ser reenviado quando ele voltar. Isso é coberto pela garantia at-least-once ([ADR-005](./ADR-005-entrega-at-least-once-com-x-event-id.md)); a recuperação está no [FDD](../FDD.md).

### Trade-off explícito

> **CONS-1:** Aceitamos até 2s de espera adicional, consultas periódicas ao banco e um único worker (sem escala horizontal e com ordem garantida só por pedido) em troca de uma solução simples, sem infraestrutura nova e isolada do ciclo de vida da API.

## Referências

- Transcrição: [09:02] Marcos, [09:06] Diego, [09:08] Diego, [09:09] Diego, [09:09] Bruno, [09:10] Marcos, [09:10] Larissa, [09:11] Diego, [09:11] Larissa, [09:12] Larissa, [09:12] Diego, [09:13] Diego, [09:13] Larissa, [09:14] Marcos, [09:28] Bruno, [09:28] Diego, [09:43] Diego, [09:48] Larissa.
- Código: `src/server.ts` (L6-27, entry-point atual; L13-21, shutdown), `package.json` (L11 e L13, scripts `dev` e `start`).
- Arquivos novos citados: `src/worker.ts` **(novo, a criar)**, `src/modules/webhooks/webhook.worker.ts` **(novo, a criar)**.
