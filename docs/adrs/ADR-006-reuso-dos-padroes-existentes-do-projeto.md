# ADR-006: Reuso dos padrões existentes do projeto

- **Data:** 2026-10-15
- **Decisores:** Larissa (Tech Lead), Bruno (Engenheiro Pleno, Pedidos), Diego (Engenheiro Sênior, Plataforma)
- **Consultada:** Sofia (Engenheira de Segurança), só na role ADMIN do replay ([09:36] Sofia) e na autorização do CRUD ([09:37] Sofia)
- **Relacionados:** [RFC](../RFC.md) · [FDD](../FDD.md) (integração com o sistema existente, arquivos novos, matriz de erros) · [PRD](../PRD.md) · [ADR-001](./ADR-001-outbox-transacional-no-mysql.md) · [ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md) · [ADR-003](./ADR-003-retry-com-backoff-exponencial-e-dlq.md)

## Status

Aceito. Fechado em [09:30] Larissa ("Decisão: reuso máximo do que já existe. AppError, Pino, error middleware, padrão de módulos, padrão de schemas Zod, padrão de códigos de erro. Webhook fica como módulo igual aos outros") e confirmado no resumo de [09:48] Larissa. Itens complementares: prefixo `WEBHOOK_` em [09:29] Larissa, `requireRole` em [09:36] Larissa, função com o client da transação aceita em [09:41] Diego e UUID em [09:51] Larissa.

## Contexto

- O time é pequeno ([09:07] Diego) e o código tem convenções claras: "Cada domínio é um módulo em src/modules com controller, service, repository, routes e schemas" ([09:27] Bruno). Em `src/modules/orders/` estão `order.controller.ts`, `order.service.ts`, `order.repository.ts`, `order.routes.ts` e `order.schemas.ts`.
- Os erros têm uma base comum, `AppError` (`src/shared/errors/app-error.ts:3-16`), com status HTTP e código, e classes específicas como `InvalidStatusTransitionError` e `InsufficientStockError` (`src/shared/errors/http-errors.ts:45-63`), com os códigos `INVALID_STATUS_TRANSITION` e `INSUFFICIENT_STOCK` ([09:28] Bruno).
- O `errorMiddleware` (`src/middlewares/error.middleware.ts:14-65`) já trata `AppError` (L15-24), `ZodError` (L26-35) e erros do Prisma (L38-53): "Vai pegar nossos erros sem precisar mudar nada" ([09:29] Bruno).
- O logger é o Pino, em `src/shared/logger/index.ts`, usado no projeto inteiro ([09:29] Bruno).
- A autenticação usa `authenticate` e `requireRole` (`src/middlewares/auth.middleware.ts:27-47` e `49-61`). O único uso de `requireRole` hoje é `src/modules/users/user.routes.ts:12-18`.
- O acesso ao banco passa por `createPrismaClient()` e pelo singleton `prisma` (`src/config/database.ts:4-10`).
- Todos os models usam id `String @id @default(uuid()) @db.Char(36)` (por exemplo, `prisma/schema.prisma:26` e `prisma/schema.prisma:75`).
- O ponto de integração é o `changeStatus` (`src/modules/orders/order.service.ts:126-179`). O `OrderService` recebe o repository e o `PrismaClient` no construtor (L26-30) e é montado em `src/app.ts:42-44`.

## Decisão

**Construir o módulo de webhooks reaproveitando a estrutura, as classes, os middlewares e as convenções que o projeto já tem, sem stack nova: "Webhook fica como módulo igual aos outros" ([09:30] Larissa).**

