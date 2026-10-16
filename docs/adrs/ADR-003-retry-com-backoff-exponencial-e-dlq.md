# ADR-003: Retry com backoff exponencial e DLQ em tabela separada

- **Data:** 2026-10-15
- **Decisores:** Larissa (Tech Lead), Diego (Engenheiro Sênior, Plataforma), Bruno (Engenheiro Pleno, Pedidos), Marcos (Product Manager)
- **Consultada:** Sofia (Engenheira de Segurança), só na role ADMIN e na auditoria do replay ([09:36] Sofia) e no timeout de 10s ([09:42] Sofia)
- **Relacionados:** [RFC](../RFC.md) · [FDD](../FDD.md) (fluxos de retry e DLQ, estratégias de resiliência) · [PRD](../PRD.md) · [ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md) · [ADR-005](./ADR-005-entrega-at-least-once-com-x-event-id.md) · [ADR-006](./ADR-006-reuso-dos-padroes-existentes-do-projeto.md)

## Status

Aceito. A progressão foi fechada em [09:17] Larissa ("Decidido: 5 tentativas, backoff 1m/5m/30m/2h/12h"), a DLQ com replay em [09:19] Larissa ("Anota também") e a role ADMIN no replay em [09:36] Larissa ("Decidido"). Tudo foi confirmado no resumo de [09:48] Larissa. A contagem "1 envio inicial + 5 retentativas" é a interpretação adotada e segue para confirmação dos revisores no [RFC](../RFC.md).

## Contexto

- O worker do [ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md) vai encontrar clientes fora do ar: "Se o cliente tá offline, o que a gente faz?" ([09:14] Larissa).
- Indisponibilidades longas acontecem: "Já tinha cliente nosso com indisponibilidade de duas horas em manutenção planejada" ([09:16] Diego).
- Retentar para sempre também é problema: o evento pode "ficar pendurado pra sempre se o cliente sumiu" ([09:15] Diego).
- A expectativa de cobertura dita na reunião era "uma janela de até 12 ou 24 horas" ([09:15] Diego), e o PM aceitou que um cliente fora por 15 horas "já tá com problema sério dele" ([09:17] Marcos).
- A Sofia levantou o tempo máximo de espera da chamada HTTP do worker: "Timeout do HTTP call do worker. Quanto?" ([09:42] Sofia).

## Decisão

**Retentar as entregas com falha com backoff exponencial (1 envio inicial e até 5 retentativas, com esperas de 1m, 5m, 30m, 2h e 12h) e, esgotadas as tentativas, mover o evento para uma tabela separada de dead letter, `webhook_dead_letter`, de onde ele só sai por replay manual de um ADMIN.**

- Backoff exponencial com teto de tentativas; esgotado o teto, a falha é permanente e o evento vai para a DLQ ([09:15] Diego).
- Progressão 1m/5m/30m/2h/12h ([09:17] Diego), fechada em [09:17] Larissa.
- **Contagem adotada: 1 envio inicial + 5 retentativas**, ou seja, no máximo 6 envios de cada evento para cada endpoint. A conta da janela entre a primeira falha e a última tentativa é:

  > 1m + 5m + 30m + 2h + 12h = 1 + 5 + 30 + 120 + 720 = 876 min = **14h36m**

  É a única leitura que usa os cinco intervalos e bate com "Total de quase 15 horas entre primeira falha e última tentativa" ([09:17] Diego). O resumo final diz "total 5 tentativas" ([09:48] Larissa). Se isso fosse lido como cinco envios no total, sobrariam só 4 esperas: a de 12h nunca seria usada e a janela cairia para 2h36m, o que contradiz as quase 15 horas. Por isso o "5 tentativas" de [09:17] Larissa é lido como 5 retentativas, e o ponto vai para confirmação no [RFC](../RFC.md).
- Timeout de 10s por chamada: "Cliente lento que não responde em 10s a gente trata como falha e marca pra retry" ([09:42] Diego), com o aceite em [09:42] Sofia. O critério completo de sucesso e falha da resposta fica no [FDD](../FDD.md).
- DLQ em tabela separada, `webhook_dead_letter`, guardando o payload, o motivo da falha e o timestamp ([09:18] Diego).
- Replay manual por um endpoint admin, `POST /admin/webhooks/dead-letter/:id/replay`, que "Recoloca na outbox como pendente" ([09:18] Diego; rota confirmada em [09:35] Diego).
- O replay exige role ADMIN, porque "Mexer em fila de entrega de notificação não é coisa de operador", e registra em log quem fez, para auditoria ([09:36] Sofia). A Larissa fechou reaproveitando "o requireRole que já existe" ([09:36] Larissa); o reuso está no [ADR-006](./ADR-006-reuso-dos-padroes-existentes-do-projeto.md).
- Erro determinístico não passa pelo retry. Um payload acima de 64KB, que a reunião decidiu tratar como erro ([09:24] Larissa), vai direto para a DLQ, porque reenviar daria o mesmo resultado. Essa regra é decisão de interpretação posterior à reunião (o limite nunca bloqueia a mudança de status) e está detalhada no [FDD](../FDD.md).

