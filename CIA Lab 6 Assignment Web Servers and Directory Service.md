###### tags: `CIA`
:::success
# CIA Lab 6 Assignment: Web Servers and Directory Service

**Name: Radomskiy Ilia**
:::

---
---
# Section 1:Web Server

[aaa.st10.sne21.ru](https://aaa.st10.sne21.ru/)
[bbb.st10.sne21.ru](https://bbb.st10.sne21.ru/)


## Task 1 - Install & Configure Virtual 

:::info
1. Fetch, verify, build and install the webserver daemon from the source. 
Note: Some features of your web server may be built-in or modularized. 
Enable at least SSL/TLS during your installation.
:::

Getting apache2 from the source
```
sudo wget https://dlcdn.apache.org//httpd/httpd-2.4.49.tar.bz2
```
and getting its certificate
```
sudo wget https://downloads.apache.org/httpd/httpd-2.4.49.tar.bz2.asc
```

I get information about signature but we need further more investigation on it like finding information about its public key
```
 gpg --verify ~/appache2/httpd-2.4.49.tar.bz2.asc ~/appache2/httpd-2.4.49.tar.bz2
 ```
 
 <center>
 
 ![](https://i.imgur.com/ArWpR2m.png)
 Information about signature
 </center>
 
 Now I have downloaded its public key created by `stefan`
```
sudo gpg --keyserver pgpkeys.mit.edu --recv-key 26F51EF9A82F4ACB43F1903ED377C9E7D1944C66
```

<center>

![](https://i.imgur.com/E3J0TaL.png)
Publick key importing
</center>

Now we can verify the signature with public key

```
 sudo gpg --verify ~/appache2/httpd-2.4.49.tar.bz2.asc ~/appache2/httpd-2.4.49.tar.bz2
```

<center>

![](https://i.imgur.com/oUljKwA.png)
Verifying signature
</center>
 
[https://www.apache.org/info/verification.html] 

From this point, we can start installing apache form the source

Unpacking
```
sudo tar xvf httpd-2.4.49.tar.bz2
```

Installing  dependencies
```
sudo apt-get install libapr1-dev libaprutil1-dev
sudo apt install libpcre3-dev
sudo apt install libssl-dev
```

Configuring apache into the special library with SSL, and with additional utils
``` 
 sudo ./configure --with-apt-util=/home/ubuntu/apache2/apr-util/apu-config.in --with-apr=/home/ubuntu/apache2/apr/apr-1-config --enable-ssl --prefix=/usr/local/apache
``` 

Installation  process

```
sudo apt install make
sudo make 
sudo  make install
``` 

Main config file is `sudo nano conf/httpd.conf` and after editing it we can apply changes by printing `sudo bin/apachectl -k start`
 
<center>

![](https://i.imgur.com/RyJBCiv.png)
Apache is working on port 80 
![](https://i.imgur.com/fEODBr2.png)
Preinstalled site is already running
</center>

[https://httpd.apache.org/docs/2.4/install.html]

:::info
2. Define the root directory and then two virtual hosts (and configure DNS records or wildcard accordingly):
aaa.st10.sne21.ru
bbb.st10.sne21.ru
:::

After apache is running we can  add records about the sites in the DNS zone, and sign them after that

<center>

![](https://i.imgur.com/j8wfppA.png)
Zone records
</center>

Virtual hosts allowing us to use several sites on one apache server.
COnfiguration file is located in `sudo nano conf/extra/httpd-vhosts.conf` file

<center>

![](https://i.imgur.com/UvrmmvK.png)
Configuration file of apache server
</center>

:::info
3. Create a simple, unique HTML page for each virtual host to make sure that the server can correctly serve it.
:::

We have seen preinstalled HTML web page via curl output, now I have created two HTML pages, for two signs in the zone, and we are able to visit the through web browser

<center>

![](https://i.imgur.com/KeginXP.png)
![](https://i.imgur.com/BXe9iOY.png)
Two web pages for `aaa` and `bbb` records
</center>

Two web pages can see in the picture below

<center>

![](https://i.imgur.com/mbte2oe.png)
![](https://i.imgur.com/EqSKLIf.png)
html code
</center>

[https://www.digitalocean.com/community/tutorials/apache-ubuntu-18-04-ru]

:::info
4. Check the configuration syntax , start the daemon and enable it at boot time.
:::

We can use build-in features of apache like `apachectl`, it can be found in.
```
sudo systemctl start apache2
sudo /usr/local/apache2/bin/apachectl
```
Some of the features like configuration checking and the status list is shown in the picture below

<center>

![](https://i.imgur.com/OsWFwkX.png)
Checking syntax
![](https://i.imgur.com/gliRYmg.png)
checking configuration
![](https://i.imgur.com/XIJg6TS.png)
status of apache throuh systemctl
</center>

:::info
5. Use curl to display the contents of a full HTTP/1.1 session served by your server.
:::

Content of both sites can be displayed or through the browser

<center>

![](https://i.imgur.com/bWyI7T9.png)
Web browser
</center>

Or with curl
<center>

![](https://i.imgur.com/V3Ca7kb.png)
Curl used to display web page
</center>


:::info
6. Explain the meaning of each request and reply header.
:::

We can find information about headers in the burp suite

<center>

![](https://i.imgur.com/7Gpf5AW.png)
Headers that was located by burpsuite
</center>

Request

GET - ask about a resource
POST - send data to the server
PUT - creates a new resource or replaces a representation of the target resource with the request
HOST - who is hosting
USER-AGENT - identification of the application/os vendor
HTTP - a type of data
DATE - date
SERVER - server name
ETAG - resource version



---
## Task 2 - SSL/TLS
:::info
1. Enable SSL/TLS and tune the various settings to make it as secure as possible
:::

SSL is almost required nowadays, so let's sign our website, in order to do that I changed some config files. Enable 443 port, and enable SSL support.

<center>

![](https://i.imgur.com/AsFKIwH.png)
Ports
![](https://i.imgur.com/E5MBaDQ.png)
SSL packet
![](https://i.imgur.com/oHXgFAg.png)
Virtual host addition lines
</center>

[https://www.linode.com/docs/guides/ssl-apache2-debian-ubuntu/]

:::info
2. Describe how you created your own certificate(s) e.g. with Let’s encrypt (certbot) or self-signed and re-validate every virtual-host . Explain your security tuning process.
:::

The first method is to self sign your site
For that, I created certificates for sites, a key you can see on the picture

<center>

![](https://i.imgur.com/tJWvrte.png)
![](https://i.imgur.com/IlKw6s2.png)
![](https://i.imgur.com/Cc5mHpl.png)
![](https://i.imgur.com/fHOcyxZ.png)
Sertificate and key for the site
</center>

From this point, we can try to find information about the site with an SSL checker.
Our site is now signed but other users cant trust it because it is self-signed, and you can see `connection not secure` 


<center>

![](https://i.imgur.com/irCLffH.png)
![](https://i.imgur.com/HdqBMYW.png)
SSLlabs check
</center>

So if you want to improve that you can sign site with someones authority like certbot

<center>

![](https://i.imgur.com/RwYF2ys.png)
![](https://i.imgur.com/FSZYSg1.png)
![](https://i.imgur.com/FbRqKlo.png)

</center>

<center>

![](https://i.imgur.com/Vwwtqdf.png)
Site is signed
</center>

---

## Task 3 - Choose one of the options from the following:

:::info
1.  Web Server Security
:::

:::info
2. Web Server Performance
:::

:::info
3. Logging
:::

Logging is very important, by checking log files you can collect important information about the server
And, for the apache server, it can be done with installation goaccess as one of the examples

Inltallinglibraries required for goaccess
```
sudo apt install libncursesw5-dev libgeoip-dev libtokyocabinet-dev build-essential
```

Goaccess was installed from the source


```
sudo wget https://tar.goaccess.io/goaccess-1.5.2.tar.gz --no-check-certificate
./configure --enable-utf8 --enable-geoip=legacy
sudo make
sudo make install
```

goaccess can be started with `sudo goaccess /usr/local/apache2/logs/` that window with logs, are visible

<center>

![](https://i.imgur.com/hREc3ar.png)
![](https://i.imgur.com/hIBKx5e.png)
Goaccess on the aaa.st10.sne21.ru web site
</center>

Logfile  that processed by goaccess can be downloaded whom web server with the command

```
scp -i "V6.pem" ubuntu@ec2-18-217-220-180.us-east-2.compute.amazonaws.com:stats.html ~/stats.html
```

And it looks like

<center>

![](https://i.imgur.com/vAu5RFb.png)
Goaccess html generated page
</center>

we can actually put this information as s web page on our server
for that, I have just added a new record in the DNS zone, a new HTML page with logs, and a new config file for the new page, you can see the new page by clicking on the link
This information will update, within 30 minutes because of the cron script script 

```
http://log.st10.sne21.ru/log.html
```

<center>

![](https://i.imgur.com/UTDPyzR.png)
Cron update script
</center>

[https://www.digitalocean.com/community/tutorials/how-to-install-and-use-goaccess-web-log-analyzer-on-ubuntu-20-04]

:::info
4. GeoIP
:::


---
# Section 2: Directory server (Optional)

For this task, I will be using OpenLDAP + phpLDAPadmin

## Task 1: Directory Server installation and configuration

:::info
1. Get familiar with your directory server. What features and capabilities?
:::

Directory Service is a service that allows to monitor and manage some kind of the recourse, the resource can be different like stuff, network recourse (devices, IP addresses), computing powers

LDAP (Lightweight Directory Access Protocol) - a protocol that is used for managing and access to the hierarchy via a network. It is used for information maintenance, like centralized authentification systems and etc.

phpLDAPadmin - web client of LDAP. It allows to get access to the data and provides some additional features

:::info
2. Prepare the environment, create/use two VMs (i.e server and client)
:::

I will be using 2 AWS machines in order to complete the task, one as a server, another as a client. There are 3 machines are used in this task, the third one is my AWS DNS server



|N Instance| Purpose | IP            |
| --------  | --------| --------     |
|1         | Apache   |18.217.220.180|
|2         | LDAPserver|3.13.3.234|
|3         | DNS  |3.129.178.180|
|4         | Client  |18.222.21.140|

<center>

![](https://i.imgur.com/AnYGqxp.png)
AWS instances
</center>

:::info
3. Install the directory server. you DON’T need to install the directory server from the source. (e.g. use apt-get). 
Hint: you can add FQDN for your server in the /etc/hosts file to make it resolvable, in
case you don’t have access to your previews lab or DNS server.
:::

installation

```
sudo apt-get update
sudo apt-get install slapd ldap-utils
sudo dpkg-reconfigure slapd
```

The most important part of the reconfiguration of the slap is a choice of DNS

<center>

![](https://i.imgur.com/16SttC3.png)
Reconfiguration
</center>

Checking open ports

```sudo ufw allow ldap```

Installing `phpldapadmin`
```
sudo apt-get install phpldapadmin
```

in file locatid in 

```
sudo nano /etc/phpldapadmin/config.php
```

I have added/changed several records
First two record tells to the `phpldapadmin` where it should provide service, on what domain, `ipa.st10.sne21.ru` in my case
The third line is used for auto-filling the user forms.
```
$servers->setValue('server','name','ipa LDAP Server');
$servers->setValue('server','base',array('dc=st10,dc=sne21,dc=ru'));
$config->custom->appearance['hide_template_warning'] = true;
```

This value can be changed, but I haven't change it
```
$servers->setValue('login','bind_id','cn=admin,dc=example,dc=com');
```

From this point, I am ready to open it in a browser

:::info
4. (For FreeIPA) get a Kerberos ticket for admin and list it. Check the admin account exists in the FreeIPA server
:::

:::info
5. Access/log-in to the web dashboard of FreeIPA server. Create a test user (e.g. your name) for the next task
:::

I can access ```http://ipa.st10.sne21.ru/phpldapadmin``` web address in order to login in to LDAP, you may see the login page in the picture below

<center>

![](https://i.imgur.com/155Fs0o.png)
Ldap login page
</center>

In the picture below you can see how does the forms of a new user look like

<center>

![](https://i.imgur.com/sNJ2KL3.png)
Creating of the user
</center>

Everything that was created is available in the left window, like in the picture below
<center>

![](https://i.imgur.com/ZDogzVN.png)
List of the created things
</center>

---

## Task 2: Configure client

:::info
1. Install the client package using a package manager
:::

:::info
2. Check both client-server can ping using their FQDN.
:::

:::info
3. Authenticate FreeIPA client to server and verify using the test user that you created in the preview task.
:::

---

