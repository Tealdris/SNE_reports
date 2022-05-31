###### tags: `AS`
:::success
# AS Lab 3 Assignment: Identity and Access Management (IAM) - 1
**Name: Radomskiy Ilia**
:::

## Preparation


1. On this web page we can find more information about the [`Keycloak`](https://www.keycloak.org/getting-started/getting-started-docker)

2. With this command I can start `Key lock` docker container

```
docker run -d -p 8081:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:17.0.1 start-dev -Dkeycloak.profile.feature.upload_scripts=enabled
```

As a result we can reach `keyloack` in the browser

<center>

![](https://i.imgur.com/qNI3mr4.png)
    
</center>

3. It is required by the assignment, and it is easier to access web page with domain name `sso` so i will configure `/etc/hosts` in order to point at `127.0.0.1`

<center>

![](https://i.imgur.com/J8cMlx7.png)
    
</center>


4. Access keycloak at `sso:8080`

Now we can access our docker container

<center>

![](https://i.imgur.com/XQfq6TK.png)
    
</center>

5. Create realm `as_lab`

We need to specify `realm`, or sime kinda of workspace for this lab: `as_lab`

<center>
    
![](https://i.imgur.com/6TvKqJl.png)
    
</center>

6. Create sample user in this `realm`

Inside realm i will create user for this `realm`

<center>

![](https://i.imgur.com/vewhdMH.png)
    
</center>

7. Launch sample app - installation instructions inside `iam-lab-part1.zip`

Now, with the purpose of `IAM` understanding, we will connect keycloak to the web application. It is a ready app, so we need just to follow the script

<center>

![](https://i.imgur.com/aIluEnH.png)

</center>

Set of commands below was used to set up this app

```
sudo npm install semver
sudo npm install --global yarn
sudo yarn install
sudo yarn dev
```

From this point, we can access application in a web browser

<center>
    
![](https://i.imgur.com/t8ZcV5c.png)

</center>

:::spoiler
help `Command failed with exit code 127.`
https://github.com/qraimbault/vue-js-starter-scss/issues/1
:::

---

## Authentication and Single-Sign On

1. Accessing those web pages is easy. It is accessible via those links

```
http://localhost:3068/pokemon/:pokemon-name
http://localhost:3068/trainer
```

<center>

![](https://i.imgur.com/ITkcnsi.png)
    
</center>

But providing the usage is impossible on this point, because there is no user in the `keycloak`

<center>

![](https://i.imgur.com/I2jJ8yj.png)

</center>

### 1 Questions

*    a) What is client?

Agent/service or application that can ask for the credentials to authorization.

*    b) What client types exist?

That may be an application that wants `single-sign-on`
Or, another type of client is requesting an `access token`

*    c) What authentication flows exist?

authentication flows is a process of authentification, that may include itself entering particular credentials.
There are several types of flows
`REgistration flow` - information required for the new user to specify for creating account name, email, password and etc
`Login Flow` - information required for the user to enter his account email, and password. There may be additional steps. Like two-factor authentification
`Credential reset flow` - information required for the user to reset his data, like password in case of driven account, or forgotten password

Or **`OIDC auth flows`** like
`Authorization Code Flow` authentification via browser uses requests and refresh tokens for the security (single time used). REdirections for other URLs are registered and controlled with mechanisms. Three parameters: `identity` and `access` and `refresh` token

`Implicit Flow` same as the previous one, but less secure because there are no refresh tokens and less amount of requests. Two parameters: `identity` and `access` and token

`Resource owner password credentials grant (Direct Access Grants)` in this case REST is used for the authentification and tocken acquisition. Three parameters: `identity` and `access` and `refresh` token
`etc`

It may look like this
You -> Client -> Autherization server (keycloack) included (Redirect URI, client id, Response type (code), scope (something client wants list of contacts) ) -> Resource server -> You (login window) -> Resource server -> Autherization server (client id, authorization code,  client secret, access token) <-> Client 

Client, in the end, has access token, and it can be used as a credential for the authentification


*    d) Why is Authorization Code Flow preferred over the others?

As a best practice, you should consider your communication with three parameters, access, refresh token, and identity of user. 

*    e) What is Single-Sign On (SSO)?

The authentification method, that allows to the users authenticate themselves easily in several applications at the same time.

*    f) What is the difference between OAuth 2.0 and OpenID Connect?

`OAuth 2.0` standard is used for the passing your data to other applications. Often in used between application communication. Key is the result of the communication, and the client doesn't know anything about user

`OpenID`  is used for the authorization, for example, if you need to register somewhere with your email service provider. REsult, some credentials about the user is know to the client, so sure can be recognized. SSO is configured with the power of OpenID, thin layer, that


### 2 Setup authentication

1. Сreate corresponding client for `app-pokemon` and `app-trainer`

Inside `keycloak` we can create users for our applications, trainer, and pokemon. In the picture below you can see those users

<center>

![](https://i.imgur.com/WeqlVRz.png)
    
</center>

NOw since they are created, when we will try to login into apps.

<center>

![](https://i.imgur.com/9By1SSE.png)
    
</center>

we will be redirected to the login page of the `keycloak`

<center>

![](https://i.imgur.com/mQ3LwRV.png)
    
</center>

If the credentials are valid, we are authenticated

<center>

![](https://i.imgur.com/7kQpIz6.png)

</center>


### 3 Analyze SSO workflow


*    a) What authentication flow is used?

it is Authorization code flow it is 3 side communication with the client resource server

*    b) What is the difference between `id_token` and `access_token`?

`ID Token` granted by `OpenID` contains information about a User. 

`Access token` is used for authentification and does not contain any information about the user.

*    c) What is purpose of `refresh_token`?

For security purposes, refresh tokens was created. ID token and Access token are available for some period of time, and after those token expires you have to ask for another token, for the further connection with the server, that is why refresh tokens are used

*    d) How is the session persisted? What will happen if you use incognito mode or another browser?

Your session is persistent if you periodically send requests to the same backend of application. To this end is used `cookie-based persistence` technique. There might be different data in the cookie, like session iD, of IP.

Actually, when I enter `http://localhost:3068/` in private mode, i will enter my user credentials again. No significant change in behavior
Yes, in incognito mode cookies are deleted, within window closing, but that is the only difference
LOgin on 1 page > (cookie created) > autologin on the different page of the same browser
It not working for the different browsers, because there are different cookies. For incognito mode, you have to enter credentials every time, when you enter private mode because cookies are deleted

---

## Resource access with role-based authorization

###  4 Enabling authorization

#### 4.1 Questions

*    a) What is the format of JWT token? How can you decode it, is there any standard tools?

`ID token` in `JSON web token` format is `JWT`. Basically, this is some kind of peace of data, that can be interpreted by the client (application) as useful data.
It can be decoded with online tools like [jwt](jwt.io)
Since it is encoded data, it can be decoded by the server provider of the `JWT`. You need to have an algorithm and a secret key.

*    b) How is JWT token issued and verified?

After your login via the `Authorization server`  user will get `JWT` in return. This is used as a token, in order for the server to verify the client.
Issuer `authentification server` creates a secret and encrypts with it credentials of the user, and shares it with the application, then, the client can send a request to he `API` directly. Because the application already has a key, user data can be decoded, and used in the authorization of the another resource


*    b) What is claim and what is scope?

`Claim` is something is inside of JWT and this data is used as credentials, in the format of name-value
`Scope` Data requested by the one part of the communication, usually can contain, or request several claims. Or in other words set of rights, you get from resource

*    c) What is aud claim?

