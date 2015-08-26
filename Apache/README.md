# Create VirtualHost and DocumentRoot

Add the below to /etc/apache2/sites-available$NAME.blaketest.co.uk.conf

```apache
<VirtualHost *:80>
ServerAdmin webmaster@$NAME.blaketest.co.uk
ServerName $NAME.blaketest.co.uk
ServerAlias www.$NAME.blaketest.co.uk
DocumentRoot /var/www/vhosts/$NAME.blaketest.co.uk/httpdocs
ErrorLog /var/log/apache2/$NAME.blaketest.co.uk-error.log
CustomLog /var/log/apache2/$NAME.blaketest.co.uk-access.log common
<Directory $DOCROOT>
    AllowOverride All
    <Files xmlrpc.php>
        order deny,allow
        deny from all
    </Files>
</Directory>

</VirtualHost>
```

**_Please note $NAME should be replaced with your name!_**

VirtualHost has been created, now we need to enable it and create the DocumentRoot:

```
mkdir -p /var/www/vhosts/$NAME.blaketest.co.uk/httpdocs
a2ensite $NAME.blaketest.co.uk
apache2ctl -t && service apache2 reload
```
