###### tags: `AS`
:::success
# AS Lab 4 Assignment: Identity and Access Management (IAM) - 2
**Name: Radomskiy Ilia**
:::

---

## Task 1: Fine-grained access

### 1. Create corresponding clients and enable Authorization services

To retrieve token from `api-trainer` we need to create it in the keycloak. Important to notice, that we need to set `access type` to the `confidential` type, as well as enable `authorization`.

<center>

![](https://i.imgur.com/9pHQmkb.png)
    
</center>

Also role `trainer` was created in the domain of the client `api-trainer`. `User` was attached within this role

<center>

![](https://i.imgur.com/aWiNgjA.png)

</center>

All of the above gives us the configuration file, that will be used in the application of the `api-trainer`

<center>

![](https://i.imgur.com/4P931B2.png)

</center>


### 2. Implement authorization rules

We are specifying resource `deck_create`, in the `api_trainer`.

<center>

![](https://i.imgur.com/uS1POoP.png)
![](https://i.imgur.com/L5hkrdp.png)
    
</center>

Scopes, that are used in credentials, are shown in the picture below

<center>

![](https://i.imgur.com/2rbFhzj.png)

</center>

We also need to create a policy. Policies are telling, what conditions must be included in the data before it is possible to grant access to resources.

<center>

![](https://i.imgur.com/chRcdXH.png)

</center>

Those policies are protecting from the not authorized access to the resources. policies, associated with the permissions are the main choice mechanism, whether should request be granted with the access or not

<center>

![](https://i.imgur.com/yLgaBn9.png)
List of permissions

![](https://i.imgur.com/LMzztK1.png)
policiy "tt", associated with the permission "deck_create"
</center>

After eveything is configured we can `add/change/delete` decks.

<center>

![](https://i.imgur.com/8GQnLwN.png)
Created deck "qwer"
</center>

---

## Task 2: User-Managed Access

### 3. Demonstrate resource sharing

Sometimes users need to share resources with others, that is why we may add this possibility to the application, and `realm`
For that, we need to enable this possibility in the `realm`

<center>

![](https://i.imgur.com/sl8eLki.png)

</center>

We can check what available resources we have in the domain of the `user`. For that just enter the web page `http://sso:8081/realms/as_lab/account/#/resources
`. This data will be shared.

<center>

![](https://i.imgur.com/boQkkbd.png)

</center>

After we enabled the user management access, we are able to share created resources with the other users, `another_user` in my case. In the picture below, we can see, that is this created deck is available for the new user

<center>

![](https://i.imgur.com/NUTK4Kg.png)

</center>


---

## Task 3: Audit

### 4. Audit capabilities

Audits, logs, and event management is important for nonrepudiation, security and maintenance of the system, That is why it is worth mentioning configuring it in the `keycloack`.
It is simple, set `Save events`, in the position on, and choose the spector of the saved actions

<center>

![](https://i.imgur.com/36D7RCC.png)

</center>

In the picture below you can see, that logging information is displayed in the, history of the events

<center>

![](https://i.imgur.com/RbeqAkh.png)

</center>

---

## **Bonus: UMA 2.0 protocol**

### 5. Learn about UMA, its main concepts and flows

**a) What is the protection API (PAT) token?**

The main reason for the `protection API` is the only resource server is allowed to access this API. This protection is used for managing resources, scopes, permissions, and policies in the resource server. PAT may be used for `Resource Management`, `Permission Management`, and `Policy API`.

**b) What is `permission ticket`?**

If a client wants to do something with the `resource server`, the client must have a ticket that allows him to make any changes, a `permission ticket` in particular. This ticket is defined by `UMA`. AS it stands in the name, `permission tickets` are used by clients to access a protected resource
conditions for ticket:
*    Singe-use
*    Secret
*    Data in tickets random
*    Should be sent back to the client by the authorization server
*    Becomes invalid if the ticket is compromised

**c) How does `resource server` create `permission ticket`?**

`resource server` is the server that is used for storing and grinding the protected data\resource. `resource server` request some secret (security token). 
[Quote](https://docs.kantarainitiative.org/uma/wg/rec-oauth-uma-grant-2.0.html#rs-tokenless-response)
`resource server MUST obtain a permission ticket from the authorization server`
In general `resource server` is asking from the `authorization server` permissions (resource identifiers and corresponding scopes) on the client's behalf, and recieve a `permission ticket` from the `authorization server`

<center>

![](https://i.imgur.com/ylanZiV.png)

</center>

**d) How does the client use `permission ticket`?**

Requested by the `resource server` permissions are given by the `authorization server`. And, after the exchange is over client can modify the information in the `resource server`

**e) What should `authorization server` do with `permission ticket`?**

In the picture above you can see process of ticket-granting, `authorization server` is giving this ticket depending on the requested `scopes` and `permissions`. So `authorization server` should accept request from the `resource server`, and decide, whether the ticket should be granted or not. 
Decision-based the comparison of the requested `scopes` and `permissions` to the client's resource request

**f) What is `RPT`?**

`Requesting Party Token` - access token with permissions, and it is a token we was speaking about in the  `permissions tickets` section


### 6. Perform UMA Grant flow using postman (or any tool you like)


## Reference
1. [resource overview keycloak](https://www.keycloak.org/docs/latest/authorization_services/index.html#_resource_overview)
2. [server administration guide from redhat](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on_continuous_delivery/7/html/server_administration_guide/clients)
3. [service protection permission api](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.2/html/authorization_services_guide/service_overview#service_protection_permission_api_papi)