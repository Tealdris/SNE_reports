###### tags: `LS`
:::success
# LS Lab 2 Assignment: Container Orchestration & Load-balancing
**Name: Radomskiy Ilia**
:::


## Task 1: Task 1 - Choose Container Engine & Orchestration
:::info
Choose a container engine. Some suggestions:
:::


* **Docker** (default choice)
* Singularity
* chroot
* OpenVZ / Virtuozzo
* other exotic choices (be sure that you will be available to deal with it and firstly discuss with TA)
:::info
Deploy a true container cluster farm, across several team machines. 
It is however recommended to proceed in a virtual machine environments so the worker nodes can have the exact same system patch level (which makes it easier).
:::

For this lab, my choice is `Docker`, because of is the most popular containerization tool, in my opinion
I will use the topology below as my cluster. Below you can find commands that are used in order to install `docker`. [How to install Docker swarm](https://itisgood.ru/2020/10/19/kak-ustanovit-docker-swarm-na-ubuntu-20-04/)

<center>

![](https://i.imgur.com/EmuHz4d.png)

</center>

add to the `/etc/hosts` every cluster node. Below you can see this file after the change

<center>

![](https://i.imgur.com/0raemy4.png)

</center>

Below commands are used in order to install `docker`

```bash=
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt-get update
apt-cache policy docker-ce
sudo apt install docker-ce -y 
sudo usermod -aG docker ${USER}
sudo usermod -aG docker $USER
sudo systemctl status docker
```




In order to separate my application into different containers, I will use `docker-compose`.

```bash=
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo docker-compose --version
```

**Bonus**: if you choose Docker, play with alternate storage drivers e.g. BTRFS or ZFS instead of OverlayFS.



:::info
Choose an orchestration engine then:
:::

* **Docker Swarm**
* Kubernetes or local k8s cluster:
* minikube: works on all major operating systems and uses providers like Docker,
* Virtualbox and KVM to power the node
* k3s: lightweight Kubernetes can work on Linux (WSL2 should also work steps here)
* MicroK8s: lightweight Kubernetes compatible with Ubuntu (Linux) distribution (can
* work with MacOS and Windows when using mutlipass?).
* Rancher (advanced)
* other exotic choices (be sure that you will be available to deal with it and firstly discuss with TA)

As for my orchestration tool choice, it will be `docker-swarm`.

:::info
Analyze and describe the best practices of usage the chosen container engine.
:::

If we want to use Docker Swarm in the best possible way, first of all, we need to think about container security. One of them:
1. Rootless containers
2. No constant UID for container
3. Root user of files and they can't be changed
4. Multistage build
5. Build minimalistic contAINERS
6. Use verified builds
7. keep up to date
8. LEft only used ports
9. no sensible data in docker file
10. etc

Second, `docker swarm` best practice itself. 
`docker swarm` allows the defined node in a `swarm` as one of the two members of cluster worker and manages. So, of course it is better if you manage them accordingly, `workers` run containers, and `manager` orchestrate them
Set up logging correctly, because it will be a great help to you in the future. Not only logs for the containers but the orchestration itself. Swam can group logging files.

```bash=
docker service logs nginx_nginx
```

Docker also provided some additions like `Docker Registry`. It is used for:

    tightly control where your images are being stored,
    fully own your images distribution pipeline,
    integrate image storage and distribution tightly into your in-house development workflow;
And you probably should include something like that in your project
Use cloud storage instead of `semi-stateless` (has some shared, not important files) containers
Set up `ENV variables` in the environment section
It is better when your containers are limited in computational powers, so you might use limitation techniques

* But in order to start using those techniques we should start a swarm itself

```bash=
sudo docker swarm init --advertise-addr 192.168.122.100
sudo docker swarm init --advertise-addr 10.1.1.92
```

After implementing those `swarm` is initialized

<center>

![](https://i.imgur.com/hWZzzai.png)

</center>

Those tokens can be used on the worker node if we want to include it to the `swarm`

```bash=
docker swarm join --token Sc 10.1.1.37:2377
```

You can see success message on the worker station on the picture below

<center>

![](https://i.imgur.com/HvBB2h2.png)

</center>

This command below may be used on the `managers` node if we want to check workers

```bash=
sudo docker node ls
```

![](https://i.imgur.com/yHI4ebr.png)


## Task 2 - Distributing an Application/Microservices


:::info
Base level (it means that this task will be evaluated very meticulously): deploy at least a simple application (e.g. a simple web page showing the hostname of the host node it is running upon) and validate that its instances are spreading across the farm. It is literally not necessary to create your own service/application: you can use something from a public repository, for example, from DockerHub. However, no one forbids you to work with self-written projects.
:::

Hint: creating such application is particularly easy to achieve with Swarm when the nodes does not share storage. Same goes for K8s, but you need use a workaround e.g. volumes with hostPath.


If we want to deploy a simple `Nginx` container we can do this with the command below.

```bash=
sudo docker service create --name web-server --publish 8090:80 nginx:1.13-alpine
sudo docker service ls
```

Now container is running

<center>

![](https://i.imgur.com/CLy3dgy.png)

</center>

we can multiply amount of the containers by this command

```bash=
sudo docker service scale web-server=3
sudo docker service ls
```

That is how replicated containers is looking now

<center>

![](https://i.imgur.com/UHEBZ9f.png)

![](https://i.imgur.com/iSxfEd4.png)

</center>


**Semi-Bonus 1** : deploy a microservices instead of standalone application, e.g. full stack web application with web server, database...

To set microservice architecture, i will use the commands below

```bash=
docker-compose up
#or
docker stack deploy --compose-file docker-compose.yml ls2
docker stack rm ls2 # deliting
docker service update ls2_flaskapp # updating service
```

After deploying we can see the results of the service. On the web page, we can see hello message and viewership counter 

<center>

![](https://i.imgur.com/1NNIoKJ.png)
![](https://i.imgur.com/DL3U8gm.png)
    
</center>

In the visualizer container working application is looking in that way

<center>

![](https://i.imgur.com/Gug2vrb.png)

</center>

Microservices included in the app are shown in the picture below.

<center>

![](https://i.imgur.com/tEQpaFx.png)
    
</center>

[How to Deploy Microservices with Docker](https://www.linode.com/docs/guides/deploying-microservices-with-docker/)

**Bonus 2**: use Veewee/Vargant to build and distribute your development environment.

**Bonus 3**: if you use k8s, `prepare` a helm chart for your application.

## 3 - Validation

:::info
Validate that when a node goes down a new instance is launched. Show how the redistribution of the instances can happen when the broken node comes back alive. Describe all steps of tests in your report in details
:::

Now let's try and turn off the `workers` with the running containers

In the three pictures below you can see me turning the containers off.
When the `manager` notice the downstate of the `workers`, he will run those containers on his own machine

<center>

![](https://i.imgur.com/eLtvRCH.png)

![](https://i.imgur.com/Sp3r0lY.png)

![](https://i.imgur.com/t9qW6Xs.png)

</center>

After that, I have turned the `workers` on.
At this point, you have two options to change things back
1. Manual, with the command below
`docker service update --force web-server`
2. Automatic, with the configuration file of `docker compose` 

Below you can see part of the `docker-compose` configuration file. It can be configured in a lot of ways, but I will not cover all of them

```bash=
# you can specify the type of node in claster for the specific work
constraints:
  - "node.role==worker"

# You can set condition and time in which your manager should restart containers on workers
restart_policy:
    condition: on-failure
    delay: 5s
    max_attempts: 3
    window: 120s
```

On the two pictures below you can see the `restart` process

<center>

![](https://i.imgur.com/WcGkeB5.png)

![](https://i.imgur.com/eLtvRCH.png)

</center>

Also everything above were implemented with the help of my groupmate Iseoluwa. Some of my containers are running on this machine `is 3`

<center>

![](https://i.imgur.com/f9i93na.png)

</center>

[docker-compose documentation](https://docs.docker.com/compose/compose-file/compose-file-v3/)

## Task 4 - Load-balancing

:::info
Choose a load-balancing facility, which will be instlled and tested against your orchestrator. 

You can choose either l3 or l7 solutions. The disadvantage of layer-3 is that web sessions may break against statefull applications. 

There is a trick to deal with that, though, for example with OpenBSD Packet Filter.
:::

Note: for K8S, you can try to do Ingress.

Choose a load-balancer to distribute the load to the worker nodes, for example either layer 3:

* layer-3 BSD pf / npf / ipfilter/ipnat
* layer-3 Linux Netfilter ( iptables )
* layer 3 Linux nftables ( nft )
* layer-3+7 HAProxy ybiidekue928 or HTTP reverse-proxy:

* layer 7 **NGINX** / NGINX Plus (dynamic objects?)
* layer 7 Apache Traffic Server? (static objects?)
* layer 7 OpenBSD Relayd
* K8S Ingress / Ingress-NGINX



:::info
Swarm or K8S’s network overlay is already spreading the requests among the farm, right? So why would a load-balancer still be needed? Explain and show briefly how a real-life network architecture look like with a small diagram.
:::

[Docker Swarm Load Balancing with NGINX](https://www.nginx.com/blog/docker-swarm-load-balancing-nginx-plus/)


`Swarm` load balancer is based on Layer 4 (TCP). Many applications require additional features, like these:

```
    SSL/TLS termination
    Content‑based routing (based, for example, on the URL or a header)
    Access control and authorization
    Rewrites and redirects
```
"

We might need another balancer if we want to add protocols, balancing algorithm or configure logging
In order for NGINX to be configured in the balanced mode, we have to edit the configuration file of the application. You can see the configuration file in the picture below

<center>

![](https://i.imgur.com/K5UhQBn.png)

</center>

As a result of the balancing we can see, that we can access different stations from the same ip address

<center>

![](https://i.imgur.com/867Rg61.png)

</center>

## Bonus - Autoscaling

:::info
1. For those who chose Kubernetes, setup a Horizontal Pod Autoscaler. Perform a stress test and validate that the number of pods is changing depending on10.1.1.77/24 the load.
:::

Hint: to perform a stress test you can use the stress utility.

:::info
2. Bonus for the others: how to scale instances in the Docker Swarm or another facility? Could it be done automatically?
:::

According to this answer [answer](https://stackoverflow.com/questions/41668621/how-to-configure-autoscaling-on-docker-swarm), there is no such possibility out of the box. But it can be done via scripting

But [orbiter](https://github.com/gianarb/orbiter) or others, for example  may be your choice. 

In the two below examples you can see implementation none of the [solutions](https://github.com/jcwimmer/docker-swarm-autoscaler). It is written in `bash`
In the first picture there are 3 nodes working, and, at the moment of high load on the service, auto scaling starts on all of thee nodes

<center>

![](https://i.imgur.com/f48kHFz.png)

</center>

On the second picture there is situation when only one node is working. In that case all additional containers appears on the the only node alive

<center>

![](https://i.imgur.com/ZeBHhXt.png)

</center>

## Bonus - Updates & Monitoring (theoretical)

1. Suppose your application must be updated. Describe the process of applying image or instance updates with your orchestrators of choice, and do it with a minor change in your sample application. In other words, how to ship the containers depending on your engine and orchestrator?

If i need to update application and deploy changes i can do it in the way it is shown below.
* On the first picture i have made i small change
* On second picture i am updating application
* Third picture is showing the result

<center>

![](https://i.imgur.com/7VleKpv.png)

![](https://i.imgur.com/V5waTP3.png)

![](https://i.imgur.com/ZvMDBRu.png)

</center>

2. It is always a good practice to do monitoring and logging. How can this be done with your orchestrators of choice? Are there built-in tools or must you use third-party ones?

Most of the monitoring tools are docker containers, they are running on the server and collecting data.
There is simple one, that included in the `dockersamples` list.

You already have seen it in work, some of the screenshots are made with the use of it. And it can be diployed with the command below.

```bash=
sudo docker service create --name=vizualization --publish=8080:8080/tcp --constraint=node.role==manager --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock dockersamples/visualizer
```

There are more complex solutions like:
[cAdvisor](https://github.com/google/cadvisor),
[InfluxDB](https://github.com/influxdata/influxdb),
[Grafana](https://github.com/grafana/grafana),
[WordPress](https://sysdig.com/blog/monitor-docker-swarm/) with Sysdig Monitor,
[etc](https://youtu.be/3uiea53t1L4);

For example you can see `cAdvisor` web page in the picture below. Plugin allow to monitor performance of the machines and containers, and usage of computational powers

<center>
    
![](https://i.imgur.com/1DhJMir.png)
![](https://i.imgur.com/jEMkYPI.png)

</center>

