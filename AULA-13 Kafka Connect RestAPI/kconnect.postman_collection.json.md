# kconnect.postman_collection.json (AULA 13)

## Contexto e arquitetura
Coleção do **Postman** para interagir com a **REST API do Kafka Connect**. Serve como documentação executável de endpoints para consultar o cluster, listar connectors e gerenciar instâncias.

## Trecho anotado
```json
{
  "info": {
    "_postman_id": "4d23e385-b847-4068-8d25-e26509e7f9c1",
    "name": "kconnect",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json",
    "_exporter_id": "4318915"
  },
  "item": [
    {
      "name": "InfoCluster",
      "request": {
        "auth": { "type": "noauth" },
        "method": "GET",
        "header": [],
        "url": { "raw": "localhost:8083", "host": ["localhost"], "port": "8083" }
      },
      "response": []
    },
    {
      "name": "Connectors",
      "request": {
        "auth": { "type": "noauth" },
        "method": "GET",
        "header": [],
        "url": {
          "raw": "localhost:8083/connectors",
          "host": ["localhost"],
          "port": "8083",
          "path": ["connectors"]
        }
      },
      "response": []
    },
    {
      "name": "ConnectorsStatus",
      "request": {
        "auth": { "type": "noauth" },
        "method": "GET",
        "header": [],
        "url": {
          "raw": "localhost:8083/connectors/meu-conector-source/status",
          "host": ["localhost"],
          "port": "8083",
          "path": ["connectors", "meu-conector-source", "status"]
        }
      },
      "response": []
    },
    {
      "name": "ConnectorsDetails",
      "request": {
        "auth": { "type": "noauth" },
        "method": "GET",
        "header": [],
        "url": {
          "raw": "localhost:8083/connectors/meu-conector-source/status",
          "host": ["localhost"],
          "port": "8083",
          "path": ["connectors", "meu-conector-source", "status"]
        }
      },
      "response": []
    },
    {
      "name": "ConnectorPlugins",
      "request": {
        "auth": { "type": "noauth" },
        "method": "GET",
        "header": [],
        "url": {
          "raw": "localhost:8083/connector-plugins",
          "host": ["localhost"],
          "port": "8083",
          "path": ["connector-plugins"]
        }
      },
      "response": []
    },
    {
      "name": "Connectors",
      "request": {
        "auth": { "type": "noauth" },
        "method": "POST",
        "header": [],
        "body": {
          "mode": "raw",
          "raw": "{\r\n    \"name\": \"gcs-test\",\r\n    \"config\": {\r\n\r\n            \"connector.class\": \"io.aiven.kafka.connect.gcs.GcsSinkConnector\",\r\n            \"tasks.max\": \"1\",\r\n            \"key.converter\": \"org.apache.kafka.connect.storage.StringConverter\",\r\n            \"value.converter\": \"org.apache.kafka.connect.json.JsonConverter\",\r\n            \"topics\": \"gcs-test-topic-source\",\r\n            \"gcs.credentials.path\": \"/data/gcs-key.json\",\r\n            \"gcs.bucket.name\": \"mey-bucket\",\r\n            \"format.output.type\": \"json\"\r\n    }\r\n}",
          "options": { "raw": { "language": "json" } }
        },
        "url": {
          "raw": "localhost:8083/connectors/meu-conector-source/status",
          "host": ["localhost"],
          "port": "8083",
          "path": ["connectors", "meu-conector-source", "status"]
        }
      },
      "response": []
    },
    {
      "name": "DeleteConnectors",
      "request": {
        "auth": {
          "type": "basic",
          "basic": [
            { "key": "password", "value": "GWftQJRQP90c4K35", "type": "string" },
            { "key": "username", "value": "connect", "type": "string" }
          ]
        },
        "method": "DELETE",
        "header": [],
        "url": {
          "raw": "localhost:8083/connectors/meu-conector-source",
          "host": ["localhost"],
          "port": "8083",
          "path": ["connectors", "meu-conector-source"]
        }
      },
      "response": []
    }
  ]
}
```

## Anotações técnicas
- **`info`**: metadados usados pelo Postman para identificar a coleção.
- **`item`**: lista de requisições. Cada item mapeia um endpoint do Kafka Connect.
- **InfoCluster (`GET /`)**: retorna metadados do cluster do Kafka Connect.
- **Connectors (`GET /connectors`)**: lista conectores registrados.
- **ConnectorsStatus (`GET /connectors/{name}/status`)**: status operacional do connector e tarefas.
- **ConnectorPlugins (`GET /connector-plugins`)**: lista plugins disponíveis no worker.
- **Connectors (`POST /connectors`)**:
  - Corpo cria um connector `GcsSinkConnector` com configurações básicas.
  - Exemplo de uso de conversores (`StringConverter`, `JsonConverter`).
  - Define o tópico de origem e parâmetros do GCS.
- **DeleteConnectors (`DELETE /connectors/{name}`)**:
  - Remove o connector; a autenticação básica indica cenário com segurança habilitada.

## Boas práticas e padrões
- **API REST como contrato**: usar coleções Postman garante documentação viva e reproduzível.
- **Separar ambientes**: prefira variáveis de ambiente no Postman para host/porta (ex.: `{{connect_host}}`).
- **Segurança**: credenciais nunca devem ser fixas em repositórios públicos; usar variáveis seguras ou secrets.
