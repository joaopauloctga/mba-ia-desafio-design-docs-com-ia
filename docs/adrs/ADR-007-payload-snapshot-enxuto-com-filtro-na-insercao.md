# ADR-007: Payload snapshot enxuto, renderizado e filtrado na inserção

- **Data:** 2026-10-15
- **Decisores:** Larissa (Tech Lead), Diego (Engenheiro Sênior, Plataforma), Bruno (Engenheiro Pleno, Pedidos), Marcos (Product Manager, só no filtro e no formato do payload; saiu às [09:50], antes da decisão do snapshot)
- **Relacionados:** [RFC](../RFC.md) · [FDD](../FDD.md) (contrato de saída com o JSON de exemplo, fluxo de criação do evento) · [PRD](../PRD.md) · [ADR-001](./ADR-001-outbox-transacional-no-mysql.md) · [ADR-004](./ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md) · [ADR-005](./ADR-005-entrega-at-least-once-com-x-event-id.md)

## Status

Aceito. Decisão secundária, que o enunciado permite registrar como ADR adicional. O snapshot foi fechado em [09:52] Bruno ("Beleza, snapshot. Decidido."), depois de [09:52] Larissa e [09:52] Diego. O filtro na inserção foi aceito em [09:34] Diego ("Concordo") e o formato enxuto, em [09:44] Bruno ("Bom, mantém payload enxuto").

## Contexto

- Cada endpoint escolhe quais mudanças quer receber: "Filtro de eventos é uma lista dos status que o webhook quer ouvir. Tipo 'só quero saber quando vira SHIPPED e DELIVERED', e a gente filtra na hora de inserir na outbox" ([09:33] Marcos). O Diego perguntou se o filtro seria aplicado na inserção ou no envio ([09:34] Diego).
- O formato do payload foi pedido pelo PM ([09:43] Marcos).
- Depois do encerramento, o Bruno levantou a dúvida: "o evento da outbox guarda o payload renderizado já, ou guarda só order_id e renderiza na hora do envio?" ([09:51] Bruno).
- No código, o `changeStatus` lê o pedido antes de atualizá-lo (`src/modules/orders/order.service.ts:132-135`) e já tem `from` e `to` (L138-139). O model `Order` traz `orderNumber`, `customerId` e `totalCents` (`prisma/schema.prisma:74-97`).
- O limite de payload é 64KB, com erro se ultrapassar ([09:24] Larissa), e "Nenhum evento nosso vai chegar perto disso" ([09:24] Diego).

## Decisão

**Renderizar o payload do evento no momento da inserção na outbox (snapshot), com um conjunto enxuto de campos e sem os items do pedido, e aplicar o filtro de status de cada endpoint também na inserção: se nenhum endpoint ativo quer aquele status, nada é inserido.**

- Snapshot na inserção: "Eu prefiro renderizado já, na hora da inserção. Se o pedido mudar depois, o evento ainda reflete o estado de quando o status mudou" ([09:52] Larissa); "Concordo, snapshot na inserção" ([09:52] Diego); "Decidido" ([09:52] Bruno).
- Campos do payload, em snake_case como foram falados ([09:43] Diego):
  - `event_id`
  - `event_type`, com o valor `order.status_changed`
  - `timestamp`, em ISO 8601
  - `order_id`
  - `order_number`
  - `from_status`
  - `to_status`
  - `customer_id`
  - campos básicos do pedido, como `total_cents`
- Sem items: "Não manda items pra não inflar. Se o cliente quiser detalhes, ele bate no GET /orders/:id depois" ([09:43] Diego).
- O `timestamp` do corpo é o momento da mudança de status, gravado no snapshot. O `X-Timestamp` é o "timestamp do envio" ([09:44] Diego) e muda a cada tentativa. O Diego só definiu o `X-Timestamp`; essa separação é interpretação adotada depois da reunião, porque com o payload renderizado na inserção o `timestamp` do corpo só pode ser o da mudança.
- Filtro na inserção: "Se nenhum webhook do customer quer aquele status, nem insere. Economiza linha na tabela" ([09:34] Bruno). Um webhook desativado conta como "não quer": a configuração guarda o "estado ativo" ([09:21] Bruno), e tratar o inativo assim é interpretação adotada depois da reunião, detalhada no [FDD](../FDD.md).
- Só a mudança de status gera evento, porque o ponto de integração decidido é o `changeStatus` ([09:40] Bruno). A criação do pedido, que grava o histórico inicial em `PENDING` (`src/modules/orders/order.service.ts:106-113`), não emite evento nesta fase, e o filtro aceita apenas status que podem ser destino de uma transição (`src/modules/orders/order.status.ts:3-10`). Essa exclusão é interpretação adotada depois da reunião e está no [FDD](../FDD.md).
- O JSON de exemplo completo e o mapeamento de cada campo ficam no [FDD](../FDD.md).

