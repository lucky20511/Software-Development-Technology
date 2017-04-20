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

Step5. Login by the temporary password

