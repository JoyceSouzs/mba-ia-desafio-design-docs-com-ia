# ADR-006: Reuso máximo dos padrões existentes do projeto

- **Status:** Aceito
- **Data da decisão:** reunião técnica de quinta-feira, [09:30] Larissa
- **Decisores:** Larissa (Tech Lead), Bruno (Pedidos), Diego (Plataforma), Sofia (Segurança)
- **Relacionados:** [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-004](ADR-004-hmac-sha256-secret-por-endpoint.md)

## Contexto

A codebase já tem padrões claros ([09:27] Bruno):

- **Módulos por domínio** em `src/modules/<dominio>/` com `*.controller.ts`, `*.service.ts`, `*.repository.ts`, `*.routes.ts` e `*.schemas.ts` (ex.: `src/modules/orders/`, `src/modules/customers/`).
- **Hierarquia de erros**: `AppError` (`src/shared/errors/app-error.ts`) com `statusCode`, `errorCode` e `details`; subclasses em `src/shared/errors/http-errors.ts` como `InsufficientStockError` (`INSUFFICIENT_STOCK`) e `InvalidStatusTransitionError` (`INVALID_STATUS_TRANSITION`) ([09:28] Bruno).
- **Middleware de erro centralizado** em `src/middlewares/error.middleware.ts`, que já trata `AppError`, `ZodError` e `Prisma.PrismaClientKnownRequestError` ([09:29] Bruno).
- **Logger Pino** em `src/shared/logger/index.ts`, usado no projeto inteiro ([09:29] Bruno).
- **Validação Zod** via `validate()` em `src/middlewares/validate.middleware.ts`.
- **Autorização** via `authenticate` e `requireRole` em `src/middlewares/auth.middleware.ts` ([09:36] Larissa).
- **Rotas** registradas em `src/routes/index.ts` (`buildApiRouter`) sob `/api/v1` (`src/app.ts`).

## Decisão

O módulo de webhooks é "um módulo igual aos outros" ([09:30] Larissa):

1. Pasta **`src/modules/webhooks/`** com controller, service, repository, routes e schemas ([09:27] Bruno), mais o processador do worker ([09:28] Bruno).
2. Erros como subclasses de `AppError` com **prefixo `WEBHOOK_`** em todos os códigos do módulo, por exemplo `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED` ([09:28] Bruno, [09:29] Larissa).
3. **Nenhuma mudança** em `src/middlewares/error.middleware.ts`: ele já serializa qualquer `AppError` ([09:29] Bruno).
4. **Pino** como único logger; nenhuma biblioteca nova de log ([09:29] Bruno).
5. **Schemas Zod** para entrada, incluindo a validação de URL `https` ([09:23] Sofia, [09:30] Larissa).
6. **`requireRole('ADMIN')`** reaproveitado no endpoint de replay da DLQ; o CRUD usa apenas `authenticate` ([09:36] Larissa, [09:37] Sofia).
7. Integração com pedidos por uma **função** `publishWebhookEvent(tx, order, fromStatus, toStatus)` que recebe o `Prisma.TransactionClient` da transação corrente, em vez de injetar um repository inteiro no `OrderService` ([09:41] Bruno, [09:41] Diego).

## Alternativas Consideradas

| Alternativa | Por que foi descartada | Fonte |
|---|---|---|
| Injetar o repository de webhooks inteiro no `OrderService` | Acopla mais do que o necessário; uma função pura que recebe o `tx` resolve | [09:41] Bruno, [09:41] Diego |
| Introduzir bibliotecas/padrões novos (outro logger, outro formato de erro) | Contraria a decisão de reuso máximo; o que existe já atende | [09:29] Bruno, [09:30] Larissa |

## Consequências

**Positivas**
- Curva de aprendizado mínima: quem conhece `orders` navega em `webhooks`.
- Respostas de erro com o mesmo envelope `{ error: { code, message, details } }` produzido por `errorMiddleware`.
- Menos dependências e menos superfície de revisão.

**Negativas / trade-offs**
- O módulo herda as limitações atuais: não há métricas nem tracing distribuído no projeto, só logs Pino e `requestId` (`src/middlewares/request-logger.middleware.ts`).
- `OrderService` passa a importar uma função do módulo de webhooks (acoplamento pequeno e explícito).
- **Trade-off explícito:** consistência com a codebase em vez de escolher a ferramenta "ideal" para cada parte do problema.

## Referências

- Transcrição: [09:23] Sofia, [09:27] Bruno, [09:28] Bruno, [09:29] Larissa, [09:29] Bruno, [09:30] Larissa, [09:36] Larissa, [09:37] Sofia, [09:41] Bruno, [09:41] Diego
- Código: `src/shared/errors/app-error.ts`, `src/shared/errors/http-errors.ts`, `src/middlewares/error.middleware.ts`, `src/middlewares/auth.middleware.ts`, `src/middlewares/validate.middleware.ts`, `src/shared/logger/index.ts`, `src/routes/index.ts`, `src/app.ts`, `src/modules/orders/`
