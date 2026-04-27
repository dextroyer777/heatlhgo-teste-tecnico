# Visão Geral de Infraestrutura

## Contexto Operacional
Considerando a limitação de um time de infraestrutura com apenas 2 pessoas (R5), esta arquitetura prioriza:
1. **Serviços Gerenciados:** Uso de MQTT Broker gerenciado e bancos de dados como serviço (DBaaS) para reduzir o *toil* operacional.
2. **Imutabilidade:** Todos os componentes rodam em containers configurados via Docker, garantindo que o ambiente de dev seja idêntico ao de produção.
3. **Observabilidade:** Centralização de logs e métricas (Prometheus/Grafana) para agilizar o diagnóstico de incidentes sem a necessidade de intervenção manual complexa.