# ADR-004: Autenticação das entregas com HMAC-SHA256 e secret por endpoint

- **Status:** Aceito
- **Data da decisão:** reunião técnica de quinta-feira, [09:22] Sofia
- **Decisores:** Sofia (Segurança), Larissa (Tech Lead), Diego (Plataforma), Bruno (Pedidos)
- **Relacionados:** [ADR-005](ADR-005-at-least-once-com-x-event-id.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)

## Contexto

Os webhooks são **outbound**: só saem da plataforma para o cliente ([09:02] Marcos, [09:03] Sofia). Os eventos carregam dados de pedidos e vão para um endpoint fora da nossa infraestrutura. O cliente precisa validar que a requisição veio de nós e que o payload não foi adulterado no caminho ([09:19] Sofia).

Já houve caso de cliente que vazou uma secret em log da própria aplicação ([09:22] Diego).

## Decisão

1. **HMAC-SHA256 sobre o corpo do request**, enviado no header `X-Signature`. O cliente recalcula e compara do lado dele ([09:20] Sofia, [09:22] Sofia).
2. **Secret única por endpoint de webhook**, nunca global da plataforma ([09:21] Sofia). A secret é **gerada pela plataforma e devolvida na criação** do webhook ([09:31] Marcos).
3. A configuração do webhook guarda `url`, `secret`, `customer_id` e estado ativo ([09:21] Bruno, [09:21] Sofia).
4. **Rotação de secret** via API, sob demanda do cliente. A secret anterior continua válida por **24 horas em paralelo** (grace period) e depois expira ([09:21] Sofia, [09:22] Sofia).
5. **TLS obrigatório**: a URL tem que ser `https`. URL `http` é recusada com erro de validação no schema Zod ([09:23] Sofia).
6. A geração de secret e o HMAC passam por **revisão de segurança da Sofia, com pelo menos 2 dias úteis**, antes do deploy ([09:46] Sofia).

## Alternativas Consideradas

| Alternativa | Por que foi descartada | Fonte |
|---|---|---|
| Secret global da plataforma | "Se vaza uma, vaza tudo" | [09:21] Sofia |
| Rotação com corte imediato (sem grace period) | O cliente precisa de tempo para migrar os sistemas dele; por isso a antiga vale 24 h em paralelo | [09:21] Sofia |
| Aceitar URL `http` | TLS é obrigatório; tratado como validação de schema | [09:23] Sofia |

## Consequências

**Positivas**
- Integridade e autenticidade verificáveis com bibliotecas padrão de mercado ([09:20] Sofia).
- Vazamento de uma secret afeta um único endpoint.
- Rotação sem downtime para o cliente.

**Negativas / trade-offs**
- A secret precisa ser armazenada de forma recuperável (o worker precisa dela para assinar) e nunca pode aparecer em logs. O logger atual (`src/shared/logger/index.ts`) redige `password`, `token` e `authorization`, mas **não** redige `secret`; o módulo precisa estender o redact.
- Durante o grace period de 24 h existem duas secrets válidas para o mesmo endpoint, o que exige modelar a secret anterior e a data de expiração.
- O cliente passa a ter a responsabilidade de validar a assinatura.
- **Trade-off explícito:** mais complexidade de modelagem (duas secrets, expiração) em troca de isolamento de vazamentos e rotação sem interrupção.

## Referências

- Transcrição: [09:02] Marcos, [09:03] Sofia, [09:19] Sofia, [09:20] Sofia, [09:20] Bruno, [09:21] Sofia, [09:21] Bruno, [09:22] Diego, [09:22] Sofia, [09:23] Sofia, [09:31] Marcos, [09:46] Sofia
- Código: `src/shared/logger/index.ts` (`redactPaths`)