`aud claim` refers to the audience of the claim, not the receivers of the data behind `aud`. 
People are often confused about the `aud` and `client_id`.
`AUD` is for the consuming side, and `client_id` is for obtaining the party

#### 4.4 Verification in the application

The code below is requesting for the token of the `API` to be able to provide with the `role` of the user, `certificate` and specific access type of the header, `Bearer` one. `Bearer` is used for the clients without the login process. Those three parameters i will configure in the new `API` client. The API is another code, at it is requested, specific roles in the `roles` field

<center>

![](https://i.imgur.com/wgiDKUw.png)
Code of the verification, befor API connection
</center>

As we can say from the analysis, role, type of access and certificate is a must in the communication, all of this makes a access token

#### 4.5 Create corresponding clients and add roles

In order to create an API connection in our application, i will create a new client API-pokemon, in the `bearer-only` mode

<center>

![](https://i.imgur.com/fxL7eSH.png)
Client creation
</center>

Inside this new client, I need to set roles, for the users, of this application

<center>

![](https://i.imgur.com/oo7JWbA.png)
roles of the api
</center>

Now, I need to set roles for the user, it can be done in the user configuration window.

<center>

![](https://i.imgur.com/0mKaeu3.png)
    
</center>

The last thing is to take a certificate, in order to make communication possible.

<center>

![](https://i.imgur.com/XFcuYgf.png)
    
</center>

At this point, everything is configured, and we can access web resources with the usage of the api. In that way, we will get the ability to search the pokemons in our web application

<center>
    
![](https://i.imgur.com/xDpa40m.png)
    
</center>


1. app-pokemon is working?

Yes

2. what is included inside access token

My particular token consists of fields, and they are common for such requests, i will specify some of them

<center>

![](https://i.imgur.com/bxefBXr.png)
    
</center>

In the token, we can see all rollers settled for the API and for the application.
Also, we are asked by `API-pokemon` (aud), an application that asked for the information `app-pokemon` (azp), and the address of the giver of the token (iss). And the others parameters

##### Questions

*    a) What are realm roles and client roles in Keycloak?

`Realm` manages the security of the metadata of users and `clients`, can be shared between clients, within `realm`

`Clients` as well as `clients roles`, exist in the domain of the one `realm`, and their purpose is to work with the credentials and data of the `user` within one `client`

In this case difference between them is a matter of  scale

*    b) What is a composite role?

`Composite Role` that can associate users with several roles. That makes the administration easier

*    c) How does browser-side app `app-pokemon` understand that form input should be enabled/disabled?

Since API-client is configured as read and edit mode, we can change values in the characteristicks fields of pokemon.

<center>

![](https://i.imgur.com/VGm2Z7t.png)
    
</center>

Those fields are possible to change since we have editor role enabled

<center>

![](https://i.imgur.com/wwNBrYm.png)
    
</center>

After we disable this role, it is not possible to edit those values, only read

<center>

![](https://i.imgur.com/HArTWba.png)
    
</center>



### 5 Issuing fine-grained access token

#### 5.1

1. Create another `test-client` with `some-role` and add this role to the sample `user`

New user and the role created in the same, as previous users, way

<center>

![](https://i.imgur.com/qkSJ7SR.png)
    
</center>

2. inspect `access_token` token for `app-pokemon`

Now, it is visible, that, every created rule is visible inside the token. We can remoove some of them

<center>

![](https://i.imgur.com/c2Fi5c0.png)
    
</center>

3. Is the new role automatically included in it?

yes, that is visible on the picture abowe


#### 5.2 

1. how exactly user information possesses is mapped inside to access token do this only `api-pokemon` roles are mapped to the access token, even if he has other roles

In the `scope` setting window, we can edit this and set only important and useful tokens
In the picture below you can see, how i specified only the 2 roles to send

<center>

![](https://i.imgur.com/0sB7Uxf.png)

</center>

In the picture below you can see the difference between, turning `full scope allowed` turned on (left) and turned off (right)

<center>
    
![](https://i.imgur.com/THbt7zZ.png)
    
</center>

* a) How is `claim` added to access token?

token generated with the data, that is located in one `realm`. As we can see it from creation of `test-client`.

### 6 Custom scopes

1. access token for issued for `app-pokemon` will include `favorite_color` user attribute.

Custom scopes are useful in some cases, so we should consider the possibility of making such information fields in `JWT` 

First, we have to create a new custom scope with the custom name `favorite_color`

<center>

![](https://i.imgur.com/8opUAkj.png)
    
</center>

Then, add this new scope to the list of the scopes, that will be used during communication

<center>

![](https://i.imgur.com/NJ4z1aD.png)
    
</center>

When the new scope is added, every token will be generated with this scope. In the picture below you can see two options, one without a scope, another with

<center>

![](https://i.imgur.com/NI3oK6d.png)
Access token without custom scope
    
![](https://i.imgur.com/DQd1mZr.png)
Access token with a custom scope
    
</center>


