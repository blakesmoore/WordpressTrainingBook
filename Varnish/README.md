# Install Varnish

**Run this on both servers!**

Install Varnish and configure apache to listen on port 8080 (As Varnish will be in front on port 80)

```
apt-get install varnish -y
mv /etc/varnish/default.vcl /etc/varnish/default.vcl.backup
sed -i 's/6081/80/g' /etc/default/varnish
cp /etc/apache2/ports.conf /etc/apache2/ports.conf.bak$(date "+_%Y%m%d")
cp /etc/apache2/sites-enabled/000-default /etc/apache2/sites-enabled/000-default.back$(date "+_%Y%m%d")
sed -i 's/Listen 80/Listen 8080/g' /etc/apache2/ports.conf
sed -i 's/NameVirtualHost \*:80/NameVirtualHost \*:8080/g' /etc/apache2/ports.conf
sed -i 's/VirtualHost \*:80/VirtualHost \*:8080/g' /etc/apache2/sites-available/$NAME.blaketest.co.uk.conf
```

**Before you run this, please change $NAME to your name!**

Make the configuration files

```
curl -s https://raw.githubusercontent.com/blakesmoore/varnish-vcl-collection/master/wordpress-example.vcl > /etc/varnish/default.vcl
mkdir /etc/varnish/lib/
curl -s https://raw.githubusercontent.com/blakesmoore/varnish-vcl-collection/master/lib/purge.vcl > /etc/varnish/lib/purge.vcl
curl -s https://raw.githubusercontent.com/blakesmoore/varnish-vcl-collection/master/lib/static.vcl > /etc/varnish/lib/static.vcl
curl -s https://raw.githubusercontent.com/blakesmoore/varnish-vcl-collection/master/lib/xforward.vcl > /etc/varnish/lib/xforward.vcl
```

You can monitor the Varnish log by running the below on both servers, We'll cover this later in the training.

```
varnishncsa -F "%h %u \"%r\" %s %b \ %{Varnish:hitmiss}x" | egrep 'PURGE|testing'
```
