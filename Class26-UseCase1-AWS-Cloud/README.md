# How To Create EC2 Instances Using Ansible

## 1: AWS Automation
-  **Automation:** you will learn to launch an EC2 instance using Ansible from the local machine. Before starting, you can understand Ansible as a radically simple IT automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and for many other IT needs.
So if you are using Ansible to launch EC2 instance you can set this up with CI/CD, dynamic creation on the instance. There are many use cases you can implement using Ansible.
So let’s get started.

For working on Ansible we need to first set up a few things,

- AWS user account
- Ansible
- Python
- Boto

## 4: Prerequisites

1.	**server.cnl.com** – 1 CPU – 1GB RAM (Python 2.7) - **Ansible Server**
2.	**node1.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 1**
3.	**node2.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 2**

from ansible server login as an ansible user as per class 4.From ansible user execute below command

ansible all -m ping

this above ping command should return with ping / pong green color.


## steps 1: Create EC2 Instance using Playbook

```
ssh-copy-id ansible@localhost
```

```
ssh ansible@localhost
```

- the above command shouldnt ask any password.


```
sudo yum install awscli python2-boto python2-botocore python-boto3
```

```
aws configure
```



## scenario 1: Create EC2 Instance using Playbook


```
vim ec2.yml
```

- create normal playbook

```
---
- name: Create a new Demo EC2 instance
  hosts: localhost
  gather_facts: False

  vars:
      region: us-east-1
      instance_type: t2.micro
      ami: ami-02e136e904f3da870  # Amazon Linux
      keypair: cnl_key # pem file name
      key: XXXXXXXXXXXXXXXXXXXXXXXXXX
      secret: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

  tasks:
    - name: create keypair
      ec2_key:
       name: "{{ keypair }}"
       aws_access_key: "{{ key }}"
       aws_secret_key: "{{ secret }}"
       region: "{{ region }}"
    - name: Create an ec2 instance
      ec2:
         aws_access_key: "{{ key }}"
         aws_secret_key: "{{ secret }}"
         key_name: "{{ keypair }}"
         #group: einsteinish  # security group name
         instance_type: "{{ instance_type}}"
         image: "{{ ami }}"
         wait: true
         region: "{{ region }}"
         count: 1  # default
         count_tag:
            Name: vijay
         instance_tags:
            Name: vijay
         vpc_subnet_id: subnet-953cb79b
         assign_public_ip: yes
```
- save the file

- To execute the file.

```
ansible-playbook ec2.yml
```

