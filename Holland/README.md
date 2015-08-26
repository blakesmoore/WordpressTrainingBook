# Configure Holland
As we've already installed Holland, we just need to configure it.

First, open /etc/holland/holland.conf and set the backup directory to /var/lib/mysqlbackup

```bash
sed -i 's/spool\/holland/lib\/mysqlbackup/g' /etc/holland/holland.conf
```

Next, edit /etc/holland/backupsets/default.conf and add / modify the below:

```
backups-to-keep = 1
databases       = *
exclude-tables  = mysql.events
[mysql:client]
user = root
host = mydatabasehost.example.com OR 123.456.789.123
password = MySecurePa$$word
```

Set Holland to run at 01:00 daily.

```
echo '00 02 * * * root /usr/sbin/holland bk' > /etc/cron.d/holland
```
