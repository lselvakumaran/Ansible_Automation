# cron jobs within Ansible


## Prerequisites

1.	**server.cnl.com** – 1 CPU – 1GB RAM (Python 2.7) - **Ansible Server**
2.	**node1.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 1**
3.	**node2.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 2**

from ansible server login as an ansible user as per class 4.From ansible user execute below command

ansible all -m ping

this above ping command should return with ping / pong green color.





## scenario 1: Cron Job for frequently


```
vim cron.yml
```

- create normal playbook

```
---
- name: Ensure monitoring cron jobs are present
  hosts: node1,node2
  tasks:
    - name: "Comment  cron jobs"
      cron:
       user: ansible
       name: "{{ item.name }}"
       job: "{{ item.job }}"
       state: present
       disabled: True
      loop:
       - { job: 'job1', name: "app1" }
       - { job: 'job2', name: "app2" }
      tags: stop_cron
```
- save the file

- To execute the file.

```
ansible-playbook cron.yml
```

## scenario 2: Ensure monitoring cron jobs are present


```
vim cron.yml
```

- create normal playbook

```
---
- name: Ensure monitoring cron jobs are present
  hosts: node1,node2
  tasks:
    - name: Send emails about expiry dates
      cron:
        user: ansible
		name: "Send expiry date emails (runs each day, 15 minutes after midnight)"
        minute: "15"
        hour: "0"
        job: "php path/to/console email-expiry-dates/send-email"  
```
- save the file

- To execute the file.

```
ansible-playbook cron.yml
```

## scenario 2.1: Ensure monitoring cron jobs are present - Update the existing Cron JOB

- Updating cron job

- Let's say that server's current time zone is UTC+00:00, and a change request comes up for the existing cron job, something like:

- Emails should be delivered right after midnight in New York's time zone

- What did we do to implement necessary changes? Updated cron job's execution time, but also its name (or we can say, the description):

```
vim cron.yml
```

- create normal playbook

```
---
- name: Ensure monitoring cron jobs are present
  hosts: node1,node2
  tasks:
    - name: Send emails about expiry dates
      cron:
        user: ansible
		name: "Send expiry date emails (runs each day, 15 minutes after midnight in New York's time zone)"
        minute: "15"
        hour: "5"
        job: "php path/to/console email-expiry-dates/send-email"
```
- save the file

- To execute the file.

```
ansible-playbook cron.yml
```


- Open new SSH Putty window and login node1 or node2.

- login as an ansible user

```
crontab -l
```


## scenario 2.2: Ensure monitoring cron jobs are present - Fixing the unexpected problem

```
vim cron.yml
```

- create normal playbook

```
---
- name: Fixing the unexpected problem
  hosts: node1,node2
  tasks:
    - name: Old (Send emails about expiry dates)
      cron:
        user: ansible
		name: "Send expiry date emails (runs each day, 15 minutes after midnight)"
        minute: "15"
        hour: "0"
        job: "php path/to/console email-expiry-dates/send-email"
        state: absent
    - name: Send emails about expiry dates
      cron:
        user: ansible
		name: "Send expiry date emails (runs each day, 15 minutes after midnight in New York's time zone)"
        minute: "15"
        hour: "5"
        job: "php path/to/console email-expiry-dates/send-email"
```
- save the file

- To execute the file.

```
ansible-playbook cron.yml
```


- Open new SSH Putty window and login node1 or node2.

- login as an ansible user

```
crontab -l
```

# Conclusion
**Changing a cron job's name will not result in an update of the existing cron job. Its name is some sort of an ID, and therefore should not be changed (in most cases). But, what we can do, is play around with tasks' name property, since it will not lead to the problem we occurred :) **