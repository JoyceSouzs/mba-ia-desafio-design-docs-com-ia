# Tracker de Rastreabilidade

Mapeia cada item registrado em PRD, RFC, FDD e ADRs à sua origem na reunião (`TRANSCRICAO.md`) ou no código do repositório. Quando um item tem mais de uma origem, a localização lista todas, separadas por `;`.

- **Fonte = TRANSCRICAO:** localização no formato `[hh:mm] Nome`, conferível em [TRANSCRICAO.md](../TRANSCRICAO.md).
- **Fonte = CODIGO:** caminho de arquivo real do repositório.
- Itens marcados **[HIPÓTESE]** nos documentos não têm origem direta. Estão listados à parte, na seção [Hipóteses explícitas](#hipóteses-explícitas-sem-origem-direta), com a justificativa de cada uma.

## 1. ADRs

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Outbox `webhook_outbox` no MySQL, na mesma transação da mudança de status | TRANSCRICAO | [09:06] Diego; [09:08] Larissa |
| ADR-001-CTX | docs/adrs/ADR-001-outbox-no-mysql.md | Restrição | Transação de `changeStatus` atualiza orders, history e estoque | CODIGO | src/modules/orders/order.service.ts |
| ADR-001-ALT-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa descartada | Disparo síncrono no service de orders | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| ADR-001-ALT-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa descartada | Redis Streams / Redis Cluster | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| ADR-001-IDX | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Índices em status e created_at | TRANSCRICAO | [09:08] Diego |
| ADR-001-UUID | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Ids em UUID seguindo o padrão do projeto | TRANSCRICAO | [09:51] Larissa |
| ADR-001-UUID-COD | docs/adrs/ADR-001-outbox-no-mysql.md | Padrão existente | `@id @default(uuid()) @db.Char(36)` em todos os models | CODIGO | prisma/schema.prisma |
| ADR-002 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Worker em processo separado com polling de 2 s | TRANSCRICAO | [09:09] Diego; [09:10] Larissa; [09:11] Diego |
| ADR-002-ENTRY | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Entry-point `src/worker.ts` e script `npm run worker` | TRANSCRICAO | [09:11] Larissa |
| ADR-002-ENTRY-COD | docs/adrs/ADR-002-worker-separado-em-polling.md | Padrão existente | Bootstrap e shutdown do entry-point HTTP como modelo | CODIGO | src/server.ts |
| ADR-002-PRISMA | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | PrismaClient próprio no worker, mesma DATABASE_URL | TRANSCRICAO | [09:11] Bruno; [09:30] Bruno |
| ADR-002-PRISMA-COD | docs/adrs/ADR-002-worker-separado-em-polling.md | Padrão existente | `createPrismaClient()` | CODIGO | src/config/database.ts |
| ADR-002-ALT-01 | docs/adrs/ADR-002-worker-separado-em-polling.md | Alternativa descartada | Trigger de banco / NOTIFY-LISTEN | TRANSCRICAO | [09:09] Bruno; [09:09] Diego |
| ADR-002-ALT-02 | docs/adrs/ADR-002-worker-separado-em-polling.md | Alternativa descartada | Worker dentro do processo da API | TRANSCRICAO | [09:11] Diego |
| ADR-002-ALT-03 | docs/adrs/ADR-002-worker-separado-em-polling.md | Alternativa descartada | Múltiplos workers em paralelo | TRANSCRICAO | [09:12] Diego; [09:13] Diego |
| ADR-002-LIM | docs/adrs/ADR-002-worker-separado-em-polling.md | Trade-off | Ordering só por order_id e com single-worker | TRANSCRICAO | [09:13] Larissa; [09:14] Marcos |
| ADR-003 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Decisão | 5 tentativas com backoff 1m/5m/30m/2h/12h | TRANSCRICAO | [09:15] Diego; [09:17] Diego; [09:17] Larissa |
| ADR-003-DLQ | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Decisão | DLQ em tabela `webhook_dead_letter` com payload, motivo e timestamp | TRANSCRICAO | [09:18] Diego |
| ADR-003-ALT-01 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Alternativa descartada | Retry indefinido | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Alternativa descartada | Apenas 3 tentativas | TRANSCRICAO | [09:16] Bruno; [09:16] Diego |
| ADR-003-ALT-03 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Alternativa descartada | DLQ como status `failed` na própria outbox | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| ADR-003-TO | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Trade-off | Cliente fora por mais de ~15 h exige replay manual | TRANSCRICAO | [09:17] Marcos |
| ADR-004 | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Decisão | HMAC-SHA256 sobre o corpo, header `X-Signature` | TRANSCRICAO | [09:20] Sofia; [09:22] Sofia |
| ADR-004-SECRET | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Decisão | Secret única por endpoint | TRANSCRICAO | [09:21] Sofia |
| ADR-004-ROT | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Decisão | Rotação com grace period de 24 h | TRANSCRICAO | [09:21] Sofia |
| ADR-004-TLS | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Restrição | URL obrigatoriamente https, validada em Zod | TRANSCRICAO | [09:23] Sofia |
| ADR-004-ALT-01 | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Alternativa descartada | Secret global da plataforma | TRANSCRICAO | [09:21] Sofia |
| ADR-004-REDACT | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Restrição | Logger atual não redige `secret` | CODIGO | src/shared/logger/index.ts |
| ADR-005 | docs/adrs/ADR-005-at-least-once-com-x-event-id.md | Decisão | At-least-once com dedup por `X-Event-Id` | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa |
| ADR-005-ALT-01 | docs/adrs/ADR-005-at-least-once-com-x-event-id.md | Alternativa descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| ADR-005-TO | docs/adrs/ADR-005-at-least-once-com-x-event-id.md | Trade-off | Responsabilidade de dedup transferida ao cliente | TRANSCRICAO | [09:25] Sofia |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Reuso máximo: AppError, Pino, error middleware, módulos, Zod | TRANSCRICAO | [09:30] Larissa |
| ADR-006-MOD | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Módulo `src/modules/webhooks` | TRANSCRICAO | [09:27] Bruno |
| ADR-006-ERR | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Prefixo `WEBHOOK_` nos códigos de erro | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa |
| ADR-006-ERR-COD | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Padrão existente | `AppError` com `statusCode`, `errorCode`, `details` | CODIGO | src/shared/errors/app-error.ts |
| ADR-006-ERR-COD2 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Padrão existente | `InsufficientStockError`, `InvalidStatusTransitionError` | CODIGO | src/shared/errors/http-errors.ts |
| ADR-006-MW | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Padrão existente | Middleware de erro trata AppError, Zod e Prisma | CODIGO | src/middlewares/error.middleware.ts |
| ADR-006-ROLE | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Reuso de `requireRole` no replay | TRANSCRICAO | [09:36] Larissa |
| ADR-006-ROLE-COD | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Padrão existente | `authenticate` e `requireRole` | CODIGO | src/middlewares/auth.middleware.ts |
| ADR-006-PUB | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Função `publishWebhookEvent(tx, ...)` em vez de injetar repository | TRANSCRICAO | [09:41] Bruno; [09:41] Diego |
| ADR-007 | docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md | Decisão | Payload renderizado (snapshot) na inserção | TRANSCRICAO | [09:52] Larissa; [09:52] Diego; [09:52] Bruno |
| ADR-007-FILTRO | docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md | Decisão | Filtro de status aplicado na inserção da outbox | TRANSCRICAO | [09:34] Bruno; [09:34] Diego |
| ADR-007-ALT-01 | docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md | Alternativa descartada | Renderizar payload no envio | TRANSCRICAO | [09:51] Bruno; [09:52] Larissa |
| ADR-007-ALT-02 | docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md | Alternativa descartada | Truncar payload acima do limite | TRANSCRICAO | [09:23] Sofia |

## 2. RFC

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| RFC-META-01 | docs/RFC.md | Metadado | Autora Larissa, que abre o doc de design | TRANSCRICAO | [09:50] Larissa |
| RFC-META-02 | docs/RFC.md | Metadado | Revisores: participantes da reunião | TRANSCRICAO | [09:00] Larissa |
| RFC-CTX-01 | docs/RFC.md | Problema | Clientes fazem polling em GET /orders; risco de churn da Atlas | TRANSCRICAO | [09:00] Marcos |
| RFC-CTX-02 | docs/RFC.md | Requisito Não Funcional | "Tempo real" significa menos de 10 s | TRANSCRICAO | [09:02] Marcos |
| RFC-CTX-03 | docs/RFC.md | Restrição | `changeStatus` já executa transação pesada | CODIGO | src/modules/orders/order.service.ts |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada | Disparo síncrono em `changeStatus` | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada | Redis Streams | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa descartada | Trigger de banco | TRANSCRICAO | [09:09] Bruno; [09:09] Diego |
| RFC-ALT-04 | docs/RFC.md | Alternativa descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| RFC-ALT-05 | docs/RFC.md | Alternativa descartada | Retry indefinido ou 3 tentativas | TRANSCRICAO | [09:15] Diego; [09:16] Bruno |
| RFC-Q-01 | docs/RFC.md | Questão em aberto | Rate limiting de saída | TRANSCRICAO | [09:38] Diego; [09:39] Larissa |
| RFC-Q-02 | docs/RFC.md | Questão em aberto | Escala para múltiplos workers | TRANSCRICAO | [09:13] Diego; [09:13] Larissa |
| RFC-Q-03 | docs/RFC.md | Questão em aberto | Email de alerta ao cliente | TRANSCRICAO | [09:37] Marcos; [09:37] Larissa |
| RFC-Q-04 | docs/RFC.md | Questão em aberto | Endurecer permissões do CRUD | TRANSCRICAO | [09:37] Sofia |
| RFC-Q-05 | docs/RFC.md | Questão em aberto | `customer_id` no body ou no path | TRANSCRICAO | [09:32] Larissa |
| RFC-Q-06 | docs/RFC.md | Questão em aberto | Arquivamento de eventos entregues | TRANSCRICAO | [09:08] Diego |
| RFC-RISCO-01 | docs/RFC.md | Risco | Clientes não deduplicam | TRANSCRICAO | [09:25] Sofia; [09:26] Marcos |
| RFC-RISCO-02 | docs/RFC.md | Risco | Vazamento de secret | TRANSCRICAO | [09:22] Diego |
| RFC-RISCO-03 | docs/RFC.md | Risco | Rajada de eventos de um cliente | TRANSCRICAO | [09:38] Diego |
| RFC-RISCO-04 | docs/RFC.md | Risco | Prazo de fim de novembro | TRANSCRICAO | [09:45] Marcos; [09:46] Larissa |
| RFC-IMP-01 | docs/RFC.md | Impacto | Novo processo `npm run worker` | TRANSCRICAO | [09:11] Larissa |
| RFC-PROX-01 | docs/RFC.md | Próximo passo | Revisão de design com Bruno e Diego | TRANSCRICAO | [09:50] Larissa |
| RFC-PROX-02 | docs/RFC.md | Próximo passo | Revisão de segurança, 2 dias úteis | TRANSCRICAO | [09:46] Sofia |

## 3. PRD

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-CTX-01 | docs/PRD.md | Público-alvo | Atlas Comercial, MaxDistribuição, Nova Cargo | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-02 | docs/PRD.md | Público-alvo | Usuários que representam o cliente, autenticados por JWT | TRANSCRICAO | [09:32] Marcos |
| PRD-CTX-03 | docs/PRD.md | Público-alvo | ADMIN opera a DLQ | TRANSCRICAO | [09:36] Sofia |
| PRD-CTX-04 | docs/PRD.md | Restrição | Webhooks apenas outbound | TRANSCRICAO | [09:02] Marcos; [09:03] Sofia |
| PRD-CTX-05 | docs/PRD.md | Problema | OMS não tem eventos, filas ou webhooks | CODIGO | src/app.ts |
| PRD-OBJ-01 | docs/PRD.md | Objetivo/Métrica | Notificação p95 < 10 s | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | docs/PRD.md | Objetivo/Métrica | 3 de 3 clientes recebendo webhooks | TRANSCRICAO | [09:00] Marcos |
| PRD-OBJ-03 | docs/PRD.md | Objetivo/Métrica | Entrega em 3 sprints, fim de novembro | TRANSCRICAO | [09:45] Marcos; [09:47] Larissa |
| PRD-OBJ-04 | docs/PRD.md | Objetivo/Métrica | Zero mudanças de status sem evento | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| PRD-OBJ-05 | docs/PRD.md | Objetivo/Métrica | Janela de retry de ~15 h | TRANSCRICAO | [09:17] Diego |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Cadastrar webhook com URL, status e secret gerada | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Listar webhooks de um cliente | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Editar webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Remover webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Rotacionar secret com grace de 24 h | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Gerar notificação atômica na mudança de status, com filtro | TRANSCRICAO | [09:34] Bruno; [09:40] Bruno |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Entregar notificação assinada com headers | TRANSCRICAO | [09:43] Diego; [09:44] Diego; [09:44] Sofia |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Retentar com backoff e mover para DLQ | TRANSCRICAO | [09:15] Diego; [09:17] Diego |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Replay da DLQ por ADMIN, com auditoria | TRANSCRICAO | [09:35] Diego; [09:36] Sofia |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Histórico das últimas 100 entregas | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-PRIO | docs/PRD.md | Decisão | Prioridade alta para itens do resumo final; histórico como média | TRANSCRICAO | [09:48] Larissa |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | Polling de 2 s, pior caso aceito | TRANSCRICAO | [09:10] Larissa; [09:10] Marcos |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | Timeout de 10 s por tentativa | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Limite de 64 KB com erro | TRANSCRICAO | [09:23] Sofia; [09:24] Diego; [09:24] Larissa |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | Mudança de status não depende do cliente | TRANSCRICAO | [09:04] Bruno |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | Worker não cai com restart da API | TRANSCRICAO | [09:11] Diego |
| PRD-NFR-06 | docs/PRD.md | Requisito Não Funcional | CRUD liberado para qualquer role autenticada | TRANSCRICAO | [09:37] Sofia |
| PRD-NFR-07 | docs/PRD.md | Requisito Não Funcional | Logs no Pino existente | TRANSCRICAO | [09:29] Bruno |
| PRD-NFR-08 | docs/PRD.md | Requisito Não Funcional | Ordem garantida por pedido com single-worker | TRANSCRICAO | [09:13] Larissa |
| PRD-NFR-09 | docs/PRD.md | Requisito Não Funcional | Secret nunca em logs | TRANSCRICAO | [09:22] Diego |
| PRD-NFR-10 | docs/PRD.md | Compatibilidade | API sob `/api/v1` | CODIGO | src/app.ts |
| PRD-OUT-01 | docs/PRD.md | Fora de escopo | Email de alerta ao cliente | TRANSCRICAO | [09:37] Larissa |
| PRD-OUT-02 | docs/PRD.md | Fora de escopo | Dashboard visual | TRANSCRICAO | [09:40] Larissa |
| PRD-OUT-03 | docs/PRD.md | Fora de escopo | Rate limiting de saída | TRANSCRICAO | [09:39] Larissa |
| PRD-OUT-04 | docs/PRD.md | Fora de escopo | Arquivamento após 30 dias | TRANSCRICAO | [09:08] Diego |
| PRD-OUT-05 | docs/PRD.md | Fora de escopo | Webhooks inbound | TRANSCRICAO | [09:02] Marcos |
| PRD-OUT-06 | docs/PRD.md | Fora de escopo | Ordering global e múltiplos workers | TRANSCRICAO | [09:13] Larissa; [09:14] Marcos |
| PRD-OUT-07 | docs/PRD.md | Fora de escopo | Itens do pedido no payload | TRANSCRICAO | [09:43] Diego |
| PRD-DEP-01 | docs/PRD.md | Dependência | Revisão de segurança da Sofia | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-02 | docs/PRD.md | Dependência | Documentação no portal de desenvolvedor | TRANSCRICAO | [09:26] Marcos; [09:40] Marcos |
| PRD-DEP-03 | docs/PRD.md | Dependência | Confirmação de prazo com a Atlas | TRANSCRICAO | [09:47] Marcos |
| PRD-DEP-04 | docs/PRD.md | Dependência | Ambiente para o processo worker | TRANSCRICAO | [09:11] Diego |
| PRD-DEP-05 | docs/PRD.md | Dependência | Revisão de design com Bruno e Diego | TRANSCRICAO | [09:50] Larissa |
| PRD-RISCO-01 | docs/PRD.md | Risco | Atraso leva à perda da Atlas | TRANSCRICAO | [09:00] Marcos |
| PRD-RISCO-02 | docs/PRD.md | Risco | Cliente não deduplica | TRANSCRICAO | [09:25] Sofia |
| PRD-RISCO-03 | docs/PRD.md | Risco | Vazamento de secret | TRANSCRICAO | [09:22] Diego |
| PRD-RISCO-04 | docs/PRD.md | Risco | Rajada de eventos | TRANSCRICAO | [09:38] Diego |
| PRD-RISCO-05 | docs/PRD.md | Risco | Cliente fora por mais de 15 h | TRANSCRICAO | [09:17] Marcos |
| PRD-TESTE-01 | docs/PRD.md | Estratégia de teste | Testes ponta a ponta da integração com order.service | TRANSCRICAO | [09:46] Larissa |
| PRD-TESTE-02 | docs/PRD.md | Estratégia de teste | Padrão Vitest + Supertest | CODIGO | tests/orders.test.ts |

## 4. FDD

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| FDD-OBJ-01 | docs/FDD.md | Objetivo técnico | Atomicidade status/evento | TRANSCRICAO | [09:06] Diego; [09:40] Bruno |
| FDD-OBJ-02 | docs/FDD.md | Objetivo técnico | Latência p95 < 10 s, espera ≤ 2 s | TRANSCRICAO | [09:02] Marcos; [09:10] Larissa |
| FDD-OBJ-03 | docs/FDD.md | Objetivo técnico | Nenhuma chamada HTTP dentro da transação | TRANSCRICAO | [09:04] Bruno |
| FDD-DADOS-01 | docs/FDD.md | Modelo de dados | `webhook_endpoints`: url, secret, customer_id, ativo | TRANSCRICAO | [09:21] Bruno |
| FDD-DADOS-02 | docs/FDD.md | Modelo de dados | `events` como lista de status | TRANSCRICAO | [09:33] Marcos |
| FDD-DADOS-03 | docs/FDD.md | Modelo de dados | `previousSecret` + expiração para o grace period | TRANSCRICAO | [09:21] Sofia |
| FDD-DADOS-04 | docs/FDD.md | Modelo de dados | Status da outbox pendente/processando/falhou/entregue | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-05 | docs/FDD.md | Modelo de dados | `webhook_deliveries`: sucesso, payload, resposta, tempo | TRANSCRICAO | [09:34] Marcos |
| FDD-DADOS-06 | docs/FDD.md | Modelo de dados | `webhook_dead_letter`: payload, motivo, timestamp, quem fez replay | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| FDD-DADOS-07 | docs/FDD.md | Modelo de dados | Enum `OrderStatus` e relação com `Customer` | CODIGO | prisma/schema.prisma |
| FDD-FLUXO-01 | docs/FDD.md | Fluxo | Inserção na outbox dentro de `changeStatus` | TRANSCRICAO | [09:40] Bruno; [09:41] Bruno |
| FDD-FLUXO-02 | docs/FDD.md | Fluxo | Worker busca pendentes mais antigos em lote | TRANSCRICAO | [09:08] Diego; [09:09] Diego |
| FDD-FLUXO-03 | docs/FDD.md | Fluxo | Processamento sequencial por `created_at` | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-04 | docs/FDD.md | Fluxo | Retry com backoff 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| FDD-FLUXO-05 | docs/FDD.md | Fluxo | DLQ e replay recolocando como pendente | TRANSCRICAO | [09:18] Diego; [09:35] Diego |
| FDD-FLUXO-06 | docs/FDD.md | Fluxo | Rotação de secret | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-07 | docs/FDD.md | Fluxo | Shutdown com `prisma.$disconnect()` | CODIGO | src/server.ts |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | POST /webhooks | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato | GET /webhooks por customer | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-02-COD | docs/FDD.md | Contrato | Formato paginado `paginated()` | CODIGO | src/shared/http/response.ts |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | PATCH /webhooks/:id | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04 | docs/FDD.md | Contrato | DELETE /webhooks/:id | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-05 | docs/FDD.md | Contrato | Rotação de secret via API | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | GET /webhooks/:id/deliveries, últimos 100 | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | POST /admin/webhooks/dead-letter/:id/replay | TRANSCRICAO | [09:18] Diego; [09:35] Diego |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | Payload outbound: campos e sem items | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRATO-09 | docs/FDD.md | Contrato | Headers X-Event-Id, X-Signature, X-Timestamp, Content-Type | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-10 | docs/FDD.md | Contrato | Header X-Webhook-Id | TRANSCRICAO | [09:44] Sofia |
| FDD-CONTRATO-11 | docs/FDD.md | Contrato | `publishWebhookEvent(tx, order, fromStatus, toStatus)` | TRANSCRICAO | [09:41] Bruno |
| FDD-CONTRATO-12 | docs/FDD.md | Contrato | Envelope de erro `{ error: { code, message, details } }` | CODIGO | src/middlewares/error.middleware.ts |
| FDD-CONFIG-01 | docs/FDD.md | Parâmetro | `WEBHOOK_POLL_INTERVAL_MS` = 2000 | TRANSCRICAO | [09:09] Diego |
| FDD-CONFIG-02 | docs/FDD.md | Parâmetro | `WEBHOOK_HTTP_TIMEOUT_MS` = 10000 | TRANSCRICAO | [09:42] Diego |
| FDD-CONFIG-03 | docs/FDD.md | Parâmetro | `WEBHOOK_MAX_PAYLOAD_BYTES` = 65536 | TRANSCRICAO | [09:24] Larissa |
| FDD-CONFIG-04 | docs/FDD.md | Parâmetro | `WEBHOOK_SECRET_GRACE_HOURS` = 24 | TRANSCRICAO | [09:21] Sofia |
| FDD-CONFIG-05 | docs/FDD.md | Integração | Variáveis validadas no `envSchema` | CODIGO | src/config/env.ts |
| FDD-ERRO-01 | docs/FDD.md | Erro | `WEBHOOK_NOT_FOUND` | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-02 | docs/FDD.md | Erro | `WEBHOOK_INVALID_URL` | TRANSCRICAO | [09:23] Sofia; [09:28] Bruno |
| FDD-ERRO-03 | docs/FDD.md | Erro | `WEBHOOK_SECRET_REQUIRED` | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-04 | docs/FDD.md | Erro | `WEBHOOK_INVALID_EVENTS` | TRANSCRICAO | [09:33] Marcos |
| FDD-ERRO-05 | docs/FDD.md | Erro | `WEBHOOK_CUSTOMER_NOT_FOUND` | TRANSCRICAO | [09:21] Bruno |
| FDD-ERRO-06 | docs/FDD.md | Erro | `WEBHOOK_DEAD_LETTER_NOT_FOUND` | TRANSCRICAO | [09:35] Diego |
| FDD-ERRO-07 | docs/FDD.md | Erro | `WEBHOOK_PAYLOAD_TOO_LARGE` | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa |
| FDD-ERRO-08 | docs/FDD.md | Erro | `WEBHOOK_DELIVERY_TIMEOUT` | TRANSCRICAO | [09:42] Diego |
| FDD-ERRO-09 | docs/FDD.md | Erro | `WEBHOOK_DELIVERY_FAILED` | TRANSCRICAO | [09:14] Larissa; [09:15] Diego |
| FDD-ERRO-10 | docs/FDD.md | Erro | `WEBHOOK_MAX_ATTEMPTS_EXCEEDED` | TRANSCRICAO | [09:15] Diego; [09:17] Larissa |
| FDD-ERRO-11 | docs/FDD.md | Restrição | `validate()` converte ZodError em `VALIDATION_ERROR` | CODIGO | src/middlewares/validate.middleware.ts |
| FDD-ERRO-12 | docs/FDD.md | Erro | `UNAUTHORIZED` / `FORBIDDEN` existentes | CODIGO | src/shared/errors/http-errors.ts |
| FDD-OBS-01 | docs/FDD.md | Observabilidade | Logs estruturados com Pino | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-01-COD | docs/FDD.md | Observabilidade | Config do logger e `redactPaths` | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-02 | docs/FDD.md | Observabilidade | Log de auditoria do replay com admin | TRANSCRICAO | [09:36] Sofia |
| FDD-OBS-03 | docs/FDD.md | Observabilidade | `requestId` por requisição para correlação | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-OBS-04 | docs/FDD.md | Observabilidade | Meta de latência usada na métrica | TRANSCRICAO | [09:02] Marcos |
| FDD-DEP-01 | docs/FDD.md | Dependência | Node >= 20, versões de Prisma, Express, Zod, Pino | CODIGO | package.json |
| FDD-DEP-02 | docs/FDD.md | Dependência | MySQL do ambiente | CODIGO | docker-compose.yml |
| FDD-RISCO-01 | docs/FDD.md | Risco | Cliente não deduplica | TRANSCRICAO | [09:25] Sofia |
| FDD-RISCO-02 | docs/FDD.md | Risco | Vazamento de secret | TRANSCRICAO | [09:22] Diego |
| FDD-RISCO-03 | docs/FDD.md | Risco | Quebra de ordem com retry | TRANSCRICAO | [09:13] Larissa |
| FDD-RISCO-04 | docs/FDD.md | Risco | Crescimento da outbox | TRANSCRICAO | [09:07] Bruno; [09:08] Diego |
| FDD-RISCO-05 | docs/FDD.md | Risco | Rajada de eventos | TRANSCRICAO | [09:38] Diego |
| FDD-RISCO-06 | docs/FDD.md | Risco | Worker parado | TRANSCRICAO | [09:11] Diego |
| FDD-INT-01 | docs/FDD.md | Integração | `changeStatus` chama `publishWebhookEvent` | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Integração | Máquina de estados `canTransition` | CODIGO | src/modules/orders/order.status.ts |
| FDD-INT-03 | docs/FDD.md | Integração | Classes de erro estendem `AppError` | CODIGO | src/shared/errors/app-error.ts |
| FDD-INT-04 | docs/FDD.md | Integração | Registro das rotas novas | CODIGO | src/routes/index.ts |
| FDD-INT-05 | docs/FDD.md | Integração | Instanciação em `buildControllers` | CODIGO | src/app.ts |
| FDD-INT-06 | docs/FDD.md | Integração | `requireRole('ADMIN')` como em users | CODIGO | src/modules/users/user.routes.ts |
| FDD-INT-07 | docs/FDD.md | Integração | PrismaClient próprio do worker | CODIGO | src/config/database.ts |
| FDD-INT-08 | docs/FDD.md | Integração | Script `worker` espelhando `start` e `dev` | CODIGO | package.json |
| FDD-INT-09 | docs/FDD.md | Integração | Factories de teste reaproveitadas | CODIGO | tests/helpers/factories.ts |
| FDD-INT-10 | docs/FDD.md | Integração | Nome `webhook.worker.ts` / `webhook.processor.ts` | TRANSCRICAO | [09:28] Bruno |
| FDD-INT-11 | docs/FDD.md | Integração | Controller de pedidos como ponto de entrada | CODIGO | src/modules/orders/order.controller.ts |

## Hipóteses explícitas (sem origem direta)

Itens marcados **[HIPÓTESE]** nos documentos. Não foram decididos na reunião; são detalhes necessários para o FDD ficar acionável, propostos para validação na revisão de design ([09:50] Larissa) ou na revisão de segurança ([09:46] Sofia).

| ID | Documento | Hipótese | Por que foi necessária |
|---|---|---|---|
| HIP-01 | docs/FDD.md | Colunas `attempts` e `nextAttemptAt` e índice `(status, nextAttemptAt)` | Sem elas não há como agendar o backoff com polling |
| HIP-02 | docs/FDD.md | Lote de 10 eventos por ciclo | Reunião disse só "batch pequeno" |
| HIP-03 | docs/FDD.md | Resetar `PROCESSING` para `PENDING` no start do worker | Crash do worker não foi discutido |
| HIP-04 | docs/FDD.md | Não 2xx e erro de rede contam como falha | Reunião citou só o timeout |
| HIP-05 | docs/FDD.md | Payload > 64 KB vai direto para a DLQ, sem retry | O tamanho não muda entre tentativas |
| HIP-06 | docs/FDD.md | Formato `sha256=<hex>` e dupla assinatura no grace period | Reunião definiu o algoritmo e a convivência de 24 h, não o formato |
| HIP-07 | docs/FDD.md | Rota `POST /webhooks/:id/rotate-secret` | Reunião não nomeou a rota |
| HIP-08 | docs/FDD.md, docs/PRD.md | Remoção lógica com `deletedAt` | Preservar histórico de entregas e FKs |
| HIP-09 | docs/FDD.md | Erros `WEBHOOK_ROTATION_IN_PROGRESS`, `WEBHOOK_DEAD_LETTER_ALREADY_REPLAYED`, `WEBHOOK_ENDPOINT_INACTIVE` | Casos de borda não discutidos |
| HIP-10 | docs/FDD.md | Métricas derivadas de tabelas e logs; sem OpenTelemetry | Projeto não tem lib de métricas/tracing e a reunião não definiu |
| HIP-11 | docs/FDD.md | Criação do pedido não gera evento | Reunião só cita `changeStatus` |
| HIP-12 | docs/FDD.md | `fetch` nativo e `node:crypto`, sem dependência nova | Coerente com ADR-006, mas não discutido |
| HIP-13 | docs/FDD.md | Nomes de arquivo além de `webhook.worker.ts` | Seguem o padrão de `src/modules/orders/` |
| HIP-14 | docs/PRD.md | Disponibilidade de 99,9% | Default do prompt de PRD; meta não discutida |
| HIP-15 | docs/PRD.md | Contingência de priorizar a Atlas | Não discutida |
| HIP-16 | docs/FDD.md | `responseBody` limitado a 4 KB | Evitar inflar a tabela de entregas |
