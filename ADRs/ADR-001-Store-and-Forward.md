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
- **Positivo:** Garante a integridade dos dados clínicos independente da rede.
- **Negativo:** Aumenta a complexidade de gestão do dispositivo na ponta (necessita de monitoramento de disco e gestão de arquivos).
- **Impacto Sistêmico:** Exige que o time de infraestrutura crie mecanismos de *auto-update* e monitoramento de saúde para os Agentes de borda.