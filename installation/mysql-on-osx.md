Step1. Download the MySQL Installation File

[https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)

Step2. Install and Keep the temporary password

Step3. Export the binary file to mysql

```
export PATH=$PATH:/usr/local/mysql/bin/
```

Step4. Run and Terminate the mysql CLI

```
sudo launchctl load -F /Library/LaunchDaemons/com.oracle.oss.mysql.mysqld.plist
```

```
sudo launchctl unload -F /Library/LaunchDaemons/com.oracle.oss.mysql.mysqld.plist
```

Step5. Login by temporary password

```
mysql -u root -h 127.0.0.1 -p
```

Step6. Change the password of root

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'lucky20511';
```

Step7. Logout and Login to check

```
mysql -u root -p
```

Some Easy Commands of MySQL:

Show existing databases:

```
SHOW DATABASES;
```

Select database:

```
USE some-database;
```

Show existing tables:

```
SHOW TABLES; 
```



