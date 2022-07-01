# Data Generator

## Mongo

### Events in format json to mongo

1\. Create job to generate data
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/users01_datagen.config
```
2\. Read events from topic
```bash
docker exec -it kafka01 \
  kafka-console-consumer --bootstrap-server localhost:9092 \
  --topic t.topic01 --from-beginning  \
  --property print.key=true --property key.separator="-"
```

3\. Create job sink mongo
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/users01-sink-mongo.config
```

4\. Review mongo database
```bash
docker exec -it mongo-db mongo admin -u admin -p pwd123456!
show dbs
use data
show collections
db["history"].find()
db["history"].count()
db["history"].remove({})
```

5\. Delete job
```bash
docker exec -it connect-1 \
  curl -X DELETE http://localhost:8083/connectors/Users01SinkMongo
```
## Events in format avro to mongo

1\. Create job to generate data
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/users02_datagen.config
```

2\. List subjects in schema-registry
```bash
docker exec -it schema-registry curl http://localhost:8081/subjects/
docker exec -it schema-registry curl http://localhost:8081/subjects/t.topic02-value/versions
docker exec -it schema-registry curl http://localhost:8081/subjects/t.topic02-value/versions/1
```

3\. Read events from topic
```bash
docker exec -it kafka01 \
  kafka-console-consumer --bootstrap-server localhost:9092 \
  --topic t.topic02 --from-beginning  \
  --property print.key=true --property key.separator="-"
```

4\. Create job sink mongo
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/users02-sink-mongo.config
```

5\. Review mongo database
```bash
docker exec -it mongo-db mongo admin -u admin -p pwd123456!
show dbs
use data
show collections
db["history"].find()
db["history"].find().pretty()
db["history"].count()
db["history"].remove({})
```

6\. Delete job
```bash
docker exec -it connect-1 \
  curl -X DELETE http://localhost:8083/connectors/Users02SinkMongo
```

## Kafka connect commands
```bash
docker exec -it connect-1 /bin/bash
export JOB_NAME="DatagenUsers02"
curl -X GET http://localhost:8083/
curl -X GET http://localhost:8083/connectors
curl -X GET http://localhost:8083/connectors/${JOB_NAME}
curl -X GET http://localhost:8083/connectors/${JOB_NAME}?expand=status
curl -X GET http://localhost:8083/connectors/${JOB_NAME}?expand=info
curl -X GET http://localhost:8083/connectors/${JOB_NAME}/config
curl -X GET http://localhost:8083/connectors/${JOB_NAME}/status

curl -X POST http://localhost:8083/connectors/${JOB_NAME}/restart
curl -X PUT http://localhost:8083/connectors/${JOB_NAME}/pause
curl -X PUT http://localhost:8083/connectors/${JOB_NAME}/resume

curl -X GET http://localhost:8083/connectors/${JOB_NAME}/tasks
curl -X GET http://localhost:8083/connectors/${JOB_NAME}/tasks/0/status
curl -X POST http://localhost:8083/connectors/${JOB_NAME}/tasks/0/restart
curl -X GET http://localhost:8083/connectors/${JOB_NAME}/topics
curl -X DELETE http://localhost:8083/connectors/${JOB_NAME}
```

5\. Review mongo database
```bash
docker exec -it mongo-db mongo admin -u admin -p pwd123456!
show dbs
use data
show collections
db["history"].find()
db["history"].find().pretty()
db["history"].count()
db["history"].remove({})
```

## REDIS

### Events in format json to mongo

1\. Create job sink mongo
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/users01-sink-redis.config
```

2\. View redis
```bash
docker exec -it redis redis-cli -n 2
auth password
keys *
get 3884777984946001977
```

### Events in format avro to mongo

1\. Create job sink mongo
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/users02-sink-redis.config
```

2\. View redis
```bash
docker exec -it redis redis-cli -n 2
auth password
keys *
get -9043479238903486796
```

### Transactions events

1\. Create job to generate data
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/transaction03_datagen.config
```

2\. Read events from topic
```bash
docker exec -it schema-registry \
  kafka-avro-console-consumer --bootstrap-server kafka01:9092 \
  --topic t.topic03 --from-beginning  \
  --property schema.registry.url=http://localhost:8081

```

3\. Delete schema
```bash
docker exec -it schema-registry curl http://localhost:8081/subjects/
docker exec -it schema-registry curl http://localhost:8081/subjects/t.topic03-value/versions
docker exec -it schema-registry curl -X DELETE http://localhost:8081/subjects/t.topic03-value
docker exec -it schema-registry curl -X DELETE http://localhost:8081/subjects/t.topic03-value/versions/1
```

### Cards events
1\. Create job to generate data
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/cards04_datagen.config
```
2\. Read events from topic
```bash
docker exec -it schema-registry \
  kafka-avro-console-consumer --bootstrap-server kafka01:9092 \
  --topic t.topic04 --from-beginning  \
  --property schema.registry.url=http://localhost:8081

```

### Ratings events
1\. Create job to generate data
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/ratings05_datagen.config
```
2\. Read events from topic
```bash
docker exec -it schema-registry \
  kafka-avro-console-consumer --bootstrap-server kafka01:9092 \
  --topic t.topic05 --from-beginning  \
  --property schema.registry.url=http://localhost:8081

```

### Stores events
1\. Create job to generate data
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/stores06_datagen.config
```
2\. Read events from topic
```bash
docker exec -it schema-registry \
  kafka-avro-console-consumer --bootstrap-server kafka01:9092 \
  --topic t.topic06 --from-beginning  \
  --property schema.registry.url=http://localhost:8081

```

### Stock trades events
1\. Create job to generate data
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/stock_trades07_datagen.config
```
2\. Read events from topic
```bash
docker exec -it schema-registry \
  kafka-avro-console-consumer --bootstrap-server kafka01:9092 \
  --topic t.topic07 --from-beginning  \
  --property schema.registry.url=http://localhost:8081
```


### Purchase events
1\. Create job to generate data
```bash
docker exec -it connect-1 \
  curl -X POST http://localhost:8083/connectors \
    -H "Content-Type: application/json" \
    --data @/tmp/data-gen/config/purchase08_datagen.config
```
2\. Read events from topic
```bash
docker exec -it schema-registry \
  kafka-avro-console-consumer --bootstrap-server kafka01:9092 \
  --topic t.topic08 --from-beginning  \
  --property schema.registry.url=http://localhost:8081
```




```bash

## Data Generator
https://github.com/confluentinc/kafka-connect-datagen/tree/master/config
https://github.com/confluentinc/kafka-connect-datagen/blob/master/README.md

https://github.com/confluentinc/kafka-connect-datagen/tree/master/src/main/resources
https://github.com/confluentinc/avro-random-generator/blob/master/README.md

https://developer.confluent.io/learn-kafka/kafka-connect/metrics-and-monitoring/

```

```
curl -X POST \
  http://localhost:8083/connectors \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d '{
  "name": "DatagenConnector01",
  "config": {
    "connector.class": "io.confluent.kafka.connect.datagen.DatagenConnector",
    "kafka.topic": "t.topic01",
    "max.interval": 500,
    "iterations": 100,
    "tasks.max": "100"
    "schema.filename": "/tmp/data-gen/shemas/user.avsc",
    "schema.keyfield": "userId",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "false",
  }
}'


```


