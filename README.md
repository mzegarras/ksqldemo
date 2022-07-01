# KSQL

![Architecture](/resources/ksqldb-architecture.png?raw=true "Employee Data title")


## 1 Clean
```bash
cd /Users/manuelzegarra/Proyectos/Presentaciones/ksql
docker-compose down
docker volume rm $(docker volume ls -qf dangling=true)
```

## 2 Startup

```bash
docker-compose up -d
```

## 3 Registrar schemas avros

```bash
jq '. | {schema: tojson}' ./schemas/TransactionEvent.avsc  | \
    curl -X POST http://localhost:8081/subjects/t.transactions-value/versions \
         -H "Content-Type:application/json" \
         -d @-

jq '. | {schema: tojson}' ./schemas/Otp.avsc  | \
    curl -X POST http://localhost:8081/subjects/t.otp-value/versions  \
         -H "Content-Type:application/json" \
         -d @-
         
jq '. | {schema: tojson}' ./schemas/Otp.avsc  | \
    curl -X POST http://localhost:8081/subjects/t.opt.user-value/versions  \
         -H "Content-Type:application/json" \
         -d @-

curl -s http://localhost:8081/subjects | jq "."
curl -s http://localhost:8081/subjects/t.transactions-value/versions | jq "."
curl -s http://localhost:8081/subjects/t.transactions-value/versions/1 | jq "."

```

## 4 Registrar topics

```bash
docker exec kafka01 kafka-topics --bootstrap-server kafka01:9092 \
    --create \
    --topic t.transactions \
    --partitions 3 \
    --replication-factor 1

docker exec kafka01 kafka-topics --bootstrap-server kafka01:9092 \
    --create \
    --topic t.opt \
    --partitions 3 \
    --replication-factor 1

docker exec kafka01 kafka-topics --bootstrap-server kafka01:9092 \
    --create \
    --topic t.opt.user \
    --partitions 3 \
    --replication-factor 1
    
```

## 5 Connect ksqlDB
```bash
docker exec -it ksqldb-cli ksql http://ksqldb-server:8088
SET 'auto.offset.reset' = 'earliest';
show topics;

print "t.transactions" limit 10;
print "t.transactions" from beginning;
```

## 6 Crear streams transactions

```bash
CREATE STREAM TRANSACTIONS_ST WITH (
    kafka_topic='t.transactions', 
    value_format='avro',
    timestamp = 'created'
  );

SELECT * FROM transactions_st EMIT CHANGES;
describe TRANSACTIONS_ST extended;
```

## 7 Transacciones por cuenta
```bash
CREATE OR REPLACE TABLE cta_transactions_tb AS
select originId,SUM(amount) AS TOTAL,COUNT(1) AS COUNT 
FROM transactions_st  
where status ='CREATED' 
group by originId 
emit changes;

select * from cta_transactions_tb emit changes;
SELECT * FROM transactions_st where TRANSACTIONID='3' EMIT CHANGES;
```

## 8 Transacciones por canal
```bash
CREATE OR REPLACE TABLE channel_transactions_tb AS  
select channel,SUM(amount) AS TOTAL,COUNT(1) AS COUNT 
FROM transactions_st
where status ='CREATED' 
group by channel 
emit changes;   

select * from channel_transactions_tb emit changes;
```

## 9 Crear data dummy
```bash

https://docs.ksqldb.io/en/0.10.2-ksqldb/developer-guide/test-and-debug/generate-custom-test-data/

docker exec ksqldb-examples ksql-datagen \
  schema="/tmp/data-gen/shemas/transactions.avsc" \
  format=avro \
  topic="t.transactions" \
  key=transactionId \
  maxInterval=50 \
  bootstrap-server="kafka01:9092" \
  schemaRegistryUrl="http://schema-registry:8081" \
  iterations=100
  
  #msgRate=-1

docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/transaction_datagen.config

docker exec -it connect-1 \
  curl -X DELETE http://localhost:8083/connectors/DatagenUsers01
```

## 10 Analizar 
```bash
describe channel_transactions_tb;

show queries;

EXPLAIN CTAS_CHANNEL_TRANSACTIONS_TB_7;

https://zz85.github.io/kafka-streams-viz/
```