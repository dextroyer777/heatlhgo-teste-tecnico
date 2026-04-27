# Desafio Técnico: Arquiteto de Software - HealthGo

## 1. Visão Geral da Arquitetura
Esta solução foi desenhada priorizando a resiliência em cenários de conectividade instável e o desacoplamento para permitir a evolução para um módulo de IA/ML sem redesenho sistêmico.

### Diagrama (C4 Model)
```mermaid
C4Container
    title Arquitetura de Coleta de ECG - HealthGo

    Person(medico, "Profissional de Saúde", "Visualiza exames e indicadores")
    
    System_Boundary(edge, "Clínica (Borda)") {
        Container(ecg, "Aparelho ECG", "Serial USB", "Captura 500 amostras/s")
        Container(agent, "Edge Gateway", "Go", "Store-and-Forward (SQLite), PII Masking")
    }
    
    System_Boundary(cloud, "HealthGo Cloud") {
        Container(broker, "Broker MQTT", "Gerenciado", "Ingestão de alta frequência")
        Container(backend, "Ingestion Service", "Go", "Validação, Normalização, Auditoria")
        Container(tsdb, "Time-Series DB", "InfluxDB/Timescale", "Hot Storage (30 dias)")
        Container(s3, "Object Storage", "S3", "Cold Storage (Histórico 1 ano)")
    }
    
    Container(dash, "Dashboard", "React", "Tempo Real (WebSocket)")
    Container(ml, "ML Service", "Python", "Detecção de Arritmia (Futuro)")

    Rel(ecg, agent, "Serial")
    Rel(agent, broker, "MQTT (QoS 1)")
    Rel(broker, backend, "Consume")
    Rel(backend, tsdb, "Write")
    Rel(backend, ml, "Event Stream")
    Rel(dash, backend, "API/WS")
    Rel(backend, s3, "Async Archive")
    Rel(medico, dash, "Monitora")
