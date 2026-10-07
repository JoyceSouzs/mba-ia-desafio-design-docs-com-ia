# Architectural Decision Records

Este diretório armazena os ADRs (Architectural Decision Records) do projeto, no formato MADR.
Cada decisão arquitetural relevante é registrada em um arquivo individual, nomeado
`ADR-NNN-titulo-em-kebab-case.md`.

## Índice: Sistema de Webhooks de Notificação de Pedidos

| ADR | Decisão | Status |
|---|---|---|
| [ADR-001](ADR-001-outbox-no-mysql.md) | Padrão Outbox no MySQL existente | Aceito |
| [ADR-002](ADR-002-worker-separado-em-polling.md) | Worker em processo separado, polling de 2 s | Aceito |
| [ADR-003](ADR-003-retry-backoff-e-dlq.md) | Retry com backoff (5 tentativas) e DLQ em tabela separada | Aceito |
| [ADR-004](ADR-004-hmac-sha256-secret-por-endpoint.md) | HMAC-SHA256 com secret por endpoint e rotação | Aceito |
| [ADR-005](ADR-005-at-least-once-com-x-event-id.md) | At-least-once com deduplicação por `X-Event-Id` | Aceito |
| [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) | Reuso máximo dos padrões do projeto | Aceito |
| [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md) | Payload snapshot e filtro de status na inserção | Aceito |
