sudo docker-compose exec kafka kafka-console-producer --topic DatabaseA.Users --broker-list kafka:29092  --property parse.key=true --property key.separator=:
-------
key1:{"user_id":"1","first_name":"bao","last_name":"nguyen"}
key2:{"user_id":"2","first_name":"john","last_name":"brown"}
key3:{"user_id":"3","first_name":"peter","last_name":"parker"}
key4:{"user_id":"4","first_name":"frank","last_name":"hill"}
key5:{"user_id":"5","first_name":"frank 5","last_name":"hill 5"}

 

key6:{"user_id":"6","first_name":"first 6","last_name":"last 6"}
key7:{"user_id":"7","first_name":"first 7","last_name":"last 7"}
key8:{"user_id":"8","first_name":"first 8","last_name":"last 8"}
-------

sudo docker-compose exec kafka kafka-consol... by Bao Nguyen Viet
Bao Nguyen Viet
4:47 PM

sudo docker-compose exec kafka kafka-console-producer --topic DatabaseB.Wages --broker-list kafka:29092  --property parse.key=true --property key.separator=:
-------
key1:{"user_id":"1","wage":"1000"}
key2:{"user_id":"2","wage":"2000"}
key3:{"user_id":"3","wage":"3000"}

docker-compose exec kafka kafka-topics --delete --bootstrap-server \
localhost:9092 --topic DatabaseA.Users

docker-compose exec kafka kafka-topics --create --bootstrap-server \
localhost:9092 --replication-factor 1 --partitions 1 --topic DatabaseA.Users \
--property parse.key=true --property key.separator=:

### Kafka

docker-compose exec kafka kafka-console-producer --bootstrap-server localhost:9092 --topic DatabaseA.Users --property parse.key=true --property key.separator=:

docker-compose exec kafka kafka-console-producer --bootstrap-server localhost:9092 --topic DatabaseB.Wages --property parse.key=true --property key.separator=:

cd /usr/bin
connect-standalone /config/connect-standalone.properties /config/connect-postgres-source.properties
kafka-topics --list --bootstrap-server localhost:9092


### KSQL

Connect to KSQL
```
docker exec -it ksqldb-cli ksql http://ksqldb-server:8088

show topics;

CREATE STREAM stream_users (user_id int, first_name varchar, last_name varchar) WITH (kafka_topic='DatabaseA.Users', value_format='JSON');

DESCRIBE stream_users;

select * from stream_users; by Bao Nguyen Viet
select * from stream_users;


CREATE STREAM stream_wages (user_id int, wage double) WITH (kafka_topic='DatabaseB.Wages', value_format='JSON');



CREATE STREAM stream_user_wages WITH (KAFKA_TOPIC='DatabaseC.UserWages', PARTITIONS=1, REPLICAS=1) AS SELECT stream_users.user_id, first_name + ' ' + last_name as full_name, stream_wages.wage FROM stream_users INNER JOIN stream_wages WITHIN 1 HOURS GRACE PERIOD 15 MINUTES ON stream_users.user_id = stream_wages.user_id;



### PostgreSQL

show wal_level ;
ALTER SYSTEM SET wal_level = logical;


