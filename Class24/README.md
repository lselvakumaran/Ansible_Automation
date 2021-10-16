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


## scenario 1: Bulk User Creation & Bulk user Deletion Using Role ?


```
cd roles
ansible-galaxy init usermgmt
```

```
vim usermgmt/vars/main.yml
```

```
staff:
        - alice
        - fred
        - robin
        - natasha
        - eric
guests:
        - rob
        - bob
        - michael
        - williams
        - julia
revoke:
        - sara
        - frank
        - larry
        - lisa
        - roger
```

- save the file


```
vim usermgmt/tasks/main.yml
```

```
---
- name: User Creation progress
  debug:
   msg: "All users will be created and added to the respective groups"
```

- go to parent directory where you have ansible.cfg file is there.
- create new playbook like below

```
vim user.yml
```

```
---
- name: create user and groups
  hosts: all
  roles:
   - usermgmt
  tasks:
   - name: create group
     group: 
      name: "{{ item }}"
      state: present
     with_items:
      - administrators
      - developers
      - managers
   - name: create STAFF USER
     user:
      name: "{{ item }}"
      state: present
      groups: administrators,developers
     with_items:
      - "{{ staff }}"
   - name: create guests USER
     user:
      name: "{{ item }}"
      state: present
      groups: developers,managers
     with_items:
      - "{{ guests }}"
   - name: revoke the USER
     user:
      name: "{{ item }}"
      state: absent
     with_items:
      - "{{ revoke }}"
```

```
ansible-playbook user.yml
```


## scenario 2: Bulk Patch installation & Bulk patch removal Using Role ?


```
cd roles
ansible-galaxy init urllist
```

```
vim urllist/vars/main.yml
```

```
devurls:
        - "http://cloudnloud.com/"
        - "http://youtube.com/"
        - "http://www.tcs.com/"
        - "http://www.infosys.com/"
        - "http://www.wipro.com/"

app2urls:
        - "http://www.irctc.com/"
        - "http://www.redbus.com/"
        - "http://www.emirates.com/"
        - "http://www.skyscanner.com/"
        - "http://www.redhat.com/"
        - "http://www.airfrance.com/"
```

- save the file


```
vim urllist/tasks/main.yml
```

```
---
- name: url check is in progress
  debug:
   msg: "all environment urls will be validated."
```

- go to parent directory where you have ansible.cfg file is there.
- create new playbook like below

```
vim urlcheck.yml
```

```
---
- name: Check urls for applications
  hosts: all
  roles:
   - urllist
  tasks:
   - name: check url is reachable or not
     uri:
      url: "{{ item }}"
     with_items:
      - "{{ devurls }}"
   - name: print the working urls.
     debug:
      msg: "{{ item }} is working and reachble"
     with_items:
       - "{{ devurls }}"
   - name: check url is reachable or not
     uri:
      url: "{{ item }}"
     with_items:
      - "{{ app2urls }}"
   - name: print the working urls.
     debug:
      msg: "{{ item }} is working and reachble"
     with_items:
      - "{{ app2urls }}"
```

```
ansible-playbook urlcheck.yml
```
