# ADR-005: Entrega at-least-once com `X-Event-Id`

- **Data:** 2026-10-15
- **Decisores:** Diego (Engenheiro Sênior, Plataforma), Larissa (Tech Lead), Bruno (Engenheiro Pleno, Pedidos), Sofia (Engenheira de Segurança), Marcos (Product Manager)
- **Relacionados:** [RFC](../RFC.md) · [FDD](../FDD.md) (contrato de saída, headers, resiliência) · [PRD](../PRD.md) · [ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md) · [ADR-003](./ADR-003-retry-com-backoff-exponencial-e-dlq.md) · [ADR-007](./ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md)

## Status

Aceito. Fechado em [09:26] Larissa ("At-least-once com X-Event-Id pra dedup do lado do cliente. Decisão.") e confirmado no resumo de [09:48] Larissa ("Idempotência por X-Event-Id, garantia at-least-once").

## Contexto

- O retry do [ADR-003](./ADR-003-retry-com-backoff-exponencial-e-dlq.md) e o replay da DLQ reenviam eventos. O Diego deixou o efeito explícito: "Pode acontecer de o cliente receber o mesmo evento duas vezes. Ele tem que estar preparado" ([09:24] Diego).
- Para se preparar, o cliente precisa reconhecer a repetição: "E como ele diferencia?" ([09:25] Bruno).
- Um mesmo cliente pode ter vários endpoints cadastrados e precisa saber qual cadastro recebeu cada envio ([09:44] Sofia).

## Decisão

**Garantir entrega at-least-once e identificar cada evento com um UUID no header `X-Event-Id`, gerado quando o evento entra na outbox, para que o cliente deduplique do lado dele.**

- Garantia at-least-once: o mesmo evento pode chegar mais de uma vez ([09:24] Diego).
- "A gente manda um event_id no header, X-Event-Id, com um UUID gerado quando o evento entra na outbox. É único por evento" ([09:25] Diego). O mesmo `event_id` também vai no corpo da entrega ([09:43] Diego; ver [ADR-007](./ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md)).
- Deduplicação no cliente: "Se o cliente recebeu duas vezes, ele dedupica pelo event_id do lado dele" ([09:25] Diego).
- O id não muda nas retentativas nem no replay da DLQ, porque é ele que permite ao cliente reconhecer a repetição ([09:25] Diego). Manter o mesmo id no replay é decisão de interpretação posterior à reunião, detalhada no [FDD](../FDD.md).
- Quando um evento vai para mais de um endpoint inscrito, todos recebem o mesmo `event_id`, e um endpoint nunca recebe dois eventos diferentes com o mesmo id. Também é decisão de interpretação, detalhada no [FDD](../FDD.md). O header `X-Webhook-Id` identifica qual cadastro recebeu a entrega ([09:44] Sofia, aceito em [09:45] Diego).
- O PM documenta a garantia "bem destacado no portal de desenvolvedor" ([09:26] Marcos).

## Alternativas Consideradas

### Alternativa 1: exactly-once (ALT-09, discutida)

- **Descrição:** garantir que cada evento seja processado uma única vez pelo cliente.
- **Prós:** o cliente não precisaria deduplicar, o que responde à objeção de [09:25] Sofia.
- **Contras:** "exigiria coordenação dos dois lados e fica muito mais complexo" ([09:25] Diego).
- **Motivo do descarte:** "At-least-once com event_id resolve 99% dos casos" ([09:25] Diego), fechado em [09:26] Larissa.

### Alternativa 2: at-most-once, sem reenvio (plausível, não discutida)

- **Descrição:** enviar cada evento uma única vez e nunca repetir.
- **Prós:** nunca duplica, e o cliente não precisa deduplicar.
- **Contras:** qualquer falha de rede, timeout ou indisponibilidade do cliente perde o evento.
- **Motivo do descarte:** contradiz o retry com backoff já decidido ([09:17] Larissa; [ADR-003](./ADR-003-retry-com-backoff-exponencial-e-dlq.md)) e o caso real de cliente com duas horas de manutenção planejada ([09:16] Diego).

## Consequências

### Positivas

- Simples e alinhado ao mercado: "Stripe faz assim, GitHub faz assim" ([09:25] Diego).
- Reenviar é sempre seguro do nosso lado. Retry, replay da DLQ e a recuperação depois de uma queda do worker ([ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md)) não precisam de coordenação com o cliente.
- `X-Event-Id` e `X-Webhook-Id` dão ao cliente o que ele precisa para correlacionar cada entrega com o evento e com o cadastro ([09:25] Diego, [09:44] Sofia).

### Negativas

- "Isso joga responsabilidade pro cliente" ([09:25] Sofia): cada cliente precisa guardar os `event_id` já processados e descartar as repetições.
- A integração depende de documentação clara no portal do desenvolvedor ([09:26] Marcos). Um cliente que não deduplicar vai processar o mesmo evento mais de uma vez.
- At-least-once não resolve ordem: um evento reenviado pode chegar depois de um evento mais novo do mesmo pedido ([ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md), [ADR-003](./ADR-003-retry-com-backoff-exponencial-e-dlq.md)).

### Trade-off explícito

> **CONS-1:** Aceitamos transferir para o cliente o trabalho de deduplicar pelo `X-Event-Id` em troca de uma entrega simples, alinhada ao mercado, que pode reenviar sem risco em retry, replay e recuperação de falhas.

## Referências

- Transcrição: [09:16] Diego, [09:17] Larissa, [09:24] Diego, [09:25] Bruno, [09:25] Diego, [09:25] Sofia, [09:26] Marcos, [09:26] Larissa, [09:43] Diego, [09:44] Sofia, [09:45] Diego, [09:48] Larissa.
