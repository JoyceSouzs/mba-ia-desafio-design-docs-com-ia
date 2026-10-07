# ADR-005: Garantia de entrega at-least-once com deduplicação por X-Event-Id

- **Status:** Aceito
- **Data da decisão:** reunião técnica de quinta-feira, [09:26] Larissa
- **Decisores:** Diego (Plataforma), Larissa (Tech Lead), Sofia (Segurança), Marcos (PM)
- **Relacionados:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-backoff-e-dlq.md)

## Contexto

Com outbox ([ADR-001](ADR-001-outbox-no-mysql.md)) e retry ([ADR-003](ADR-003-retry-backoff-e-dlq.md)), existem cenários em que o cliente recebe o mesmo evento mais de uma vez. Por exemplo: o cliente processa a requisição, mas a resposta não chega ao worker dentro do timeout de 10 s ([09:42] Diego), e o evento é reenviado. O replay manual da DLQ também reenvia eventos ([09:18] Diego).

## Decisão

1. A plataforma garante entrega **at-least-once**: o cliente pode receber o mesmo evento mais de uma vez e precisa estar preparado ([09:24] Diego).
2. Cada evento recebe um **UUID gerado no momento da inserção na outbox**, enviado no header **`X-Event-Id`** e no campo `event_id` do payload. É único por evento ([09:25] Diego, [09:43] Diego).
3. O cliente **deduplica pelo `event_id`** do lado dele ([09:25] Diego, [09:26] Larissa).
4. O comportamento será documentado em destaque no portal de desenvolvedor ([09:26] Marcos).

## Alternativas Consideradas

| Alternativa | Por que foi descartada | Fonte |
|---|---|---|
| Exactly-once | Exige coordenação dos dois lados e fica muito mais complexo; at-least-once com event_id resolve a grande maioria dos casos | [09:25] Diego |
| At-most-once (sem retry) | Incompatível com a política de retry decidida para cobrir indisponibilidade do cliente | [09:15] Diego, [09:17] Larissa |

## Consequências

**Positivas**
- Nenhum evento é perdido silenciosamente: ou é entregue, ou vai para a DLQ.
- Padrão de mercado conhecido pelos clientes ("Stripe faz assim, GitHub faz assim", [09:25] Diego).
- Implementação simples do nosso lado.

**Negativas / trade-offs**
- **Responsabilidade transferida ao cliente**, como Sofia apontou ([09:25] Sofia). Cliente que não deduplica pode processar uma mudança de status duas vezes.
- O `event_id` precisa ser estável: o replay da DLQ deve reaproveitar o mesmo `event_id`, senão a deduplicação do cliente não funciona.
- **Trade-off explícito:** simplicidade e robustez do nosso lado em troca de exigir idempotência no consumidor.

## Referências

- Transcrição: [09:18] Diego, [09:24] Diego, [09:25] Bruno, [09:25] Diego, [09:25] Sofia, [09:26] Marcos, [09:26] Larissa, [09:42] Diego, [09:43] Diego
