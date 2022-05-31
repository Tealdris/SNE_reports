###### tags: `CIA`
:::success
# CIA Lab 5 Assignment: Mail Transfer Agents (MTA)
**Name: Radomskiy Ilia**
:::

Postfix or **Exim**

## Task 1 - Install

:::info
Install from  source code
:::

***a) make sure that your system does not contain a pre-installed version of the 
MTA of your choice***

In this lab, we are covering email services, but before start, we must ensure that there are no installed email services on the machine.
 By taping apt-cache policy exim4 we may take a look at the sources package, and if it is installed or not

<center>

![](https://i.imgur.com/8I2oOpF.png)
reaching for installed exim4
</center>

***b) Make sure the source code is retrieved from a secure location. Use the official 
website for the MTA of your choice.***

Picture of the official web site of the exim mail service you can see on the picture below
<center>

![](https://i.imgur.com/4tTU14H.png)
Official exim web site
</center>

At the same web page it is can be found links to the FTP server where Exim itself and the signatures to it can be found

***c) Because it is important that an MTA be correct and secure it is often signed using a digital PGP signature. If your MTA is signed then make sure you have downloaded the correct sources by checking the validity of the key and the signature.***

With the downloaded compressed file and key, we can verify them
In the picture below you can see how does signature look like

<center>

