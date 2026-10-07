# Registro da entrevista: FDD

Execução do prompt "Prompt para geração de um FDD" com a regra adicional de que **toda resposta do usuário vem da `TRANSCRICAO.md` ou do código**. Quando a transcrição não responde, a resposta é marcada como **[HIPÓTESE]** ou **[EM ABERTO]**, nunca preenchida por invenção.

Legenda: **P** = pergunta do entrevistador (IA) · **R** = resposta extraída da fonte.

---

## Etapa 1. Contexto e motivação técnica

**P:** Qual problema técnico real a feature resolve?
**R:** Clientes fazem polling em `GET /orders` para detectar mudança de status, o que é lento e caro ([09:00] Marcos). O OMS não tem nenhum mecanismo de eventos: não há filas, eventos ou webhooks no código (`src/`).

**P:** Como ela se encaixa nos sistemas existentes?
**R:** Entra no ponto em que o status muda, `OrderService.changeStatus` em `src/modules/orders/order.service.ts`, dentro da transação que já atualiza `orders`, `order_status_history` e estoque ([09:04] Bruno, [09:40] Bruno).

**P:** Quem são os atores?
**R:** Usuários autenticados que representam o cliente e cadastram webhooks pela API ([09:32] Marcos); operadores que mudam status de pedidos (`src/modules/orders/order.controller.ts`); ADMIN que faz replay de DLQ ([09:36] Sofia); o endpoint HTTP do cliente; o worker.

**Resumo confirmado:** feature outbound ([09:02] Marcos), acoplada a `changeStatus` por outbox, com API de gestão e worker separado.

## Etapa 2. Objetivos técnicos

**P:** Quais resultados mensuráveis?
**R:** Notificação em menos de 10 s ([09:02] Marcos); espera de polling de no máximo 2 s ([09:10] Larissa); timeout de 10 s por chamada ([09:42] Diego); payload de até 64 KB ([09:24] Larissa).

**P:** Quais garantias determinísticas?
**R:** Atomicidade status/evento ([09:06] Diego, [09:40] Bruno); at-least-once ([09:24] Diego); ordem por `order_id` com single-worker ([09:13] Larissa); 5 retentativas e DLQ ([09:17] Larissa).

## Etapa 3. Escopo e exclusões

**P:** O que fica fora?
**R:** Email de alerta ([09:37] Larissa); dashboard ([09:40] Larissa); rate limiting de saída ([09:39] Larissa); arquivamento após 30 dias ([09:08] Diego); webhooks inbound ([09:02] Marcos); múltiplos workers ([09:13] Diego); `items` no payload ([09:43] Diego).

**P:** A criação do pedido (status inicial PENDING) gera evento?
**R:** A transcrição fala só de mudança de status e aponta `changeStatus` como o ponto de integração ([09:40] Bruno). A criação em `OrderService.create` grava histórico com `fromStatus: null`, mas não é citada. **[HIPÓTESE]**: criação não gera evento nesta fase; confirmar na revisão.

## Etapa 4. Fluxos

**P:** Como o worker escolhe o que processar?
**R:** Pendentes mais antigos, em lote pequeno, a cada 2 s, por ordem de `created_at` ([09:08] Diego, [09:09] Diego, [09:12] Diego). Tamanho do lote: **[HIPÓTESE]** 10, configurável.

**P:** Como o backoff é agendado?
**R:** 1m/5m/30m/2h/12h ([09:17] Diego). A coluna `next_attempt_at` é **[HIPÓTESE]** de modelagem necessária para implementar o backoff com polling.

**P:** O que acontece se o worker morrer com eventos em PROCESSING?
**R:** Não discutido. **[HIPÓTESE]**: no start, o worker devolve eventos `PROCESSING` para `PENDING`. Gera possível duplicidade, coberta pela garantia at-least-once ([09:24] Diego).

## Etapa 5. Contratos públicos

**P:** Quais endpoints?
**R:** POST cadastro ([09:31] Marcos); PATCH, DELETE, GET lista por customer ([09:33] Bruno); rotação de secret ([09:21] Sofia); `GET /webhooks/:id/deliveries` ([09:34] Marcos); `POST /admin/webhooks/dead-letter/:id/replay` ([09:35] Diego).

**P:** Rota da rotação de secret?
**R:** A transcrição só diz "endpoint pro cliente conseguir pedir nova secret pela API" ([09:21] Sofia). **[HIPÓTESE]** `POST /webhooks/:id/rotate-secret`.

**P:** Onde vai o `customer_id`?
**R:** "No body ou no path. Não vem do JWT" ([09:32] Larissa). **[EM ABERTO]**; FDD propõe body na criação e query na listagem.

**P:** Headers e payload da chamada ao cliente?
**R:** `X-Event-Id`, `X-Signature`, `X-Timestamp`, `Content-Type: application/json` ([09:44] Diego) e `X-Webhook-Id` ([09:44] Sofia). Payload: `event_id`, `event_type`, `timestamp`, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id`, `total_cents` ([09:43] Diego).

**P:** Formato do valor de `X-Signature` e o que fazer no grace period?
**R:** Só foi decidido "HMAC-SHA256 sobre o corpo" ([09:22] Sofia) e que a secret antiga "fica válida por 24 horas em paralelo" ([09:21] Sofia). **[HIPÓTESE]** hex do HMAC; durante o grace period enviar as duas assinaturas separadas por vírgula. Item para a revisão da Sofia ([09:46] Sofia).

## Etapa 6. Erros e fallback

**P:** Quais códigos de erro?
**R:** Prefixo `WEBHOOK_` ([09:29] Larissa). Citados: `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED` ([09:28] Bruno). Os demais foram derivados de restrições da reunião (timeout, 64 KB, DLQ) e estão mapeados no TRACKER.

**P:** Quais respostas do cliente contam como falha?
**R:** Timeout de 10 s conta como falha ([09:42] Diego). **[HIPÓTESE]**: qualquer resposta fora de 2xx e erros de rede também contam.

**P:** Payload acima de 64 KB?
**R:** Não envia; erro, sem truncar ([09:23] Sofia, [09:24] Larissa). **[HIPÓTESE]**: vai direto para a DLQ sem retry, pois o tamanho não muda entre tentativas.

## Etapa 7. Observabilidade

**P:** Quais logs?
**R:** Pino, já usado no projeto ([09:29] Bruno, `src/shared/logger/index.ts`). Replay precisa logar quem fez ([09:36] Sofia).

**P:** Métricas e tracing?
**R:** A reunião não definiu e o código não tem biblioteca de métricas nem de tracing. **[HIPÓTESE]**: métricas calculadas a partir das tabelas e dos logs; correlação por `event_id`/`order_id` no lugar de tracing distribuído. Exportador e OpenTelemetry ficam fora desta fase para manter o [ADR-006](../adrs/ADR-006-reuso-dos-padroes-do-projeto.md).

## Etapa 8. Dependências

**P:** Dependências novas?
**R:** Nenhuma obrigatória: mesmo MySQL e Prisma ([09:11] Diego). **[HIPÓTESE]**: HTTP via `fetch` nativo do Node 20 (`package.json` exige `node >= 20`) e HMAC via `node:crypto`.

## Etapa 9. Critérios de aceite

Derivados das garantias das etapas 2, 5 e 6.

## Etapa 10. Riscos

**P:** Riscos técnicos?
**R:** Duplicidade no cliente ([09:25] Sofia); vazamento de secret ([09:22] Diego); quebra de ordem com retry (decorrência de [09:13] Larissa); rajada de eventos ([09:38] Diego); crescimento da outbox ([09:07] Bruno, [09:08] Diego).
