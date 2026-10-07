# Registro da entrevista: PRD

Execução do "Prompt de Entrevista para Gerar PRD" com a regra adicional de que **toda resposta do usuário vem da `TRANSCRICAO.md` ou do código**. O que a reunião não responde fica marcado como **[HIPÓTESE]** (default do prompt, explícito) ou **[EM ABERTO]**.

Legenda: **P** = pergunta do entrevistador (IA) · **R** = resposta extraída da fonte.

---

## 1. Contexto e visão geral

**P:** Qual é o produto ou sistema em que essa feature entra?
**R:** O Order Management System (OMS) em produção: API Node.js + TypeScript, MySQL via Prisma (`package.json`, `prisma/schema.prisma`).

**P:** Sistema existente ou novo?
**R:** Existente. Novo módulo dentro da mesma aplicação, mais um processo worker ([09:27] Bruno, [09:11] Larissa).

**P:** Quem é o público-alvo?
**R:** Clientes B2B que integram via API: Atlas Comercial, MaxDistribuição e Nova Cargo ([09:00] Marcos), por meio de usuários que os representam ([09:32] Marcos). Secundário: ADMINs que operam a DLQ ([09:36] Sofia).

**P:** Objetivo de negócio em duas ou três frases?
**R:** Notificar em tempo real as mudanças de status dos pedidos para tirar os clientes do polling e reter a Atlas, que ameaçou migrar para um concorrente ([09:00] Marcos).

> Resumo confirmado: webhooks outbound ([09:02] Marcos) no OMS existente, para clientes B2B, motivados por pedido formal e risco de churn.

## 2. Problema e oportunidade

**P:** O que acontece hoje?
**R:** Clientes batem em `GET /orders` periodicamente, o que deixa a integração lenta e cara ([09:00] Marcos).

**P:** Exemplo com números?
**R:** Três clientes fizeram pedido formal; a Atlas condicionou a permanência à entrega até o fim do trimestre ([09:00] Marcos) e depois pediu fim de novembro ([09:45] Marcos). Custos de polling não foram quantificados: **[EM ABERTO]**.

**P:** O que já foi tentado?
**R:** Nada. O código não tem eventos, filas nem webhooks.

## 3. Objetivos e métricas

**P:** Que resultado mensurável?
**R:** Notificação abaixo de 10 s ([09:02] Marcos). Polling de 2 s aceito pelo PM ([09:10] Marcos).

**P:** Outras metas?
**R:** Os três clientes integrados ([09:00] Marcos); entrega em 3 sprints, incluindo revisão de segurança ([09:47] Larissa); nenhuma mudança de status sem evento ([09:40] Bruno).

## 4. Escopo

**P:** O que precisa estar pronto?
**R:** Cadastro e gestão de webhooks, filtro de status, entrega assinada, retry, DLQ com replay e histórico de entregas ([09:31] a [09:36]).

**P:** O que está fora?
**R:** Email de alerta ([09:37] Larissa); dashboard ([09:40] Larissa); rate limiting ([09:39] Larissa); arquivamento ([09:08] Diego); inbound ([09:02] Marcos); ordering global ([09:13] Larissa).

## 5. Requisitos funcionais

**P:** Liste os requisitos.
**R:** Lista ditada pelo Marcos e pelo Bruno em [09:31] a [09:35], mais rotação de secret ([09:21] Sofia), retry/DLQ ([09:17] Larissa) e o enfileiramento na mudança de status ([09:40] Bruno). Prioridades: tudo que a Larissa repetiu no resumo final ([09:48] Larissa) entrou como alta; o histórico de entregas, ausente do resumo, entrou como média.

**P:** Como o customer é identificado?
**R:** Endpoint autenticado normal; `customer_id` no body ou no path, não pelo JWT ([09:32] Larissa). Forma final **[EM ABERTO]**.

## 6. Requisitos não funcionais

**P:** Performance?
**R:** < 10 s ([09:02] Marcos); timeout de 10 s por chamada ([09:42] Diego); payload ≤ 64 KB ([09:24] Larissa).

**P:** Disponibilidade?
**R:** Não discutida em número. Discutido: mudança de status não pode depender do cliente ([09:04] Bruno) e o worker não pode cair junto com a API ([09:11] Diego). **[HIPÓTESE]** default do prompt para sistema voltado a cliente externo: 99,9%.

**P:** Segurança?
**R:** HMAC-SHA256, secret por endpoint, rotação com 24 h, TLS obrigatório ([09:20] a [09:23] Sofia); ADMIN no replay com auditoria ([09:36] Sofia).

**P:** Observabilidade?
**R:** Histórico de entregas para o cliente ([09:34] Marcos); logs Pino ([09:29] Bruno); auditoria de replay ([09:36] Sofia). Métricas: **[HIPÓTESE]** derivadas, ver FDD.

## 7 e 8. Arquitetura, decisões e trade-offs

**R:** Sete decisões registradas como ADR. Ver `docs/adrs/`.

## 9. Dependências

**R:** Revisão de segurança da Sofia, 2 dias úteis ([09:46] Sofia); documentação no portal pelo Marcos ([09:26] Marcos); confirmação de prazo com a Atlas ([09:47] Marcos); ambiente para rodar o worker como processo separado ([09:11] Diego).

## 10. Riscos

**R:** Churn da Atlas ([09:00] Marcos); cliente não deduplica ([09:25] Sofia); vazamento de secret ([09:22] Diego); rajada de eventos ([09:38] Diego); cliente fora do ar por mais de 15 h ([09:17] Marcos).

## 11. Critérios de aceitação

Derivados dos itens acima, em formato verificável.

## 12. Testes e validação

**R:** Revisão de segurança ([09:46] Sofia); testes ponta a ponta da integração com `order.service` ([09:46] Larissa); sessão de revisão de design ([09:50] Larissa). Tipos de teste seguem o padrão Vitest/Supertest do repositório (`tests/`).
