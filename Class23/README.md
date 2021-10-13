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


## 5: before install some package open firewall port ?
**Ansible roles - resuable role - we can open port multiple times for any package installation**

![ScreenShot](https://github.com/cloudnloud/Ansible_Automation/blob/main/Class23/firewall-port-opening.JPG)


## scenario 1: create dummy role - Allow service in firewall ?


```
cd roles
ansible-galaxy init myfirewall
```

```
vim myfirewall/defaults/main.yml
```

```
---
firewall_service: ssh

```

- save the file

```
vim myfirewall/handlers/main.yml
---
- name: restart firewalld
  service: name=firewalld state=restarted enabled=yes 
```

- save the file

```
vim myfirewall/tasks/main.yml
```

```
---
- name: install firewalld
  yum: name=firewalld state=present
- name: start firewalld service
  service: name=firewalld state=restarted enabled=yes
- name: open ports in firewalld
  firewalld: 
   service: "{{ firewall_service }}"
   permanent: true 
   state: enabled 
   immediate: true
```

- go to parent directory where you have ansible.cfg file is there.
- create new playbook like below

```
vim firewall.yml
```

```
---
- name: install nginx using role
  hosts: all
  become: yes
  roles:
   - myfirewall
```

```
ansible-playbook firewall.yml
```

## Scenario 2: create main package installation role 
## (call firewall opening whenever you want to install any package)?


```
cd roles
ansible-galaxy init webserver
```

```
vim webserver/files/index.html
```

```
<!DOCTYPE html>
<html>
<head>
<title>My web server</title>
</head>
<body><h1>Welcome to my webpage </h1></body>
</html>
```


- save the file

```
vim webserver/handlers/main.yml
```

```
---
- name: restart httpd
  service: name=httpd state=restarted
```

- save the file


```
vim webserver/meta/main.yml
```

```
---
dependencies:
 - { role: myfirewall, firewall_service: http }
```

- save the file


```
vim webserver/templates/vhost.conf.j2
```

```
# {{ ansible_managed }}

<VirtualHost *:80>
    ServerAdmin webmaster@{{ ansible_fqdn }}
    ServerName {{ ansible_fqdn }}
    ErrorLog logs/{{ ansible_hostname }}-error.log
    CustomLog logs/{{ ansible_hostname }}-common.log common
    DocumentRoot /var/www/vhosts/{{ ansible_hostname }}/

    <Directory /var/www/vhosts/{{ ansible_hostname }}/>
	Options +Indexes +FollowSymlinks +Includes
	Order allow,deny
	Allow from all
    </Directory>
</VirtualHost>
```

- save the file

```
vim webserver/tasks/main.yml
```

```
---
- name: install httpd
  yum: name=httpd state=latest
- name: start httpd service
  service: name=httpd state=restarted enabled=yes
- name: copy the html file
  copy:
   src: html/
   dest: "/var/www/vhosts/{{ ansible_hostname }}"
- name: vhost template file
  template:
   src: vhost.conf.j2
   dest: /etc/httpd/conf.d/vhost.conf
   owner: root
   group: root
   mode: 0644
  notify:
    - restart httpd
```

- go to parent directory where you have ansible.cfg file is there.
- create new playbook like below

```
vim web.yml
```

```
---
- name: install webserver and add webserver service in firewall using role
  hosts: all
  become: yes
  roles:
   - webserver
```

```
ansible-playbook web.yml
```

