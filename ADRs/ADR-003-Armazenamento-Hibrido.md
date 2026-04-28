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
- **Positivo:** **Otimização de custo (FinOps):** Diferencia o custo de armazenamento de dados "quentes" (processamento intenso) de dados "frios" (arquivamento de longo prazo). Garante que a performance do dashboard seja constante, independente do volume histórico acumulado.
    
- **Negativo:** Complexidade de gestão do ciclo de vida dos dados (Data Lifecycle). Exige a criação e manutenção de Jobs de ETL (Export/Archive) que, se falharem, podem causar _data gap_ entre as camadas, necessitando de alertas de reconciliação.
    
- **Impacto Sistêmico:** Permite uma arquitetura _Analytics-Ready_. Ao salvar em formato colunar (Parquet) no S3, o time de ML consegue processar anos de histórico de ECG de forma eficiente e barata, sem precisar ler bancos relacionais pesados.