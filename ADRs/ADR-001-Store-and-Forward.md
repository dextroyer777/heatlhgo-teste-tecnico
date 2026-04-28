---
tipo: ADR
status: Aceito
data: 2026-04-27
tags:
  - arquitetura
  - edge
  - resiliencia
---
---
author: Filipe S. Sabino Gomes
tipo: ADR
status: Aceito
data: 2026-04-27
tags: [arquitetura, edge, resiliencia]
---

# ADR 001: Estratégia de Store-and-Forward na Borda

## Contexto
O cenário R2 (conectividade instável, quedas de 5-30 min) exige que o sistema seja *offline-first*. O aparelho de ECG precisa continuar capturando dados sem perda de integridade clínica.

## Decisão
Implementar um agente local (Edge Gateway) com banco de dados embarcado (SQLite) para persistência temporária (buffer) na borda.

## Alternativas Descartadas
1. **Buffer em Memória:** Descartado pois quedas de energia apagariam os dados, comprometendo a auditoria clínica.
2. **Envio Direto (HTTP):** Descartado pois a instabilidade da rede causaria bloqueio no fluxo clínico e perda de pacotes durante as quedas.

## Trade-offs e Impacto
- - **Positivo:** Garante a **integridade clínica e auditoria** dos dados, transformando uma falha de rede (risco de negócio) em um problema de latência gerenciável (problema técnico).
    
- **Negativo:** Introduz um _stateful component_ na borda. Exige monitoramento ativo do disco (evitar _out-of-disk_ em ambiente sem acesso fácil) e implementação de uma estratégia de _pruning_ (limpeza de dados antigos) para evitar crescimento infinito do SQLite.
    
- **Impacto Sistêmico:** Aumenta a responsabilidade do Edge Gateway, que deixa de ser um "pass-through" para virar um nó de armazenamento. Isso requer um pipeline de _deployment_ que inclua validação de saúde do banco (schema migrations).