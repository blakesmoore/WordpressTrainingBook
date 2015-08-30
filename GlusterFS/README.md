# Configure GlusterFS

### Configure /etc/hosts

**Unless specified, run these commands on both servers**

We're adding these as an alias of the IP address for convenience. Open /etc/hosts and add the below:

```
192.168.3.1 glus1
192.168.3.2 glus2
```

*Make sure you modify the IP addressses to the correct IP.*

Now we need to ensure they can 'talk' to eachother, run the below:

```
ping -c2 glus1; ping -c2 glus2
```

### Prepare the bricks

Ensure the Block Storage volumes you created earlier are attached to the servers.

#### Create the partitiion table

You'll need to run the below in order to create the primary partition

```
fdisk /dev/xvdb
Press 'n' to add a new partition
As it's Primary, leave as default of 'p'
We want the full partition so leave first and last sector as the default.
Press 't' for type and input '8e' (Linux LVM)
Press 'p' to print the partition table
Press 'w' to write and save the changes
```

See full output below:

```
# fdisk /dev/xvdb
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0x18f286a7.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p):
Using default response p
Partition number (1-4, default 1):
Using default value 1
First sector (2048-157286399, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-157286399, default 157286399):
Using default value 157286399

Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 8e
Changed system type of partition 1 to 8e (Linux LVM)

Command (m for help): p

Disk /dev/xvdb: 80.5 GB, 80530636800 bytes
255 heads, 63 sectors/track, 9790 cylinders, total 157286400 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x18f286a7

    Device Boot      Start         End      Blocks   Id  System
/dev/xvdb1            2048   157286399    78642176   8e  Linux LVM

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

#### Create Logical Volumes

```
pvcreate /dev/xvdb1
vgcreate vgglus1 /dev/xvdb1
lvcreate -l 100%VG -n gbrick1 vgglus1
mkfs.xfs -i size=512 /dev/vgglus1/gbrick1
echo '/dev/vgglus1/gbrick1 /data/gluster/gvol0 xfs inode64,nobarrier 0 0' >> /etc/fstab
mkdir -p /data/gluster/gvol0
mount -a
mkdir -p /data/gluster/gvol0/brick1
```

#### Start the daemon

```
service glusterfs-server start
```

#### Build peer group

**From glus1**

```
gluster peer probe glus2
gluster peer status
```

#### Create a replicated volume

```
gluster volume create gvol0 replica 2 transport tcp glus1:/data/gluster/gvol0/brick1 glus2:/data/gluster/gvol0/brick1
gluster volume set gvol0 auth.allow 192.168.3.*,127.0.0.1
gluster volume set gvol0 nfs.disable off
gluster volume set gvol0 nfs.addr-namelookup off
gluster volume set gvol0 nfs.export-volumes on
gluster volume set gvol0 nfs.rpc-auth-allow 192.168.3.*
gluster volume set gvol0 performance.io-thread-count 32
gluster volume start gvol0
```

You can run the below to to get information about the volume

```
[root@gluster1 ~]# gluster volume info gvol0
 
Volume Name: gvol0
Type: Replicate
Volume ID: 8bd82bdb-cb5b-40b4-b42c-eccfb3baa37f
Status: Started
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: glus1:/data/gluster/gvol0/brick1
Brick2: glus2:/data/gluster/gvol0/brick1
Options Reconfigured:
performance.io-thread-count: 32
nfs.rpc-auth-allow: 192.168.6.*
nfs.export-volumes: on
nfs.addr-namelookup: off
nfs.disable: off
auth.allow: 192.168.6.*,127.0.0.1
performance.readdir-ahead: on
```

#### Mount the volume

**From glus1**

```
modprobe fuse
echo 'glus1:/gvol0 /var/www/vhosts glusterfs defaults,acl,_netdev,backup-volfile-servers=glus2 0 0' >> /etc/fstab
mkdir -p /var/www/vhosts
mount /var/www/vhosts
```

**From glus2**

```
modprobe fuse
echo 'glus2:/gvol0 /var/www/vhosts glusterfs defaults,acl,_netdev,backup-volfile-servers=glus1 0 0' >> /etc/fstab
mkdir -p /var/www/vhosts
mount /var/www/vhosts
```

We've now configured a replicated GlusterFS volume and mounted at /var/www/vhosts
