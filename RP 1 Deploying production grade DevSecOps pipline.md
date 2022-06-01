###### tags: `RP`
:::success
# RP 1: Deploying production grade  DevSecOps pipline 
**Name: Radomskiy Ilia**
:::

SCM trigger + jenkins master and agent + pipline script 

## Abstract

Abstract - in this project I will create a production-grade pipeline according to `DevSecOps` standards using tools that are often used by `CI/CD` engineers like `Jenkins` and etc. First of all, I need to understand how to build pipelines in the `Jenkins` tool. After that security might be added to the project in order to check library dependencies with vulnerabilities, or vulnerabilities itself inside the project.


## I. Introduction

In order to understand what `DevSecOps` means we should look closer to the `DevOps` term [1]. `DevOps` means `development & operations` this is a development process improvement technique, it is used in order to create an environment for project automated development. `DevOps` term is close to another term `CI/CD` (continuous integration / continuous delivery/deployment). So we can say that some DevOps methods are used in order to create` CI/CD`. `DevOps` was created from the process of `Agile` software development, this term describes the collaborative work of a group of people in order to improve their productivity.
Speaking of a real `DevOps` example, we can see how it is implemented in the picture below.

<center>
    
![](https://i.imgur.com/MD6gNy4.jpg)

</center>

Describing this picture it is can be told that `DevOps` is a `method to create effective and automated infrastructure for the development process`. Another definition was suggested by Len Bass, Ingo Weber, and Liming Zhu [2]`a set of practices intended to reduce the time between committing a change to a system and the change being placed into normal production while ensuring high quality`.
But at some point, engineers start to implement security features in their `DevOps` infrastructure, in that way `DevSecOps` become a thing. This is not only good practice, to implement security in your project, in the modern world it is mandatory to bother with any security issue of your project.
`DevSecOps` includes its own analysis practice: static analysis or Whitebox (`SAST`),  software composition analysis (`SCA`), and dynamic analysis (`DAST`) or black box. As you can see from the picture below, the `DevSecOps` pipeline is consisting of a bigger amount of tools used in order to deploy an application

<center>
    
![](https://i.imgur.com/IZwCPPm.jpg)

</center>

In this project, some of the `DevSecOps` practices will be described.

## II. Research goals

* Research about `DevSecOps` pipeline tools, and security tools and plugins
* Implement production grade`DevSecOps` pipeline
* Describe some of the possible variations of the `DevSecOps` pipeline

## III. Related works

[DevSecOps: A CASE STUDY ON A SAMPLE IMPLEMENTATION OF DevSecOps](https://www.researchgate.net/publication/346211873_DevSecOps_A_CASE_STUDY_ON_A_SAMPLE_IMPLEMENTATION_OF_DevSecOps)

[DevSecOps: A Boon to the IT Industry](https://deliverypdf.ssrn.com/delivery.php?ID=772029127089022107119081012100107029127088006054089053094031103111098115030093070027041060048030012023098000115066085073006072033006063021080093090065026117123090109058013055087106104121067000104088069109099089029019071093005002006071127031123115018011&EXT=pdf&INDEX=TRUE)

[DevSecOps Metrics](https://www.researchgate.net/publication/335705973_DevSecOps_Metrics/citations)

## IV. Complex Solution

In order to achieve my research goals, I need to reach several objectives.
First of all, I need to choose the `CI/CD` solution, I will provide pros and cons some of them: `jenkins`, `gitlab`, `CircleCI`. After tool have been chosen I have to do describe all of the possible methods of configuring pipelines for the particular tool: different triggers, different approaches to build pipeline, etc.

Second thing, after the main `CI/CD`tool have been chosen and described, I need to build `DevOps` pipeline. Later on, this pipeline will be used with the adding security mechanisms.
But before implementing the `DevSecOps` pipeline I need to understand what kind of security tools are existing, in order to use them.
The last one is to gather everything `CI/CD` tool, security tools, in one project, or several projects if necessary to show the difference in aproaches

* difference jenkins/oth (optional)
* jenkins description
* tools description

## V Jenkins vs git lab vs curcle ci 

There are several tools that can be used in order to create a `CI/CD` pipeline. In this section, I will cover some similarities and the differences of `Jenkins` , `gitlab` and `circleci`, during the `continuous integration` process. 

### curcle ci


Circle ci is a Cloud `CI` platform, it is Partially free and has some features that differs it from the other `CI` tools[3]:

    REST API available
    Caching of dependencies
    SSH connection available
    Requires a small amount of configuration

Some of the key features can be pinpointed:
Pros:

    it is easy and fast to start deploying your applications
    The YAML file is easy to create and work
    no need to set independent server

But, of course, there are some downsides:

Cons:

    CIrcle Ci is free only on ubuntu
    It is free only for `go`, `java`, `python`, `ruby/rails`, `scala`, and some other programming languages


### git lab

`GitLab` is a part of the Version Control System, `VCS` of GitLab. Some of the features are similar to the other `CI/CD` tools. But some of them are different from the previous, `Circle CI` tool, such as `LDAP` server can be used. Also, `GitLab` is an open-source product with additional features if you chose subscription plans[4].

Pros:

    free for the basic features
    popular tool
    GitLab Pages is a website that can be used to publish results
    access control for repositories
    ability to work with the different VCS (not only GitLab)
    no need to set independent server
    monitoring included

Cons:

    can't test results of branches `merge` before `merge` itself
    can't separate stages of ci/cd
    massive work with artifacts

### Jenkins

`Jenkins` is an Open source application written in java. Jenkins server needs to be deployed in order to provide `CI/CD` features like software development related to building, testing and deploying, facilitating continuous integration and continuous delivery[5]. `Jenkins` also supports `REST API`, as well as you can have full access to the platform.

Pros:

    free
    plugins can provide with the additional possible integration and usability
    configuration possibilities
    can be used in cybernetics (Jenkinsx)
    parallel execution are available
    ability to use nodes/slaves

Cons:

    need independent server
    requires time for configuration
    plugins are vulnerable
    

## VI Jenkins DevOps pipelines

Before building the `DevSecOps` pipeline, I should build the `DevOps` one. The difference between them is in security tests. But what ways exist in configuring a pipeline? I will answer that question in the next section

### A)

Jenkins offers a various amount of instruments in order to configure your `CI/CD` environment. You can take a look on the picture below in order to see them. List of the `items` is extensible with the plugins, that can be downloaded to `Jenkins` server

<center>

![](https://i.imgur.com/GkGzWLA.png)

</center>

`freestyle project` can be chosen as an example. In this case, you need to configure your pipeline with your mouse, and, only in some small part of the project write a script for the pipeline.

For example, there are several methods of triggering your build:

1. Trigger builds remotely (e.g., from scripts): you can use the link, created by `Jenkins`, in order to start building
2. Build after other projects are built: you can concatenate your projects in order to follow the special order
3. Build periodically: you can set up a schedule for the periodical build
4. SCMtrigger: you can check for changes in your repository, and it can be triggered for your project
5. GitHub hook trigger for Git SCM polling: your git account can tell your `Jenkins` server about changes

Adding credentials SCM of yours is very useful thing, for example, you can use code from `GitHub` in order to build and deploy applications. Of course, `Jenkins` allows to use of local storage, but SCM integration makes more use. 

<center>
    
![](https://i.imgur.com/y8Wtar3.png)

</center>

This requires adding SSH keys in order to connect to SCM via ssh, for example for your private repository
Credential data, like private keys, are also required if you want to connect to any workstation, like slave `Jenkins-node`, or your deploy workstation. Those are stored in the credentials section of the `Jenkins` settings. And you can observe this on the picture below. This is one key that is used for the ssh connection to some of the machines

<center>
    
![](https://i.imgur.com/hm6xY58.png)

</center>

After you have set up all you need for your build and deployment: your machines are working and accessible; you can start you deployment phase, so write a pipeline itself. It can be written as a part of a `freestyle project` or `pipeline` project.

**freestyle project**
In the particular case below you can see build and test stage that is happening on the `Jenkins` master server. 

<center>
    
![](https://i.imgur.com/IGFqmFt.png)

</center>
    
After that, if the `test` build is successful, it is deployed to the `test` server

<center>
    
![](https://i.imgur.com/H0yoBI5.png)

</center>

As it was told before, `Jenkins` plugins give to project more flexibility if it is required. It is also useful if you want to integrate some other technology, like `AWS Elastic Beanstalk`, for example, you can see a plugin for it in the picture below

<center>

![](https://i.imgur.com/FZ3eg4h.png)

</center>

**pipeline**

the pipeline is a little bit different approach for the configuring pipeline, and I will describe it below, in the next section

### B)

The pipeline is a script that is written to tell the `ci/cd` tool about your `way-to-production` process. It can be written in different languages, in particular, case of Jenkins it is a `groovy` language. The difference from `freestyle project` is that in the `pipeline` case you write the whole process as a script. Some parts might be configured outside of a script, like credentials, but this part was discussed above.
An example of my `DevOps` pipeline is written below and it should include several stages:
```
    agent setup
    Initialization of variables
    Building of application
    SSH Deployment
```
```java=
    pipeline {

        agent {     //agent setup
            node {
                label 'Node1'
            }   
        }

        stages {
            stage ('Initialize') {
                steps {
                    sh '''
                                echo "PATH = ${PATH}"
                                echo "M2_HOME = ${M2_HOME}"
                        ''' 
                }
            }

                stage ('Build Phase') {
                    steps {
                        echo 'Build Phase'
                        sh '''
                            mvn clean package # make war file
                        '''
                    }
                }

                stage ('Deploy-to-tomcat') {
                    steps {
                        sshagent(['tomcat']) {
                            sh 'scp -o StrictHostKeyChecking=no /home/ubuntu/jenkins/workspace/Pipeline-job-1/target/*.war ubuntu@192.168.122.103:/opt/tomcat/webapps/webapp.war'
                        }
                    }
                }
```       

If we image environment built for this papilene as a pictogram it will be look like on the picture below.

<center>
    
![](https://i.imgur.com/0HUdqTD.png)

</center>

## VII DevSecOps tools

Before start to build `DevSecOps` we have to know about tools that can be used as a part of `security operations`. Tools can be divided in different categories.

SAST - Static Application Security Testing  is used if we need to check code with the correspondence of vulnerability pattern and style. Additionally file configuration analysis is happening. In the list below there are some the tools:
```
    pvs
    checkmax
    fortify
    coverty
    veracode
    incode
    positive technologies
    bandit: python applications
```

OSA/SCA open source analysis. This stage can consist of different steps:
```
vulnerabilities in libraries
analysis of license clarity
post build analysis
```

In the list below there are some the tools:
```
sonatype 
nexus
white source
source clear
blackduck
dependency-check
```

DAST - Dynamic Application Security Testing. On this step our application is tested by the imitated user of black hat operator. DAST application can send to application `evil` patterns and analyze the responses [7]
In the list below there are some the tools:
```
burpsuite
acunetix
owasp zap
micro focus
w3af
appscan from ibm
```

In my project following application are used. In order to perform all of the described above steps of automated testing:
```
trufflehog - OSA/SCA
owasp-dependency-check - OSA/SCA
sonarqube - SAST
owasp-zap -DAST
```

`trufflehog`[8] is an application that search through git repository for secrets, that can be left in source code by the developers.
`owasp-dependency-check` [9] is an `Software Composition Analysis` tool, during its execution application tries to find publicly disclosed vulnerabilities in the project libraries
`sonarqube` [10] this is Static Application Security Testing tool, purposed to detect bugs, vulnerabilities.
`owasp-zap` [11] serve as Dynamic Application Security Testing tool. It is used in order to test deployed application by execution some of the scripted tests


## VIII DevSecOps production grade pipeline 

At this stage we are ready to write our pipeline for `DevSecOps`. It is shown in the hidden part below.

:::spoiler
```java=
pipeline {


    agent {
        node {
            label 'Node1'
        }   
    }

    tools { 
        maven "M3"
    }

    stages {
        stage ('Initialize') {
            steps {
                sh '''
                        echo "PATH = ${PATH}"
                        echo "M2_HOME = ${M2_HOME}"
                    ''' 
            }
        }

        stage ('Git-Secrits-checking') {
            steps {
                sh '''
                    rm trufflehog || true
                    docker run gesellix/trufflehog --json https://github.com/Tealdris/jenkins-pipeline.git > trufflehog
                    cat trufflehog
                '''
            }
        }

        stage ('Sourse-composition-analysis') {
            steps {
                sh '''
                    chmod +x owasp-dependency-check.sh
                    bash owasp-dependency-check.sh
                    cat /home/ubuntu//OWASP-Dependency-Check/reports/dependency-check-report.xml
                '''
            }
        }
        
        stage ('SAST') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        mvn -e clean package sonar:sonar
                        cat target/sonar/report-task.txt
                    '''
                }
            }
        }

            stage ('Build Phase') {
                steps {
                    echo 'Build Phase'
                    sh '''
                        mvn clean package
                    '''
                }
            }

            stage ('Deploy-to-tomcat') {
                steps {
                    sshagent(['tomcat']) {
                        sh 'scp -o StrictHostKeyChecking=no /home/ubuntu/jenkins/workspace/Pipeline-job-1/target/*.war ubuntu@192.168.122.103:/opt/tomcat/webapps/webapp.war'
                    }
                }
            }

            stage ('DAST') {
                steps {
                    sshagent(['zap']) {
                                 sh 'ssh -o  StrictHostKeyChecking=no ubuntu@192.168.122.93 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.122.103:8080/webapp/" || true'
                    }
                }            
        }

    }        
}
```
:::

And if we image environment built for this papilene as a pictogram it will be look like on the picture below. As you can notice there are several steps added for the security checking as well as additional `ZAP` server for the `DAST` security checking.

<center>

![](https://i.imgur.com/LmblQlq.png)

</center>

After pipeline is writen and `jenkins` server is working, i am getting quite a similar results to the `DevOps` pipeline, but, of course, with the additioan output, corresponding to the devsecops tools that have been used. Results of the every stage of the devsecops pipeline is discussed below

### trufflehog - OSA/SCA

At the picture below you can see code, that were added to the `github` repository, with the purpose of the finding `secrets`. Those secrets are `api keys` and you shouldn't left things like this in your repository. 

<center>
    
![](https://i.imgur.com/iRZANp2.png)
    
</center>

Those left secrets, hopefully have been found by the `trufflehog`, you can see output of the `trufflehog` programs on the picture below.

<center>

![](https://i.imgur.com/6DRh8p7.png)
    
</center>

Unfortunately, some of the left secrets were skipped by the `OSA` solution. You can see them in the picture below.

<center>

![](https://i.imgur.com/cV6akfh.png)

</center>

But `trufflehog`, in order for this not being a case, can be configured in order to find specific credentials. To this end, you need to add the file to `/path/to/rules`

### owasp-dependency-check - OSA/SCA

At this stage, the `OSA/SCA` tool is checking dependencies, so to libraries that are used in the project. Of course, this tool can find something that have not been documented and included in the list of vulnerable libraries. That is why you need to keep data updated.
The important thing about `dependency-check` is that, it is checking dependencies only if they have been initialized, if not they can be missed

<center>
    
![](https://i.imgur.com/yvgmYtE.png)    
    
</center>

### sonarqube - SAST

`sonarqube` analyzes the code, that are located in the repository, on the vulnerabilities. in the order for it to work we have to start `sonareqube` server. Result of the analysis can be found on the `sonarqube` server. in my scenario, `sonarqube` suggests changing two particular lines of code. Because they can `impact on the security of application`.

<center>
    
![](https://i.imgur.com/3a7G1ZF.png)    
    
</center>

<center>
    
![](https://i.imgur.com/LJfqQby.png)  
![](https://i.imgur.com/LMttm78.png)    
    
</center>


### owasp-zap -DAST

`ZAP-owasp` server was configured on another server because it can be intense for the one machine to run the `Jenkins` pipeline and security testing at the same time. `ZAP` server is running inside the docker image.
As you can see from the pictures below `DAST` is trying to do a test on the application. Some of them give warning signs. 

<center>
    
![](https://i.imgur.com/BAUqKdT.png)
    
</center>

<center>
    
![](https://i.imgur.com/oQtwl3P.png)
    
</center>

Results are written in the last line of the report. In the end, our application is only getting warning signs.

<center>
    
![](https://i.imgur.com/lGKo8CB.png)
    
</center>
## Conclusion

During the investigation on the `DevSecOps` i have covered different `CI/CD`:`circleCI` `Jenkins` and `GitLab`; tools. As a result of my comparison `Jenkins` application have been chosen, due to open source, flexibility, and other reasons. `Jenkins` application was explained, what it consists of, and how to build different types of pipelines.
Before the `DevSecOps` pipeline investigation, some of the `security operations` tools were described. like `trufflehog`, `owasp-dependency-check` and the others
In the last section results of the built `DevSecOps` pipeline has been explained. What vulnerable parts of the application have been found.



## References

0. [myGit](https://github.com/Tealdris/jenkins_test_1)
1. [DevOps](https://en.wikipedia.org/wiki/DevOps)
2. [ Bass, Len; Weber, Ingo; Zhu, Liming (2015). DevOps: A Software Architect's Perspective. ISBN 978-0134049847.](http://revall.info/devops-a-software-architect-s-perspective.html)
3. [CircleCI vs Travis CI vs Jenkins](https://habr.com/ru/company/southbridge/blog/332836/)
4. [Jenkins VS GitLab](https://habr.com/ru/company/ruvds/blog/522334/)
5. [Jenkins Wiki](https://en.wikipedia.org/wiki/Jenkins_(software))
6. [Jenkins Documentation](https://www.jenkins.io/doc/)
7. [Fear and Loathing DevSecOps](https://habr.com/ru/company/oleg-bunin/blog/448488/)
8. [Trufflehog-github](https://github.com/trufflesecurity/truffleHog)
9. [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
10. [SonarQube Documentation](https://docs.sonarqube.org/latest/)
11. [owasp-zap documentation](https://www.zaproxy.org/docs/)

