# Tracker de Rastreabilidade

Mapeia cada item registrado em PRD, RFC, FDD e ADRs à sua origem na reunião (`TRANSCRICAO.md`) ou no código do repositório. Quando um item tem mais de uma origem, a localização lista todas, separadas por `;`.

- **Fonte = TRANSCRICAO:** localização no formato `[hh:mm] Nome`, conferível em [TRANSCRICAO.md](../TRANSCRICAO.md).
- **Fonte = CODIGO:** caminho de arquivo real do repositório.
- **Fonte = HIPÓTESE:** item marcado **[HIPÓTESE]** nos documentos, sem origem direta. Fica na seção [Hipóteses explícitas](#5-hipóteses-explícitas), com a justificativa no próprio conteúdo, para ser validado na revisão de design ([09:50] Larissa) ou de segurança ([09:46] Sofia).

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
| ADR-001-CONS | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência | Atomicidade sem infraestrutura nova | TRANSCRICAO | [09:06] Diego; [09:07] Diego |
| ADR-002-CONS | docs/adrs/ADR-002-worker-separado-em-polling.md | Consequência | Latência mínima de 2 s aceita | TRANSCRICAO | [09:10] Larissa |
| ADR-003-CONS | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Consequência | Cobre ~15 h de indisponibilidade sem intervenção | TRANSCRICAO | [09:17] Diego |
| ADR-004-CONS | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Consequência | Revisão de segurança de HMAC e geração de secret | TRANSCRICAO | [09:46] Sofia |
| ADR-005-CONS | docs/adrs/ADR-005-at-least-once-com-x-event-id.md | Consequência | Comportamento documentado no portal | TRANSCRICAO | [09:26] Marcos |
| ADR-006-CONS | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Consequência | Middleware de erro captura erros novos sem mudança | TRANSCRICAO | [09:29] Bruno |
| ADR-007-CONS | docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md | Consequência | Payload enxuto sem items | TRANSCRICAO | [09:43] Diego; [09:44] Bruno |

## 2. RFC

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| RFC-META-01 | docs/RFC.md | Metadado | Autora Larissa, que abre o doc de design | TRANSCRICAO | [09:50] Larissa |
| RFC-META-02 | docs/RFC.md | Metadado | Revisores: participantes da reunião, com revisão de design e de segurança combinadas | TRANSCRICAO | [09:46] Sofia; [09:50] Larissa |
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
| RFC-RISCO-01 | docs/RFC.md | Risco de arquitetura | Transação de `changeStatus`, já pesada, ganha mais uma escrita | TRANSCRICAO | [09:04] Bruno; [09:34] Bruno |
| RFC-RISCO-02 | docs/RFC.md | Risco de arquitetura | Teto de escala do single-worker | TRANSCRICAO | [09:13] Diego |
| RFC-RISCO-03 | docs/RFC.md | Risco de arquitetura | Vazamento de secret limitado a um endpoint | TRANSCRICAO | [09:21] Sofia; [09:22] Diego |
| RFC-RISCO-04 | docs/RFC.md | Risco de arquitetura | Rajada de eventos de um cliente | TRANSCRICAO | [09:38] Diego; [09:39] Larissa |
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
| PRD-CTX-06 | docs/PRD.md | Contexto | Mudança de status acontece no service de pedidos | CODIGO | src/modules/orders/order.service.ts |
| PRD-OBJ-01 | docs/PRD.md | Objetivo/Métrica | Notificação p95 < 10 s | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | docs/PRD.md | Objetivo/Métrica | 3 de 3 clientes recebendo webhooks | TRANSCRICAO | [09:00] Marcos |
| PRD-OBJ-03 | docs/PRD.md | Objetivo/Métrica | Entrega em 3 sprints, fim de novembro | TRANSCRICAO | [09:45] Marcos; [09:47] Larissa |
| PRD-OBJ-04 | docs/PRD.md | Objetivo/Métrica | Zero mudanças de status sem evento | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| PRD-OBJ-05 | docs/PRD.md | Objetivo/Métrica | Janela de retry de ~15 h | TRANSCRICAO | [09:17] Diego |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Cadastrar webhook com URL, status e secret gerada | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Listar webhooks de um cliente | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Editar webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Remover webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04-ALT | docs/PRD.md | Fluxo alternativo | Pausar envio pelo estado ativo, sem remover | TRANSCRICAO | [09:21] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Rotacionar secret com grace de 24 h | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Gerar notificação atômica na mudança de status, com filtro | TRANSCRICAO | [09:34] Bruno; [09:40] Bruno |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Entregar notificação assinada com headers | TRANSCRICAO | [09:43] Diego; [09:44] Diego; [09:44] Sofia |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Retentar com backoff e mover para DLQ | TRANSCRICAO | [09:15] Diego; [09:17] Diego |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Replay da DLQ por ADMIN, com auditoria | TRANSCRICAO | [09:35] Diego; [09:36] Sofia |
| PRD-FR-09-ALT | docs/PRD.md | Fluxo alternativo | Evento reprocessado volta ao ciclo de tentativas | TRANSCRICAO | [09:18] Diego |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Histórico das últimas 100 entregas | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-PRIO | docs/PRD.md | Decisão | Prioridade alta para itens do resumo final; histórico como média | TRANSCRICAO | [09:48] Larissa |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | Polling de 2 s, pior caso aceito | TRANSCRICAO | [09:10] Larissa; [09:10] Marcos |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | Timeout de 10 s por tentativa | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Limite de 64 KB com erro | TRANSCRICAO | [09:23] Sofia; [09:24] Diego; [09:24] Larissa |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | Mudança de status não depende do cliente | TRANSCRICAO | [09:04] Bruno |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | Worker não cai com restart da API | TRANSCRICAO | [09:11] Diego |
| PRD-NFR-05B | docs/PRD.md | Requisito Não Funcional | Sem meta de uptime; eventos não se perdem com worker parado | TRANSCRICAO | [09:06] Diego |
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
| PRD-RISCO-01 | docs/PRD.md | Risco de negócio | Atraso leva à perda da Atlas | TRANSCRICAO | [09:00] Marcos |
| PRD-RISCO-01-CONT | docs/PRD.md | Contingência | Marcos comunica desvio de prazo à Atlas | TRANSCRICAO | [09:47] Marcos; [09:49] Marcos |
| PRD-RISCO-02 | docs/PRD.md | Risco de negócio | Clientes não se adaptam à entrega duplicada | TRANSCRICAO | [09:25] Sofia; [09:26] Marcos |
| PRD-RISCO-03 | docs/PRD.md | Risco de negócio | Cliente fora por mais de 15 h | TRANSCRICAO | [09:17] Marcos |
| PRD-TESTE-01 | docs/PRD.md | Estratégia de teste | Testes ponta a ponta da integração com order.service | TRANSCRICAO | [09:46] Larissa |
| PRD-TESTE-02 | docs/PRD.md | Estratégia de teste | Padrão Vitest + Supertest | CODIGO | tests/orders.test.ts |
| PRD-CEN-01 | docs/PRD.md | Cenário de uso | Cliente escolhe receber só SHIPPED e DELIVERED | TRANSCRICAO | [09:33] Marcos |
| PRD-CEN-02 | docs/PRD.md | Cenário de uso | Notificação em segundos, sem polling | TRANSCRICAO | [09:02] Marcos |
| PRD-CEN-03 | docs/PRD.md | Cenário de uso | Cliente fora por 2 h em manutenção recebe depois | TRANSCRICAO | [09:16] Diego |
| PRD-CEN-04 | docs/PRD.md | Cenário de uso | Cliente rotaciona secret vazada e tem 24 h | TRANSCRICAO | [09:21] Sofia; [09:22] Diego |
| PRD-CEN-05 | docs/PRD.md | Cenário de uso | Cliente consulta os últimos 100 envios | TRANSCRICAO | [09:34] Marcos |
| PRD-CEN-06 | docs/PRD.md | Cenário de uso | ADMIN reprocessa notificação da DLQ | TRANSCRICAO | [09:18] Diego |
| PRD-CTX-07 | docs/PRD.md | Contexto | Novo módulo e worker separado no mesmo banco | TRANSCRICAO | [09:27] Bruno; [09:11] Diego |
| PRD-ARQ-01 | docs/PRD.md | Arquitetura | Abordagem outbox com worker separado | TRANSCRICAO | [09:06] Diego; [09:08] Larissa |
| PRD-ARQ-02 | docs/PRD.md | Arquitetura | Componente: tabelas de configuração, outbox, entregas e DLQ | TRANSCRICAO | [09:08] Diego; [09:18] Diego; [09:34] Marcos |
| PRD-ARQ-03 | docs/PRD.md | Arquitetura | Integração: `changeStatus` registra a notificação | TRANSCRICAO | [09:40] Bruno; [09:41] Bruno |
| PRD-ARQ-04 | docs/PRD.md | Arquitetura | Integração: portal de desenvolvedor documenta o contrato | TRANSCRICAO | [09:26] Marcos |
| PRD-DEC-01 | docs/PRD.md | Decisão | Outbox no MySQL e trade-off de tabela crescente | TRANSCRICAO | [09:07] Diego; [09:08] Diego |
| PRD-DEC-02 | docs/PRD.md | Decisão | Worker com polling de 2 s e trade-off de espera | TRANSCRICAO | [09:09] Diego; [09:10] Larissa |
| PRD-DEC-03 | docs/PRD.md | Decisão | 5 tentativas e DLQ; trade-off de replay manual | TRANSCRICAO | [09:16] Diego; [09:17] Marcos |
| PRD-DEC-04 | docs/PRD.md | Decisão | HMAC com secret por webhook | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia |
| PRD-DEC-05 | docs/PRD.md | Decisão | At-least-once e trade-off de dedup no cliente | TRANSCRICAO | [09:25] Diego; [09:25] Sofia |
| PRD-DEC-06 | docs/PRD.md | Decisão | Reuso dos padrões do projeto | TRANSCRICAO | [09:30] Larissa |
| PRD-DEC-07 | docs/PRD.md | Decisão | Conteúdo congelado e filtro na origem | TRANSCRICAO | [09:52] Larissa; [09:34] Bruno |
| PRD-CA-01 | docs/PRD.md | Critério de aceitação | Cadastro com URL https devolve a secret | TRANSCRICAO | [09:31] Marcos; [09:23] Sofia |
| PRD-CA-02 | docs/PRD.md | Critério de aceitação | URL http recusada | TRANSCRICAO | [09:23] Sofia |
| PRD-CA-03 | docs/PRD.md | Critério de aceitação | Notificação em menos de 10 s (p95) | TRANSCRICAO | [09:02] Marcos |
| PRD-CA-04 | docs/PRD.md | Critério de aceitação | Status não escolhido não gera notificação | TRANSCRICAO | [09:34] Bruno |
| PRD-CA-05 | docs/PRD.md | Critério de aceitação | Falha no registro desfaz a mudança de status | TRANSCRICAO | [09:40] Bruno |
| PRD-CA-06 | docs/PRD.md | Critério de aceitação | Headers e assinatura HMAC presentes em toda notificação | TRANSCRICAO | [09:44] Diego; [09:44] Sofia |
| PRD-CA-07 | docs/PRD.md | Critério de aceitação | Retentativas em 1m/5m/30m/2h/12h e depois DLQ | TRANSCRICAO | [09:17] Larissa |
| PRD-CA-08 | docs/PRD.md | Critério de aceitação | Replay por ADMIN com mesmo identificador e log de autoria | TRANSCRICAO | [09:36] Sofia; [09:18] Diego |
| PRD-CA-09 | docs/PRD.md | Critério de aceitação | Secret antiga e nova válidas por 24 h após rotação | TRANSCRICAO | [09:21] Sofia |
| PRD-CA-10 | docs/PRD.md | Critério de aceitação | Histórico com 100 tentativas, resultado, payload, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| PRD-CA-11 | docs/PRD.md | Critério de aceitação | Restart da API não interrompe envios | TRANSCRICAO | [09:11] Diego |
| PRD-CA-12 | docs/PRD.md | Critério de aceitação | Revisão de segurança antes do deploy | TRANSCRICAO | [09:46] Sofia; [09:49] Sofia |
| PRD-CA-13 | docs/PRD.md | Critério de aceitação | Portal publicado com assinatura e deduplicação | TRANSCRICAO | [09:26] Marcos |
| PRD-TESTE-03 | docs/PRD.md | Estratégia de teste | Revisão de design antes de codar | TRANSCRICAO | [09:50] Larissa |

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
| FDD-ERRO-05 | docs/FDD.md | Erro | `WEBHOOK_CUSTOMER_NOT_FOUND`, mesma checagem de customer inexistente já feita na criação de pedido | CODIGO | src/modules/orders/order.service.ts |
| FDD-ERRO-06 | docs/FDD.md | Erro | `WEBHOOK_DEAD_LETTER_NOT_FOUND`, inclusive para item que não está mais em FAILED | TRANSCRICAO | [09:35] Diego |
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
| FDD-RISCO-01 | docs/FDD.md | Risco técnico | Secret exposta em logs ou respostas | TRANSCRICAO | [09:22] Diego |
| FDD-RISCO-01-COD | docs/FDD.md | Mitigação | Estender `redactPaths` do logger | CODIGO | src/shared/logger/index.ts |
| FDD-RISCO-02 | docs/FDD.md | Risco técnico | Quebra de ordem com retry | TRANSCRICAO | [09:13] Larissa |
| FDD-RISCO-03 | docs/FDD.md | Risco técnico | Crescimento da outbox | TRANSCRICAO | [09:07] Bruno; [09:08] Diego |
| FDD-RISCO-04 | docs/FDD.md | Risco técnico | Worker parado | TRANSCRICAO | [09:11] Diego |
| FDD-RISCO-05 | docs/FDD.md | Risco técnico | Reenvio após crash, coberto por at-least-once | TRANSCRICAO | [09:24] Diego; [09:25] Diego |
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
| FDD-ALT-01 | docs/FDD.md | Fluxo alternativo | Sem webhook interessado, nada é inserido | TRANSCRICAO | [09:34] Bruno |
| FDD-ALT-02 | docs/FDD.md | Fluxo alternativo | Falha na outbox gera rollback e erro 500 | TRANSCRICAO | [09:40] Bruno |
| FDD-ALT-03 | docs/FDD.md | Fluxo alternativo | Filtro alterado não afeta eventos já gravados | TRANSCRICAO | [09:34] Bruno |
| FDD-ALT-04 | docs/FDD.md | Fluxo alternativo | Mudanças rápidas saem em ordem com single-worker | TRANSCRICAO | [09:12] Diego |
| FDD-INV-01 | docs/FDD.md | Invariante | Nenhuma mudança de status sem evento e vice-versa | TRANSCRICAO | [09:06] Diego |
| FDD-INV-02 | docs/FDD.md | Invariante | `event_id` imutável até a DLQ e o replay | TRANSCRICAO | [09:25] Diego |
| FDD-INV-03 | docs/FDD.md | Invariante | Payload imutável após a inserção | TRANSCRICAO | [09:52] Larissa |
| FDD-INV-04 | docs/FDD.md | Invariante | Nenhum HTTP dentro da transação de pedidos | TRANSCRICAO | [09:04] Bruno |
| FDD-INV-05 | docs/FDD.md | Invariante | Nenhum corpo acima de 64 KB enviado | TRANSCRICAO | [09:24] Larissa |
| FDD-INV-06 | docs/FDD.md | Invariante | Secret fora de logs e respostas | TRANSCRICAO | [09:22] Diego |
| FDD-RES-01 | docs/FDD.md | Resiliência | Timeout de 10 s por chamada | TRANSCRICAO | [09:42] Diego |
| FDD-RES-02 | docs/FDD.md | Resiliência | Fallback: DLQ e replay manual; email fica para a próxima fase | TRANSCRICAO | [09:18] Diego; [09:37] Larissa |
| FDD-RES-03 | docs/FDD.md | Resiliência | Sem circuit breaker; rate limiting em aberto | TRANSCRICAO | [09:39] Larissa |
| FDD-LOG-01 | docs/FDD.md | Observabilidade | Log `webhook_event_enqueued` | TRANSCRICAO | [09:40] Bruno |
| FDD-LOG-02 | docs/FDD.md | Observabilidade | Logs `worker_started`/`worker_stopped` | TRANSCRICAO | [09:11] Diego |
| FDD-LOG-03 | docs/FDD.md | Observabilidade | Log `webhook_delivery_succeeded` com status e duração | TRANSCRICAO | [09:34] Marcos |
| FDD-LOG-04 | docs/FDD.md | Observabilidade | Log `webhook_delivery_failed` com próximo agendamento | TRANSCRICAO | [09:15] Diego |
| FDD-LOG-05 | docs/FDD.md | Observabilidade | Log `webhook_moved_to_dlq` | TRANSCRICAO | [09:18] Diego |
| FDD-LOG-06 | docs/FDD.md | Observabilidade | Log `webhook_secret_rotated` | TRANSCRICAO | [09:21] Sofia |
| FDD-OBS-05 | docs/FDD.md | Observabilidade | Alerta para cada evento na DLQ, sem email ao cliente | TRANSCRICAO | [09:37] Larissa |
| FDD-OBS-06 | docs/FDD.md | Observabilidade | Painel para o cliente fora de escopo | TRANSCRICAO | [09:40] Larissa |
| FDD-CA-01 | docs/FDD.md | Critério técnico | Uma linha PENDING por webhook interessado no mesmo commit | TRANSCRICAO | [09:40] Bruno |
| FDD-CA-02 | docs/FDD.md | Critério técnico | Erro em `publishWebhookEvent` faz rollback completo | TRANSCRICAO | [09:41] Diego |
| FDD-CA-03 | docs/FDD.md | Critério técnico | URL http gera detalhe `WEBHOOK_INVALID_URL` | TRANSCRICAO | [09:23] Sofia |
| FDD-CA-04 | docs/FDD.md | Critério técnico | Secret só na criação e na rotação | TRANSCRICAO | [09:31] Marcos |
| FDD-CA-05 | docs/FDD.md | Critério técnico | p95 abaixo de 10 s com 100 eventos | TRANSCRICAO | [09:02] Marcos |
| FDD-CA-06 | docs/FDD.md | Critério técnico | Endpoint lento gera `WEBHOOK_DELIVERY_TIMEOUT` | TRANSCRICAO | [09:42] Diego |
| FDD-CA-07 | docs/FDD.md | Critério técnico | 6 tentativas com intervalos de backoff e DLQ | TRANSCRICAO | [09:17] Diego |
| FDD-CA-08 | docs/FDD.md | Critério técnico | Replay: 403 para OPERATOR, 202 para ADMIN | TRANSCRICAO | [09:36] Sofia |
| FDD-CA-09 | docs/FDD.md | Critério técnico | Ordem PAID, PROCESSING, SHIPPED preservada | TRANSCRICAO | [09:12] Diego |
| FDD-CA-10 | docs/FDD.md | Critério técnico | Restart da API não interrompe o worker | TRANSCRICAO | [09:11] Diego |
| FDD-CA-11 | docs/FDD.md | Critério técnico | Deliveries limitado a 100, do mais novo ao mais antigo | TRANSCRICAO | [09:34] Marcos |
| FDD-DEP-03 | docs/FDD.md | Dependência | Revisão de segurança da Sofia | TRANSCRICAO | [09:46] Sofia |
| FDD-DEP-04 | docs/FDD.md | Dependência | Portal de desenvolvedor pelo Marcos | TRANSCRICAO | [09:26] Marcos |
| FDD-INT-12 | docs/FDD.md | Integração | Mesmo Prisma e mesmo banco para API e worker | TRANSCRICAO | [09:11] Bruno |

## 5. Hipóteses explícitas

Itens marcados **[HIPÓTESE]** nos documentos. Não foram decididos na reunião; são detalhes necessários para o FDD ficar acionável. A justificativa de cada um está no conteúdo.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| HIP-01 | docs/FDD.md | Hipótese | Colunas `attempts`/`nextAttemptAt` e índice `(status, nextAttemptAt)`: sem elas não há como agendar backoff com polling | HIPÓTESE | docs/FDD.md, seção 4.1 |
| HIP-02 | docs/FDD.md | Hipótese | Lote de 10 eventos por ciclo: a reunião disse só "batch pequeno" | HIPÓTESE | docs/FDD.md, seção 4.3 |
| HIP-03 | docs/FDD.md | Hipótese | Resetar `PROCESSING` para `PENDING` no start do worker: crash não foi discutido | HIPÓTESE | docs/FDD.md, seção 4.3 |
| HIP-04 | docs/FDD.md | Hipótese | Não 2xx e erro de rede contam como falha: a reunião citou só o timeout | HIPÓTESE | docs/FDD.md, seção 4.4 |
| HIP-05 | docs/FDD.md | Hipótese | Payload acima de 64 KB vai direto para a DLQ, sem retry: o tamanho não muda entre tentativas | HIPÓTESE | docs/FDD.md, seção 4.3 |
| HIP-06 | docs/FDD.md | Hipótese | Formato `sha256=<hex>` e dupla assinatura no grace period: a reunião definiu algoritmo e convivência de 24 h, não o formato | HIPÓTESE | docs/FDD.md, seção 5.8 |
| HIP-07 | docs/FDD.md | Hipótese | Rota `POST /webhooks/:id/rotate-secret`: a reunião não nomeou a rota | HIPÓTESE | docs/FDD.md, seção 5.5 |
| HIP-08 | docs/FDD.md | Hipótese | Remoção lógica com `deletedAt`: preserva histórico de entregas e FKs | HIPÓTESE | docs/FDD.md, seção 5.4 |
| HIP-09 | docs/FDD.md | Hipótese | `WEBHOOK_ENDPOINT_INACTIVE` encerra evento de webhook removido sem retry: caso de borda não discutido | HIPÓTESE | docs/FDD.md, seção 6 |
| HIP-10 | docs/FDD.md | Hipótese | Métricas derivadas de tabelas e logs, sem OpenTelemetry: o projeto não tem lib de métricas/tracing e a reunião não definiu | HIPÓTESE | docs/FDD.md, seção 7 |
| HIP-11 | docs/FDD.md | Hipótese | Criação do pedido não gera evento: a reunião só cita `changeStatus` | HIPÓTESE | docs/FDD.md, seção 3 |
| HIP-12 | docs/FDD.md | Hipótese | `fetch` nativo e `node:crypto`, sem dependência nova: coerente com ADR-006, mas não discutido | HIPÓTESE | docs/FDD.md, seção 8 |
| HIP-13 | docs/FDD.md | Hipótese | Nomes de arquivo além de `webhook.worker.ts`, seguindo o padrão de `src/modules/orders/` | HIPÓTESE | docs/FDD.md, seção 11 |
| HIP-14 | docs/FDD.md | Hipótese | Bloquear envio por par (webhook, pedido) com evento anterior pendente, como contingência de ordem | HIPÓTESE | docs/FDD.md, seção 10 (R2) |
| HIP-15 | docs/FDD.md | Hipótese | Payload versionado por `event_type` sem remoção de campos | HIPÓTESE | docs/FDD.md, seção 8 |
