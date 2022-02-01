# Connecting Hasura to Redshift

Connecting Hasura to Redshift requires:
- A Postgres RDS Database
- Configuration of the `postgres_fdw` extension
- A Redshift cluster located **in the same zone as the RDS database**

## 1. Create Postgres extensions

On your Postgres RDS database, run the following statements to create the required extensions for Redshift support:
```sql
CREATE EXTENSION postgres_fdw;
CREATE EXTENSION dblink;
```

## 2. Add Redshift as a foreign server to Postgres

On your Postgres RDS database, run the following statements to add your Redshift instance as a new server and grant user permissions:
- **NOTE: Make sure to replace the `OPTIONS` values with those of your own Redshift cluster and Postgres user**
  
```sql
CREATE SERVER redshift_foreign_server
    FOREIGN DATA WRAPPER postgres_fdw
    OPTIONS (host 'redshift-cluster-1.xxxxxx.us-east-1.redshift.amazonaws.com', port '5439', dbname 'dev', sslmode 'require');

CREATE USER MAPPING FOR postgres
    SERVER redshift_foreign_server
    OPTIONS (user 'awsuser', password 'Password123');
```

## 3. Create views for the Redshift data you would like to query

Since `dblink` is not capable of creating foreign table definitions as other fdw implementations can, you must instead either:
- A) Create views/materialized views that represent the results of your query
- B) Create tables via `CREATE TABLE mytable AS SELECT...` that query Redshift data

As an example, here is the creation of a view and materialized view:

```sql
CREATE OR REPLACE VIEW v_sales AS
    SELECT *
    FROM dblink ('redshift_foreign_server', $REDSHIFT$
        SELECT sellerid, sum(pricepaid) sales
        FROM sales
        WHERE saletime >= '2008-01-01'
          AND saletime < '2008-02-01'
        GROUP BY sellerid
        ORDER BY sales DESC
        $REDSHIFT$)
    AS t1 (sellerid int, sales decimal);

CREATE MATERIALIZED VIEW v_users_likes_by_state AS
    SELECT *
    FROM dblink('redshift_foreign_server', $REDSHIFT$
        SELECT state, sum(likesports::int) sports_like_count
        FROM users
        GROUP BY state
        $REDSHIFT$)
    AS t1 (state text, sports_like_count int);
```

## 4. Track the newly-created views in your Hasura instance

From the `You are here: Data -> (DB Name) -> (Schema Name)` page on the Hasura web console, you should see the following under the `Untracked tables` section:
- `v_sales`
- `v_users_likes_by_state`

Press `Track` next to each view name.

They should now be visible under your DB tables/views on the lefthand side.

## 5. Query the views

You can now query the views using GraphQL, or browse them from your data dashboard


## More information

For more information, see the following link:

https://aws.amazon.com/blogs/big-data/join-amazon-redshift-and-amazon-rds-postgresql-with-dblink/