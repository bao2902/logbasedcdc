# Context 1

- We have a Users table (fields: user_id, first_name, last_name) in a source PostgreSQL database.
- We have a Wages table (fields: user_id, wage) in a soure PostgreSQL database.
- We have a Users table (fields: user_id, first_name, last_name) in a sink PostgreSQL database.
- We have a Wages table (fields: user_id, wage) in a sink PostgreSQL database.
- We want to synchronize the Change Data Capture (CDC) to the tables Users and Wages in a sink PostgreSQL database when the tables Users and Wages are inserted, updated, or deleted in the source PostgreSQL database.
- We expect this synchonization happenning in near-real-time.

# Context 2

- We have a Users table (fields: user_id, first_name, last_name) in a source PostgreSQL database.
- We have a Wages table (fields: user_id, wage) in a soure PostgreSQL database.
- We have a User_Wages table (fields: user_id, full_name, wage) in a sink PostgreSQL database.
- We want to synchronize the Change Data Capture (CDC) to the table User_Wages in a sink PostgreSQL database when the tables Users and Wages are inserted, updated, or deleted in the source PostgreSQL database.
- This synchonization includes the data tranforming as following:
+ User_Wages.user_id = Users.user_id
+ User_Wages.full_name = Users.first_name + ' ' + Users.last_name
+ User_Wages.wage = Wages.wage
- We expect this synchonization happenning in near-real-time.

# Architecture

![alt text](https://github.com/bao2902/logbasedcdc/blob/main/LogBasedCDC.png?raw=true)

# Build Dockerfile

sudo docker build -t nashtech/kafka .

# Start Docker Compose

sudo docker-compose up -d

# Access Kafka container and run Kafka

sudo docker exec -t -i kafka /bin/bash

cd /bin

connect-standalone /config/connect-standalone.properties /config/connect-postgres-source.properties /config/connect-postgres-sink-users.properties /config/connect-postgres-sink-wages.properties /config/connect-postgres-sink-user-wages.properties

# Access kSQL client

sudo docker-compose exec ksqldb-cli ksql http://ksqldb-server:8088

# Create kSQL stream for users

CREATE STREAM stream_users (
schema varchar, 
payload STRUCT<
	before varchar,
	after STRUCT<
		user_id int,
		first_name varchar,
		last_name varchar
	>,
	source varchar,
	op varchar,
	ts_ms bigint
>
)
WITH (kafka_topic='localhost.public.users', value_format='JSON');


# Create kSQL stream for wages

CREATE STREAM stream_wages (
schema varchar, 
payload STRUCT<
	before varchar,
	after STRUCT<
		user_id int,
		wage bigint
	>,
	source varchar,
	op varchar,
	ts_ms bigint
>
)
WITH (kafka_topic='localhost.public.wages', value_format='JSON');


# Create kSQL stream for user_wages

CREATE STREAM stream_user_wages 
WITH (KAFKA_TOPIC='sink_database.user_wages', value_format='KAFKA', PARTITIONS=1, REPLICAS=1) 
AS 
SELECT 
'{"schema":{"type":"struct","fields":[{"type":"int32","optional":false,"field":"user_id"}],"optional":false,"name":"sink_database.user_wages_1.Key"},"payload":{"user_id":' + CAST(stream_users.payload->after->user_id AS VARCHAR) + '}}' as key,
'{"schema":{"type":"struct","fields":[{"type":"struct","fields":[{"type":"int32","optional":false,"field":"user_id"},{"type":"string","optional":true,"field":"full_name"},{"type":"int32","optional":true,"field":"wage"}],"optional":true,"name":"sink_database.user_wages_1.Value","field":"before"},{"type":"struct","fields":[{"type":"int32","optional":false,"field":"user_id"},{"type":"string","optional":true,"field":"full_name"},{"type":"int32","optional":true,"field":"wage"}],"optional":true,"name":"sink_database.user_wages_1.Value","field":"after"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"version"},{"type":"string","optional":false,"field":"connector"},{"type":"string","optional":false,"field":"name"},{"type":"int64","optional":false,"field":"ts_ms"},{"type":"string","optional":true,"name":"io.debezium.data.Enum","version":1,"parameters":{"allowed":"true,last,false"},"default":"false","field":"snapshot"},{"type":"string","optional":false,"field":"db"},{"type":"string","optional":false,"field":"schema"},{"type":"string","optional":false,"field":"table"},{"type":"int64","optional":true,"field":"txId"},{"type":"int64","optional":true,"field":"lsn"},{"type":"int64","optional":true,"field":"xmin"}],"optional":false,"name":"io.debezium.connector.postgresql.Source","field":"source"},{"type":"string","optional":false,"field":"op"},{"type":"int64","optional":true,"field":"ts_ms"}],"optional":false,"name":"sink_database.user_wages_1.Envelope"},"payload":{"before":null,"after":{"user_id":' + CAST(stream_users.payload->after->user_id AS VARCHAR) 
+ ',"full_name":"' + stream_users.payload->after->first_name + ' ' + stream_users.payload->after->last_name 
+ '","wage":' + CAST(stream_wages.payload->after->wage AS VARCHAR) 
+ '},"source":{"version":"0.10.0.Final","connector":"postgresql","name":"localhost","ts_ms":1682324643158,"snapshot":"false","db":"postgres","schema":"public","table":"user_wages","txId":831,"lsn":23268184,"xmin":null},"op":"c","ts_ms":1682324669819}}'
FROM stream_users 
INNER JOIN stream_wages 
WITHIN 1 HOURS GRACE PERIOD 15 MINUTES 
ON stream_users.payload->after->user_id = stream_wages.payload->after->user_id
PARTITION BY '{"schema":{"type":"struct","fields":[{"type":"int32","optional":false,"field":"user_id"}],"optional":false,"name":"sink_database.user_wages_1.Key"},"payload":{"user_id":' + CAST(stream_users.payload->after->user_id AS VARCHAR) + '}}'
;


# Access source Postgres and insert/update/delete data for users and wages

sudo docker exec -it  postgres-source /bin/bash

psql -U postgres

INSERT INTO users VALUES(1, 'first 1', 'last 1');

UPDATE users SET first_name = 'first 1 updated' WHERE user_id = 1;

DELETE FROM users WHERE user_id = 1;

INSERT INTO wages VALUES(1, '1000');

UPDATE wages SET wage = 1000 + 1 WHERE user_id = 1;

DELETE FROM wages WHERE user_id = 1;


# Access kSQL client and check data

select * from stream_users;

select * from stream_wages;

select * from stream_user_wages;

PRINT 'localhost.public.wages' FROM BEGINNING LIMIT 100;

PRINT 'localhost.public.users' FROM BEGINNING LIMIT 100;

PRINT 'sink_database.user_wages' FROM BEGINNING LIMIT 100;


# Access sink Postgres and check data

sudo docker exec -it  postgres-sink /bin/bash

psql -U postgres

select * from users;

select * from wages;

select * from user_wages;
