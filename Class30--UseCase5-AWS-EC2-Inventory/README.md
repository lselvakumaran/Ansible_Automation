# Collect EC2 instances inventory

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
- name: ec2 instances information
  hosts: localhost
  tasks:
   - name: collect ec2 information
     ec2_instance_facts:
      aws_access_key: AKIAWUWPB7NKFOP6WYJS
      aws_secret_key: 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
      region: "us-east-1"
      filters:
       instance-state-name: running
     register: ec2_facts

   - name: metadata
     debug: 
      var: ec2_facts
```
- save the file

- go to your AWS Cloud account (aws.amazon.om) and make sure you created some ec2 instances.Make sure ec2 instances are running.

- To execute the file.

```
ansible-playbook ec2.yml
```

