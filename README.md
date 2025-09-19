# ksql-poc

## Passo a Passo

### 1. Criar o tópico

```bash
docker-compose exec broker kafka-topics --create --topic pedidos --bootstrap-server broker:29092 --partitions 1 --replication-factor 1
```

### 2. Criar o stream e a query

```sql
CREATE STREAM pedidos_stream (
    id VARCHAR KEY,
    produto VARCHAR,
    valor DECIMAL(10, 2)
) WITH (
    KAFKA_TOPIC = 'pedidos',
    VALUE_FORMAT = 'JSON'
);

CREATE STREAM pedidos_caros_v2
WITH (
    KAFKA_TOPIC = 'processado-pedidos-caros-v1',
    VALUE_FORMAT = 'JSON'
) AS SELECT
    *
FROM
    pedidos_stream -- Nosso stream de origem
WHERE
    valor > 200.00;
```

### 3. Produzir dados no tópico

```bash
docker-compose exec broker kafka-console-producer --topic pedidos --bootstrap-server broker:29092
```

Exemplo de mensagens para produzir:

```json
{"id":"1","produto":"Notebook","valor":2500.00}
{"id":"2","produto":"Mouse","valor":50.00}
{"id":"3","produto":"Monitor","valor":800.00}
{"id":"4","produto":"Teclado","valor":150.00}
{"id":"5","produto":"Cadeira Gamer","valor":1200.00}
{"id":"6","produto":"Pen Drive","valor":80.00}
{"id":"7","produto":"Smartphone","valor":1800.00}
{"id":"8","produto":"Fone de Ouvido","valor":300.00}
{"id":"9","produto":"Webcam","valor":220.00}
{"id":"10","produto":"Roteador","valor":400.00}
```

### 4. Consumir os dados processados

```bash
docker-compose exec broker kafka-console-consumer --topic processado-pedidos-caros-v1 --bootstrap-server broker:29092 --from-beginning --property print.key=true --property value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
```