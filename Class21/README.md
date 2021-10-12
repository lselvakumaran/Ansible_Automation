# Ansible Roles

## 1: What is role
-  **Role:** Ansible role is a set of tasks to configure a host to serve a certain purpose like configuring a service. Roles are defined using YAML files with a predefined directory structure. A role directory structure contains directories: defaults, vars, tasks, files, templates, meta, handlers.

## 2: use of Ansible roles?
 - **What is the use of Ansible roles??:** Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. After you group your content in roles, you can easily reuse them and share them with other users.
 
 - **What is the default behavior when an ansible task fails?:** When Ansible receives a non-zero return code from a command or a failure from a module, by default it stops executing on that host and continues on other hosts. However, in some circumstances you may want different behavior. Sometimes a non-zero return code indicates success.
 
## 3: What is the difference between roles and playbook in Ansible?
- ** Role is a set of tasks and additional files to configure host to serve for a certain role. Playbook is a mapping between hosts and roles **.

## 4: How do you write an Ansible role?
- ** To create a Ansible roles, use ansible-galaxy command which has the templates to create it. This will create it under the default directory /etc/ansible/roles and do the modifications else we need to create each directories and files manually. where, ansible-glaxy is the command to create the roles using the templates **.

## 5: What is role and task in Ansible?

- Roles provide a framework for fully independent, or interdependent collections of variables, tasks, files, templates, and modules. In Ansible, the role is the primary mechanism for breaking a playbook into multiple files. This simplifies writing complex playbooks, and it makes them easier to reuse.

## 6: What is the file structure of Ansible roles?
- Roles are defined using YAML files with a predefined directory structure. A role directory structure contains directories: defaults, vars, tasks, files, templates, meta, handlers. Each directory must contain a main. yml file which contains relevant content.

## 7:  How do I install Ansible roles?
**Installing roles **
- Set the environment variable ANSIBLE_ROLES_PATH in your session.
- Use the --roles-path option for the ansible-galaxy command.
- Define roles_path in an ansible. cfg file.

## 7:  Folder Structure ?
**Ansible roles Directory Structure**
https://github.com/cloudnloud/Ansible_Automation/blob/main/Class21/Ansible-Roles-Structure.png

To start this handson lab,you need following resources.



## 4: Prerequisites

1.	**server.cnl.com** – 1 CPU – 1GB RAM (Python 2.7) - **Ansible Server**
2.	**node1.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 1**
3.	**node2.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 2**

from ansible server login as an ansible user as per class 4.From ansible user execute below command

ansible all -m ping

this above ping command should return with ping / pong green color.



## scenario 1: How to create ansible role folder structure

- create file called tasks.yml [vim tasks.yml].
```
ansible-galaxy init test-role-1
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```

## scenario 2: Ignore error in entire playbook

- create file called tasks.yml [vim tasks.yml].
```
---
- name: Repeated tasks can be written as standard loops
  hosts: all
  become: true
  ignore_errors: yes
  tasks:
  - name: this will not be counted as a failure 1
    command: /bin/false
  - name: this will not be counted as a failure 2
    command: /bin/false
  - name: this will not be counted as a failure 3
    debug: 
     msg: "i am now able to clear all the interview on my own" 	
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```

## scenario 3: changed_when Statement

- Example 1:  Start the HTTPD (or) Apache Server which is already started
- Having said that, Let’s start with our trialWe are going to start the HTTPD (or) Apache Server which is already running. 
- Ideally, If it is already running it should not report as changed


- Step 1: Direct register and see how register output in json format

```
--- 
- hosts: all
  ignore_errors: yes
  tasks:
    - name: "Start the Apache HTTPD Server"
      shell: "httpd -k start"
      register: starthttpdout     

    - debug:
        var: starthttpdout.stdout
```		

```
ansible all -m shell -a "ps -eaf|grep -i httpd"
```

- Step 2: Modified Playbook with Debug Enabled.To know the truth, you should modify the playbook with a debug and register as follows

