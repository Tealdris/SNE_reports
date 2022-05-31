###### tags: `LS`
:::success
# LS Lab 3 Assignment: Infrastructure as Code (IaC)
**Name: Radomskiy Ilia**
:::

---

## Task 1: IaC Theory
:::info
Briefly answer for the following questions what is and for what:
:::

**git** repository:
* /.git - it is a software purposed for tracking changes in your working project. Usually used for the collaborative development of products.
* /.github - it is a website that is being used for storing git-repositories
* .gitignore - set of rules. That rules are specified in order to ignore some of the files in the local repository because they are not useful for the others. `Log` files for example
* .gitmodules - file located on the top level of repository, and it is used in order to set requirements of the application. LIke a symbolic link to any other resource, may be another repository

[git documentation](https://git-scm.com/doc)

**ansible** directory:
* ansible.cfg - main configuration file, used in order to set some parameters in it, in order to keep everything clean and understandable this file include inside other files (links). 
* inventory folder - folder with the files that contain information about machines included in the ansible network
* roles folder - ansible is a complicated and powerful instrument, that it is a good practice to keep everything as separate as possible
    * tasks - main command
    * defaults - contain variables, but those variables have small priority, that is why they can be overwritten by the other variables
    * files - it is used when we need to use additional files in our ansible project
    * handlers - may be described as some kind of script that can be used, so instead of writing the same 3 line command all over again you can replace it with the 1 like a link to the handler
    * templates - may store any example or sample that may be used in the project
* playbooks folder - contains sequential instruction sets for the workers nodes to execute

[Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)

**terraform** folder:
* main.tf - master file in the terraform folder, instructions written there are used to deploy machines
* variables.tf - they are often used as input variables. The file allows you to set some of the variables in order to execute `main.tf` file
* outputs.tf - data that is stored by terraforming in order to use later, may be another terraform script

[Terraform documentation](https://www.terraform.io/docs)

---

## Task 2: Choose your application/microservices

:::info
The base level (it means that this task will be evaluated very meticulously): find (much better to write) a simple application with **REST** interface. For example, it could be a web server that returns `"Hello to SNE family! Time is: < current time in Innopolis >"`. Use whatever programming language that you want (python, golang, C#, java...).
**Semi-Bonus:** prepare microservices instead of a standalone application, e.g. full-stack web application with a web server, database...
:::

For this task, I will choose this [Aplication](https://github.com/Tealdris/webnotes), located in my repository. This application is a web note page, that allows the users to leave notes to it. The application consists of the `Postgres` database, `Nginx` server, and `API` written in `python` with the addition of the `fast API` module. All described modules are separated from each other in the containers with the help of `docker-compose`. This microservice architecture will be deployed with the help of the `docker swarm` orchestration technique on the two `AWS` cloud machines with the different os on it: `ubuntu` and `amazon Linux` (analog of RedHat).
In order to deliver services to the server, I will use `Terraform` and `ansible`. `Terraform` will be used in order to create AWS instance machines. And ansible will be used to configure them: set up parameters, install required applications, services, etc. All the configuration files are located in my [Git repository](https://github.com/Tealdris/ls3_infr_as_code.git).


---

## Task 3: Dockerize your application

:::info
1. Build **Docker** image for your application (make Dockerfile). 
2. Look for the best **Docker** practices, try to follow them and put into report.
**Bonus:** use **docker-compose** in the case of microservices.
:::

To make containers we need to write `docker file`. This file describes the docker-container environment, inside which will be running my services.

```dockerfile=
#dockerfile python API
FROM python:3.7-slim # take configuration of this container
WORKDIR /usr/src/app # set working directory
COPY . . # copy file from the local directory
RUN pip install -r requirements.txt # install requirements of application
```

```dockerfile=
#
FROM nginx:1.18-alpine # take configuration of this container
COPY nginx.conf /etc/nginx/nginx.conf # copy this nginx configuration file
```
and official docker hub container `postgres:13-alpine`

In order to build containers locally, I performed this command `docker build -t tealdris/api:ls3 .`. After that, they can be pushed to the `docker hub` with `docker push tealdris/api:ls3`

Because the application is built with the microservice architecture we need to write a `docker-composr.yml` file

This file contains the following general structure of used settings 

```dockerfile=
version '3'
services:
    service name:
        image: of container db\api\nginx
        enviroment_file: of database
        volumes: for database or server
        build: con be performed for the container
        command: uvicorn API_Server:app --host api --port 80
        depends_on: other services
        ports: for the services
```

---

## Task 4: Deliver your app using Software Configuration Management


:::info
1. Get your personal cloud account. To avoid some payment troubles, take a look on free tier that e.g. **AWS** or GCP offers for you (this account type should be fit for a lab).
:::

I will use `AWS` for this task because I have already been using it for other projects.

<center>

![](https://i.imgur.com/9LEksgy.png)

</center>

In order for the next step to be possible, I need to configure access tokens for the terraform. It can be done in this, `IAM` `AWS` section.

<center>

![](https://i.imgur.com/a95QCca.png)

</center>

It is better not to use those in any file, because of leakage possibility. But you can store them as variables.
```bash=
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
```

From now on i will be configuring `AWS` instances through `terraform`

:::info
2. Use **Terraform** to deploy your required cloud instance.
Look for the best **Terraform** practices, try to follow them and put into report.
**Bonus:** use **Packer** to deploy your cloud image/docker container.
:::

Terraform file for my case contain every configuration setting that will be required in the project. I will explain terraform file below

```yaml=
provider "aws" {
    region = "us-east-2" # set aws region
}
variable "key_name" {} #variable for the key
resource "tls_private_key" "pk" { # generate key
  algorithm = "RSA"
  rsa_bits  = 4096
}
resource "aws_key_pair" "kp" { #use key to store in in file and ssh connection to servers
  key_name   = var.key_name
  public_key = tls_private_key.pk.public_key_openssh
  provisioner "local-exec" { # Create "myKey.pem" to your computer!!
    command = "echo '${tls_private_key.pk.private_key_pem}' > ./myKey.pem"
  }
}
resource "aws_instance" "back-lx" { # set up instance RedHat
    ami = "ami-0231217be14a6f3ba" # number-code in AWS
    instance_type = "t2.micro"
    vpc_security_group_ids = [aws_security_group.webserver.id] # add to security group
    key_name = aws_key_pair.kp.key_name # add ssh key
    provisioner "local-exec" {
    command = "echo ${aws_instance.back-lx.public_ip} >> ip.txt" # execute this command, send ip address to local file
    }
}
resource "aws_instance" "front-ub" { # set up instance Ubuntu
    ami = "ami-0fb653ca2d3203ac1"
    instance_type = "t2.micro"
    vpc_security_group_ids = [aws_security_group.webserver.id]
    key_name = aws_key_pair.kp.key_name
    provisioner "local-exec" {
    command = "echo ${aws_instance.front-ub.public_ip} >> ip.txt"
    }
}
resource "aws_security_group" "webserver" { # set security group
  name        = "webserver-security-group" # set security name
  description = "Allow all inbound traffic" 
  ingress { # input rule for 80 port
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ...
  egress { # output rule for any port
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    cidr_blocks     = ["0.0.0.0/0"]
  }
}
```

Before executing the rule we need to prepare our folder to communicate with `terraform init` or `terraform init -upgrade`, it is done after `.tf` file creation because some of commands may require additional configuration
After that, we may check what changes we will apply to machines with `terraform plan`
In order to send changes to `AWS` `terraform apply` can be performed
Last, in order to terminate all machines, you may use `terraform destroy`

`Terraform` also allow us to increase its usability  with the ability to: include files (user_data =) or scripts, `tagging` instances, and debugging with `terraform console`

After running this scrip two new instances are created in the specific security group. They are accessible via ssh with a key.

<center>

![](https://i.imgur.com/9LEksgy.png)

</center>

**Packer**: But those instances are clean. In case you have created a huge amount OS empty instances it will be inconvenient to send commands to every machine to install something, docker for example.
If we want to make the process easier we might use `packer`
`Packer` is used to create images in a cloud provider with the installed requisites in it. In the picture below you can see three images, two of them created by `packer`.

<center>

![](https://i.imgur.com/bU5nSkO.png)

</center>

The first one is empty, but it is not very useful. It was created with the following command

```json=
{
    "variables": {
      "aws_access_key": "",
      "aws_secret_key": ""
    },
    "builders": [
      {
        "type": "amazon-ebs", # provider
        "access_key": "{{user `aws_access_key`}}",
        "secret_key": "{{user `aws_secret_key`}}",
        "region": "us-east-2",
        "source_ami_filter": {
          "filters": {
            "virtualization-type": "hvm",
            "name": "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*", # instance name
            "root-device-type": "ebs" # used by amazon
          },
          "owners": ["099720109477"],
          "most_recent": true
        },
        "instance_type": "t2.micro",
        "ssh_username": "ubuntu",
        "ami_name": "packer-example {{timestamp}}" # tag name of instance
      }
    ],
```

The second image was created with the installed docker in it. Installation of docker is simple and can be found in the official documentation. The most important part of the script below it is shell initialization.

```json=
    "provisioners": [{
      "type": "shell", # start shell command on machine
      "inline": [
          "sleep 30",
          "sudo apt-get update",
          "sudo apt install -y  apt-transport-https ca-certificates curl software-properties-common",
          "wget https://download.docker.com/linux/ubuntu/gpg -O gpg",
          "sudo apt-key add gpg",
          "sudo add-apt-repository \"deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable\"",
          "rm gpg",
          "sudo apt update",
          "sudo apt install -y docker-ce",
          "sudo curl -L \"https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)\" -o /usr/local/bin/docker-compose",
          "sudo chmod +x /usr/local/bin/docker-compose"
      ]
  }]
  }
```

In the picture below you can see the console output of the packer installation image. After image creation, we can use `AMI` name of it in our infrastructure

<center>
    
![](https://i.imgur.com/hY0w7O9.png)
    
![](https://i.imgur.com/KkQklW0.png)

</center>



:::info
3. Choose SCM tool. Suggestions:
* **Ansible** (default choice)
* SaltStack
* Puppet
* Chef
* etc
:::

For the `ansible` it is a little bit complicated because `ansible` is used in order to the configuration of machines after initialization, so the variety on configuration files is bigger.
My `ansible` folder looks like this.

<center>
    
![](https://i.imgur.com/y2cYDZ5.png)

</center>

`Group` vars contains information about machines.

```
ansible_user: ec2-user # radhat user
ansible_ssh_private_key_file: /home/st10/Documents/git/ls3_infr_as_code/terraform/myKey.pem # ssh key
```

`Playbooks` folder contains information about plays, that can be performed. Below you can see a simple playbook that allows checking connection to the servers

```yaml=
- name: Test connection on server
  hosts: all
  become: yes
  tasks: 
  - name: Check and print linux dist
    debug: var=ansible_os_family   
  - block: # UBUNTU block
    - name: Ping ub
      remote_user: ubuntu
      ping:
    when: ansible_os_family == "Debian"
  - block: # ReedHat block
    - name: Ping rh
      remote_user: ec2-user
      ping:
    when: ansible_os_family == "RedHat"
```
The out put of this playbook is shown below. As you can see it is successful

<center>

![](https://i.imgur.com/3jpN2ui.png)

</center>

other playbooks are bigger and will be explained when it is needed
 
`Ansible.cft` contains itself, in my case

```yaml=
[defaults]
host_key_checking = false # ask about fingerprint befor ssh connection
inventory = ./hosts.txt # host file of the configurable machines
```
 
The last file is `hosts.txt`. this file provides information about instances and location of the key 


:::info
4. Using SCM tool, write a playbook tasks to deliver your application to cloud instance. 
Try to separate your configuration files into inventory/roles/playbooks files. In real practice, we almost newer use poor playbooks where everything all in one.
Also try to use the best practices for you SCM tool and put them into report.
Use **Molecula** and **Ansible Lint** to test your application before to deliver to cloud (or their quivalents)
:::

Before delivery itself, we might need to check our files, with `ansible lint`, for example 
In the picture below you can see the result of the checking two playbooks.
1 one is a simple ping playbook, and there are no mistakes
second is the purpose to deliver docker to the AWS instances. And `ansible lint` marks some of the week spots. For example, in this case, an ansible link marked every `command` line with the `bash` command in it. That means it is not a `true` way to set your ansible playbooks. And you probably should.

<center>

![](https://i.imgur.com/n6KSs7y.png)

</center>

Delivery of applications can be separated into stages.

1. Docker installation | Playbook `install_docker2.yml`
At this stage, if our instances are clean (before packer), that is why we might need to install `docker` on them
Playbook for this stage can be divided into two parts, installation for the `Ubuntu` and for the `RedHat`. Most of the time `name` of the `task` speaks for itself. But long story short, the whole book can be listed like this

* installation docker from source on ubuntu
    * install packages for docker
    * install gpg key
    * Verify key
    * set repository
    * update repository
    * install docker
    * set permissions for `ubuntu` user
    * install docker-compose
    * install pip
    * install docker-compose from pip
* installation docker from the repository on RedHat
    * install pip
    * install docker
    * install docker-compose
    * install docker-compose from pip
    * start docker & enable it to work on boot
    * install git (not installed by default)

In the picture below you can see the result of the installation

<center>

![](https://i.imgur.com/B8KDdqO.png)

</center>

2. Docker swarm initialization | Playbook `install_d_swarm.yml`
Now we want to start manager-worker relationships between two machines in order to communicate in a swarm. That is why we can list steps like this

* initialization docker swarm- manager on ubuntu
    * set a swarm
    * get token for the worker to join
    * set token as a variable
* initialization docker swarm- manager on RedHat
    * set a worker in swarm 

3. Running the application in the swarm | Playbook `run_d_swarm.yml`

At this stage, we need to clone repositories on the machines and run `docker stack deploy --compose-file docker-compose.yml ls3` on the worker node


After that, we can see running service on the cloud machine

<center>
    
![](https://i.imgur.com/53WJEPb.png)
    
![](https://i.imgur.com/bteaSyO.png)
    
![](https://i.imgur.com/MtjoxR6.png)
    
![](https://i.imgur.com/PRPC6gZ.png)

![](https://i.imgur.com/v6r299S.png)

</center>
    
---

## Task 5: - Teamwork with the version control system (optional)


:::info
You can do this task if you have a different projects with your cooperation team and you want to be more familiar with git.
In production, before deploying the version of the service/application, we receive a review from colleagues.
**Bonus:** implement steps 2 and 4 via **Terraform**.
:::

:::info
1. Create and log in to your personal version control system account on the git engine:
* **github** (choice)
* gitlab
* bitbucket
* etc
:::

My [Github](https://github.com/Tealdris) repository. As a part of this task we, in the team, will create a repository with `Terraform`, share administrator access in it, and, also work with the credentials

:::info
2. Create a repository with your application/microservices and all required scripts/configs. With `terraform`
:::

In this part  `terraform.tf` file was used in order to configure the repository, permissions in it, collaborators and branches
For example in the section below you can see part of the terraform file. It is used in order to create a repository.

```json=
resource "github_repository" "default" {
 name        = "example-repo1"
 description = "My ls project"
 visibility  = "public"
 auto_init   = true
}
```

<center>

![](https://i.imgur.com/pC2YqfS.png)

</center>

:::info
3. Synchronize your local and remote repository.
:::

In order to synchronize the local repository and the repository in GitHub we need to perform the following steps

```bash=
git clone repository
#create files
git add files
git commit -m "" # for files
git push # files (this way in main if not specifiend othervise)
```


:::info
4. Create a separate branch for development, as well as protect your master branch in the repository from direct commits to it.
:::

Part below is used in order to create branch `development` in the repository through terraform

```
resource "github_branch" "development" {
 repository = github_repository.default.name
 branch     = "development"
}
```

Or it can be done via website

<center>

![](https://i.imgur.com/eM7rqPI.png)

</center>

But if we want to set permissions on the main repository we can do it in `terraform` to

```json=
resource "github_branch_protection" "default" {
  repository_id  = github_repository.default.name
  pattern          = "main"
  enforce_admins   = true
   required_pull_request_reviews {
    dismiss_stale_reviews  = true
    restrict_dismissals    = true
   }
 }
```

After all configuration, we cant push to the main repository 
In the picture below I am trying to push into Ivan repository.

<center>

![](https://i.imgur.com/00lcqLq.png)

</center>

On the picture below Vladimir is not able to push into main repository

<center>
    
![](https://i.imgur.com/K3H2auT.jpg)

</center>
    
:::info
5. Create a Pull Request from your developer branch to the master branch.
:::

On the picture below Ivan allows my commit to the main brunch

<center>

![](https://i.imgur.com/PJvjEsM.png)

</center>

:::info
6. Your colleague should get explanation about your work and conduct a review of your PR.
:::

If i want to review some kind of project it will look like this

<center>

![](https://i.imgur.com/In04Fza.png)

</center>

:::info
7. Receive an approvement, merge your PR and synchronize the local and remote repositories
:::

After reciveng commits you  can aprove them to your main branch

<center>

![](https://i.imgur.com/T6yABow.png)

</center>

---

## Task 6: Play with SCM cases


:::info
The goal is to easily, reliably and quickly maintain different kinds of systems at once. Prepare two different guest systems e.g. an Ubuntu Server and Fedora. Then play with some Software Configuration Management (SCM) to automate system preparations on those. You might create and provide your own ideas or follow to some suggestions:
* NTP
* default shell
* Apache Web Server
* Docker
* Nginx & PHP-FPM & MariaDB & WordPress
* Ansible Vault (definitely good choice)
* IPtables (close all machine ports exclude ssh, http, https)
* development environments organization (ssh keys management...)
* etc
Never forget to try to use the best practices. Complete as many tasks as possible (at least two cases for a team of two people, one case for an individual work).
:::

The difference during the installation of the different OS is shown below.
In the example below there is the installation of the same packet but for a different OS. The difference in commands is very small only `user` and `packet manager`

```yaml=
    - name: RH Install python3-pip
      remote_user: ec2-user $
      yum: $
        name: python3-pip
        state: present
        update_cache: yes


    - name: UB Install python3-pip
      remote_user: ubuntu $
      apt: $
        name: python3-pip
        state: present
        update_cache: yes
```

but sometimes it is harder to make similar configuration files
For example, during my docker on Redhat, I was not able to install docker from the docker repository. But I manage to install it from the repository of RedHat.  At this time packet is different.


```yaml=
    - name: UB Install docker
      remote_user: ubuntu
      apt:
        name: docker-ce
        state: present
        update_cache: yes
        
    - name: RH docker
      remote_user: ec2-user 
      yum: $
        name: docker
        state: present
        update_cache: yes
```
      
In general, I didn't understand the part with `playing with SCM cases`
But in my lab, there are several parts that cover some of the topics
default shell may be used by `terraform` in the `user data` section, the sh script can be stored there.
Also, shell were used in the ansible and packer sections because sometimes it is easier to just print command instead of a particular program structured block of code
The application was dockerized and separated into the different services
I also have a playbook with the installation of the apache web server, I have made it when I want to learn ansible.
I have already covered it, but my terraform file has part with the managing of the ports and ssh keys. And I don't know will this count as paying



---

:::spoiler
### Terraform Best Practices

- Make understandable naming
- Don’t commit the .tfstate file
- Back up state files
- Manipulate state only through commands
- Use variables
- Use validate and plan commands frequently
- Abstract code to maximize reuse
- Use modules, where it is necessary


### Docker Best Practices

- Do not use :latest
- Create ephemeral containers
- Exclude with .dockerignore
- Minimize the number of layers
- Build the smallest image possible
- Properly tag your images
- Don't be root
:::

:::spoiler
worldpress db pass
ilyaradomskiy@bk.ru


sudo apt install python3-pip
python3-pip install flask
mkdir microblog
cd microblog
apt install python3.8-venv
python3 -m venv venv
source venv/bin/activate
#inside venv
pip install flask
mkdir app
:::