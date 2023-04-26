Create kSQL stream for users

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
