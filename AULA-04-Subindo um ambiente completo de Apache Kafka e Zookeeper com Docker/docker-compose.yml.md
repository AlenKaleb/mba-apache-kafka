# docker-compose.yml (AULA 04)

## Contexto e arquitetura
Este `docker-compose` define um ambiente mínimo para desenvolvimento local com **Zookeeper**, **Kafka Broker** e a UI **AKHQ**. É um setup clássico para estudar o ecossistema Kafka em single-node, usando imagens da Confluent.

**Componentes principais**
- **Zookeeper**: coordenação e metadados do cluster (em versões antigas do Kafka).
- **Broker (cp-server)**: serviço Kafka responsável por tópicos, partições e armazenamento.
- **AKHQ**: interface web para inspecionar tópicos, consumidores e mensagens.

## Trecho anotado
```yaml
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.2.1
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  broker:
    image: confluentinc/cp-server:7.2.1
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_CONFLUENT_LICENSE_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_CONFLUENT_BALANCER_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_JMX_HOSTNAME: localhost
      CONFLUENT_SUPPORT_CUSTOMER_ID: 'anonymous'

  akhq:
    image: tchiotludo/akhq
    environment:
      AKHQ_CONFIGURATION: |
        akhq:
          connections:
            kafka:
              properties:
                bootstrap.servers: "broker:29092"
    depends_on:
     - broker
    ports:
      - 8080:8080
```

## Anotações técnicas
- **`version: '2'`**: especifica o formato do Compose. É simples e compatível com Docker Compose clássico.
- **Serviço `zookeeper`**:
  - `ZOOKEEPER_CLIENT_PORT`: porta padrão (2181) para clientes/servidores se conectarem.
  - `ZOOKEEPER_TICK_TIME`: base de tempo em ms, usada por timeouts de sessão.
- **Serviço `broker`**:
  - `depends_on`: garante que o Zookeeper inicie primeiro (boa prática para ordem de boot, mas não espera *health*).
  - `KAFKA_ZOOKEEPER_CONNECT`: aponta para o Zookeeper do Compose.
  - `KAFKA_ADVERTISED_LISTENERS`: expõe dois listeners:
    - `PLAINTEXT://broker:29092` para comunicação interna entre containers.
    - `PLAINTEXT_HOST://localhost:9092` para clientes locais na máquina host.
  - `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP`: mapeia listeners para protocolos (aqui, sem TLS).
  - `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1`: necessário em single-node para tópicos internos.
  - Configurações de transações e balanceamento com fator 1 evitam falhas em ambiente pequeno.
  - `KAFKA_JMX_HOSTNAME`: facilita monitoramento local.
- **Serviço `akhq`**:
  - `AKHQ_CONFIGURATION` embute YAML com a definição de cluster, apontando para o broker interno.
  - Portas `8080:8080` para acesso via browser.

## Boas práticas destacadas
- **Separação de listeners interno/externo** para evitar problemas de conectividade dentro/fora da rede Docker.
- **Replicação mínima configurada** em tópicos internos para suportar ambiente de estudo sem múltiplos brokers.
- **Ferramenta de observabilidade (AKHQ)** para facilitar entendimento do fluxo de mensagens.
