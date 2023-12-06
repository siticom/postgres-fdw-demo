# Demo for a Postgres Foreign Data Wrapper

This repo shows a minimal example how to create a foreign data wrapper (fdw) in the same database cluster with the same user.

```shell
# start the database
docker compose up -d

# connect to the database via psql
docker compose exec postgres psql -U postgres
```

then add the following SQL statements:

```sql
-- create two databases
CREATE DATABASE alpha;
CREATE DATABASE beta;

-- connect to alpha database
\c alpha

-- create a demo table with dummy values
CREATE TABLE demo_table (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    age INT
);
INSERT INTO demo_table (name, age) VALUES
    ('John Doe', 30),
    ('Jane Smith', 25),
    ('Bob Johnson', 40);


-- create the fdw extensions in both databases
\c alpha
CREATE EXTENSION IF NOT EXISTS postgres_fdw;
\c beta
CREATE EXTENSION IF NOT EXISTS postgres_fdw;

-- in beta create a fdw to alpha and a user mapping
-- we do not need to provide credentials, because we use the same use in both databases
\c beta
CREATE SERVER alpha_server
   FOREIGN DATA WRAPPER postgres_fdw
   OPTIONS (dbname 'alpha');
CREATE USER MAPPING FOR CURRENT_USER SERVER alpha_server;

-- create a read-only link to the the original table in alpha
CREATE FOREIGN TABLE alpha_demo_table (
   id SERIAL,
   name VARCHAR(50),
   age INT
)
SERVER alpha_server
OPTIONS (table_name 'demo_table', updatable 'false');

-- show all foreign tables
\dE

-- show values from linked table
SELECT * FROM  alpha_demo_table;
```

