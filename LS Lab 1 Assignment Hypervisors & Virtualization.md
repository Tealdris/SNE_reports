###### tags: `LS`
:::success
# LS Lab 1 Assignment: Hypervisors & Virtualization
**Name: Radomskiy Ilia**
:::

---

## Task 1 - Choose virtualization technology:

:::info
Linux KVM (default choice, but guest configs are XML and harder to synchronize among the
farm...)
:::

My chose for this lab is Linux KVM, because it is standard choose


---

## Task 2 - Local implementation:

:::info
Agree on which host system version you’re going to work on with your teammate. Different OS and versions may work and will make task 2 even more interesting. But if you don’t want to take any risks, check uname −r and lsb_release −a first. Proceed with the following steps INDIVIDUALLY: everybody needs to know how to set up a single and true virtualization host.
:::

I will be using  `system version ubuntu-20.04.3-live-server-amd64.iso` as my host system. It can be found on the `http://nz.archive.ubuntu.com/ubuntu` mirror

**Host install** – Install the hypervisor and host tools on a physical machine (make sure VT/AMD-V is enabled).

Before installation, we have to check if your machine supports virtualization or not
test.
There are several methods, some of them are shown below.

```
grep -e 'vmx' /proc/cpuinfo
```

<center>

![](https://i.imgur.com/a681d9t.png)

</center>

```
kvm-ok
```

<center>

![](https://i.imgur.com/ho82i8N.png)

</center>

```
uname -m
```

<center>

![](https://i.imgur.com/xIwG3da.png)

</center>

After some tests, we can start installing packages

```
sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
```

Users are required to be configured for the KVM server to work normally

```
sudo adduser `id -un` libvirt
sudo adduser `id -un` kvm
```

With the command below we can verify that our environment works fine
```
virsh list --all
```

<center>

![](https://i.imgur.com/sDyOKx5.png)

</center>

After that we can install GUI manages for the KVM.

```
sudo apt-get install virt-manager
```

[ KVM/Installation](https://help.ubuntu.com/community/KVM/Installation)

**Guest install** – Install a guest with a local virtual disk with a method that fits best, just to validate that you get a VM up and running. If using XEN, be clear and provide details on what type of guest you chose.

**First-way**, create disk and use it inside of the VMM.

1. the First step is to create the disk. There are 2 options: `raw` and `qcow2`.

<center>

![](https://i.imgur.com/cfrY1dl.png)
    
</center>

<center>

![](https://i.imgur.com/jZWJsmD.png)

</center>

The difference between them is that raw disk occupies all the memory that has been given to it
In the case of qcow2, the disk is smaller because unused parts of the disk is available for the others

After `iso` of the OS and `img` of disk are chosen new VM instance appears in the VMM

<center>

![](https://i.imgur.com/Sqyza7Y.png)

</center>

If we start this machine, the process of the installation will begin. After installation is finished we can enter to the new VM

<center>

![](https://i.imgur.com/7WAboVi.png)

</center>

[How to create virtual machines on Linux using KVM](https://wiki.merionet.ru/servernye-resheniya/58/kak-sozdavat-virtualnye-mashiny-na-linux-s-pomoshhyu-kvm/)

**Sparse-file virtual disk** – Install another guest hard and DIY way. Create a SPARSE file (either with dd seek or QCOW2, for example), mount it in a folder, and use `debootstrap` to get a Ubuntu Server system over there, quick & dirty. For VMware/HyperV users, try to do something similar.

**Second-way**, create the disk, mount it, and after that copy your OS to the mounted disk. After unmounting you will receive `img` disk with the OS on it.

In the pictures below you can see difference between raw and `qcow` disks with the commands that were used in order to create them

```
sudo dd if=/dev/zero of=ubuntu-box1-vm-disk1-1G bs=1M count=1120 status=progress
```

<center>

![](https://i.imgur.com/MD2K28D.png)

</center>

```
sudo qemu-img create -f qcow2 ubuntu-box2-vm-disk1-5G 5G
```

<center>

![](https://i.imgur.com/sZn5hsb.png)

</center>

Below you can see steps that are performed in order to set VM

Creation of the disk
```
dd if=/dev/zero of=ubuntu-server.img bs=1M count=1 seek=4095 sudo losetup --show -f ubuntu-server.img
```

Creating partitions
```
sudo parted /dev/loop29 mklabel msdos
sudo parted /dev/loop29 mkpart pri ext2 0% 100%
sudo mkfs.ext4 /dev/loop29p1 
```

Mounting and downloading ubuntu os
```
sudo mount /dev/loop29p1 /mnt
sudo debootstrap focal /mnt http://nz.archive.ubuntu.com/ubuntu
sudo mount -o bind /dev /mnt/dev
```

Starting console in order to set some setting inside the ubuntu

```
LANG=C sudo chroot /mnt
```

<center>

![](https://i.imgur.com/qUOSksN.png)

</center>

Enabling Virsh Console Access For KVM Guests, this step will be useful later
```
systemctl enable serial-getty@ttyS0.service
```

These mounting commands are used in order for the environment to looks normal, and for the success next commands
```
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t dev dev /dev
```

Setting up entry for the root filesystem
```
cat << EOF > /etc/fstab
UUID=$(blkid /dev/loop20p1 | cut -d\" -f2) / ext4 errors=remount-ro 0 1
EOF
```

Set repositories and update OS
```
cat << EOF > /etc/apt/sources.list
deb http://nz.archive.ubuntu.com/ubuntu focal main restricted universe multiverse
deb http://nz.archive.ubuntu.com/ubuntu focal-updates main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu focal-security main restricted universe multiverse
EOF
apt update && apt -y dist-upgrade
```

Install additional programs and cleaning after installation
```
apt -y install linux-image-generic grub2-common openssh-server bridge-utils ethtool nano

apt clean
```

These lines were added to the `/etc/default/grub` in order to enable serial console

```
GRUB_CMDLINE_LINUX_DEFAULT="console=tty0 console=ttyS0,115200n8"
 
GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"
```

Then we need to remove `rm /etc/grub.d/30_os-prober` file on order for grub not to add entries for operating system

```
update-grub
grub-install --force /dev/loop29
```

<center>

![](https://i.imgur.com/KBmb5W8.png)

</center>

adding user for the os 
```
groupadd admin
useradd -s /bin/bash -m -d /home/user -G admin user
passwd user
passwd user
```

After everything is setted up we can exit this console and unmount disk
```
exit
sudo umount /mnt/{dev,proc,sys} /mnt
```

in order to compress our `img` we can perform the next command
```
sudo zerofree /dev/loop29p1
```
and remove the loop interface because we don't need it in our machine
```
sudo losetup -d /dev/loop 29
```

now we can start this machine in two different ways
with CLI or with GUI
The command for the installation is shown below
```

virt-install --virt-type kvm --name ubuntu-server --vcpus 2 --memory 2048 --os-variant ubuntu20.04 --disk=ubuntu-server.img --import --network bridge=virbr0
```

This command can be used in order to start the console 
```
virsh console ubuntu20.04
```

But, if everything was configured correctly, we can see console window with the ubuntu running in it

<center>

![](https://i.imgur.com/R8L3ERi.png)

</center>

[Exinda 4010 re-purposing with Ubuntu](https://jeremy.geek.nz/tag/debootstrap/)

**Network** – Setup the network manually (always show how you do it in the report). For Community XEN you need to setup a bridge. For `KVM/libvirt`, please get rid of the default setup and do it yourself. In other words, make your guests capable to obtain a DHCP lease from the SAME network (or any other local network in the case of remote work).

If we want to configure network settings inside VM we need to edit `/etc/netplan/01-default.yaml` file.
This file can be modified during installation (previous step), inside VM, or from VM manager. In the picture below you can see what does network configuration file looks like from the VM manager. In this case i am using network `new` and interface `virbr0`

<center>

![](https://i.imgur.com/oFUAsIR.png)

</center>

Somehow interfaces on my VM weren't turned on, so I have to perform additions steps

```
sudo ip link set enp1s0 up
sudo dhclient
```

In the pictures below you can see ip addresses of the VM's. And the ability ot pin each other, so they are located in the same network

<center>

![](https://i.imgur.com/WaMOxc4.png)

![](https://i.imgur.com/mhOzJWS.png)

</center>

or  VM's can be isolated in the new network. For that purpose, I need to create a new bridge and interface.
```
sudo brctl addbr br0
sudo ip l s dev br0 up
sudo tunctl -t tap0 -u st10
sudo ip l s dev tap0 up
sudo brctl addif br0 tap0
sudo brctl addif br0 eth0
sudo ifconfig eth0 0.0.0.0 promisc
sudo ip a a 192.168.100.1/24 dev br0
```
[Connecting the router in GNS3 to the network.](https://mik17.wordpress.com/2009/11/02/%D0%BF%D0%BE%D0%B4%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%BD%D0%B8%D0%B5-%D1%80%D1%83%D1%82%D0%B5%D1%80%D0%B0-%D0%B8%D0%B7-gns3-%D0%BA-%D1%81%D0%B5%D1%82%D0%B8/)

In the picture below you can see that network has changed for the two of the machines, and they can ping each other. It is possible to configure several ip addresses on one machine you can see it on the second machine.

<center>

![](https://i.imgur.com/DaHp7K1.png)

![](https://i.imgur.com/d3tSA5R.png)

![](https://i.imgur.com/yfL5HiY.png)

</center>

[Creating an isolated network for KVM virtual machines](https://bozza.ru/art-266.html)

The last example is about how we can obtain the IP address from the same network as our machine. For this purpose I have changed the setting in the `/etc/netplan/01-networkmanager-all.yaml` file, you can see it in the picture below.

<center>

![](https://i.imgur.com/jb2OIBQ.png)
</center>

Second, I need to change the network settings for the VM. It can be done with VMM

<center>
    
![](https://i.imgur.com/3IKD4CZ.png)

</center>
    
As you can see IP address has changed to the SAME network
<center>
    
![](https://i.imgur.com/mTZS7o6.png)
    
</center>

**Text console** – Make sure you can reach the text console of the guest from the host. What configurations allow for both, the kernel and the userland system to show up there?
Eventually, disable the graphical console.

As we have settled before `grub` setting are changed like it is shown in the picture below

<center>

![](https://i.imgur.com/exT0d1H.png)

</center>

Another way with which we can disable graphical console it is ` --graphics none`, during the process of installation, it is shown below

```
virt-install --virt-type kvm --name Guest2 --vcpus 2 --memory 4000 --os-variant ubuntu20.04 --disk ~/Guest2.img --import --network bridge=br12 --graphics none
```

In the picture below you can see the difference with the graphical console being on and off

<center>

![](https://i.imgur.com/i49ybHg.png)
![](https://i.imgur.com/UbW7tLC.png)

</center>

It is also can be disabled inside `yml` file

<center>

![](https://i.imgur.com/Ox0JDD9.png)

</center>

**Snapshot** – Proceed with a hot-snapshot, meaning while the guest is running, take it. Attempt to make sure that the file system was properly dealt with…

Hot snapshot can be done if the system is running on the `qcow2` file system, but `raw` can be converted into one

```
qemu-img convert -f raw -O qcow2 image-name.img image-name.qcow2
```

If we want to create a snapshot we can perform the command below
```
virsh snapshot-create-as --domain ubuntu-serverq
```

<center>

![](https://i.imgur.com/LMvqC71.png)

</center>

And restore machine with this one
```
virsh snapshot-revert ubuntu-serverq --snapshotname ubuntu-serverq_snap
```

In the picture below you can see the process of restoring VM from snapshot.
In the beginning, there is no file `1234`
After I have created this file
Then restore the machine from the snapshot
After this step, there is no such file

<center>

![](https://i.imgur.com/l3KHvtk.png)

</center>

Or, we can show this in the other example. The command below will delete everything on the system
```
sudo rm --no-preserve-root -rf /
```
After execution it is not possible to run any command, system is destroyed

<center>

![](https://i.imgur.com/IaWUBUE.png)

</center>

After restoration, we can see that everything is back to normal
```
virsh snapshot-revert ubuntu-serverq --snapshotname ubuntu-serverq_snap
```

<center>

![](https://i.imgur.com/dwifSvj.png)

</center>

[How to Create KVM Virtual Machine Snapshot with Virsh Command](https://www.linuxtechi.com/create-revert-delete-kvm-virtual-machine-snapshot-virsh-command/)

---

## Task 3 - Cluster validation:

:::info
Now as a team, choose which one of the two machines is also going to provide the shared storage for virtual disks to live in. Set it up and share both ideally, guests virtual disks and configurations.

* Take team member 1’s favorite guest (virtual disk and configuration) and put it in the shared storage
* Take team member 2’s favorite guest (virtual disk and configuration) and put it in the shared storage
* Eventually fix the pathes in the configuration and validate that both guests run as well as before
* Now shut them down and run them on the other team member’s machine/host (cold-migration)
* Now don’t even shut them down while migrating… (hot-migration/live-migration)

You have to show a live DEMO of task 3 and bonuses. So, record a video of your process and upload among with report. By the way, you still need to include this task in your report as usual.
:::

The first thing to do is to install the application for the shared library
```
sudo apt-get install sshfs
```
After that, I have added a new user in sake of my groupmate-Ivan convenience. Also some additional things were done, for example new directory was made as well as new group and etc.
```
sudo adduser shared_storage
mkdir ~/shared_storage
chmod 777 /etc/fuse.conf
sudo groupadd fuse
sudo usermod -a -G fuse shared_storage
sudo chown fuse:fuse /etc/fuse.conf
sudo chown -R libvirt-qemu:kvm /home/shared_storage
```
Inside file `/etc/fuse.conf` i have added the line `user_allow_other` because i this allows for the others users to work in a shared directory. All things that were made above are similar for the host of my groupmate
```
sudo nano /etc/fuse.conf
```

<center>

![](https://i.imgur.com/2hp1Tmj.png)

</center> 

After that, we are ready to mount a directory from the host machine of my groupmate

```
sudo sshfs -o allow_other,default_permissions shared_storage@10.1.1.211:/home/shared_storage ~/shared_storage
```

On the picture below you can see mounted, shared directory on my PC, with the images of my groupmate

<center>

![](https://i.imgur.com/fLjzbsK.png)

</center>

On the screenshot below you can see my machines: `my_ubuntu_qc` and `my_ubuntu_im`; and machines of my groupmate: `ubuntuint-qcow` and `ubuntuint-img`

<center>

![](https://i.imgur.com/sTlK2BC.png)

</center>

On the two screenshots below you can see them running

<center>
    
![](https://i.imgur.com/6jpN2yc.png)
    
![](https://i.imgur.com/MnO4wvW.png)

</center>

On the two pictures you can see procee of migarion:

![](https://i.imgur.com/oGNJgOK.png)
Ivan creating file on VM

![](https://i.imgur.com/6t0m8l1.png)
cold migration on my workstation, i am having same file


![](https://i.imgur.com/AbqXwFO.png)
![](https://i.imgur.com/UmQVtsP.png)
i am changing file 

![](https://i.imgur.com/palGQhR.png)
![](https://i.imgur.com/UAmA4uA.png)
Ivan can get content of the file with out we turning the machine off from my side, hot migration

 video file will be provided in moodle



I have done migration with the Alexander too. 

By adding connection his machine to my vmm i can see VM that was created by alexander

![](https://i.imgur.com/YNkrDh1.png)

and there is an option of migration in it

![](https://i.imgur.com/Rg8QfGS.png)

Video will be provided in the moodle (timeline2)

[Connecting and configuring SSHFS on Linux](https://losst.ru/podklyuchenie-i-nastrojka-sshfs-v-linux)

---

## Bonuses:

:::info
Still as a team, choose a solution to orchestrate your guests among the farm, as described in the introduction: enable HA (when a host crashes, dead guests are restarted elsewhere) and DRS (live-migrations shuffles). Some examples:
:::



---
## References:

1: [link's short description (e.g. Installation guide)](https://phoenixnap.com/kb/install-ubuntu-20-04)

:::spoiler

## Bonuses Paolo:

* Where do you find the Ubuntu distribution? 

Official website https://ubuntu.com/download/desktop

What are the steps that take a Linux kernel distribution and create an Ubuntu distribution? 

In order to create your own Linux distribution you can take several approaches:
Distribution creation tool
Or create if from the scratch if you want the journey

Who provides the funding for the creation of the Ubuntu distribution? 

Ubuntu is developed by the British company Canonical, and supported by the people who use premium services or make donations.

*  Enumerate different package managers used by the different distributions of Linux. What are the differences between them? 

DPKG - Debian, base package system management:
APT supports all necessary functions,
Synaptic graphical version;
RPM - Redhead:
yum similar to apt,
DNF yum next generation;
PackMan - Arch, used for sake of arch users convenience, have a system to synchronize a list of packages with the server
Portage - Gentoo, you can build packages from sources during installation, also you can install several versions of the package
Flatpack - fedora, same as snap, but you can add dependencies to the installation package

SNAP - universal, programs are self-sufficient, dependencies included

*  Two VMs hosted on the same physical computer are isolated, but it is still possible for one VM to affect the other VM. How can this happen? 

those two machines share resources: memory, computational powers. If one VM is running a heavy process/program it can cause a performance drop on the other machine
can be located in the same network, so the data, transferred through it can interfere 

*  We’ve focused on isolation among VMs that are running at the same time on a hypervisor. VMs shut down and stop executing, and new VMs startup. What does a hypervisor do to maintain isolation, or prevent leakage, between virtual machines?

CPU, network, and disk scheduling are used to give to the VM isolation. This technique allows us to share physical machine resources with between VM without or with a very small impact on each other

**create disk**
1
qemu-img create -f qcow2 vdisk2 20G
qemu-img info vdisk1

2
raw(not sparse)
qemu-img create -f raw raw-sparce.img 20G

3
sudo qemu-img create -f raw ubuntu-box1-vm-disk1-5G 5G

sudo dd if=/dev/zero of=ubuntu-box1-vm-disk1-5G bs=1M count=5120 status=progress


sudo qemu-img create -f qcow2 ubuntu-box2-vm-disk1-5G 5G

**attach**

virsh attach-disk ubuntu-20_server --source /home/st10/vdisk3 --target vdc --driver qemu

virsh attach-disk ubuntu-20_server --source /home/st10/raw-sparce.img --target vdc --driver qemu

**install**

sudo virt-install --virt-type kvm --name ubuntu-20 --ram 2048 --cdrom=/home/st10/Downloads/ubuntu-20.04.3-live-server-amd64.iso --disk path=/var/lib/libvirt/images/vdisk2.img


only 1 line but it is to easy
virt-install --name=ubuntu-vm-20G --memory=2048 --cdrom=/home/st10/Downloads/ubuntu-20.04.3-live-server-amd64.iso --disk size=20
https://linuxconfig.org/how-to-create-and-manage-kvm-virtual-machines-from-cli

add ssh
https://linuxhandbook.com/add-ssh-public-key-to-server/
+
exec ssh-agent bash
ssh-add /root/.ssh/id_rsa

:::