## Alternativas Consideradas

### Alternativa 1: retry indefinido com backoff (ALT-05, discutida)

- **Descrição:** retentar sem teto, só aumentando o intervalo.
- **Prós:** nenhum evento seria abandonado, por mais longa que fosse a indisponibilidade.
- **Contras:** "isso traz o problema de evento ficar pendurado pra sempre se o cliente sumiu" ([09:15] Diego).
- **Motivo do descarte:** o Diego sugeriu um teto de tentativas no lugar ([09:15] Diego), fechado em [09:17] Larissa.

### Alternativa 2: 3 tentativas (ALT-06, discutida)

- **Descrição:** teto menor, "mais agressivo" ([09:16] Bruno).
- **Prós:** a falha permanente seria detectada mais cedo.
- **Contras:** "a gente retentaria três vezes em 30 minutos e mataria", o que não cobre a manutenção planejada de duas horas que um cliente já teve ([09:16] Diego).
- **Motivo do descarte:** "3 é pouco" ([09:16] Diego); "Cinco fica bom" ([09:16] Larissa).

### Alternativa 3: DLQ como status "failed" na própria outbox (ALT-07, discutida)

- **Descrição:** marcar o evento como falho na própria `webhook_outbox`, sem tabela separada ([09:17] Larissa levantou as duas opções).
- **Prós:** uma tabela a menos.
- **Contras:** falhas permanentes ficariam misturadas com os eventos que o worker ainda processa.
- **Motivo do descarte:** a tabela separada deixa "Mais limpa a leitura da outbox principal, e fica como evidence pra debug e reprocessamento" ([09:18] Diego).

## Consequências

### Positivas

- Cobre indisponibilidades de até 14h36m, o que inclui com folga a manutenção planejada de duas horas citada em [09:16] Diego.
- Nada fica pendurado para sempre: todo evento termina entregue ou na DLQ ([09:15] Diego).
- A outbox continua enxuta para o worker, e a DLQ concentra a evidência para debug e reprocessamento ([09:18] Diego).
- O reprocessamento é restrito a ADMIN e auditado ([09:36] Sofia).

### Negativas

- Um cliente fora do ar por mais de 14h36m perde a entrega automática; os eventos só voltam por replay manual. Foi aceito pelo PM: "Se um cliente meu cair por 15 horas, ele já tá com problema sério dele" ([09:17] Marcos).
- O replay é manual, evento a evento, pelo id da DLQ. Como o ADMIN descobre esse id não foi discutido; fica como ponto para os revisores no [RFC](../RFC.md).
- O cliente não é avisado de que está falhando: o email depois de falhas seguidas ficou para a próxima fase ([09:37] Larissa).
- Um evento em backoff pode ser ultrapassado por um evento mais novo do mesmo pedido, o que limita a ordem por pedido descrita no [ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md).
- Cada retentativa e cada replay podem entregar um evento que o cliente já processou (por exemplo, quando a resposta se perde depois do timeout); a deduplicação fica com o cliente ([ADR-005](./ADR-005-entrega-at-least-once-com-x-event-id.md)).
- Se os revisores decidirem que "5 tentativas" são 5 envios no total, a janela cai para 2h36m e este ADR precisa ser revisto.

### Trade-off explícito

> **CONS-1:** Aceitamos que um cliente fora do ar por mais de 14h36m dependa de replay manual de um ADMIN, sem aviso automático, em troca de uma janela de retry finita e previsível, sem eventos pendurados para sempre e com a DLQ como evidência separada da outbox.

## Referências

- Transcrição: [09:14] Larissa, [09:15] Diego, [09:16] Bruno, [09:16] Diego, [09:16] Larissa, [09:17] Diego, [09:17] Marcos, [09:17] Larissa, [09:18] Diego, [09:19] Larissa, [09:24] Larissa, [09:35] Diego, [09:36] Sofia, [09:36] Larissa, [09:37] Larissa, [09:42] Sofia, [09:42] Diego, [09:48] Larissa.
- Código: `src/middlewares/auth.middleware.ts` (L49-61, `requireRole`, reaproveitado no replay; ver [ADR-006](./ADR-006-reuso-dos-padroes-existentes-do-projeto.md)).
