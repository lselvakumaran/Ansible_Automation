# Ansible Jinja2 Template

## Step-1: What is Jinja2 template
-  **Jinja2:** Jinja2 templates are simple template files that store variables that can change from time to time. When Playbooks are executed, these variables get replaced by actual values defined in Ansible Playbooks. This way, templating offers an efficient and flexible solution to create or alter configuration file with ease.

## Step-2: Jinja2
 - **What is Template:** A template is a file that contains all your configuration parameters, but the dynamic values are given as variables in the Ansible. During the playbook execution, it depends on the conditions such as which cluster you are using, and the variables will be replaced with the relevant values.
 
 - **Ansible Use ?:** Ansible uses Jinja2 templating to enable dynamic expressions and access to variables. Ansible includes a lot of specialized filters and tests for templating. ... All templating happens on the Ansible controller before the task is sent and executed on the target machine.
 
 - **Jinja Format?:** A Jinja template is simply a text file. Jinja can generate any text-based format (HTML, XML, CSV, LaTeX, etc.).xml , or any other extension is just fine. A template contains variables and/or expressions, which get replaced with values when a template is rendered; and tags, which control the logic of the template.

## scenario 1: Jinja2 file with just copy module

- create file called hello_world.j2 [vim hello_world.j2].
```
{{ variable_to_be_replaced }}
This line won't be changed
Variable given as inline - {{ inline_variable }} - :)
```
- save this hello_world.j2 file

- create file called file.yml [vim file.yml].
```
- hosts: all
  vars:
    variable_to_be_replaced: 'Hello world'
    inline_variable: 'hello again'
  tasks:
    - name: Ansible Template Example
      copy:
        src: hello_world.j2
        dest: /var/tmp/hello_world.txt
```
- save this fie.yml file

- Execute the ansible playbook

```
ansible-playbook file.yml
```

- Verify the file from ansible client machines

```
ansible all -m command -a "cat /var/tmp/hello_world.txt"
```




## scenario 2: Jinja2 file with template module

- create file called hello_world.j2 [vim hello_world.j2].
```
{{ variable_to_be_replaced }}
This line won't be changed
Variable given as inline - {{ inline_variable }} - :)
```
- save this hello_world.j2 file

- create file called file.yml [vim file.yml].
```
- hosts: all
  vars:
    variable_to_be_replaced: 'Hello world'
    inline_variable: 'hello again'
  tasks:
    - name: Ansible Template Example
      template:
        src: hello_world.j2
        dest: /var/tmp/hello_world.txt
```
- save this fie.yml file

- Execute the ansible playbook

```
ansible-playbook file.yml
```

- Verify the file from ansible client machines

```
ansible all -m command -a "cat /var/tmp/hello_world.txt"
```

- **Now you see the differences right between just copy module and template module :) **


## Ansible template with_items for multiple files


## scenario 2: Multiple Jinja2 files with template module


- create file called ex.j2 [vim ex.j2].

```
{{ variable_to_be_replaced }}
This line won't be changed
Variable given as inline - {{ inline_variable }} - :)
```
- save this hello_world.j2 file

- create file called file.yml [vim file.yml].
```
- hosts: all
  vars:
    variable_to_be_replaced: 'Hello world'
    inline_variable: 'file1'
  tasks:
    - name: Ansible Template Example
      template:
        src: hello_world.j2
        dest: /var/tmp/hello_world.txt
```
- save this fie.yml file

- Execute the ansible playbook

```
ansible-playbook file.yml
```

- Verify the file from ansible client machines

```
ansible all -m command -a "cat /var/tmp/hello_world.txt"
```


- **Install AWS CLI V2.x**
   - Documentation Reference: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html
```
Mac OS (for all users)
$ curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
$ sudo installer -pkg AWSCLIV2.pkg -target /
$ which aws
/usr/local/bin/aws 
$ aws --version
aws-cli/2.0.6 Python/3.7.4 Darwin/18.7.0 botocore/2.0.0
```   

- **Install Docker CLI** 
   - Docker Desktop for MAC: https://docs.docker.com/docker-for-mac/install/
   - Docker Desktop for Windows: https://docs.docker.com/docker-for-windows/install/
   - Docker on Linux: https://docs.docker.com/install/linux/docker-ce/centos/

- **On AWS Console**
   - Create Authorization Token for admin user if not created
- **Configure AWS CLI with Authorization Token**
```
aws configure
AWS Access Key ID: ****
AWS Secret Access Key: ****
Default Region Name: ap-south-1
```   

## Step-4: Create ECR Repository
- Create simple ECR repo via AWS Console 
- Repository Name: aws-ecr-nginx
- Explore ECR console. 
- **Create ECR Repository using AWS CLI**
```
aws ecr create-repository --repository-name aws-ecr-nginx --region us-east-1

```

## Step-5: Create Docker Image locally
```
wget https://raw.githubusercontent.com/stacksimplify/aws-fargate-ecs-masterclass/master/04-ECR-Elastic-Container-Registry/index.html
wget https://raw.githubusercontent.com/stacksimplify/aws-fargate-ecs-masterclass/master/04-ECR-Elastic-Container-Registry/Dockerfile
```

```
docker build -t 456774515540.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-nginx:1.0.0 . 
docker run --name aws-ecr-nginx -p 80:80 --rm -d 456774515540.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-nginx:1.0.0
```


## Step-6: Push Docker Image to AWS ECR
- Push the docker image to ECR
- **AWS CLI Version 1.x**
```
AWS CLI Version 1.x
aws ecr get-login --no-include-email --region <your-region>
aws ecr get-login --no-include-email --region us-east-1
Use "docker login" command from previous command output
docker push 456774515540.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-nginx:1.0.0
```
- **AWS CLI Version 2.x**
```
AWS CLI Version 2.x
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <your-ecr-repo-url>

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 456774515540.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-nginx

docker push 456774515540.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-nginx:1.0.0
```


## Step-7: Using ECR Image with Amazon ECS
- Create Task Definition: aws-ecr-nginx
   - Container Image: 456774515540.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-nginx:1.0.0
   - Container Port - 80
- Create Service: aws-ecr-nginx-svc
- Test it


## Now delete all images and containers (Stop and Delete)


## Step-8: To stop any running container
```
docker stop $(docker ps -q)  
```

## Step-9: To remove all the containers

```
docker rm $(docker ps -a -q)
```

## Step-10: To remove all docker images in a single command
```
docker rmi $(docker images -a -q)
```

## Step-11: Now you have clean environment

```
docker pull cloudnloud/nginxapp2:latest

docker run -dit --name cloudnloud-web -p 80:80 cloudnloud/nginxapp2:latest
```