## Alternativas Consideradas

### Alternativa 1: guardar só o `order_id` e renderizar na hora do envio (ALT-16, discutida)

- **Descrição:** a outbox guarda a referência do pedido, e o worker monta o payload ao enviar ([09:51] Bruno levantou a opção).
- **Prós:** linhas menores na outbox.
- **Contras:** se o pedido mudar entre a inserção e o envio, inclusive durante uma janela de retry de até 14h36m ([ADR-003](./ADR-003-retry-com-backoff-exponencial-e-dlq.md)), o evento descreveria um estado diferente do momento da mudança.
- **Motivo do descarte:** "Senão tem caso esquisito" ([09:52] Larissa).

### Alternativa 2: payload com os items do pedido (ALT-17, discutida)

- **Descrição:** incluir a lista de items em cada evento.
- **Prós:** o cliente teria todos os dados numa única entrega.
- **Contras:** infla o payload a cada mudança de status.
- **Motivo do descarte:** "Não manda items pra não inflar" ([09:43] Diego), com o aceite em [09:44] Bruno.

### Alternativa 3: filtrar na hora do envio (ALT-11, discutida)

- **Descrição:** inserir todo evento na outbox e decidir no worker quem recebe ([09:34] Diego levantou a opção).
- **Prós:** uma mudança no filtro valeria também para eventos já registrados e ainda não enviados.
- **Contras:** a outbox ganharia linhas que ninguém vai receber.
- **Motivo do descarte:** filtrar na inserção "Economiza linha na tabela" ([09:34] Bruno); "Concordo" ([09:34] Diego).

## Consequências

### Positivas

- O evento reflete o estado do pedido no momento da mudança, mesmo que o envio aconteça horas depois por causa do retry ([09:52] Larissa).
- A outbox recebe menos linhas: status sem inscritos não geram registro ([09:34] Bruno).
- O payload fica pequeno, bem abaixo do limite de 64KB ([09:24] Diego).
- O worker não precisa ler o pedido na hora do envio: o evento já está pronto para assinar ([ADR-004](./ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md)) e enviar.

### Negativas

- Uma mudança no filtro, a ativação de um webhook ou um cadastro novo só valem para eventos futuros. Mudanças de status que não geraram linha não podem ser recuperadas depois.
- O snapshot repete dados do pedido em cada linha, o que soma ao crescimento da outbox sem arquivamento nesta feature ([ADR-001](./ADR-001-outbox-transacional-no-mysql.md)).
- O cliente precisa chamar `GET /orders/:id` para ver os items ([09:43] Diego).
- A consulta dos endpoints inscritos do customer entra na transação do `changeStatus`, que já é pesada ([09:04] Bruno): é mais uma leitura no caminho crítico.

### Trade-off explícito

> **CONS-1:** Aceitamos dados duplicados na outbox, uma leitura a mais na transação de status e um filtro que só vale para eventos futuros, em troca de eventos fiéis ao momento da mudança, de uma outbox menor e de um payload enxuto.

## Referências

- Transcrição: [09:04] Bruno, [09:21] Bruno, [09:24] Larissa, [09:24] Diego, [09:33] Marcos, [09:34] Diego, [09:34] Bruno, [09:40] Bruno, [09:43] Marcos, [09:43] Diego, [09:44] Bruno, [09:44] Diego, [09:51] Bruno, [09:52] Larissa, [09:52] Diego, [09:52] Bruno.
- Código: `src/modules/orders/order.service.ts` (L132-135, L138-139, L106-113), `src/modules/orders/order.status.ts` (L3-10), `prisma/schema.prisma` (L74-97, model `Order`).
