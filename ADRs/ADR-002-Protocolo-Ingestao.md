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
- **Positivo:** Alta eficiência de banda (baixo overhead de headers), essencial para a variabilidade de qualidade das redes de clínicas. O padrão Pub/Sub desacopla totalmente a origem (Edge) do consumo (Backend), permitindo que você adicione novos consumidores (ex: serviço de auditoria, serviço de ML) sem alterar o Agente de borda.
    
- **Negativo:** Introduz uma dependência crítica: o Broker MQTT. Se o Broker cair, todo o fluxo para. Exige estratégias de _High Availability_ (HA) e gestão de certificados (TLS) em escala para +1.000 clínicas.
    
- **Impacto Sistêmico:** A maturidade do sistema dependerá da gestão do Broker. Ao optar por um serviço gerenciado, transferimos a complexidade operacional da infraestrutura para o provedor, alinhando-se à limitação da squad de 2 pessoas.