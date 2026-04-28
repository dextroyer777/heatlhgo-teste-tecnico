# Documento de Arquitetura de Alto Nível (HLD): Sistema HealthGo

## 1. Visão Geral

O sistema HealthGo é uma **Arquitetura Distribuída Híbrida (Edge-to-Cloud)** projetada para coleta contínua de ECG. O sistema prioriza a integridade dos dados na ponta (clínica) e a escalabilidade elástica na nuvem, otimizando custos e reduzindo o esforço operacional de um time enxuto.

## 2. Princípios Arquiteturais (Drivers)

- **Resiliência (Offline-First):** O sistema deve tolerar quedas de rede de até 30 minutos sem perda de dados, utilizando armazenamento local na borda.
    
- **Eficiência Operacional:** Foco em serviços gerenciados e automação (IaC) para minimizar o _toil_ de uma equipe de apenas 2 pessoas.
    
- **Escalabilidade de Armazenamento:** Uso de _Tiered Storage_ (dados quentes no InfluxDB, dados frios no S3) para sustentar alto volume de dados sem degradação de performance.
    

## 3. Arquitetura do Sistema (C4 Model)

```
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
```

## 4. Decomposição Tecnológica e ADRs

Para garantir a rastreabilidade do projeto, cada componente principal está vinculado a uma Decisão de Arquitetura (ADR):

| **Camada**        | **Tecnologia** | **ADR** | **Justificativa**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ----------------- | -------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Borda**         | SQLite         |         | Buffer local para garantir integridade durante quedas de internet.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Ingestão**      | MQTT (QoS 1)   |         | Protocolo leve, ideal para banda instável e eficiência de cabeçalho.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **Armazenamento** | InfluxDB + S3  |         | _Tiered Storage_ evita _write amplification_ e otimiza custos operacionais.<br><br>**Hot Storage (Camada Quente - InfluxDB/Timescale):**<br><br>- **Finalidade:** Armazena os dados de ECG mais recentes (ex: últimos 30 dias).<br>    <br>- **Por que:** Banco de dados de séries temporais (TSDB) otimizado para _queries_ de alta frequência e baixa latência, essenciais para o dashboard de monitoramento em tempo real.<br><br>**Cold Storage (Camada Fria - AWS S3):**<br><br>- **Finalidade:** Armazena o histórico completo (1 ano ou mais).<br>    <br>- **Por que:** Otimização de custo severa. O S3 permite arquivamento em formato colunar (ex: Parquet), que é extremamente eficiente para análise de grandes volumes de dados (batch/ML) sem o custo de licenciamento ou infraestrutura de um banco de dados relacional. |
