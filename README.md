# Context 1: only CDC

- We have a Users table (fields: user_id, first_name, last_name) in a source PostgreSQL database.
- We have a Wages table (fields: user_id, wage) in a soure PostgreSQL database.
- We have a Users table (fields: user_id, first_name, last_name) in a sink PostgreSQL database.
- We have a Wages table (fields: user_id, wage) in a sink PostgreSQL database.
- We want to synchronize the Change Data Capture (CDC) to the tables Users and Wages in a sink PostgreSQL database when the tables Users and Wages are inserted, updated, or deleted in the source PostgreSQL database.
- We expect this synchonization happenning in near-real-time.

# Context 2: CDC + transform

- We have a Users table (fields: user_id, first_name, last_name) in a source PostgreSQL database.
- We have a Wages table (fields: user_id, wage) in a soure PostgreSQL database.
- We have a User_Wages table (fields: user_id, full_name, wage) in a sink PostgreSQL database.
- We want to synchronize the Change Data Capture (CDC) to the table User_Wages in a sink PostgreSQL database when the tables Users and Wages are inserted, updated, or deleted in the source PostgreSQL database.
- This synchonization includes the data tranforming as following:
+ User_Wages.user_id = Users.user_id
+ User_Wages.full_name = Users.first_name + ' ' + Users.last_name
+ User_Wages.wage = Wages.wage
- We expect this synchonization happenning in near-real-time.

# Architecture for both Context 1 and Context 2

![alt text](https://github.com/bao2902/logbasedcdc/blob/main/LogBasedCDC.PNG)

# Environment

"docker-compose.yml" includes the following containers:
- Zookeeper
- Kafka
- kSQL database server
- kSQL database client
- Source PostgreSQL database
- Sink PostgreSQL database

"config" folder includes the following configurations:
- Source PostgreSQL config for Users and Wages tables (connect-postgres-source.properties)
- Sink PostgreSQL config for Users table (connect-postgres-sink-users.properties)
- Sink PostgreSQL config for Wages table (connect-postgres-sink-wages.properties)
- Sink PostgreSQL config for User_Wages table (connect-postgres-sink-user-wages.properties)

"plugins" folder includes the following plugins:
- Debezium PostgreSQL source connector (debezium-connector-postgres)
- Debezium JDBC sink connector (confluentinc-kafka-connect-jdbc)


# Steps to start containers:

1. Build Dockerfile

sudo docker build -t nashtech/kafka .

2. Start Docker Compose

sudo docker-compose up -d


# Steps to create source PosgreeSQL tables:

1. Access source PostgreSQL container

sudo docker exec -it  postgres-source /bin/bash

psql -U postgres

2. Create Users table and insert 1 record

CREATE TABLE users(user_id INTEGER, first_name VARCHAR(200), last_name VARCHAR(200), PRIMARY KEY (user_id));

INSERT INTO users VALUES(1, 'first 1', 'last 1');

3. Create Wages table and insert 1 record

CREATE TABLE wages(user_id INTEGER, wage integer, PRIMARY KEY (user_id));

INSERT INTO wages VALUES(1, '1000');



# Steps to create sink PosgreeSQL tables:

1. Access sink PostgreSQL container

sudo docker exec -it  postgres-sink /bin/bash

psql -U postgres

2. Create Users table

CREATE TABLE users(user_id INTEGER, first_name VARCHAR(200), last_name VARCHAR(200), PRIMARY KEY (user_id));

3. Create Wages table 

CREATE TABLE wages(user_id INTEGER, wage integer, PRIMARY KEY (user_id));

4. Create User_Wages table 

CREATE TABLE user_wages(user_id INTEGER, full_name VARCHAR(200), wage integer, PRIMARY KEY (user_id));



# Steps to create source Kafka topics:

1. Access Kafka container

sudo docker exec -t -i kafka /bin/bash

2. Start Kafka standalone cluster

cd /bin

connect-standalone /config/connect-standalone.properties /config/connect-postgres-source.properties /config/connect-postgres-sink-users.properties /config/connect-postgres-sink-wages.properties /config/connect-postgres-sink-user-wages.properties

3. Check source Kafka topics

kafka-topics --list --bootstrap-server localhost:9092
