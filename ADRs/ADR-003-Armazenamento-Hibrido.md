---
author: Filipe S. Sabino Gomes
tipo: ADR
status: Aceito
data: 2026-04-27
tags: [arquitetura, storage, dados]
---

# ADR 003: Armazenamento Híbrido (Tiered Storage)

## Contexto
O volume de 500 amostras/s por aparelho gera uma carga de escrita massiva. Precisamos de performance para o dashboard (30s) e conformidade histórica (1 ano).

## Decisão
Arquitetura *Tiered*: InfluxDB (Hot Storage - 30 dias) para alta performance e escrita rápida, com exportação assíncrona para S3 (Cold Storage - 1 ano) em formato Parquet.

## Alternativas Descartadas
1. **RDBMS único (Postgres):** Descartado devido à amplificação de escrita (*write amplification*) e lentidão em consultas temporais com alto volume.

## Trade-offs e Impacto
- **Positivo:** O InfluxDB absorve a carga de escrita constante sem bloquear as leituras do dashboard. O S3 reduz drasticamente os custos de retenção de longo prazo.
- **Negativo:** Aumenta a complexidade operacional, exigindo um job de ETL/Archive para mover dados entre os Tiers.
- **Impacto Sistêmico:** Permite que o time de ML (que chegará em 6 meses) consuma dados estruturados e otimizados (Parquet) sem impactar a performance da aplicação em tempo real.