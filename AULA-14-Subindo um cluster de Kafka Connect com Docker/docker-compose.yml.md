# docker-compose.yml (AULA 14)

## Contexto e arquitetura
Este Compose cria um ambiente completo com **Zookeeper**, **Kafka Broker**, **Kafka Connect** (worker standalone) e **AKHQ**. A arquitetura segue o padrão clássico de pipelines de integração usando Kafka Connect.

## Trecho anotado
```yaml
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.2.1
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

  kconnect:
    image: confluentinc/cp-kafka-connect:6.2.4
    ports:
     - 8083:8083
     - 5005:5005
    environment:
     - CONNECT_BOOTSTRAP_SERVERS=broker:29092
     - CONNECT_GROUP_ID=kconnect
     - CONNECT_PLUGIN_PATH=/data/plugins
     - CONNECT_CLIENT_ID=kconnect
     - CONNECT_CONFIG_STORAGE_TOPIC=_kconnect-configs
     - CONNECT_OFFSET_STORAGE_TOPIC=_kconnect-offsets
     - CONNECT_STATUS_STORAGE_TOPIC=_kconnect-status
     - CONNECT_REST_ADVERTISED_HOST_NAME=kconnect
     - KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1
     - CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR=1
     - CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR=1
     - CONNECT_STATUS_STORAGE_REPLICATION_FACTOR=1
     - CONNECT_PRODUCER_COMPRESSION_TYPE=lz4
     - CONNECT_CONSUMER_FETCH_MIN_BYTES=10000
     - CONNECT_CONSUMER_MAX_POLL_INTERVAL_MS=1800000
     - CONNECT_OFFSET_FLUSH_TIMEOUT_MS=600000
     - TZ=America/Sao_Paulo
     - CONNECT_KEY_CONVERTER=org.apache.kafka.connect.storage.StringConverter
     - CONNECT_VALUE_CONVERTER=org.apache.kafka.connect.storage.StringConverter
     - KAFKA_OPTS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005
    volumes:
     - "./data:/data/"
    healthcheck:
      disable: true
    depends_on:
     - broker

  akhq:
    image: tchiotludo/akhq
    environment:
      AKHQ_CONFIGURATION: |
        akhq:
          connections:
            kafka:
              properties:
                bootstrap.servers: "broker:29092"
              connect:
                - name: kafka-connect
                  url: http://kconnect:8083
    depends_on:
     - broker
     - kconnect
    ports:
      - 8080:8080
```

## Anotações técnicas
- **Kafka Connect (`kconnect`)**:
  - `CONNECT_*` define o worker e seus tópicos internos (configs, offsets, status).
  - `CONNECT_PLUGIN_PATH` aponta para diretório onde plugins adicionais são montados via volume.
  - `CONNECT_KEY_CONVERTER` e `CONNECT_VALUE_CONVERTER` definem como keys/values são serializados.
  - `KAFKA_OPTS` habilita depuração remota Java (porta 5005).
  - `healthcheck: disable` evita que Compose tente interpretar um healthcheck inexistente na imagem.
- **AKHQ**:
  - Configura conexão ao Kafka e ao Connect REST API para inspecionar conectores.

## Boas práticas e padrões
- **Tópicos internos com replicação 1** em ambiente single-node.
- **Volumes** para plugins do Connect, permitindo extensões sem rebuild de imagem.
- **Separação de listeners** para evitar problemas de advertized listeners no Docker.