| ID | Padrão reaproveitado | Onde está hoje | Como o módulo de webhooks usa | Fala |
| --- | --- | --- | --- | --- |
| ADR-006a | Módulo por domínio | `src/modules/orders/` | `src/modules/webhooks/` **(novo, a criar)** com controller, service, repository, routes e schemas | [09:27] Bruno |
| ADR-006b | Erros com código | `src/shared/errors/app-error.ts` (L3-16); `src/shared/errors/http-errors.ts` (L45-63) | Subclasses com prefixo `WEBHOOK_` (por exemplo, `WEBHOOK_NOT_FOUND` e `WEBHOOK_INVALID_URL`); a lista completa fica na matriz de erros do [FDD](../FDD.md) | [09:28] Bruno, [09:29] Larissa |
| ADR-006c | Error middleware centralizado | `src/middlewares/error.middleware.ts` (L15-24) | Sem alteração: os novos erros saem no formato atual | [09:29] Bruno |
| ADR-006d | Logger Pino | `src/shared/logger/index.ts` | Logs da API e do worker, incluindo o registro de quem fez o replay; nenhum logger novo | [09:29] Bruno |
| ADR-006e | Schemas Zod e middleware `validate` | `src/middlewares/validate.middleware.ts` (L11-37); `src/modules/orders/order.schemas.ts` | Schemas do módulo no mesmo padrão | [09:30] Larissa |
| ADR-006f | Autorização por role | `src/middlewares/auth.middleware.ts` (L49-61); `src/modules/users/user.routes.ts` (L12-18) | `requireRole('ADMIN')` no replay da DLQ; o CRUD aceita qualquer role autenticada "por enquanto" ([09:37] Sofia) | [09:36] Larissa |
| ADR-006g | Transação do `changeStatus` | `src/modules/orders/order.service.ts` (L126-179) | `publishWebhookEvent(tx, order, fromStatus, toStatus)`: função que recebe o client da transação, sem dependência nova no construtor do `OrderService` | [09:41] Bruno, [09:41] Diego |
| ADR-006h | Client do Prisma | `src/config/database.ts` (L4-10) | O worker cria a sua própria instância, com a mesma `DATABASE_URL` | [09:30] Bruno |
| ADR-006i | IDs UUID | `prisma/schema.prisma` (L26 e L75) | Tabelas novas com id UUID | [09:51] Larissa |

- O prefixo `WEBHOOK_` vale para os códigos novos do módulo ([09:29] Larissa). Erros genéricos que já existem (`VALIDATION_ERROR`, `UNAUTHORIZED`, `FORBIDDEN` e o `NOT_FOUND` de customer) continuam sendo usados como estão, na leitura de "reuso máximo do que já existe" ([09:30] Larissa).
- A entry-point do worker, `src/worker.ts` **(novo, a criar)**, segue o molde de `src/server.ts` ([09:11] Larissa); ver [ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md).
- Nomes de arquivos, assinaturas e a composição em `src/app.ts` e `src/routes/index.ts` ficam no [FDD](../FDD.md), seção "Integração com o sistema existente".

## Alternativas Consideradas

### Alternativa 1: injetar o repository de webhook no `OrderService` (ALT-13, discutida)

- **Descrição:** passar um repository de webhook pelo construtor do `OrderService`, como os services já recebem os seus repositories (`src/app.ts:42-44`) ([09:41] Bruno levantou a opção).
- **Prós:** segue o estilo de injeção por construtor que o projeto já usa.
- **Contras:** muda o construtor do `OrderService` e a composição em `src/app.ts`. Além disso, os repositories recebem o `PrismaClient` no construtor (`src/modules/orders/order.repository.ts:18-19`), e a inserção precisa usar o client da transação.
- **Motivo do descarte:** "Boa, função pura recebendo o tx. Não precisa injetar repository inteiro" ([09:41] Diego).

### Alternativa 2: o worker usar o mesmo PrismaClient da API (ALT-14, discutida)

- **Descrição:** compartilhar a mesma instância do client entre API e worker ("usar o mesmo Prisma client", [09:11] Bruno; a pergunta voltou em [09:29] Diego).
- **Prós:** um pool de conexões só.
- **Contras:** API e worker são processos diferentes ([ADR-002](./ADR-002-worker-em-processo-separado-com-polling.md)), e uma instância não atravessa processos.
- **Motivo do descarte:** "Separado. PrismaClient é por processo. Mesmo banco, mesma DATABASE_URL, mas instância nova porque é outro processo Node" ([09:30] Bruno).

### Alternativa 3: id auto-incremental na outbox (ALT-15, discutida)

- **Descrição:** chave numérica sequencial na tabela de outbox ([09:51] Diego perguntou).
- **Prós:** a chave já sairia em ordem de inserção.
- **Contras:** destoa de todos os models, que usam UUID `@db.Char(36)` (`prisma/schema.prisma:26`).
- **Motivo do descarte:** "UUID, segue o padrão do resto do projeto. Tudo é uuid" ([09:51] Larissa).

