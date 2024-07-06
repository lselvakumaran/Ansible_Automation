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

![ScreenShot](https://github.com/cloudnloud/Ansible_Automation/blob/main/Class21/Ansible-Roles-Structure.png)

To start this handson lab,you need following resources.



## 4: Prerequisites

1.	**server.cnl.com** – 1 CPU – 1GB RAM (Python 2.7) - **Ansible Server**
2.	**node1.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 1**
3.	**node2.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 2**

from ansible server login as an ansible user as per class 4.From ansible user execute below command

ansible all -m ping

this above ping command should return with ping / pong green color.

## scenario 1: How to monitor many urls ?

- create roles dirdctory
```
vim urlmonitor.yml
```

- create sample role and learn how directory structure are ?.
```
---
- name: check the url's reachable and working
  hosts: all
  tasks:
   - name: check url1
     uri:
      url: http://www.google.com/
     register: googleurl
   - name: print json output
     debug:
      var: googleurl
   - name: print google url1 is working
     debug:
      msg: google url1 is reahable and working
     when: googleurl.status == 200
   - name: check url2
     uri:
      url: http://www.microsoft.com/
     register: microsofturl
   - name: print json output
     debug:
      var: microsofturl
   - name: print microsoft url2 is working
     debug:
      msg: microsoft url2 is reahable and working
     when: microsofturl.status == 200
```

- execute the playbook

```
ansible-playbook urlmonitor.yml
```


## scenario 2: How to create ansible role folder structure

- create roles dirdctory
```
mkdir roles
cd roles
```

- create sample role and learn how directory structure are ?.
```
ansible-galaxy init singleurl
```

- Type below command to see how the folder structure looks like ?.
- make sure you are in roles directory.

```
tree
```

- Declare your roles path in your environnment related ansible.cfg file.


```
vim ansible.cfg
[defaults]
inventory=hosts
remote_user=ansible
timeout=3000
roles_path=/home/ansible/dev/roles

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False
```

## scenario 3: single url test

![ScreenShot](https://github.com/cloudnloud/Ansible_Automation/blob/main/Class21/multi-url.JPG)

- single url is just dummy skeleton

- goto singleurl folder.
- we are going to test http://www.google.com/ url is rechable and working or not ?.

- go to defaults folder and set the variable

```
cd singleurl
cd defaults
vim roles/singleurl/defaults/main.yml 
---
url: http://www.google.com/

```
Now go to tasks folder and enter what you want to test and print.

```
cd singleurl
cd tasks
vim roles/singleurl/tasks/main.yml 
---
- name: check url is reachable or not
  uri:
   url: "{{ url }}"
- name: print the {{ url }} is working
  debug:
   msg: "{{ url }} is working and reachble"

```

- come out of roles folder.

- create new yaml playbook

```
vim /home/ansible/dev/singleurltesting.yml
---
- name: test the url reachablity
  hosts: all
  roles:
   - singleurl
```

## scenario 4: Now perform the same above test and need to test 100's of urls

![ScreenShot](https://github.com/cloudnloud/Ansible_Automation/blob/main/Class21/multi-url.JPG)

- multiurl is just actual where you need to mention all the urls you want to monitor.

- goto multiurl folder.
- we are going to test many url's are rechable and working or not ?.

- go to meta folder and list all the urls which you want to monitor

```
cd roles
mkdir multiurl
cd multiurl
mkdir meta
cd meta
vim /home/ansible/dev/roles/multiurl/meta/main.yml 

in dependencies section remove []

then add the below lines

dependencies: 
 - { role: singleurl, url: "http://cloudnloud.com/"}
 - { role: singleurl, url: "http://youtube.com/"}
 - { role: singleurl, url: "http://www.tcs.com/"}
 - { role: singleurl, url: "http://www.infosys.com/"}
 - { role: singleurl, url: "http://www.wipro.com/"}
 - { role: singleurl, url: "http://www.irctc.com/"}
 - { role: singleurl, url: "http://www.redbus.com/"}
 - { role: singleurl, url: "http://www.emirates.com/"}
 - { role: singleurl, url: "http://www.skyscanner.com/"}
 - { role: singleurl, url: "http://www.redhat.com/"}
 - { role: singleurl, url: "http://www.airfrance.com/"}

```
Now go to tasks folder and enter what you want to test and print.

```
cd multiurl
mkdir tasks
cd tasks
vim /home/ansible/dev/roles/multiurl/tasks/main.yml 
---
- name: print all url monitoring tasks has been completed.
  debug:
   msg: "we now have good confidence on what is role and what is dummy role and what is meta depedencies"
```

- come out of roles folder.

- create new yaml playbook

```
vim /home/ansible/dev/multiurltesting.yml
---
- name: test the url reachablity
  hosts: all
  roles:
   - multiurl
```
