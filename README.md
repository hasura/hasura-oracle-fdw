# Using oracle_fdw with Hasura Cloud and AWS RDS

## Requirements

- Hasura Cloud application
- AWS RDS Postgres database
- Oracle database

## Instructions

1. Connect the RDS Postgres to your Hasura Cloud application
2. Run the following commands to install the `oracle_fdw` extension, and connect your Oracle database (filling in the correct connection string and credentials for your Oracle database)

```sql
CREATE EXTENSION oracle_fdw;

CREATE SERVER oradb
    FOREIGN DATA WRAPPER oracle_fdw
    OPTIONS (dbserver '//database-2.xxxxxxxx.us-east-1.rds.amazonaws.com:1521/mydb');

CREATE USER MAPPING FOR postgres
    SERVER oradb
    OPTIONS (user 'admin', password 'mypassword');
```

3. For each table you would like to access through Hasura, use the `CREATE FOREIGN TABLE()` statement to wire up the foreign table connection. The below example creates a foreign table in Postgres called `genre_oracle` that maps to the remote table `GENRE` in the Oracle database:

```sql
CREATE FOREIGN TABLE genre_oracle (
    GenreId int NOT NULL,
    Name varchar(120)
)
    SERVER oradb
    OPTIONS (table 'GENRE');
```

4. Now, all foreign tables created should be available for querying and insert/delete mutations.