![](https://i.imgur.com/hiO1DaB.png)
exim signature
</center>

during the verification process of the file I get the message ```good signature```, which means files are not substituted[1]

<center>

![](https://i.imgur.com/l7iMsqx.png)
Signature verification
</center>

***d) There are a number of options that you will have to enter before compilation, so that the functionality can be compiled into the program. Make sure the basic install holds all the necessary functionality. Show the options you configured.***

:::info
2. Most of the options for an MTA can be found in a configuration file that will be loaded when the MTA starts. It is recommended to start with an example configuration that looks a lot like what you need for now.  Show how you adapt it to your needs.
:::

Most of the added configuration parameters were published on the official Exim manual web page [2]

According to it, I did several things[3]:
have added group in the ```/etc/group``` file
```exim:*:90:```
have created a user with
```sudo useradd exim -g exim```
Next thing copying ```EDITME``` to the Local/directory with the following command
```cat ./src/EDITME | grep -ve "^#" -ve "^$" > Local/Makefile```
This command dilates every comment in the new file
Additional records in the ```Makefile``` file
```
USE_DB=yes
USE_OPENSL=yes
TLS_LIBS=-lssl -lcrypto
USE_OPENSSL=yes
USE_OPENSSL_PC=openssl
```

in the end, the file will be looking like that

<center>

![](https://i.imgur.com/2TAZobK.png)
File ```'Makefile'```
</center>

From this point we are some kind of ready to make installation, there might be some additional features in the configuration file, that requires additional libraries or your system might have not installed some utilities.
 In the case with Exim you might use
 ```
 sudo apt-get build-dep exim4
 ```
command, but this command requires ```dpkg-dev``` utility
so, write in advance
```
sudo apt-get install dpkg-dev 
```

With
```checkinstall pkg-config```

sudo cp /etc/apt/sources.list /etc/apt/sources.list~
sudo sed -Ei 's/^# deb-src /deb-src /' /etc/apt/sources.list
sudo apt-get update

At this point, we are ready to perform ```make``` and ```make install``` commands


After installation there is an option to change the configuration of the exim, it can be done in two ways:
```sudo dpkg-reconfigure exim4-config```
Or editing: ```/etc/exim4/update-exim4.conf.conf``` file

<center>

![](https://i.imgur.com/Ul44i9Y.png)
```/etc/exim4/update-exim4.conf.conf``` file
</center>

:::info
3. Configure
:::

***a) Add a local account on your experimental machine and make sure that the MTA can 
deliver mail to it. Show the required configuration.***
local

User has been added on the previous step with, but for the testing purposes we might add another ```test``` one
```sudo useradd test -g exim```

Exim allows printing information about users.

<center>

![](https://i.imgur.com/KYj4Mv2.png)
Information about user with exim
</center>

Let's make an easiest test on the mail - send mail from one user to the same user

<center>

![](https://i.imgur.com/CMObfau.png)
Sending mail to "ubuntu" from "ubuntu"
</center>

We can see sent mail in the ubuntu's mailbox, that located in the ```/var/mail/ubuntu```

<center>

![](https://i.imgur.com/LZnnZnf.png)
Starage of the emails to the user "ubuntu"
</center>



***b) Add to your log an email received by this account.  Do not forget the full headers!***

Our added user also can receive main on its own mailbox, so I send mail to the "test" user from "root"

<center>

![](https://i.imgur.com/FGbbnJk.png)
sending mail to the "test" user
</center>

<center>

![](https://i.imgur.com/4QINjxS.png)
recived mail on the "test' user mailbox
</center>

***c) Also make sure that any email intended for  ```postmaster@st10.sne21.ru``` is delivered to this account. Show the full email as delivered to the new account and the required configuration.***

![](https://i.imgur.com/VSzxRby.png)


---
## Task 2 - Sending‌ ‌mail‌ ‌-‌ ‌email‌ ‌validation‌ ‌-‌ ‌SPF‌ ‌&‌ ‌DKIM‌ 

:::info
4.‌ ‌Write‌ ‌a‌ ‌small‌ ‌paragraph‌ ‌that‌ ‌highlights‌ ‌the‌ ‌advantages‌ ‌and‌ ‌disadvantages‌ ‌of‌ ‌SPF‌ ‌and‌‌
DomainKeys‌ ‌Identified‌ ‌Mail‌ ‌(DKIM).‌ ‌What‌ ‌would‌ ‌you‌ ‌choose‌ ‌at‌ ‌a‌ ‌first‌ ‌glance‌ ‌and‌ ‌why?‌
:::

SPF (Sender Policy Framework) - addition to the DNS records, TXT type. In this record, there is a list of IP addresses, that are allowed to send information from this mailbox

SPF breaks plain message forwarding

DKIM (Domain Keys Identified Mail) - a method that allows checking the domain name of the mail sender, it is also a DNS record

One of the disadvantages of DKIM is that spam servers can support DKIM, so DKIM doesn't guarantee safety

Personally, I would like to use both options in order to increase security, so to use DMARC (Domain-based Message Authentication, Reporting and Conformance). This method uses both SPF and DKIM.

But if I have to choose, I would pick DKIM, because of the usage of the cryptographic algorithms

:::info
5.‌ ‌Set‌ ‌up‌ ‌your‌ ‌mail‌ ‌server‌ ‌to‌ ‌use‌ ‌a‌ ‌relay‌ ‌server‌ ‌to‌ ‌be‌ ‌able‌ ‌to‌ ‌send‌ ‌outgoing‌ ‌mail‌ ‌for‌ ‌different‌‌
domains‌ ‌(contact‌ ‌your‌ ‌TA‌ ‌to‌ ‌obtain‌ ‌address/credentials‌ ‌to‌ ‌the‌ ‌relay‌ ‌server‌ ‌and‌ ‌SPF/DKIM‌‌ records).‌ ‌
:::

in my case, I will be using my server as a relay to the ```mailgun``` service. Also, i will verify DkiM on the same service


After verification on mail gun
Mailgun recommends using ```swaks``` to send emails, so the command that will send emails to my personal mail is listed below.
In this command used credentials that was received from mailgun, and port that different from standard


```/usr/swaks/./swaks --auth --server smtp.mailgun.org --au postmaster@st10.sne21.ru --ap 1bf790b7f20bac352e8445f091cd9c64-90346a2d-4dd75249 --to ilyaradomskiy@mail.ru --from "postmaster@st10.sne21.ru" --h-Subject: "Subject" --body 'Hello SNE' --p 587```

But there also other ways to send an email, with the echo command, and with using Exim itself

```echo Message | mail -s Subject ilyaradomskiy@mail.ru```
```exim -v postmaster@st10.sne21.ru```

:::info
6.‌ ‌Test‌ ‌how‌ ‌your‌ ‌mail‌ ‌is‌ ‌delivered‌ ‌to‌ ‌commonly‌ ‌known‌ ‌mail‌ ‌servers‌ ‌(f.e.‌ ‌gmail).‌ ‌Provide‌ ‌full‌‌
email/MTA‌ ‌headers‌ ‌to‌ ‌see‌ ‌how‌ ‌SPF/DKIM‌ ‌were‌ ‌delivered.‌ ‌
:::

Let's test, how does the process of verification go
For that, i send mail to my server, and I expect to see some DKIM record
Result you may see on the picture below

<center>

![](https://i.imgur.com/BXbrt77.png)
![](https://i.imgur.com/VSzxRby.png)
sending email to the postmaster with dkim reconrd in it
</center>


---
## Task‌ ‌3‌ ‌-‌ ‌Mail‌ ‌backup‌

:::info
7.‌ ‌You‌ ‌have‌ ‌backup:
:::

Backup is very important in terms of fault tolerance, so it might be useful to have some

**a) Adapt‌ ‌the‌ ‌DNS‌ ‌information‌ ‌for‌ ‌your‌ ‌domain,‌ ‌so‌ ‌that‌ ‌the‌ ‌backup‌ ‌MTA‌ ‌on‌ ‌your‌‌ partner’s‌ ‌server‌ ‌can‌ ‌be‌ ‌found.‌**

I have added records of my partner in my  zone config file ```db.st10.sne21.ru```, and it looks like this

<center>

![](https://i.imgur.com/p3ZciUE.png)
Zone file
</center>

So record means that mail have to go to email service of my partner if it can't reach my service

**b) Validate‌ ‌by‌ ‌shutting‌ ‌your‌ ‌service‌ ‌down‌ ‌and‌ ‌sending‌ ‌a‌ ‌message‌ ‌to‌ ‌your‌ ‌domain‌ ‌**

In order to check how does everything works, I stopped my Exim service, and have to send emails to my server via my personal mailbox
In the picture below you can see that his mail service is trying to send to me an email, that was sent to him, because my Exim is turned off

<center>

![](https://i.imgur.com/ooWXbYB.jpg)
Postfix log file of my partner
</center>

**c) Bring‌ ‌your‌ ‌service‌ ‌back‌ ‌up‌ ‌and‌ ‌wait.‌**

So, now I turned my mail server on, waited for a couple of minutes, and successfully received a message from my partner

<center>

![](https://i.imgur.com/cZYkARS.png)
Mail from my partner's backup  server 
</center>

:::info
8.‌ ‌You‌ ‌provide‌ ‌backup:‌
:::

**a) Make‌ ‌your‌ ‌MTA‌ ‌act‌ ‌as‌ ‌a‌ ‌backup‌ ‌for‌ ‌your‌ ‌partner’s‌ ‌domain.‌ ‌**

In this step, I am providing backup to my partner, st9. In order to do that, I am adding records in several files. One of them is on the picture below[how to set up Exim as backup MX server]
This record allows taking email messages from my partner 

<center>

![](https://i.imgur.com/rS2Aew4.png)
</center>

**b) Show‌ ‌the‌ ‌logs‌ ‌while‌ ‌doing‌ ‌your‌ ‌mate’s‌ ‌acceptance‌ ‌test‌ ‌and‌ ‌show‌ ‌where‌ ‌the‌ ‌message‌‌v is‌ ‌temporarily‌ ‌stored.‌ ‌**

In the picture below you can see part of the logging session. In this case, we can see that my server is trying to send an email to the email server of my partner.

<center>

![](https://i.imgur.com/B0wBQU4.png)
</center>

**c) Once‌ ‌your‌ ‌partner’s‌ ‌MTA‌ ‌is‌ ‌back‌ ‌online,‌ ‌eventually‌ ‌force‌ ‌an‌ ‌immediate‌ ‌delivery‌ ‌and‌‌ show‌ ‌your‌ ‌mail‌ ‌logs.‌ ‌**

When the server of my partner is available again, the message that was stored in my server sent to him
In the picture below you can see log files that my partner have sent to me

<center>

![](https://i.imgur.com/68uguBq.jpg)
![](https://i.imgur.com/KTruHIM.jpg)
Log file of postfix server of my partner 
</center>

---

## Task‌ ‌4‌ ‌-‌ ‌Mailing‌ ‌loops‌

:::info
9.‌ ‌Create‌ ‌an‌ ‌email‌ ‌loop‌ ‌with‌ ‌your‌ ‌partner‌ ‌from‌ ‌domain‌ ‌to‌ ‌domain‌ ‌using‌ ‌email‌ ‌aliases.‌
:::

I accedentealy have created maik loop by my self when i was trying to check is my mail server working at all

<center>

![](https://i.imgur.com/H1EdtQp.png)
Exim log file
</center>

:::info
10.‌ ‌Send‌ ‌an‌ ‌email‌ ‌to‌ ‌the‌ ‌loop‌ ‌using‌ ‌your‌ ‌own‌ ‌email‌ ‌address‌ ‌and‌ ‌see‌ ‌what‌ ‌happens‌ ‌on‌ ‌your‌‌ MTA.‌ ‌
:::

But in order to know more about possible email problems we, i and my partner(st9) have crated mailing loop, you can see on thi picture below

<center>

![](https://i.imgur.com/kTiBkyI.png)
Exim log file
</center>

:::info
11.‌ ‌Can‌ ‌you‌ ‌change‌ ‌the‌ ‌behaviour‌ ‌of‌ ‌your‌ ‌MTA‌ ‌in‌ ‌response‌ ‌to‌ ‌this‌ ‌loop?‌ ‌
:::

Basik way to avoid loops is to do not to create them
But the second way is to create filter rules like:
local domains exception
`domains = ! +st10.sne21.ru : ! +local_domains`

--
## Task‌ ‌5‌ ‌-‌ ‌Virtual‌ ‌Domains‌

:::info
12.‌ ‌Create‌ ‌a‌ ‌new‌ ‌subdomain‌ ‌within‌ ‌your‌ ‌domain‌ ‌and‌ ‌add‌ ‌an‌ ‌MX‌ ‌entry‌ ‌to‌ ‌it.‌ ‌
:::

In order to create a virtual domain have added records in my DNS zone config file.

<center>

![](https://i.imgur.com/qObBg1l.png)
db.st10.sne21.ru
</center>

:::info
13.‌ ‌Then‌ ‌extend‌ ‌your‌ ‌MTA‌ ‌configuration‌ ‌to‌ ‌handle‌ ‌virtual‌ ‌domains,‌ ‌and‌ ‌have‌ ‌it‌ ‌handle‌ ‌the‌‌ email‌ ‌for‌ ‌the‌ ‌newly‌ ‌created‌ ‌domain.‌ ‌
:::

But not only zone file needs configuration, I also have to add some additional configuration in the Exim files
[Virtual domains with Exim 4]
```lsearch;/etc/exim4/domains.virtual``` this line tells the Exim that he has to look at the file ```domains.virtual``` additionally, in order to find new domains

<center>

![](https://i.imgur.com/bEXSp4B.png)
main config file
</center>

the new file is a file with ```virtual_aliases```, this config file contains a file with names, that used for virtual domains

<center>
 
![](https://i.imgur.com/ZDqpeMN.png)
The file that contains a link to the  file with aliases
</center>

Of course, files, that was previously announced is also configured, I have added the name of my virtual domain

<center>

![](https://i.imgur.com/n25wOsq.png)
![](https://i.imgur.com
/fmMxp63.png)
Files with innformation about domains
</center>


:::info
14.‌ ‌Validate‌ ‌that‌ ‌you‌ ‌are‌ ‌now‌ ‌receiving‌ ‌emails‌ ‌for‌ ‌both‌ ‌domains.‌
:::

So now we can send email to a newly created domain, as well as on the old one
I can receive an email as a local user

<center>

![](https://i.imgur.com/OTcCC6Q.png)
Local emails
</center>

Or email can be sent from other mail providers

<center>

![](https://i.imgur.com/dpewoyq.png)
![](https://i.imgur.com/u8Ssit6.png)
Email from my personal mail account
</center>

---
## Task‌ ‌6‌ ‌-‌ ‌Transport‌ ‌Encryption‌

:::info
15.‌ Questions‌ ‌
:::

**a) Which‌ ‌one‌ ‌is‌ ‌better,‌ ‌SSL/TLS‌ ‌or‌ ‌STARTTLS,‌ ‌why?‌ ‌**

SSL (Secure Sockets Layer) - the protocol used for increasing security, to this end 
asymmetric cryptography (exchange), symmetric encryption (storage), message authentication (integrity). NExt generation


TLS (Transport Layer Security) - the successor of SSL, based on version 3, Made in order to provide privacy and data integrity between two or more computers

STARTTLS - uses or TLS or SSL for encrypted message exchange.

The difference is that is, in the case of TLS and SSL, if one side does not support encrypted exchange the there will be no exchange, StartTLS will exchange messages in plain text, case if one side does not support encrypted exchange.

So because STARTTLS won't help against an active attacker,  and because SSL is and old version of encryption method, i think TLS if better
[SMTP wikipedia]

**b) Which‌ ‌one‌ ‌is‌ ‌actually‌ ‌in‌ ‌use‌ ‌for‌ ‌SMTP?‌ ‌**

All of them is used for SMTP encryption. For example Yandex.mail uses both TLS\SSL and starttls
[Yandex now supports encryption of outgoing and incoming mail]

:::info
16.‌ Task‌ ‌
:::

**a) Add‌ ‌transport‌ ‌encryption‌ ‌to‌ ‌your‌ ‌MTA.‌ ‌**


To create certificate i preformed this line.

`openssl req -x509 -sha256 -days 9000 -nodes -newkey rsa:4096 -keyout /etc/exim.key -out /etc/exim.cert
Generating a 4096 bit RSA private key`

then i have changed permission on the  files with

`exim:exim /etc/exim.key`
and
`chmod /etc/exim.*`

after that i have added this to the config file

![](https://i.imgur.com/C3yTbYX.png)

 
**b) Eventually‌ ‌force‌ ‌the‌ ‌transport‌ ‌to‌ ‌be‌ ‌encrypted‌ ‌only‌ ‌(refuse‌ ‌non‌ ‌encrypted‌ ‌transport).‌ ‌**

``tls-on-connect`` option to forse others to use TLS connectoins

it can be change in the file

![](https://i.imgur.com/HiEPF3n.png)



**c) Proceed‌ ‌with‌ ‌validation‌ ‌(proof‌ ‌or‌ ‌acceptance‌ ‌testing),‌ ‌as‌ ‌usual.**

And on this stage we can check is it available for my server to use TLS protocol

![](https://i.imgur.com/VbXfp1f.png)

It is also can be checket with creation of connection via  telnet

![](https://i.imgur.com/35Q91E4.png)


---
## BONUS‌ ‌(2‌ ‌points)‌ ‌-‌ ‌Task‌ ‌7‌ ‌-‌ ‌Anti-spam‌ ‌filters‌

:::info
17.‌ ‌Investigate‌ ‌what‌ ‌generic‌ ‌anti-spam‌ ‌open‌ ‌source‌ ‌software‌ ‌packages‌ ‌are‌ ‌out‌ ‌there,‌ ‌choose‌‌
one,‌ ‌download‌ ‌it‌ ‌(compile‌ ‌it‌ ‌if‌ ‌necessary)‌ ‌and‌ ‌configure‌ ‌your‌ ‌MTA‌ ‌to‌ ‌use‌ ‌it.‌ ‌Make‌ ‌sure‌ ‌that‌‌ in‌ ‌your‌ ‌MTA‌ ‌group‌ ‌there‌ ‌are‌ ‌2‌ ‌different‌ ‌anti-spam‌ ‌solutions‌ ‌implemented!
:::

---
## References

[Key verification](https://itsecforu.ru/2021/07/07/%F0%9F%94%90-%D0%BA%D0%B0%D0%BA-%D0%BF%D1%80%D0%BE%D0%B2%D0%B5%D1%80%D0%B8%D1%82%D1%8C-pgp-%D0%BF%D0%BE%D0%B4%D0%BF%D0%B8%D1%81%D1%8C-%D0%B7%D0%B0%D0%B3%D1%80%D1%83%D0%B6%D0%B5%D0%BD%D0%BD%D0%BE%D0%B3/)
[Exim official guid](https://www.exim.org/exim-html-current/doc/html/spec_html/ch-building_and_installing_exim.html)
[AfNOG 2006 Exim Practical](https://nsrc.org/wrc/data/2006/928523445448219c5a7017/PracticalNotes.pdf)
[Virtual domains with Exim 4](https://muzso.hu/2009/01/15/virtual-domains-with-exim-4-and-debian-updated)
[How to setup exim as a backup MX server](https://help.directadmin.com/item.php?id=167)
[SMTP wikipedia](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol#Security_extensions)
[Yandex now supports encryption of outgoing and incoming mail](https://habr.com/ru/company/yandex/blog/203882/)



---
## Some felpful liks

