---
author: Filipe S. Sabino Gomes
tipo: ADR
status: Aceito
data: 2026-04-27
tags: [arquitetura, iot, mensageria]
---

# ADR 002: Protocolo de Ingestão MQTT

## Contexto
Precisamos transportar 500 amostras/s por aparelho com baixo *overhead* para evitar consumo excessivo de banda em clínicas com internet instável.

## Decisão
Uso de MQTT (QoS 1) para a telemetria entre o Agente de borda e o Broker central.

## Alternativas Descartadas
1. **Kafka (na ponta):** Descartado pela complexidade de infraestrutura. Exigir um cluster Kafka em +1.000 clínicas é inviável para uma squad de infra de apenas 2 pessoas.
2. **REST/HTTP:** Descartado pelo alto custo de *header* por mensagem, que sobrecarrega a rede em conexões lentas.

## Trade-offs e Impacto
- **Positivo:** Protocolo nativo para IoT, com baixo consumo de banda e suporte a desconexões (funcionalidade *Last Will*).
- **Negativo:** Exige a implementação de uma lógica de *ack* (QoS) na aplicação para garantir a entrega.
- **Impacto Sistêmico:** Simplifica a infraestrutura na nuvem ao permitir o uso de Brokers gerenciados, reduzindo o *toil* do time de infra.