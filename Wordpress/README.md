# Install Wordpress

### Install base components

```
wget --quiet -O /var/www/vhosts/$NAME.blaketest.co.uk/httpdocs/latest.tar.gz http://wordpress.org/latest.tar.gz
cd /var/www/vhosts/$NAME.blaketest.co.uk/httpdocs/
tar -xzf latest.tar.gz --strip-components 1 && rm latest.tar.gz
mv wp-config-sample.php wp-config.php
```

Open up wp-config.php and input your database details
```
vim wp-config.php
[...]
# Below are the details you need to change

 define( 'DB_NAME',     'database_name_here' );
 define( 'DB_USER',     'username_here' );
 define( 'DB_PASSWORD', 'password_here' );
 define( 'DB_HOST',     'localhost' );
```

Configure ACLs to allow the apache and FTP user to both write to the DocumentRoot
```
useradd -d /var/www/vhosts/$NAME.blaketest.co.uk/httpdocs -s /sbin/nologin $NAME
chown -R $NAME:$NAME /var/www/vhosts/$NAME.blaketest.co.uk/httpdocs
setfacl -R -d -m u:${NAME}:rwx /var/www/vhosts/$NAME.blaketest.co.uk/httpdocs
setfacl -R -m u:www-data:rwx /var/www/vhosts/$NAME.blaketest.co.uk/httpdocs
setfacl -R -d -m u:www-data:rwx /var/www/vhosts/$NAME.blaketest.co.uk/httpdocs
```
