###### tags: `OT`
:::success
# OT Lab 3 Assignment: DevSecOps
**Name: Radomskiy Ilia**
:::


## Task 1: Setting up your environment

:::info
1. Create two VMs, for the Jenkins server and its agent.
:::

For this lab i will be using `GNS3`, and two machines in it in order to configure network with one `Jenkins` server and one slave-node

<center>

![](https://i.imgur.com/igNPcdQ.png)
Img 1 - Network topology
</center>


* a. Install Docker Engine on both VMs.

Docker was installed in that way

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

sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo docker-compose --version
```

<center>

![](https://i.imgur.com/F4vzAGf.png)
Img 2 - Docker check
</center>


:::info
2. Isolate your Jenkins server behind a reverse proxy and access it through port 80 (e.g Nginx Reverse Proxy).
:::

[**Why should we do this?**](https://www.jenkins.io/doc/book/system-administration/reverse-proxy-configuration-nginx/)

It is a matter of security, it is better to run your environment, `jenkins` in our case, in unexpected way, or hide behind the proxy.

Install `nginx`
```
sudo apt install nginx-full
```

Change configuration in the file
```
sudo nano /etc/nginx/sites-available/default
```

<center>

![](https://i.imgur.com/NmfrjK1.png)
Img 3 - configuration file
</center>

To check configuration we might use

```
nginx -t
```

<center>

![](https://i.imgur.com/AnVkGMC.png)
Img 4 - nginx check
    
</center>

Next, install `Java`, and `jenkins` itself

```
sudo apt-get update
sudo apt-get install default-jdk

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
  
sudo apt-get update
sudo apt-get install jenkins
```

And change configuration in order to configure redirection

```bash=
sudo nano /etc/default/jenkins
#add this line in jenkins config
--httpListenAddress=127.0.0.1
```

Then restart both services

```
sudo systemctl restart jenkins
sudo systemctl restart nginx
```

On the picture below you can see difference between `Jenkins` working with proxy and without

<center>

![](https://i.imgur.com/iB4uLxh.png)
Img 5 - Comparison if Jenkins with and without proxy
</center>

[1] 

Now, lets configure client.

```bash=
#create ssh keys
ssh-keygen -t rsa
cat id_rsa.pub > ~/.ssh/authorized_keys
cat id_rsa
ssh-copy-id ubuntu@10.1.1.21
```

And add client on the `Jenkins` server

<center>

![](https://i.imgur.com/8LW6WEx.png)
Img 6 - Adding new node to master
</center>

[2]



:::info
3. You will be working on the following Maven project. Go through it and familiarize yourself (git clone https://github.com/HamidullahMuslih/ot-lab-sdlc.git )
:::

OK, got it.
```
git clone https://github.com/HamidullahMuslih/ot-lab-sdlc.git
```
Nice repo, awsome commits
<center>
    
![](https://i.imgur.com/Kf8xybl.png)
Img 7 - Smoking Arnold Alois Schwarzenegger
</center>

:::info
3. Read and explain in one line sentence the Maven lifecycle. Understand commands for each phase (e.g build, unit test, integration test, packaging and etc.) since you will need to use them during the pipeline tasks.
:::

`Maven` is used instead of manual compilation/build of the project in `java`.
Default life cycle is `validate`, `compile`, `test`, `package`, `verify`, `install`, `deploy`. Some of the steps are missed. because they are surplus

`build`: code + os = compiled version
`unit test`: test of separated modules
`integration test`: test of modules combined, and tested in that way
`packaging`: creation of handyarchive for different purposes, like `.jar` (`.zip` archive with `.jar` extention) and sending `classes`, `interfaces`, `resources`, and `metadata` in it
[3] 

---

## Task 2: Implement DevSecOps Pipeline

:::info
1. Create the first job to fetch and build the `Maven project`. Note: skip the testing in this job.
:::

First i download plugin of `maven` inside `jenkins` and restart it. Also new item as `maven project` have been created

<center>

![](https://i.imgur.com/819QFzv.png)
Img 8 - New maven project
    
</center>

And i will build it with `clean` , `install` and `-Dmaven.test.skip` commands. Last one is used to skip the test parts.

<center>

![](https://i.imgur.com/DI9ODdS.png)
Img 9 - Setting parameters for project
    
</center>

<center>

![](https://i.imgur.com/bHzkZKo.png)
![](https://i.imgur.com/ZYh1nOJ.png)
Img 10 - Output of the building
    
</center>

Job have completed successfully.
For `clean` stage there was deletion of previous build
And `compile` is uses source code in order to create binary file.
In this example we are not testing application with `-Dmaven.test.skip`

:::info
2. Create the next job to perform the `unit tests` of the project.
:::

Unit test is type of tests where programmer is verifying performance of the parts of the application.
If we need to run unit tests we can specify it in maven with `test`. I will set it in the new maven project `Maven-OT3-test`

<center>

![](https://i.imgur.com/rYKjIbb.png)
Img 11 - Setted 'test' parameter
    
</center>

That is lead to the test. All of the finished successfully

<center>

![](https://i.imgur.com/vSSUUiT.png)
Img 12 - Tests results
    
</center>


:::info
3. Create the next job to perform the `integration test` on the project.
:::

Integration tests are used when we need to check if the application as a whole is working normally
Maven parameter `Integration-test` is specified in order to run such tests

<center>
    
![](https://i.imgur.com/iX8VBIx.png)
Img 13 - Setting up 'integration-test' in the 'jenkins' field    
    
</center>


As a result we are got the 6 tests, all of them are successful. Additionally to the unit test, is makes 9 test in total.

<center>

![](https://i.imgur.com/AZrqkup.png)
Img 14 - Tests result
    
</center>

[4]

:::info
4. Create the next job to perform `static code analysis` on the project (e.g with the CheckStyle, PMD, FindBugs tools). Note: Read about `Warnings Next Generation` Plugin .
:::

Static code analysis is very important part of the `CI\CD` pipeline. On this stage most of the non critical errors should be fixed automatically.
For that i will be using `Warnings Next Generation` plugin in `Jenkins`

In new project and i will set up new configuration settings for the static analysis.

<center>
    
![](https://i.imgur.com/5JHxGva.png)
Img 15 - Creation of the new project
    
</center>

After run we can see reports of the static analysis

<center>

![](https://i.imgur.com/plj0owc.png)
Img 16 - checkstyle, pmd and FindBugs reports
    
</center>

*    **a.  Show the reports and share your understanding.**

In the sections below i will speak about reports in detail

`Checkstyle` is set of java coding rules, you have to follow them as your best practice coding style in java. That will make your code more readable, secure and clean

<center>

![](https://i.imgur.com/BHzKFX4.png)
Img 17 - checkstyle report
    
</center>

As an example of the 706 issues, i will show one of them. In the line 50 in the picture below, you can see code `if (!dir.exists())`. As `checkstyle` stands you have to use `{}` in `if` constructions

<center>

![](https://i.imgur.com/neYevnG.png)
Img 18 - Brackets issue    
    
</center>

`pmd` also static analyzer. It is used more as a tool of prevention of bad habbits of programmer.

<center>

![](https://i.imgur.com/wvnQCPT.png)
Img 19 - 'pmd' report
    
</center>

In the example below you can see that, `PMD` have found unused library. So it might be deleted

<center>

![](https://i.imgur.com/8rZQFyM.png)
Img 20 - pmd report
    
</center>

`FindBugs` is used, as stands in the name, to find bugs. In java code. `FindBugs` works with Java bytecode, not source code.

<center>

![](https://i.imgur.com/i3agSHe.png)
Img 21 - FindBugs report
    
</center>

Bug that have been found in the java application is problem with the encoding from byte to string.

<center>

![](https://i.imgur.com/TXBMSv4.png)
Img 22 - FindBugs report  
    
</center>
[5]

*    **b. Feed the static code analysis reports to the `Violations` Plugin and set the quality gates for the number of bugs in one of the reports, validate your work to fail/unstable the job. Note: after validation remove the quality gate restriction since it will fail the last task of the lab.**

`Violations` plugin is used based on the static analysis of the `checkstyle`, `pmd` and `findbugs`. After that short report is generated. 

We can set level of acceptance of the errors, so if we don't want we can dismiss whole pipeline, because of the amount of errors, bugs and etc.

<center>

![](https://i.imgur.com/RImNXTo.png)
Img 23 - Violation    
    
</center>

We can set number, how much errors we can get without any consequences. In the image 23, if number of checkstyle warnings is less than 10, it is ok.
It the picture 24 i setted up, such setting, 'if the value is less than 900 errors, then it is ok'

<center>

![](https://i.imgur.com/iUgyrpn.png)
Img 24 - Everything sunny in 'Jenkis'  
    
</center>

But off course it is not right. You have to specify more strict rules, like in the picture below.
As you can see, 706 errors is too much, and that makes us cry.
In case of `PMD` warnings it is okay, but you need to fix some of them
But, 10 bugs for your project, as it was specified in the `Violations` is fine

<center>

![](https://i.imgur.com/kT4lvZb.png)
Img 25 - Everything not that sunny in 'Jenkis'  
    
</center>


:::info
5. Create the next job to dockerize your artifact from target/`****`.war and push it to the docker hub.
:::

This new job is used in order to make docker container and push it to the docker hub account
In the picture 26 you can see `dockerfile`

<center>
    
![](https://i.imgur.com/HyinXT8.png)
Img 26 - Docker file
</center>

In such manere as in the picture 27, we can see, in order to build and push something to doker hub, we need to add two things, `name of account`/`name of the app` and account credentials

<center>

![](https://i.imgur.com/uAFfBR2.png)
Img 27 - Post Steps of the maven project
    
</center>

In the Console output we can see thecommand to build `docker` image with the name `java-example`

<center>

![](https://i.imgur.com/lB0Vdap.png)
Img 28 - Console output    
    
</center>

Later, this image is uploaded to the `dockerhub`

:::info
6. Create the next job to deploy the docker image from the docker hub to the Jenkins agent and validate/show that you can access the web app.
:::

Now, since our docker image is on the docker hub, we can download and run it. We will do this during build stage of the `jenkins` project `Project Maven-OT3-deploy`

<center>

![](https://i.imgur.com/YPuEXBS.png)
Img 29 - Console output  
    
</center>

Same shell commands we can see in the console output

<center>

![](https://i.imgur.com/CvjJTI4.png)
Img 30 - shell commands used during this pahse
    
</center>

After container is uploaded and running we can enter application via browser

<center>

![](https://i.imgur.com/YPn2lEp.png)
Img 31 - Login screen of the application
    
</center>



:::info
7. Finally, chain all the jobs such that when you run the 1st job it should execute the rest.
Note: Use Post-Build Actions>Build other projects.
:::

Last stage of the pipeline creation is concatenation of the created projects together. We can do this with `Post-build Actions` by typing name of the downstream project

<center>

![](https://i.imgur.com/oP0KrYg.png)
Img 32 - Post-build Actions cancatination
    
</center>

Resulted pipeline we can see on the last job of the pipeline, like in the picture 33

<center>

![](https://i.imgur.com/JlNFnC5.png)
Img 33 - Console output of the last project 'Maven-OT3-deploy'        
    
</center>


## Task 3: (Optional) I need more

:::info
1. Integrate Slack notification such that for each job the developer gets notified whether the job is a succus, failed, unstable. (Easy)
:::

In order to set slack nitifications we have to:
1. register on the slack web page (vpn)
2. create room for the lab

<center>

![](https://i.imgur.com/rH4TabW.png)

</center>

3. Create jenkins app, and configure ir, in order to give ability to write messages and others permissions, like this one on the picture

<center>

![](https://i.imgur.com/qEMK6I2.png)
    
</center>

4. create OAuth tocker in order to login Jenkins to the workspace
 
<center>

![](https://i.imgur.com/t903Hex.png)
    
</center>

5. Next, inside jenkins we need to connect Slack and jenkins, so, enter workspace name, credentials of the app, and Chanel. You have to invite your application\bot to that channel

<center>

![](https://i.imgur.com/Bax1Ew1.png)
    
</center>

6. After that you are connected

<center>

![](https://i.imgur.com/NZYP2aW.png)
    
</center>

7. next thing is to create notification in your job. It can be connected with the sme kind of condition of the progree, like success, abort, unstable and others

<center>

![](https://i.imgur.com/V05SYDm.png)
    
</center>

8. Later, after everything is setted up you will recieve notification on your server


<center>

![](https://i.imgur.com/Kc7DDmG.png)
    
</center>

:::info
2. Integrate Snyk with Jinkens and find project vulnerabilities.
:::

:::info
3. Integrate Burbsuit in your pipeline and perform vulnerability scanning against the web application.
:::

New plugins like `Burbsuit` for jenkins need to be download from the web sites of the creators of the plugins, [like this](https://portswigger.net/burp/releases#driver) and installed with advanced section of the plugin instlation section

<center>

![](https://i.imgur.com/8Okz6ft.png)
![](https://i.imgur.com/LrHkl81.png)
    
</center>

Then you need to configure new user for the `Burp Suite Enterprise Edition`. Since it is paid app, i have to take trial version.

In the `Burp Suite Enterprise Edition` i will creatu user according to the official [docementation](https://portswigger.net/burp/documentation/enterprise/integrating-with-other-tools/ci-cd/jenkins/site-driven) and its perposes, so to Jenkins CI/CD checker

<center>

![](https://i.imgur.com/ilTmn1W.png)
    
</center>

`Burp Suite Enterprise Edition` afteranalysis of the web application gives us such output, but lets use jenkins for this purpose

<center>

![](https://i.imgur.com/ksXoTUR.png)

</center>

Now we can doo same thing but with jenkins, for that we can add user credentials to the jenkins server

<center>

![](https://i.imgur.com/3AahBL7.png)
    
</center>

In the end we have got analysis, that was persormed from the jenkins server

<center>

![](https://i.imgur.com/SP1c0AQ.png)

</center>

:::info
4. Deploy the SonarQube server in a VM or Docker container and perform static code analysis and set quality gates with SonarQube.
:::

:::info
5. Write fuzz test cases and perform fuzzing on the software.
:::

:::info
6. Deploy SonaType Nexus artifact server such that, after code analysis (6th job) the artifact should be uploaded to Nexus server.
:::

:::info
7. Deploy K8s cluster and make master node as Jenkins agent.
:::

* a. Write manifest files for the docker image.
* b. Create a job (or replacement of number eighth job) to deploy the app on K8s cluster.

## References

1. [jenkins using an nginx reverse proxy](https://www.digitalocean.com/community/tutorials/how-to-configure-jenkins-with-ssl-using-an-nginx-reverse-proxy-on-ubuntu-18-04)
2. [Jenkins add client](https://devopscube.com/setup-slaves-on-jenkins-2/)
3. [Maven](https://www.baeldung.com/maven-packaging-types) 
4. [Lifecycle-tests](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)
5. [SAST](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Jenkins-Warnings-Plugin-CheckStyle-FindBugs-PMD-Example-Tutorial)\
6. [Burp](https://portswigger.net/burp/documentation/enterprise/integrating-with-other-tools/ci-cd/jenkins)
