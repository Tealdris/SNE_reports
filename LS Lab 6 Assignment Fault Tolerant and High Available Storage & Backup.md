###### tags: `LS`
:::success
# LS Lab 6 Assignment: Fault Tolerant and High Available Storage & Backup
**Name: Radomskiy Ilia**
:::

---

# Fault Tolerant Storage


## Task 1: Take a pick

:::info
Before choosing, briefly describe and explain the difference between **block storage**, **file storage**, and **object storage**.
Take a pick from the list proposed below:
* **DRBD** (preferred option)
* Ceph block device
* Gluster block device
* FreeBSD HAST (if you're familiar with *BSD)
* does anything else exist as for BDs?
:::

**File storage** it is a way to store information in the hierarchy of folders, like usual file storage, of windows for example
**Block storage** represents storage chunks of data, they are organized into volumes
**object storage** it is metadata with the links on the data itself
All of the said above can be represented like it is done in the picture below

<center>

![](https://i.imgur.com/491I0Aa.png)
    
</center>

---
## Task 2: Fault tolerant setup

:::info
* Create and configure a necessary amount of VMs for your option. Install necessary packets and then configure a distributed block device.
* Check (for all VMs) that a new block device appeared in the system and format it with usual filesystem like EXT4 or XFS and then mount. Make sure that each VM can recognize a filesystem on distributed block device.
* Validate that storage on your distributed block device is fault tolerant (create some data, destroy one node, check storage status, etc.).
:::

On the picture below you can see my topology for this lab.
QEMU machines `OriginalU4` and `OriginalU3` are used for the `DRDB` storage. `OriginalU1` and `OriginalU2` are used for the `Borg` backup

<center>

![](https://i.imgur.com/2wVdSbN.png)
    
</center>

Before configuring storage space we need to set `hostnames` for the machines in the `/etc/hostname` file

```
10.1.1.11 21 s1
10.1.1.12 22 s2
```

After that we can add repository if `DRDB` and install it
```
sudo add-apt-repository ppa:linbit/linbit-drbd9-stack
sudo apt update
sudo apt install -y drbd-dkms drbd-utils
```

To check some of the modules of the `DRDB` we might use the following command `sudo modprobe drbd` some of the packages might not include some of the modules, so you have to be sure.

After that, we need to [prepare](https://www.stableit.ru/2011/05/losetup.html) disk for the `DRDB` storage

Create block storage
```
dd if=/dev/zero of=/sdb1 bs=2024k count=1024
sudo losetup /dev/loop10 /sdb1
```

We can check, was disk preparation successfully or not with the following command `losetup -a`

<center>

![](https://i.imgur.com/XRaWEHZ.png)
    
</center>

After that, we need to configure network part, grand access to the servers to particular port through a firewall

```
sudo apt install firewalld -y
sudo firewall-cmd --permanent --add-port=7789/tcp
sudo firewall-cmd --reload
```

Next step change main configuration file of the `DRBD`: `/etc/drbd.d/global_common.conf`
Configuration is stored in the hidden part. The most important it is to add line with `protocol C;`. This stands for `synchronous replication protocol`. 

:::spoiler
```
global {
 usage-count  yes;
}
common {
 net {
  protocol C;
 }
}
```
:::

Now we need to change `test.res` so to the configuration of the data storage itself. File located in `/etc/drbd.d/test.res`

:::spoiler
```
resource test {
        on s1 {
                device /dev/drbd205;
                disk /dev/loop2;
                        address 10.1.1.11:7789;
                        meta-disk internal;
}
        on s2 {
                device /dev/drbd205;
                disk /dev/loop1;
                        address 10.1.1.12:7789;
                        meta-disk internal;
}
}
```
:::

When everything is configured we can start our data storage. The result you can see in the picture

```
sudo drbdadm create-md test
```

<center>

![](https://i.imgur.com/V4wkg1s.png)
    
![](https://i.imgur.com/dQWo1DB.png)
    
</center>

On both machines we can see this type of message because they are not synchronized

<center>

![](https://i.imgur.com/bvQ7Ut6.png)
    
![](https://i.imgur.com/eRCz6mh.png)
    
</center>

To make node of the storage primary we need to use this command

```
drbdadm primary --force test
```

After synchronization and verification we can see different message

```
sudo drbdadm up test
drbdadm status test
```

<center>

![](https://i.imgur.com/Pugh6ZC.png)
    
![](https://i.imgur.com/4l0yfwR.png)
    
</center>

Now we can set a file system on it and mount this storage on the `S1`.

```
sudo mkfs -t ext4 /dev/drbd205
```

Picture of successfully  created filesystem shown below

<center>

![](https://i.imgur.com/e4km5xz.png)
    
</center>

Now create a place where the disk will be mounted, and mount it. In order to verify the work of the storage, i will create a file on the `S1`. If everything is configured correctly i should wear it on the `S2` server

```
mkdir -p /mnt/DRDB_PRI/
mount /dev/drbd205 /mnt/DRDB_PRI/
cd /mnt/DRDB_PRI/
touch Hello.txt
```

<center>

![](https://i.imgur.com/9DuiWLH.png)

</center>

Now, in order to continue work we have to unmount the disk, and turn `S 1` in secondary node

```
umount /mnt/DRDB_PRI/
drbdadm secondary test
```

`lsblk` can be used if we want to show that disk is available for the system

<center>
    
![](https://i.imgur.com/U50rYNB.png)
    
![](https://i.imgur.com/SIusgz8.png)

</center>

For the `S2` machine we have to become primary and mount the disk in order to get access to the files

```
drbdadm primary test
mkdir -p /mnt/DRDB_SEC/
mount /dev/drbd205 /mnt/DRDB_SEC/
cd /mnt/DRDB_SEC/
```

As you can see from the pictures below we are having access to the files created on the `S1`, and vice versa

<center>

![](https://i.imgur.com/TM2ZwfS.png)
    
![](https://i.imgur.com/X82Q3Cb.png)
    
</center>

:::info
Have you lost your data after destroying one node? Was it necessary to do something on another nodes to get the data?
:::

No because of the `Protocol C`

In the pictures below you can see what will happen if we turn off one of the nodes in the cluster

<center>

![](https://i.imgur.com/WjH89k6.png)
    Notification
    
![](https://i.imgur.com/Qefy6yQ.png)
    Status message
    
![](https://i.imgur.com/X82Q3Cb.png)
    But files are still available
</center>

On those pictures below you can see S2 returning to the cluster after being down

<center>

![](https://i.imgur.com/11MJzpb.png)
    
![](https://i.imgur.com/yOJu7s8.png)
    
</center>

---

## Task 3: High Available setup

:::info
* Modify your configuration to make storage high available. Besides it, you will need to format your block device with a clustered filesystem (e.g., OCFS2, GlusterFS, MooseFS).
* Again validate the storage on your distributed block device.
:::

In order to set up [High Available](https://mirivlad.ru/ustanovka-i-nastrojka-drbd-dlya-setevoj-replikatsii-fajlovoj-sistemy-na-debian-8/) cluster we need to change configuration file of storage: ` /etc/drbd.d/test.res` New configuration file provided in the spoiler part


:::spoiler
```
        on s2 {
                device /dev/drbd205;
                disk /dev/loop10;
                        address 10.1.1.12:7789;
                        meta-disk internal;
}
        net {
                allow-two-primaries;
                after-sb-0pri discard-zero-changes;
                after-sb-1pri discard-secondary;
                after-sb-2pri disconnect;
}
        startup {
                become-primary-on both;
}
}
```
:::

We need to disconnect every device from the cluster in order to set up the in the new way.

```
drbdadm disconnect test
drbdadm connect test
drbdadm primary test
drbdadm status test
```

`ocfs2` need to be installed in the pc in order to work

```
sudo apt-get install ocfs2-tools -y
```

After installation we need to write new file system on the block device with:

```
mkfs -t ocfs2 -N 2 -L ocfs2_drbd205 /dev/drbd205
```

After successful writing we can receive this message

<center>

![](https://i.imgur.com/Kb0flw5.png)
    
</center>

In order to configure a new cluster, we need to set up `ocfs2` cluster settings. Configuration file is located in `/etc/ocfs2/cluster.conf`. The config setting is shown in the hidden part

:::spoiler
```
node:
        ip_port = 7777
        ip_address = 10.1.1.21
        number = 1
        name = s21
        cluster = ocfs2

node:
        ip_port = 7777
        ip_address = 10.1.1.22
        number = 2
        name = s22
        cluster = ocfs2
cluster:
        node_count = 2
        heartbeat_mode = local
        name = ocfs2
```
:::

But, the basic setup is the Linux kernel is not enough for cluster creation. So we have to update it.

```
sudo apt install linux-generic
```

Before cluster set up we might check modules that have to be installed now the the update pf the kernel

```
sudo modprobe ocfs2_stackglue ocfs2_dlmfs
```

To check status, reload or make programs running on the boot we might use following commands

```
o2cb cluster-status
/etc/init.d/o2cb restart
/etc/init.d/o2cb status
o2cb list-cluster ocfs2
systemctl enable o2cb
systemctl enable ocfs2
```

In order to start work with the cluster we have to register it.
```
o2cb register-cluster ocfs2
```

On the picture below we can see results of the cluster creation

<center>

![](https://i.imgur.com/x7ok1KM.png)
S1 and S2 are in disk cluster
    
![](https://i.imgur.com/3mPjEF0.png)
We can verify this with df 
    
![](https://i.imgur.com/oLLc8NM.png)
We can verify this with lsblk 
    
</center>

In order to work with the storage, we need to mount disks in both servers 

```
mount -t ocfs2 /dev/drbd205 /mnt/DRDB_PRI
mount -t ocfs2 /dev/drbd205 /mnt/DRDB_SEC
```

As you can see from the picture below, i am showing the empty folder on both servers
After that, I am creating only 1 file on each server.
In the end, I am having the same folder with two files

<center>

![](https://i.imgur.com/UXd9Wcx.png)
    
![](https://i.imgur.com/ryWCKqG.png)
    
</center>

:::info
Was it necessary to do something on another nodes to get the data in this setup?
:::

As I have shown in the pictures above you don't need to do anything to receive created files. With the exception of the mounting filesystem on the workstation

---

## Task 4 - Questions

:::info
* Explain what is fault tolerance and high availability
:::

[Fault tolerance](https://en.wikipedia.org/wiki/Fault_tolerance) - ability of the complex system to maintain its working state despite the fact the parts of the system being turned off

[High availability](https://en.wikipedia.org/wiki/High_availability) - agreement , based on the fact of workability of system throughout the year, can be measured with the downtime parameter (%) 

:::spoiler
https://www.ibm.com/docs/en/powerha-aix/7.2?topic=aix-high-availability-versus-fault-tolerance
:::

:::info
* What is a split-brain situation?
:::

[Split-brain](https://en.wikipedia.org/wiki/Split-brain_(computing)) - often used when data is separated between several servers, because of network topology, or due to availability reasons. Often is a bad thing, it often happens when data is loaded in one database from the different primary devices


:::info
* What are the advantages and disadvantages of clustered filesystems?
:::

Advantages

+ availability - when your data is stored on the different devices-server, users can make much more quick communications with you, so, the quicker user will be served, the with the much more high percentage you will get a customer
+ less faults - when your server is down, your customers are buying from somewhere else
+  back up - losing data can be crucial to business, especially when it depends on the data itself, nowadays there are plenty of companies working only with the data of the users only

Disadvantages

- price - of course, it is much more expensive to maintain such clusters
- no 100% guarantee
- it is harder to build, deploy, fix, manage and develop such systems


--- 

# Backup

## Task 1: Take a pick

:::info
Choose any tool from the list below:
**Borg**
restic
rsync
:::

**Borg** is my choice as a backup solution

---

## Task 2: Configuration & Testing

:::info
Create 2 VMs (server and client), install and configure OS (please check the list of supported OS for your solution).
:::

In the picture below you can see my topology for this lab.
QEMU machines `Original U4` and `Original U3` are used for the `DRDB` storage. `Original U 1` and `Original U2` are used for the `Borg` backup

<center>

![](https://i.imgur.com/2wVdSbN.png)
    
</center>

:::info
Configure your solution: install necessary packets, edit the configuration.
:::

In order to install borg i need to perform following commands
```
sudo wget https://github.com/borgbackup/borg/releases/download/1.1.6/borg-linux64 -O /usr/local/bin/borg
sudo chmod +x /usr/local/bin/borg
sudo useradd -m borg
```

If you don't want to enter password every time you need to worry about ssh keys

```
sudo mkdir ~borg/.ssh
sudo echo 'command="/usr/local/bin/borg serve" key.pub' > ~borg/.ssh/authorized_keys
sudo chown -R borg:borg ~borg/.ssh
```

:::info
Create a repo on the backup server which will be used to store your backups. 
**Bonus:** configure an encrypted repo
:::

This command below is used if you want to create a repository for your backup. `--encryption` is used if you want to encrypt your repository. In that way every interaction with the encrypted data requires password

```
borg init --encryption=repokey borg@10.1.1.21:MyBorgRepo
```

<center>

![](https://i.imgur.com/sCPsTrD.png)
    
![](https://i.imgur.com/WH0pyj3.png)

</center>

For this lab, I have created two repositories with backup. `MyBorgRepo` is used as repository with complete backup of the system, so to `/root` folder
`MyBorgRepo2` is used in order to backup 1 folder

```
borg init --encryption=repokey borg@10.1.1.21:MyBorgRepo2
```

<center>
    
![](https://i.imgur.com/e0lsLiN.png)
    
![](https://i.imgur.com/umJoqE2.png)

</center>

After the creation of the repositories, we can observe a new folder on the backup server

<center>

![](https://i.imgur.com/d9p80Zf.png)
    
</center>

:::info
Make a backup of `/home` directory (create some files and directories before backuping) of your client. Don't forget to make a verification of backup (some solutions provide an embedded option to verify backups). If there is no embedded option to verify backups try to make a verification on your own.
:::

To store in repository some data we need to enter the following command

This command will backup 2 folders `/etc` and `/root` on the backup server with the name `MyFirstBackup(date of the backup creation)`
```
borg create --stats --list borg@10.1.1.21:MyBorgRepo::"MyFirstBackup-{now:%Y-%m-%d_%H:%M:%S}" /etc /root
```

In this picture you can see what backups are stored in the repository, right now there is only one backup

<center>

![](https://i.imgur.com/Dsz1EHI.png)

</center>

This command will backup 1 folder `/home/ubuntu/backup` on the backup server with the name `MySecondBackup(date of the backup creation)`

```
borg create --stats --list borg@10.1.1.21:MyBorgRepo2::"MySecondBackup-{now:%Y-%m-%d_%H:%M:%S}" /home/ubuntu/backup
```

On this picture you can see files that are stored in that particular  back up

<center>

![](https://i.imgur.com/XTS6AYt.png)

</center>

:::info
Then damage your client's `/home` directory (encrypt it or forcibly remove) and restore from
backup. 
Has anything changed with the files after restoring? 
Can you see and edit them?
:::

With the command `borg list MyBorgRepo::MyFirstBackup-2022-02-28_08:44:36` you can see files that are stored in this backup. In case with `/root` it's too many. Same command applyes for second backup `borg list MyBorgRepo2::MySecondBackup-2022-02-28_08:53:23`

<center>

![](https://i.imgur.com/cA0OilQ.png)
First backup
    
![](https://i.imgur.com/p68VsCu.png)
Second backup
</center>

Now, when your data is saved we might check it

I will delete `test` file on the machine and will restore it from the backup machine

<center>

![](https://i.imgur.com/i4U9fbH.png)
File is deleted
</center>

To use backup files by appointment. I will use this command

```
borg extract borg@10.1.1.21:MyBorgRepo2::MySecondBackup-2022-02-28_08:53:23
```

<center>

![](https://i.imgur.com/FHtSzj8.png)
Command execution
</center>

On the picture below you can see that file have successfully  restored

<center>

![](https://i.imgur.com/5tLLM6S.png)

</center>

---

## Task 3: Questions

:::info
* When and how often do you need to verify your backups?
:::

It depends on the data importance, the much more valuable assets stored on your system, the more often you need to do backups
But most common practice is to do backups once a week

:::info
* Backup rotations schemes, what are they? Are they available in your solution?
:::

Backup rotations schemes can be explained with the manage and time management of backups.
There are several types of schemes

**First in, first out** - a simple one, the oldest data replaced with the new one
**Grandfather-father-son** - little bit complex, there three parallel backups happen at the same time, daily, weekly and monthly. In all of them, there are some **FIFO** methods is used
**Tower of Hanoi** - mathematics of the `tower of Hanoi` puzzle is used: 1 every day, 2 every 4 days, etc

In **borg** case `cron` can be used, to trigger backup

---

:::spoiler
centos
sudo sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-Linux-*

sudo sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-Linux-*
sudo yum update -y

yum install elrepo-release
glibc-devel
yum install drbd90-utils kmod-drbd90

https://www.2daygeek.com/install-enable-elrepo-on-centos-rhel-redhat-linux/

sudo yum update -y

https://itsecforu.ru/2019/01/14/%D0%BA%D0%B0%D0%BA-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B8%D1%82%D1%8C-drbd-%D0%B4%D0%BB%D1%8F-%D1%80%D0%B5%D0%BF%D0%BB%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D0%B8-%D1%85%D1%80%D0%B0%D0%BD%D0%B8%D0%BB/

network
https://www.dmosk.ru/miniinstruktions.php?mini=centos-network
sudo nano /etc/sysconfig/network-scripts/ifcfg-ens32

DEVICE=eth0
BOOTPROTO=static
IPADDR=10.1.1.11
NETMASK=255.255.255.0
GATEWAY=10.1.1.1
DNS1=8.8.8.8
ONBOOT=yes
NM_CONTROLLED=yes

systemctl restart NetworkManager
nmcli networking off; nmcli networking on

sudo yum install firewalld -y
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --permanent --add-port=7789/tcp
sudo firewall-cmd --reload

suod nano /etc/hosts
10.1.1.11 server1
10.1.1.12 server2

sudo nano /etc/drbd.d/data.res

sudo dd if=/dev/zero of=/dev/sdb1 bs=1024k count=1024

sudo mkfs.ext4 /sdb1
sudo mount /sdb1 /mnt
sudo umount /mnt

drbd1

sudo yum install lvm2

:::

:::spoiler

mount 
https://losst.ru/montirovanie-diska-v-linux
superblock
https://unix.stackexchange.com/questions/315063/mount-wrong-fs-type-bad-option-bad-superblock

https://mirivlad.ru/ustanovka-i-nastrojka-drbd-dlya-setevoj-replikatsii-fajlovoj-sistemy-na-debian-8/

nano /etc/hostname
10.1.1.11 21 s1
10.1.1.12 22 s2


sudo add-apt-repository ppa:linbit/linbit-drbd9-stack
sudo apt update
sudo apt install -y drbd-dkms drbd-utils
sudo modprobe drbd

https://www.stableit.ru/2011/05/losetup.html

dd if=/dev/zero of=/sdb1 bs=2024k count=1024


sudo losetup /dev/loop10 /sdb1

losetup -a # !reboot

/dev/loop10

sudo apt install firewalld -y
sudo firewall-cmd --permanent --add-port=7789/tcp
sudo firewall-cmd --reload

sudo nano /etc/drbd.d/global_common.conf

global {
 usage-count  yes;
}
common {
 net {
  protocol C;
 }
}

sudo nano /etc/drbd.d/test.res

sudo nano /etc/drbd.d/r0.res
```
resource test {
        on s1 {
                device /dev/drbd205;
                disk /dev/loop2;
                        address 10.1.1.11:7789;
                        meta-disk internal;
}
        on s2 {
                device /dev/drbd205;
                disk /dev/loop1;
                        address 10.1.1.12:7789;
                        meta-disk internal;
}

}
```

sudo drbdadm create-md test

sudo drbdadm up test

drbdadm status test


drbdadm primary --force test



sudo mkfs -t ext4 /dev/drbd205

mkdir -p /mnt/DRDB_PRI/
mount /dev/drbd205 /mnt/DRDB_PRI/
cd /mnt/DRDB_PRI/

touch Hello.txt

umount /mnt/DRDB_PRI/

s1
drbdadm secondary test

s2
drbdadm primary test
mkdir -p /mnt/DRDB_SEC/
mount /dev/drbd205 /mnt/DRDB_SEC/
cd /mnt/DRDB_SEC/

```
        on s2 {
                device /dev/drbd205;
                disk /dev/loop10;
                        address 10.1.1.12:7789;
                        meta-disk internal;
}
        net {
                allow-two-primaries;
                after-sb-0pri discard-zero-changes;
                after-sb-1pri discard-secondary;
                after-sb-2pri disconnect;
}
        startup {
                become-primary-on both;
}
}
```

https://mirivlad.ru/ustanovka-i-nastrojka-drbd-dlya-setevoj-replikatsii-fajlovoj-sistemy-na-debian-8/


drbdadm disconnect test
drbdadm connect test
drbdadm primary test

drbdadm status test

sudo apt-get install ocfs2-tools -y

mkfs -t ocfs2 -N 2 -L ocfs2_drbd205 /dev/drbd205

nano /etc/ocfs2/cluster.conf

```
node:
        ip_port = 7777
        ip_address = 10.1.1.21
        number = 1
        name = s21
        cluster = ocfs2

node:
        ip_port = 7777
        ip_address = 10.1.1.22
        number = 2
        name = s22
        cluster = ocfs2
cluster:
        node_count = 2
        heartbeat_mode = local
        name = ocfs2
```

sudo apt install linux-generic
sudo modprobe ocfs2_stackglue ocfs2_dlmfs


sudo firewall-cmd --permanent --add-port=7777/tcp
sudo firewall-cmd --reload

o2cb cluster-status
/etc/init.d/o2cb restart
/etc/init.d/o2cb status
 o2cb list-cluster ocfs2
 
systemctl enable o2cb
systemctl enable ocfs2
**o2cb register-cluster ocfs2**

mount -t ocfs2 /dev/drbd205 /mnt/DRDB_PRI
mount -t ocfs2 /dev/drbd205 /mnt/DRDB_SEC

mkfs.ocfs2 -L 'r0' /dev/loop10

:::

:::spoiler
borg

sudo wget https://github.com/borgbackup/borg/releases/download/1.1.6/borg-linux64 -O /usr/local/bin/borg
sudo chmod +x /usr/local/bin/borg

sudo useradd -m borg
sudo mkdir ~borg/.ssh
sudo echo 'command="/usr/local/bin/borg serve" ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDRsuqEewOiImzudn03tsYjGXyYkpEbsJhs5+p0Qmpr1UYwG/n2bPyaPe8rMHciAe+R2BwZOBFgj5g7ovBzvfqPnhLDB1ZzhbydGz60MVyvL2MadLYBXbh6Gc9bIkjVQ6B98qviHoDRJhEhrdcY039utxns9OdKDL4ixIPGUc4WI5qzUu5T4PjA0M2k05GpUDJ/9oXVHKKv5mM7DBFxsYPu7U0kzHdXnYvhR4QtjMm2s3F/RxyhoEK0GMFpcdLcg8STWMbaawryspuw9HWpBIxkHSyzJExqBIGYZ6+KG1V2Wiyt/OQ51gZY98JQ6fL1Qtk0shvliAzhiCne+93rCM93TCyd61PWujceRO3/7gUnQtkqHEWmmgC+GfKiZUR72k9dJLc9f+vzeRFcEvROlKFk4KPyPi35fRkKRIjj7z/8K9uHbD9Mc766a6+XuJgH9soPBYXl293zpgbtDQWW2NJML875vBR5woSvSaFGKhsjZi2KFn/fJa4GjMwEnIkAwTU= ubuntu@s22' > ~borg/.ssh/authorized_keys
sudo chown -R borg:borg ~borg/.ssh

borg init --encryption=repokey borg@10.1.1.21:MyBorgRepo
borg create --stats --list borg@10.1.1.21:MyBorgRepo::"MyFirstBackup-{now:%Y-%m-%d_%H:%M:%S}" /etc /root
borg list MyBorgRepo::MyFirstBackup-2022-02-28_08:44:36

borg init --encryption=repokey borg@10.1.1.21:MyBorgRepo2
borg create --stats --list borg@10.1.1.21:MyBorgRepo2::"MySecondBackup-{now:%Y-%m-%d_%H:%M:%S}" /home/ubuntu/backup

MySecondBackup-2022-02-28_08:53:23
borg list MyBorgRepo2::MySecondBackup-2022-02-28_08:53:23

borg extract borg@10.1.1.21:MyBorgRepo2::MySecondBackup-2022-02-28_08:53:23
:::