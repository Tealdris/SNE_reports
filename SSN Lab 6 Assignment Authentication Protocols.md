###### tags: `SSN`
:::success
# SSN Lab 6 Assignment: Authentication Protocols
(OpenIDConnect, OAuth, LDAP)
**Name: Radomskiy Ilia**
:::


## Task 1: Setting up your environment

:::info
In this task, setup commands are provided. Those were used in order to set up the environment for this lab and it is not particularly useful, just for the personal convenience
:::

:::spoiler

JAVA 11
```
sudo apt -y install openjdk-8-jdk
```
[sourse](https://losst.ru/ustanovka-java-v-ubuntu-18-04)

VS studio
Modules that were setted up:
```
Spring Boot Tools
Gradle Language Support
Extension Pack for Java
```

IntelliJ IDEA
```
sudo snap install intellij-idea-community --classic --edge
```
[sourse](https://losst.ru/ustanovka-intellij-idea-na-ubuntu-16-04)

Download in folder project
```
git clone https://github.com/andifalk/secure-oauth2-oidc-workshop.git oidc_workshop
```
RUn gradlew
```
gradlew bootRun
```

Run Keyloak in docker via script
```
sudo ./run_keycloak_docker.sh
```
[Sourse](https://github.com/andifalk/secure-oauth2-oidc-workshop/blob/master/setup/README.md)

httpie installation
```
snap install httpie
```
[Sourse](https://github.com/httpie/httpie)

postman installation
```
snap install postman
```
[Sourse](https://learning.postman.com/docs/getting-started/installation-and-updates/#installing-postman-on-linux)
:::

## Task 1.2: OAuth 2.0 Authorization Grant Flows

### Client Credentials Grant

In this step we are getting basics of the authorizations flows, such parameters (client secret) as credentials which are used for obtaining a token that are used for the session establishment

`keyloak`, which was started up in docker, and it can be entered in browser with address

```
http://localhost:8080/auth/admin
```

We can obtain session token with several methods, but in genereal we are provide server with some parameters such as client_id, client_secret and others in order to get session token.
Some of the methods are shown below:

1. Curl

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&client_id=library-client&client_secret=9584640c-3804-4dcd-997b-93593cfb9ea7" http://localhost:8080/auth/realms/workshop/protocol/openid-connect/token
```

The output of this command is shown in the picture below

<center>

![](https://i.imgur.com/2HymrPW.png)

</center>

Or we can rewrite it in different way, more human-readable

```
{
"access_token":"eyJhbGciOiJSUzI1NiIs...",
"expires_in":300,
"refresh_expires_in":0,
"token_type":"Bearer",
"not-before-policy":1571836504,
"scope":"library_curator email profile"
}
```

2. Httpie

```
http --form POST localhost:8080/auth/realms/workshop/protocol/openid-connect/token grant_type='password' username='bwayne' password='wayne' client_id='library-client' client_secret='9584640c-3804-4dcd-997b-93593cfb9ea7'
```
Output of this command is shown in the picture below

<center>
    
![](https://i.imgur.com/bRzA65y.png)

</center>

Or we can rewrite it in a different way, more human-readable, you can compare it with the last one, and notice that the only difference is output order

```
{
    "access_token": "eyJhbGciOiJSUzI1Ni...",
    "expires_in": 300,
    "not-before-policy": 1571836504,
    "refresh_expires_in": 0,
    "scope": "library_curator email profile",
    "token_type": "Bearer"
}
```

3. Postman

Below you can see parameters that have i used in the postman fields in order to get an access token:
```
get http://localhost:8080/library-server/books

Access token: Available Tokens

Token name: UserInfo
Grant Type: Password credentials
Access Token URL: http://localhost:8080/auth/realms/workshop/protocol/openid-connect/token
Client ID: library-client
Client Secret: 584640c-3804-4dcd-997b-93593cfb9ea7
Scope: library_user email profile
```

As an output, we are getting the same results as previous via `curl` or `httpie`

<center>
    
![](https://i.imgur.com/3FoZQo7.png)
![](https://i.imgur.com/2X6CY8v.png)

</center>

[Postman](https://learning.postman.com/docs/sending-requests/authorization/#requesting-an-oauth-20-token)

---

### RO Password Credentials Grant

The next step is to see how does additional data change our process or session establishment. In this case we are adding another communication party (resource owner), which knows our data such as password and login

1. Curl

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=password&username=bwayne&password=wayne&client_id=library-client&client_secret=9584640c-3804-4dcd-997b-93593cfb9ea7" http://localhost:8080/auth/realms/workshop/protocol/openid-connect/token
```
the output of this command is shown in the picture below

<center>
    
![](https://i.imgur.com/eUutMWt.png)

</center>

Or we can rewrite it in different way, more human readable

```
{
"access_token":"eyJhbGciOiJSUzI1NiI...",
"expires_in":300,
"refresh_expires_in":1800,
"refresh_token":"eyJhbGciOiJIUzI1NiI...",
"token_type":"Bearer",
"not-before-policy":1571836504,
"session_state":"097ca1d2-e9f1-48d6-aa5d-70f857143c44",
"scope":"library_user email profile"
}
```

2. Httpie

```
http --form POST localhost:8080/auth/realms/workshop/protocol/openid-connect/token grant_type='password' username='bwayne' password='wayne' client_id='library-client' client_secret='9584640c-3804-4dcd-997b-93593cfb9ea7'
```

The output of this command is shown in the picture below

<center>

![](https://i.imgur.com/5PnRWNi.png)

</center>
    
Or we can rewrite it in a different way, more human-readable, you can compare it with the last one, and notice that the only difference is output order

```
{
    "access_token": "eyJhbGciOiJSUzI1NiI...",
    "expires_in": 300,
    "not-before-policy": 1571836504,
    "refresh_expires_in": 1800,
    "refresh_token": "eyJhbGciOiJSUzI1NiI...",
    "scope": "library_user email profile",
    "session_state": "8bb62827-19ae-4032-9f3c-1e7f12ddfb93",
    "token_type": "Bearer"
}
```

3. Postman

Below you can see parameters that have i used in the postman fields in order to get an access token:
```
Token Name UserInfo
Grant Type Password Credentials
Access Token URL http://localhost:8080/auth/realms/workshop/protocol/openid-connect/token
username=bwayne
password=wayne
Client ID library-client
Client secret 9584640c-3804-4dcd-997b-93593cfb9ea7
Scope: library_user email profile
```

As an output we are getting same results as previous via `curl` or `httpie`
<center>
    
![](https://i.imgur.com/4XUZort.png)
![](https://i.imgur.com/ofcgiVm.png)

</center>

---

### Advanced: Authorization Code Grant

For this step, there are 4 parties now, client, resource owner, user agent, and authorization server. So first thing to do in this scenario is to log in with your data to the keycloak. After that we caan odtain information about our our session tocken

1. Postman

Fileds of postman was filled with:


```
Token Name UserInfo
Grant Type Authorization cod
Callback URL 
http://localhost:9090/library-client/login/oauth2/code/keycloak
Auth URL 
http://localhost:8080/auth/realms/workshop/protocol/openid-connect/auth
Access Token URL 
http://localhost:8080/auth/realms/workshop/protocol/openid-connect/token
Client ID library-client
Client Secret 9584640c-3804-4dcd-997b-93593cfb9ea7
Scope library_user email profile
State any secuence
```

As a result we are getting access token, and token that are used to refresh session
<center>
    
![](https://i.imgur.com/9nqaXNm.png)
![](https://i.imgur.com/uTd3Gtz.png)

</center>

2. Curl

In case of `curl` command it have to be made in several additional steps, than for the postman

2.1 Go to web browser with address

```
http://localhost:8080/auth/realms/workshop/protocol/openid-connect/auth?response_type=code&client_id=demo-client&redirect_uri=http://localhost:9095/client/callback
```

Enter credentials, and extract code from new page, you can fing process in the picture below, orange highlited color is our code

<center>
    
![](https://i.imgur.com/tPu6W3W.png)

</center>

<center>

![](https://i.imgur.com/7kp9TEP.png)
    
</center>

exctracted code:
```
f30c214f-f6ba-4422-ad7b-9ac3ba88ee1c.ced24c7e-7bc9-4194-8c1a-d4ec01f081e4.58f0efec-2e91-4c63-823f-85280996f8a5
```

This code goes to the our `curl` command:

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=authorization_code&code=f30c214f-f6ba-4422-ad7b-9ac3ba88ee1c.ced24c7e-7bc9-4194-8c1a-d4ec01f081e4.58f0efec-2e91-4c63-823f-85280996f8a5&redirect_uri=http://localhost:9095/client/callback&client_id=demo-client&client_secret=b3ec9d3f-d1ee-4a18-b4ba-05d832c15293" http://localhost:8080/auth/realms/workshop/protocol/openid-connect/token
```

After that we can see output of curl command in the pitcure below

<center>
    
![](https://i.imgur.com/5JPaMrU.png)
    
</center>

Or it can we writen in different way, more human readable

```
{
"access_token":"eyJhbGciOiJSUzI1NiI...",
"expires_in":300,
"refresh_expires_in":1800,
"refresh_token":"eyJhbGciOiJIUzI1NiI...",
"token_type":"Bearer",
"not-before-policy":1571836504,
"session_state":"a23e5bdf-6893-48ef-98
}
```

## Task 1.2: OAuth 2.0 Authorization Code Grant Flow in Detail

In this part, we are observing the authorization code grant - the most used flow modern of OAuth 2.0

The process of user authorization looks like this:
1. User sends his credentials to his agent, then to the server
2. Then Servers replies with auth code to the agent, and the agent redirect it to the user
3. Exchange happens between client and server: authorization code into an access token

In the pictures below you can see process of obtaining code and sending code in order to obtain tokens from the server

The authorization process begins, step 1

<center>
    
![](https://i.imgur.com/aSz0RJ6.png)
Initialize 1 step
</center>

We have received code and can push button in order to obtain a token,  step 3

<center>
    
![](https://i.imgur.com/p1PTaOU.png)
step 3
    
</center>

Received token, now we can establish session

<center>
    
![](https://i.imgur.com/X5tuF77.png)
after step 3

</center>

## Task 1.3: Client app for the GitHub API

In this Task, we can observe how does CommonOAuth2 implement by GitHub

From this page, we are taken `Client id` and `clients secret` in order to access github

<center>

![](https://i.imgur.com/DJkaOsh.png)
Github advanced settings
</center>
    
Then we run `gradlew` application with our `Client id` and `clients secret` on the web page `http://localhost:9090/` we got this redirection

<center>
    
![](https://i.imgur.com/FQKjYnq.png)

</center>
    
After authorization, we can see parameters of communication: login, id, etc

<center>
    
![](https://i.imgur.com/FAboqvk.png)

</center>



## Task 2: LDAP

For this task, I build topology in GNS3 consisting of VM's from the VirtualBox: server and two clients

<center>
    
![](https://i.imgur.com/YAiK0cX.png)

</center>
    
The LDAP server has to be configured:

Installation of the LDAP

```
sudo apt install slapd ldap-utils
```

slapd -  for dynamic configuration of ldap
ldap-utils - LDAP itself

after that installation we can reconfigure LDAP settings
```
sudo dpkg-reconfigure slapd
```

In order to our LDAP server work i have changed several configuration files:

In the file `ldap.conf` with the command
```
sudo nano /etc/ldap/ldap.conf
```
I have added my LDAP server settings, specifically the DNS name, and URI of it

<center>
    
![](https://i.imgur.com/vgtIDG0.png)

</center>

After that we can check if our server can be found by entered name, the command below will also provide some additional information

```
sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:// -b cn=config dn
```

<center>
    
![](https://i.imgur.com/kdfIHiG.png)

</center>

if you want to modify database you can add your information to the file and perform the command:

create a file with data
```
sudo nano add_content.ldif
```

Data of the user with the explanation is listed in the hidden part, on the picture you can see new user parameters have been written in the file

::: spoiler

group creation

dn: ou=People,dc=example,dc=com
objectClass: organizationalUnit
ou: People

group creation

dn: ou=Groups,dc=example,dc=com
objectClass: organizationalUnit
ou: Groups

group creation

dn: cn=miners,ou=Groups,dc=example,dc=com
objectClass: posixGroup
cn: miners
gidNumber: 5000


User creation

dn: uid=john,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: john
sn: Doe
givenName: John
cn: John Doe
displayName: John Doe
uidNumber: 10000
gidNumber: 5000
userPassword: {CRYPT}x
gecos: John Doe
loginShell: /bin/bash
homeDirectory: /home/john

:::


<center>

![](https://i.imgur.com/UREkUwQ.png)
content of the file
</center>
    
in this case, it contains information that will be added to the database with the next update

the update can be performed with the following command
```
ldapadd -x -D cd=admin,dc=example,dc=com -W -f add_content.ldif
```

<center>
    
![](https://i.imgur.com/c3TjVuS.png)
Database successfully updated, but file with new information contained old data, so it was not added to data base
</center>

the database itself located by the path, but the information there is stored in an encrypted way:

```
/var/lib/ldap
```

When database is updated we can check if the new user has been added to it or not:

```
ldapsearch -x -LLL -b dc=example,dc=com '(uid=test)' cn gidNumber
```

<center>
    
![](https://i.imgur.com/rOyu7sa.png)

</center>

Also, we can see that update was successful if we can see new user in the LDAP web page

<center>
    
![](https://i.imgur.com/LVmoFKJ.png)

</center>

---
In the section below, there is configuration that has been done for the client

Firstly, install LDAP on the both of clients machines
```
sudo apt install ldap-utils libnss-ldapd libpam-ldapd
```

Then configure it
```
sudo dpkg-reconfigure ldap-auth-config
```

The most important steps of configuration are shown below, they are: LDAP IP address, distinguished name, and information about  user `admin`

<center>
    
![](https://i.imgur.com/2CqSYLr.png)
IP address
    
![](https://i.imgur.com/lfit1PA.png)
distinguished name
    
![](https://i.imgur.com/hTPT2oF.png)
admin account information
</center>

For the client machine to become a user of our LDAP server there, some configuration should be done

Edit the `ldap.conf` file

```
/etc/ldap/ldap.conf
```

On the picture below you can see that i have added information about server, it name and IP

<center>
    
![](https://i.imgur.com/MuuompR.png)

</center>
    
Then file `common-session` have to be configured, it is used by users to create the directory if it doesn't exists
```
sudo nano /etc/pam/d/common-session
```

<center>
    
![](https://i.imgur.com/0WuI9Nl.png)
Content of the common-session file
</center>

The next file `nsswitch.conf` is used to set the priority of getting sources of IP address information

```
sudo nano /etc/nsswitch.conf
```

Setting that is shown below is added to the file, they are used for the client authentification
```
passwd: compat systemd ldap
group: compat systemd ldap
shadow: compat ldap
```

<center>

![](https://i.imgur.com/nvZaffu.png)

</center>

Then a have edited file `nslcd.conf`, it is configuration file for LDAP

```
sudo nano /etc/nslcd.conf
```

I have added such lines as and you can see them in the picture

```
uri ldap://192.168.122.176
base dc=example,dc=com
```

<center>
    
![](https://i.imgur.com/y3CQSX3.png)

</center>

At this point we are ready to see, are we able to establish a connection with the server or not.
In order to do that I use a command that is shown below

```
sudo ldapsearch -x -H 'ldap://example.com' -P 3 -LLL -b 'dc=example,dc=com'
```

also can be used LDAP's IP address instead
```

sudo ldapsearch -x -H '192.168.122.176' -P 3 -LLL -b 'dc=example,dc=com' 
```

We can see the output in the picture below

<center>
    
![](https://i.imgur.com/WpsLBeP.png)

</center>

After that i can try to login as users of that LDAP that was configured earlier. You can see successful authentification at the bottom of the pictures below

<center>
    
![](https://i.imgur.com/f61iqVq.png)
For the client 1
    
![](https://i.imgur.com/61YvMwb.png)
For the client 2
</center>

[LDAP installation](https://pro-ldap.ru/books/openldap-ubuntu-in-practice/index.htmlhttps://ubuntu.com/server/docs/service-ldap)
[](https://ubuntu.com/server/docs/service-ldap)
[](https://blog.sedicomm.com/2019/10/21/kak-ustanovit-server-openldap-dlya-tsentralizovannoj-autentifikatsii/)

## Appendix

