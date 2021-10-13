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

## scenario 1: How to do patching normally without roles ?


```
vim patching.yml
```
```
---
- name: update entire servers
  hosts: all
  tasks:
   - name: update the system
     yum:
      name: "*"
      state: latest
```
- In the first line, we give the task a meaningful name so we know what Ansible is doing. In the next line, the yum module updates the CentOS virtual machine (VM), then name: "*" tells yum to update everything, and, finally, state: latest updates to the latest RPM


```
ansible-playbook patching.yml
```

```
---
- name: Restart the servers and with new kernel
  hosts: all
  tasks:
   - name: restart system to reboot to newest kernel
     shell: "sleep 5 && reboot"
     async: 1
     poll: 0
   - name: wait for 10 seconds
     pause:
      seconds: 10
   - name: wait for the system to reboot
     wait_for_connection:
      connect_timeout: 20
      sleep: 5
      delay: 5
      timeout: 60
   - name: install epel-release
     yum:
      name: epel-release
      state: latest
```

- execute the playbook

```
ansible-playbook patching.yml
```


## scenario 2: How to install nginx without role

```
vim nginx.yml
```

```
---
- name: instll nginx
  hosts: all
  vars:
   - type_of_webserver: nginx
  tasks:
   - name: install nginx
     yum: 
	  name: nginx 
	  state: latest
   - name: copy index.html template
     template:
      src: index.html
      dest: /usr/share/nginx/html
     notify: restart nginx
   - name: enable and start service
     service:
      name: nginx
      enabled: yes
      state: started
  handlers:
   - name: restart nginx
     service:
	  name: nginx
	  state: restarted
```

```
vim index.html
```

```
<!DOCTYPE html>
<html>
<head>
<title>My Nginx web server</title>
</head>
<body><h1>Welcome to my {{ type_of_webserver }} webpage </h1></body>
</html>
```

- declare variable in inventory file as an global variable or declare in the same playbook as an local variable
```
vim hosts

[all:vars]
type_of_webserver=nginx
```

- execute the playbook

```
ansible-playbook nginx.yml
```


## scenario 3: nginx installation via resuable role

- create nginx role and learn how directory structure are ?.
```
cd roles
ansible-galaxy init nginx_role
```

```
vim nginx_role/tasks/main.yml
```
```
# tasks file for nginx_role
- name: install nginx
  yum: name=nginx state=latest
- name: copy index.html template
  template:
   src: index.html
   dest: /usr/share/nginx/html
  notify: restart nginx
- name: enable and start service
  service:
   name: nginx
   enabled: yes
   state: started
```

```
vim nginx_role/template/index.html
```
```
<!DOCTYPE html>
<html>
<head>
<title>My Nginx web server</title>
</head>
<body><h1>Welcome to my {{ type_of_webserver }} webpage </h1></body>
</html>
```

```
vim nginx_role/handlers/main.yml
```

```
# handlers file for nginx_role
- name: restart nginx service
  service: name=nginx state=restarted
  listen: “restart nginx”
```

```
vim Nginx_role/defaults/main.yml
```

```
# defaults file for nginx_role
type_of _webserver: Nginx
```

- go to parent directory where you have ansible.cfg file is there.
- create new playbook like below

```
vim nginx.yml
```

```
---
- install nginx using role
# Install nginx Playbook
- hosts: ansible_client
  become: yes
  roles:
   - nginx_role
```

```
ansible-playbook nginx.yml
```