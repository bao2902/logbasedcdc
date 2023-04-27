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
