# Build Dockerfile

sudo docker build -t nashtech/kafka .

# Start Docker Compose

sudo docker-compose up -d

# Access Kafka container and run Kafka

sudo docker exec -t -i kafka /bin/bash

cd /bin

connect-standalone /config/connect-standalone.properties /config/connect-postgres-source.properties /config/connect-postgres-sink-users.properties /config/connect-postgres-sink-wages.properties /config/connect-postgres-sink-user-wages.properties

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
