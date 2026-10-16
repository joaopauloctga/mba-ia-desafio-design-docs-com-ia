# Architectural Decision Records

Este diretório guarda os ADRs do Sistema de Webhooks de Notificação de Pedidos. Cada ADR registra uma decisão arquitetural fechada na reunião técnica ([TRANSCRICAO.md](../../TRANSCRICAO.md)), com o contexto, as alternativas e as consequências. A proposta como um todo está no [RFC](../RFC.md), e o detalhe de implementação, no [FDD](../FDD.md).

## Convenção

- Nome do arquivo: `ADR-NNN-titulo-em-kebab-case.md`, com numeração sequencial de três dígitos.
- Um arquivo por decisão. Um ADR aceito não é reescrito: se a decisão mudar, um novo ADR o substitui e o antigo passa a "Substituído por ADR-NNN".
- Toda afirmação de Contexto, Decisão e Alternativas cita a fala de origem (`[hh:mm] Nome`) ou um caminho real do código. Arquivos que ainda não existem levam a marca **(novo, a criar)**.
- Alternativas que não foram discutidas na reunião aparecem marcadas como "plausível, não discutida".
- Rastreabilidade no [tracker](../TRACKER.md): `ADR-00N` é a decisão, `ADR-00N-ALT-n` é a alternativa n da seção "Alternativas Consideradas", `ADR-00N-CONS-1` é o trade-off explícito e `ADR-006a` a `ADR-006i` são os padrões reaproveitados listados no ADR-006. Os rótulos `ALT-NN` entre parênteses são os IDs globais de alternativas, os mesmos usados no RFC.

## Template (MADR adaptado)

```markdown
# ADR-NNN: <título da decisão>

- **Data:** AAAA-MM-DD
- **Decisores:** só quem participou daquela decisão
- **Relacionados:** links para RFC, FDD e outros ADRs

## Status
Proposto | Aceito | Substituído por ADR-NNN, com a fala de fechamento

## Contexto
Problema e forças em jogo, com [hh:mm] Nome e caminhos de arquivo

## Decisão
Uma frase em negrito e o detalhamento curto

## Alternativas Consideradas
### Alternativa n: <nome> (ALT-NN, discutida | plausível, não discutida)
Descrição, prós, contras e motivo do descarte com a fala de origem

## Consequências
### Positivas
### Negativas
### Trade-off explícito

## Referências
Falas e arquivos citados
```

## Índice

| Nº | Título | Status | Decisão principal coberta |
| --- | --- | --- | --- |
| [ADR-001](./ADR-001-outbox-transacional-no-mysql.md) | Outbox transacional no MySQL | Aceito | Padrão Outbox no MySQL, na mesma transação do `changeStatus` |
| [ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md) | Worker em processo separado com polling de 2s | Aceito | Worker em processo separado em polling, single-worker |
| [ADR-003](./ADR-003-retry-com-backoff-exponencial-e-dlq.md) | Retry com backoff exponencial e DLQ em tabela separada | Aceito | Retry 1m/5m/30m/2h/12h (janela de 14h36m), DLQ e replay por ADMIN |
| [ADR-004](./ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md) | Assinatura HMAC-SHA256 com secret por endpoint | Aceito | HMAC-SHA256 com secret por endpoint e rotação com 24h de validade da antiga |
| [ADR-005](./ADR-005-entrega-at-least-once-com-x-event-id.md) | Entrega at-least-once com `X-Event-Id` | Aceito | Garantia at-least-once com `X-Event-Id` para dedup no cliente |
| [ADR-006](./ADR-006-reuso-dos-padroes-existentes-do-projeto.md) | Reuso dos padrões existentes do projeto | Aceito | Reuso de módulos, `AppError`, error middleware, Pino, Zod, `requireRole` e UUID |
| [ADR-007](./ADR-007-payload-snapshot-enxuto-com-filtro-na-insercao.md) | Payload snapshot enxuto, renderizado e filtrado na inserção | Aceito | Decisão secundária: formato do payload, snapshot e filtro na inserção |

Os seis primeiros cobrem as seis decisões principais da reunião. O ADR-007 registra uma decisão secundária que afeta a outbox e o contrato com o cliente.
