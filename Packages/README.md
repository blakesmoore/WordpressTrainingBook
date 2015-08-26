# Install the required packages
```shell
wget http://download.opensuse.org/repositories/home:/holland-backup/xUbuntu_14.04/Release.key -O - | sudo apt-key add - && echo 'deb http://download.opensuse.org/repositories/home:/holland-backup/xUbuntu_14.04/ ./' > /etc/apt/sources.list.d/holland.list && add-apt-repository ppa:gluster/glusterfs-3.7 && apt-get update && apt-get install apache2 mysql-client php5 php5-mysql php5-memcached vim unzip links vsftpd lvm2 xfsprogs python-software-properties software-properties-common glusterfs-server holland-common holland-mysqldump acl -y && apt-mark hold glusterfs*
```

Here is what you have installed:
* Apache
* MySQL-client
* php5 and modules
* GlusterFS and require packages
* Holland
