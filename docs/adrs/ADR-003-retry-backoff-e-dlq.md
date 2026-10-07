# ADR-003: Retry com backoff exponencial (5 tentativas) e DLQ em tabela separada

- **Status:** Aceito
- **Data da decisão:** reunião técnica de quinta-feira, [09:17] Larissa
- **Decisores:** Larissa (Tech Lead), Diego (Plataforma), Bruno (Pedidos), Marcos (PM)
- **Relacionados:** [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-005](ADR-005-at-least-once-com-x-event-id.md)

## Contexto

O endpoint do cliente pode estar lento ou fora do ar. O worker precisa decidir quantas vezes tentar de novo, com que intervalo e o que fazer quando desistir.

- Já houve cliente com indisponibilidade de duas horas em manutenção planejada ([09:16] Diego).
- Evento pendurado para sempre é um problema quando o cliente "sumiu" ([09:15] Diego).
- Uma requisição que não responde em 10 s é tratada como falha e vai para retry ([09:42] Diego).

## Decisão

1. **Backoff exponencial com 5 tentativas** ([09:15] Diego, [09:16] Larissa).
2. **Progressão: 1 min, 5 min, 30 min, 2 h, 12 h**, totalizando quase 15 h entre a primeira falha e a última tentativa ([09:17] Diego, [09:17] Larissa).
   - Interpretação adotada: as 5 tentativas são **retentativas após a primeira falha**, uma para cada intervalo da progressão. É a única leitura que bate com "quase 15 horas entre primeira falha e última tentativa" (1m + 5m + 30m + 2h + 12h = 14h36m). A confirmar na revisão do FDD.
3. Esgotadas as tentativas, o evento é considerado **falha permanente** e movido para a **DLQ** ([09:15] Diego).
4. A DLQ é uma **tabela separada `webhook_dead_letter`**, com o payload, o motivo da falha e o timestamp ([09:18] Diego).
5. **Reprocessamento manual** via `POST /admin/webhooks/dead-letter/:id/replay`, que recoloca o evento na outbox como pendente ([09:18] Diego, [09:35] Diego). O endpoint exige role `ADMIN` e registra quem fez o replay ([09:36] Sofia, [09:36] Larissa).

## Alternativas Consideradas

| Alternativa | Por que foi descartada | Fonte |
|---|---|---|
| Retry indefinido com backoff | Evento fica pendurado para sempre se o cliente sumiu | [09:15] Diego |
| 3 tentativas (mais agressivo) | Cobre só ~30 min; não sobrevive a uma manutenção planejada de 2 h | [09:16] Bruno, [09:16] Diego |
| Marcar como `failed` na própria outbox em vez de tabela separada | Tabela separada deixa a leitura da outbox principal mais limpa e serve de evidência para debug e reprocessamento | [09:17] Larissa, [09:18] Diego |

## Consequências

**Positivas**
- Cobre indisponibilidades de até ~15 h sem intervenção ([09:17] Diego).
- Falha permanente fica visível e auditável em uma tabela própria.
- Replay controlado e auditado, restrito a `ADMIN`.

**Negativas / trade-offs**
- Um cliente fora do ar por mais de ~15 h perde a entrega automática; Marcos considerou aceitável ("ele já tá com problema sério dele", [09:17] Marcos).
- Reprocessamento depende de ação humana; não há alerta automático para o cliente (email adiado, [09:37] Larissa).
- Eventos atrasados por retry podem ser entregues depois de eventos mais novos do mesmo pedido, se o worker seguir processando outros pendentes. A garantia de ordem é limitada por pedido e single-worker ([09:13] Larissa).
- **Trade-off explícito:** priorizamos não ter eventos pendurados para sempre, aceitando que uma janela de indisponibilidade muito longa exige replay manual.

## Referências

- Transcrição: [09:14] Larissa, [09:15] Diego, [09:16] Bruno, [09:16] Diego, [09:17] Diego, [09:17] Marcos, [09:17] Larissa, [09:18] Diego, [09:18] Bruno, [09:35] Diego, [09:36] Sofia, [09:36] Larissa, [09:42] Diego
- Código: `src/middlewares/auth.middleware.ts` (`requireRole`)