```
--- 
- hosts: all
  ignore_errors: yes
  tasks:
    - name: "Start the Apache HTTPD Server"
      shell: "httpd -k start"
      register: starthttpdout     

    - debug:
        var: starthttpdout
```	


- Step 3: Modified Playbook with Debug Enabled.To know the truth, you should modify the playbook with a debug and register as follows

```
--- 
- hosts: all
  ignore_errors: yes
  tasks:
    - name: "Start the Apache HTTPD Server"
      shell: "httpd -k start"
      register: starthttpdout     

    - debug:
        var: starthttpdout.stdout
```		

```
ansible all -m shell -a "ps -eaf|grep -i httpd"
```

- Step 4: Here comes the changed_when to explicitly tell ansible when to consider the task as Successful (or) changed

- Let us re modify our same Playbook with changed_when

```
ansible node2 -m shell -a "httpd -k stop"
```


```
--- 
- hosts: all
  tasks:
  - name: "Start the Apache HTTPD Server"
    shell: "httpd -k start"
    register: starthttpdout
    changed_when: "'already running' not in starthttpdout.stdout"

  - debug:
      msg: "{{starthttpdout.stdout}}"
```  

```
ansible all -m shell -a "ps -eaf|grep -i httpd"
```


## scenario 4: failed_when Statement

- System Requirement / Prerequisite check before Installation
This one is a more real-time scenario every one of us might have come across, During the installation of software, in midway, the installation wizard will fail stating that there is no enough memory (or) the minimum system requirements to install that specific software is not met

- As we all know, Every Software needs some minimum system requirements. When they are not met, The installation would fail.

- In our case, We are going to take weblogic application server installation as an example.

- As per oracle recommendation for weblogic 12c to function properly and for hassle-free installation, The system must meet the following requirement

- 2 GB of Physical Memory ( RAM)
- Minimum 4 GB of Disk space in Domain Directory /opt
- Minimum 1 GB of disk space in /tmp directory
- Now we are going to perform a quick pre-requisite check using ansible failed_when and determine whether the system requirement specified above are met

- Consider the following playbook


```
---
- hosts: all
  tasks:
  - name: Making sure the /tmp has more than 1gb
    shell: "df -h /tmp|grep -v Filesystem|awk '{print $4}'|cut -d G -f1"
    register: tmpspace
    failed_when: "tmpspace.stdout|float < 1"

  - name: Making sure the /opt has more than 4gb
    shell: "df -h /opt|grep -v Filesystem|awk '{print $4}'|cut -d G -f1"
    register: optspace
    failed_when: "optspace.stdout|float < 4"

  - name: Making sure the Physical Memory more than 2gb
    shell: "cat /proc/meminfo|grep -i memtotal|awk '{print $2/1024/1024}'"
    register: memory
    failed_when: "memory.stdout|float < 2"
```  

- if you dont understand the above playbook.Try to play like below.....

```
---
- hosts: all
  tasks:
  - name: Making sure the /tmp has more than 1gb
    shell: "df -h /tmp|grep -v Filesystem|awk '{print $4}'|cut -d G -f1"
    register: tmpspace
    failed_when: "tmpspace.stdout|float < 1"
  - name: print jason output
    debug:
     var: tmpspace
  - name: print /tmp file system has enough free space
    debug:
     msg: "/tmp flesystem is having enough free space"
    when: tmpspace.stdout > 15

  - name: Making sure the /opt has more than 4gb
    shell: "df -h /opt|grep -v Filesystem|awk '{print $4}'|cut -d G -f1"
    register: optspace
    failed_when: "optspace.stdout|float < 4"
  - name: print /opt file system has enough free space
    debug:
     msg: "/opt flesystem is having enough free space"
    when: optspace.stdout > 15

  - name: Making sure the Physical Memory more than 2gb
    shell: "cat /proc/meminfo|grep -i memtotal|awk '{print $2/1024/1024}'"
    register: memory
    failed_when: "memory.stdout|float < 2"
```  