### Alternativa 4: stack própria para os webhooks (plausível, não discutida)

- **Descrição:** serviço separado, ou módulo com outro logger, outro tratamento de erros e outra biblioteca de validação.
- **Prós:** liberdade para escolher ferramentas específicas de entrega.
- **Contras:** mais uma stack para um time pequeno manter.
- **Motivo do descarte:** contradiz "a gente é um time pequeno" ([09:07] Diego), "mesmo banco, mesma stack" ([09:11] Diego), "Não vamos botar nada novo" ([09:29] Bruno) e a decisão de reuso máximo ([09:30] Larissa).

## Consequências

### Positivas

- Curva de aprendizado quase nula: o módulo se lê como os outros ([09:27] Bruno).
- Os novos erros já saem no formato `{ error: { code, message, details } }` sem mudar o middleware (`src/middlewares/error.middleware.ts:15-24`; [09:29] Bruno).
- Nenhuma dependência nova: o projeto já exige Node 20 ou superior (`package.json:7-9`), que traz `fetch` e `node:crypto`, e o pacote `uuid` já está instalado (`package.json:32`).
- O construtor do `OrderService` (`src/modules/orders/order.service.ts:26-30`), a composição em `src/app.ts:42-44` e o contrato de `PATCH /orders/:id/status` (`src/modules/orders/order.routes.ts:19-23`) não mudam.

### Negativas

- O `NotFoundError` tem código fixo `NOT_FOUND` (`src/shared/errors/http-errors.ts:27-31`). `WEBHOOK_NOT_FOUND` exige uma subclasse própria de `AppError`; reaproveitar o `NotFoundError` faria a API responder `NOT_FOUND`.
- O domínio de pedidos passa a depender do módulo de webhooks dentro da sua transação mais crítica: `order.service.ts` importa e chama `publishWebhookEvent`.
- O `requireRole` só conhece role (`ADMIN` e `OPERATOR`, `prisma/schema.prisma:11-14`), e `User` não tem vínculo com `Customer` (`prisma/schema.prisma:25-38`). Na prática, qualquer usuário autenticado gerencia os webhooks de qualquer customer, o que foi aceito "por enquanto" ([09:37] Sofia); endurecer isso fica em aberto no [RFC](../RFC.md).
- O logger tem `service: 'order-management-api'` fixo (`src/shared/logger/index.ts:20`). Sem ajuste, os logs do worker saem com o mesmo nome de serviço da API (tratado no [FDD](../FDD.md)).
- Reaproveitar `src/config/database.ts` no worker carrega a configuração de `src/config/env.ts`, que exige `JWT_SECRET` (L8) para o processo subir, mesmo sem o worker usar JWT.
- API e worker mantêm dois pools de conexão no mesmo banco, um por processo ([09:29] Diego, [09:30] Bruno).

### Trade-off explícito

> **CONS-1:** Aceitamos herdar as limitações dos padrões atuais (código fixo do `NotFoundError`, autorização só por role, configuração e logger pensados para a API) e acoplar o domínio de pedidos ao módulo de webhooks, em troca de nenhuma dependência nova e de um módulo que o time já sabe ler, testar e operar.

## Referências

- Transcrição: [09:07] Diego, [09:11] Bruno, [09:11] Diego, [09:11] Larissa, [09:27] Bruno, [09:28] Bruno, [09:29] Larissa, [09:29] Bruno, [09:29] Diego, [09:30] Bruno, [09:30] Larissa, [09:36] Sofia, [09:36] Larissa, [09:37] Sofia, [09:41] Bruno, [09:41] Diego, [09:48] Larissa, [09:51] Diego, [09:51] Larissa.
- Código: `src/modules/orders/` (`order.controller.ts`, `order.service.ts`, `order.repository.ts`, `order.routes.ts`, `order.schemas.ts`), `src/shared/errors/app-error.ts`, `src/shared/errors/http-errors.ts`, `src/middlewares/error.middleware.ts`, `src/shared/logger/index.ts`, `src/middlewares/validate.middleware.ts`, `src/middlewares/auth.middleware.ts`, `src/modules/users/user.routes.ts`, `src/config/database.ts`, `src/config/env.ts`, `prisma/schema.prisma`, `src/app.ts`, `package.json`.
