###### tags: `AS`
:::success
# AS Lab 1 Assignment: Threat Modeling
**Name: Radomskiy Ilia**
:::
 
Tables with entry points, assets, trust levels
Data flow diagrams
Threats summary table
Report that contains clarifying information about your results

---

## Threat Model Information

1. **Application Name**

Youtube-like
It will be easier to talk about specific stuff, so i will use this [github](https://github.com/WWBN/AVideo) repository as an example of such application

2. **Application Version**

v0.1

3. **Description**

Features:
- upload and delete videos
- get video streams with desired quality
- search available videos (filtering and sorting by some attributes)
- display user profile and uploaded videos
- some videos are private (only specific user) and some are hidden (not shown insearch and on user page)
- display view history
- display, create and delete comments on videoSystem entities:
- user data
- user upload history
- user view history
- video data
- video objects
- comments

System components:
- Application servers
- Databases
- Video processing queue
- Video uploading servers
- Video object store

4. **Document Owner**

Nikita

5. **Participants**

SNE students

6. **Reviewer**

Radomskiy Ilia

7. **External Dependencies**

    a. **Server**: linux Ubuntu20.04.4 LTS
    b. Programming lang PHP 8
    c. Database MySQL 8.0.19
    d. Apache 2.4
    b. Comunications between the system parts will be happening in the private network
    c. Comunication with the all the other world will be happening via TLS

8. Trust level
1.	Anonymous
2.	User with invalid Credentials
3.	User with valid Credentials on the other channel
4.	User with valid credentials on its own channel (private video)
5.	Moderator
6.	Database server administrator
7.	Website administrator
8.	Web server read user (no sudo) process
9.	Database read user (no sudo)
10.	Database Read/Write User (no sudo)



9. **Entry points**

1. HTTPS Website available via tls 1-5
2. Main page entry point for all users 1-5
3. Login page availble for all users 1-5
4. Search engine availble for all users 1-5
5. User account settings personal chanel(private video) etc 4-5
6. Video can be seen by any one, if not specified 1-5

10. Exit points

1. HTTPS Website available via tls 1-5
2. Main page entry point for all users 1-5
3. Login page availble for all users 1-5
4. Search engine Availble for all users 1-5
5. User account Settings personal chanel(private video) etc 4-5
6. Video Can be seen by any one, if not specified 1-5

11. Asstets

1. ID of user
2. 

https://owasp.org/www-community/Threat_Modeling_Process#