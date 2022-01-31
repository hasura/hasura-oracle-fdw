# Using oracle_fdw with Hasura Cloud and AWS RDS

## Requirements

- Hasura Cloud application
- AWS RDS Postgres database
- Oracle database

## Instructions

1. Connect the RDS Postgres to your Hasura Cloud application
   - See steps 3 and 4 of the documentation here:
     - https://hasura.io/docs/latest/graphql/cloud/getting-started/cloud-databases/aws-postgres.html#step-3-allow-connections-to-your-db-from-hasura-cloud
     - https://hasura.io/docs/latest/graphql/cloud/getting-started/cloud-databases/aws-postgres.html#step-4-construct-the-database-connection-url

2. In your Postgres database, run the following commands to install the `oracle_fdw` extension and connect your Oracle database (filling in the correct connection string and credentials for your Oracle connection)
   -  > NOTE: You can do this via `psql`, the Hasura web console UI's "Run SQL" page, or any other database administration tool you prefer

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

4. Make sure that the newly-created foreign tables you want to query from Hasura are **"tracked"**
   - You can do this by pressing the **"Track Table"** button on the Hasura web console, under the *"You are here: Data > (Database Name) > (Schema Name)"* page on the web console
   - IE: `https://<HASURA CLOUD URL>/consoledata/default/schema/public`
   - See: https://hasura.io/docs/latest/graphql/core/databases/postgres/schema/tables.html#tracking-tables

5. Now, all foreign tables created should be available for querying and insert/delete mutations. You can run a GraphQL query against the foreign table to test that it is working and returning the data you expect.
   - For example, using the table names given above, something like:

```graphql
query {
    # Query "GENRE" table in Oracle
    genre_oracle {
        GenreId
        Name
    }
}

