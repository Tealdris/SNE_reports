###### tags: `LS`
:::success
# LS Lab 4 Assignment: NoSQL
**Name: Radomskiy Ilia**
:::


## Task 1:  Take NoSQL solution
:::info
**REDIS** is my choise for this lab
:::

---
## Task 2: NoSQL cluster theory

:::info
Give the answers for the following questions what is and for what, will be better to provide explanatory pictures/graphs:
:::

1. Redundancy: 
In [information theory](https://en.wikipedia.org/wiki/Redundancy_(information_theory)) redundancy it is additional data to actual data. With the help of this additional part, the main part can be recovered. It can be represented as Euler diagrams. In the example below, there is an explanation of Hamming code 7;4. We can use 3 additional redundancy bits in order to recover a 1-bit error in 4 information bits. In order to check messages on errors, we need to solve the equation below.

<center>

![](https://i.imgur.com/U9uu7GK.png)
    
![](https://i.imgur.com/ajT5hBt.png)

</center>

If we speak about [Data Redundancy](https://en.wikipedia.org/wiki/Data_redundancy), then it is a similar term to the previous one. It can be as additional information about the databases like in the previous case, but it also can be a complete replica of the information in the database. 

2. HA (nice): 
it is better when your data is available every time. Even during a data server crash or high load of the network. If a high-available database cluster is configured, then, during a server crash, the application reloads on another part of the cluster. Or the information flows are going on another server.
Of course, applications have to be programmed in order to have the ability to work on that kind of database.
There are several methods of configuring this type od database:
**Active/active**, if one server is down, then the whole flow is going to the others servers. Servers have to be configured similarly
**Active/passive**, a full replica of every node in the database. This passive replica starts working only if the main server is down
**N + 1**, same as **Active/passive**, but for the several applications, in order to save resources, by using one replica server in several applications
**N + M**, same as the previous one, but there are several replicas for the applications
etc.
HA can be measured with an uptime parameter. It is in percent. If this parameter is equal to 90%, then it means the server is available 90% of the time or 21,20 hours a day

3. Fault Tolerance
It is the ability of the system\application to continue its work even after the down, or down of some of the components in the system.
There are some techniques in order to reach [fault-tolerance](https://en.wikipedia.org/wiki/Fault_tolerance#Fault_tolerance_techniques) in databases: 
**Replication**, configuring cluster of nodes, in order for them to work simultaneously
**Redundancy**, copy of the database, during the down, a copy is working instead of the broken node
**Diversity** use several copies of the same server in order to avoid errors
Such architecture as **RAID** can be implemented in order to get any of above techniques


## Task 3 - NoSQL cluster deployment & validation & Fault Tolerance


1. Deploy your NoSQL cluster (e.g. with master, primary, slave, replica...) and investigate/describe its features.

For this lab, my choice is **[Redis](http://redis.io/documentation)**
Deploy cluster is easy:
Install: `sudo apt install redis-server`
Change in `/etc/redis/redis.conf` several parameters: 
```
bind 0.0.0.0 #accept connections
```
After configuration and restart of redis we can see the result of the working master database
```
redis-cli -a foobared info replication
```

<center>

![](https://i.imgur.com/QgM8rvV.png)

</center>

Information in database can be writen with `redis-cli -a foobared set test 'test'`

<center>

![](https://i.imgur.com/aa5P27a.png)

</center>

after that, we need to configure slave node. Installation is the same. But configuration of `/etc/redis/redis.conf` is little bit different.

```
replicaof ip\dns port
masterauth password
requirepass password
```

After restart, we receive information about successful replication, and can get `test` message from the server

<center>

![](https://i.imgur.com/4gAuwrY.png)

</center>


2. Validate all the features by operating with some dataset. Create new nodes/ destroy one…

By adding some data to the database we can validate your cluster

<center>

![](https://i.imgur.com/qW3bB09.png)
![](https://i.imgur.com/UNB1B1W.png)

</center>

3. Describe and validate the election process of the primary.

The master node is chosen by the administrator, but if the cluster is configured in the `Sentinel` node. The cluster will be automated, and in the moment of the down master will be changed
In order to configure monitoring additional config file was added to every monitoring node

```
sudo redis-server /etc/redis/sentinel.conf --sentinel &
```
```
sentinel monitor redis-test 10.1.1.16 6379 2
sentinel down-after-milliseconds redis-test 6001
sentinel failover-timeout redis-test 60000
sentinel parallel-syncs redis-test 1
bind 0.0.0.0
sentinel auth-pass redis-test foobared
```

After that, there are three monitoring servers

<center>

![](https://i.imgur.com/uRKgU8x.png)

</center>>

4. Destroy the nodes and describe what will happened?

If one of the slave nodes is down then, nothing happens, in this topology, there are only master matters. But if the master is down, information is available but not for the writing

<center>

![](https://i.imgur.com/pPQmzq5.png)

</center>

So after master node is dead, if `sentinel`, like in subtask 3 is described, is configured, then monitor server notice the change in behaviour, and another master server will be chosen

5. Try to add new nodes after cluster is created, show them. How hard is it?

in order to configure the slave node. I need to change only 3 parameters`/etc/redis/redis.conf` .

```
replicaof ip\dns port
masterauth password
requirepass password
```

After restart we receiving information about successful replication, and can get `test` message from the server

<center>

![](https://i.imgur.com/4gAuwrY.png)
    
![](https://i.imgur.com/PX7O6FL.png)

</center>

?. Sharding Cluster

In the example below i have configured cluster from 6 nodes (3 servers). In this cluster information is written to the servers with the sequence of changing servers. So every new set is stored on the different node.
After configuring three all nodes in the same way, and enabling cluster mode `cluster-enabled yes` we can see result on the picture below

<center>

![](https://i.imgur.com/LEa3KVL.png)
All nodes are in the cluster
    
![](https://i.imgur.com/QzoR3fE.png)
![](https://i.imgur.com/NPWv5av.png)
![](https://i.imgur.com/UMlf8UC.png)
New data is written to the different servers
    
![](https://i.imgur.com/uY8pPyg.png)
Data in the cluster is separated between the nodes

</center> 

**Bonus:** deploy your NoSQL Cluster as Ansible roles

---
## References:

1: [information theory wikipedia](https://en.wikipedia.org/wiki/Redundancy_(information_theory))
2: [Data Redundancy wikipedia](https://en.wikipedia.org/wiki/Data_redundancy)
3: [fault-tolerance wikipedia](https://en.wikipedia.org/wiki/Fault_tolerance#Fault_tolerance_techniques)
4: [Redis documentation](http://redis.io/documentation)
5:

:::spoiler

sudo ufw enable
sudo ufw allow 27017 8 9

https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-20-04-ru

install
curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
sudo apt update
sudo apt install mongodb-org
sudo systemctl start mongod.service

mongo --eval 'db.runCommand({ connectionStatus: 1 })'

mkdir /data/db/
mongo --host localhost

mkdir /data/node1
mkdir /data/node2
mkdir /data/arbiter
sudo mongod --replSet myapp --dbpath /data/node1 --port 40000 &
sudo mongod --replSet myapp --dbpath /data/node2 --port 40001 &
sudo mongod --replSet myapp --dbpath /data/arbiter --port 40002 &

mongo --port 40000
rs.initiate()
rs.add(localhost:40001)
rs.addArb('local:40002')

rs.status()
mongo --port 40001

secondaryOk()

sudo mongos --configdb circle/127.0.0.1:27100 --port 27002


work
db # current db
show collections
show dbs
use nameofdb # create db (but more like  switch to created one or new)
db.createCollection("users") # create collection incide of db

db.users.insertOne({"name":"john","email":"something","age":23,"car":true})

db.users.insertOne(
    {
        "name":"john",
        "email":"something",
        "age":23,
        "car":true,
        "favc":["zeleny","krasny","cherny"],
        "child": {"name": "jack", "surename":"charley","age":5}
    }
)

https://www.digitalocean.com/community/tutorials/how-to-configure-remote-access-for-mongodb-on-ubuntu-20-04

remote access
sudo lsof -i | grep mongo
port
sudo ufw allow from 10.1.1.67 to any port 27017
sudo ufw allow from 10.1.1.67 to any port 22

27017
sudo ufw enable
sudo ufw status
in
sudo nano /etc/mongod.conf
```
# network interfaces
net:
  port: 40002
  bindIp: 127.0.0.1,10.1.1.25
```
sudo systemctl restart mongod
nc -zv 10.1.1.13 27017


sudo mongod --dbpath /data/instance1/ --config /etc/mongod.conf --port 27000

sharding

sudo mkdir /data/instance1 x2
sudo mongod --dbpath /data/instance1 --port 27000 & x2
sudo mkdir /data/config
sudo mongod --configsvr --dbpath /data/config --port 27002 &
sudo mongos --configdb confmong/127.0.0.1:27002 --port 27100 &
mongo --port 27002


sh.addShard("127.0.0.1:27000")

sudo mongos --configdb confmong/127.0.0.1:27002 --port 27100 &



sudo apt update
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list

sudo apt-get update
sudo apt-get install -y mongodb-org=4.4.8 mongodb-org-server=4.4.8 mongodb-org-shell=4.4.8 mongodb-org-mongos=4.4.8 mongodb-org-tools=4.4.8

sudo systemctl enable mongod
sudo service mongod restart
sudo service mongod status
mongo
use admin
db.createUser( { user: "mongoadmin", pwd: "asdf1234", roles: [ "root" ] });
exit
db.createUser( { user: "mongoadmin", pwd: "asdf1234", roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] } );


server
sudo openssl rand -base64 741 > keyfile
sudo mkdir -p /opt/mongodb
sudo cp keyfile /opt/mongodb/
sudo mkdir Document
sudo cp keyfile Document/
sudo chown mongodb:mongodb /opt/mongodb/keyfile
sudo chmod 0600 /opt/mongodb/keyfile
sudo chmod 777 Document/keyfile
sudo nano /etc/mongod.conf
security: 
  keyFile: /opt/mongodb/keyfile
  authorization: enabled
sudo service mongod restart
sudo service mongod status

others
sudo mkdir -p /opt/mongodb
sudo cp keyfile /opt/mongodb/
sudo chown mongodb:mongodb /opt/mongodb/keyfile
sudo chmod 0600 /opt/mongodb/keyfile
sudo nano /etc/mongod.conf
security: 
  keyFile: /opt/mongodb/keyfile
  authorization: enabled
sudo service mongod restart
sudo service mongod status

all
sudo nano /etc/mongod.conf
replication: 
  replSetName: mongodb-rs
sudo service mongod restart
sudo service mongod status

serever
mongo --host 10.1.1.13 --username mongoadmin --password asdf1234
rs.initiate()
rs.add("10.1.1.14:27017")
rs.status()

rs.initiate( {
   _id : "rs0",
   members: [
      { _id: 0, host: "10.1.1.11:27017" },
      { _id: 1, host: "10.1.1.15:27017" },
      { _id: 1, host: "10.1.1.12:27017" },
   ]
})


sudo brctl addbr br0
sudo ip l s dev br0 up
sudo tunctl -t tap0 -u st10
sudo ip l s dev tap0 up
sudo brctl addif br0 tap0
sudo brctl addif br0 eno1
sudo ifconfig eno1 0.0.0.0
sudo ip a a 10.1.1.67 dev br0




https://success.outsystems.com/Support/Enterprise_Customers/Installation/Configuring_OutSystems_with_Redis_in-memory_session_storage/Set_up_a_Redis_Cluster_for_Production_environments


sudo apt-get update
sudo apt-get install redis-server
sudo systemctl disable redis-server.service

sudo ufw allow 7000
sudo ufw allow 7001
sudo ufw allow 17000
sudo ufw allow 17001
sudo ufw status

sudo nano /etc/rc.local
```
#!/bin/sh -e
echo never > /sys/kernel/mm/transparent_hugepage/enabled
sysctl -w net.core.somaxconn=65535

exit 0
```
sudo chmod +x /etc/rc.local
sudo nano /etc/sysctl.conf
```
vm.overcommit_memory=1
```
sudo mkdir /etc/redis/cluster
sudo mkdir /etc/redis/cluster/7000
sudo mkdir /var/lib/redis/7000
sudo mkdir /etc/redis/cluster/7001
sudo mkdir /var/lib/redis/7001
sudo nano /etc/redis/cluster/7000/redis_7000.conf
sudo nano /etc/redis/cluster/7001/redis_7001.conf

sudo chown redis:redis -R /var/lib/redis
sudo chmod 770 -R /var/lib/redis
sudo chown redis:redis -R /etc/redis

sudo nano /etc/systemd/system/redis_7000.service

sudo systemctl enable /etc/systemd/system/redis_7000.service
sudo systemctl enable /etc/systemd/system/redis_7001.service


redis-cli --cluster create 10.1.1.16:7000 10.1.1.14:7000 10.1.1.13:7000 10.1.1.16:7001 10.1.1.14:7001 10.1.1.13:7001 --cluster-replicas 1
redis-cli --cluster add-node

redis-cli -c -h 10.1.1.16 -p 7000
set a 1

CLUSTER NODES

KEYS *

redis-cli --cluster add-node 127.0.0.1:7000 10.1.1.16:7000

redis-cli --cluster add-node 127.0.0.1:7001 10.1.1.76:7001 --cluster-slave --cluster-master-id e91472e1a1d84c5935830408bbdd8e86335b1ace

sudo systemctl restart redis
redis-cli -a foobared info replication
tail -f /var/log/redis/redis-server.log

redis-cli -a foobared set test2 'test2'
redis-cli -a foobared get test


:::