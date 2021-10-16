# Ansible Optimization


## Prerequisites

1.	**server.cnl.com** – 1 CPU – 1GB RAM (Python 2.7) - **Ansible Server**
2.	**node1.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 1**
3.	**node2.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 2**

from ansible server login as an ansible user as per class 4.From ansible user execute below command

ansible all -m ping

this above ping command should return with ping / pong green color.





## scenario 1: Measuring Ansible Tasks Execution Time



- ** Before optimizing whatever it is, we should have some good baseline information about our current situation and also be able to measure how timings are changing after certain changes we’ve made during the time.**

- Measuring Ansible tasks execution time, which is called callback plugins. 

- In order to measure our timings, we will need to put the following line in the “[defaults]” section of our  ansible.cfg file 

```
vim ansible.cfg
```


```
[defaults]

# Enable timing information
callback_whitelist = timer, profile_tasks

```



- create normal playbook

```
vim httpd.yml
```


```
---
- name: install httpd
  hosts: node1,node2
  tasks:
   - name: install httpd
     yum:
      name: httpd
      state: latest
   - name: restart web service
     service:
      name: httpd
      state: restarted
   - name: install vsftpd
     yum:
      name: vsftpd
      state: latest
   - name: restart web service
     service:
      name: vsftpd
      state: restarted
```
- save the file

- To execute the file.

```
ansible-playbook httpd.yml
```


- After adding this line, you will start seeing timing information for your task execution. 

# Ansible Performance Bottlenecks
## As we already have the tools for measure timing/performance, we now have to start optimizing it. 

** Two are the most common performance killers in Ansible: 

- 1) SSH 

- 2) Facts gathering

- and here is how to fix them. 


## scenario 2: Optimizing Ansible SSH Performance 

- This could give you some huge performance benefit, especially if you are executing big number of tasks or executing on a big number of hosts, or both. 

- The idea behind SSH Multiplexing is that once a ssh connection is made to a host, the connection will stay in background for a given period of time. Whenever you need to re-execute something on the same host, the connection will be reused (if you try using the same user/host/port pair). 

- ** By default Ansible is using ControlPersist=60 **, which means ** each connection will stay alive (in the background) for 60 seconds at most **. For me this is too little and I prefer the connections to stay alive for multiple hours. 


- Add the following inside the [ssh_connection] section of your ansible.cfg config file: 

```

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=18000 -o PreferredAuthentications=publickey
control_path = %(directory)s/ansible-ssh-%%h-%%p-%%r

```


# CAUTION

- When using SSH Multiplexing with longer ControlPersist time, **there is a potential trouble**, if you sleep your notebook/pc and wake it again with existing MUX connections. By doing this, your connections will be broken, but still staying as Persistent, which will break your ssh connectivity  to the muxed hosts. 

- For fixing it, you will have to kill the SSH [mux] containing processes, could be done with something like that:

```
ps faux  | grep ssh | grep "\[mux\]"  | awk '{print $2}' | xargs kill
``` 


## scenario 3: Optimizing SSH PreferredAuthentications


- Another thing that has effect on speed is the Authentication methods SSH will try to use while connecting. 

- If you use only public keys for ssh connection with the desired hosts, you could add the following additional option “PreferredAuthentications=publickey”  to your ssh_args (ansible.cfg -> [ssh_connection])

- Now your config line should look like this:

# Adding PreferredAuthentications=publickey to the ssh_args line
ssh_args = -o ControlMaster=auto -o ControlPersist=18000 -o PreferredAuthentications=publickey




## scenario 4: Enabling Pipelining

- Another serious gain in the speed could be achieved by enabling Pipelining. 

- Enabling  happens by adding the following option in ansible.cfg [ssh_connection] section: 

```
[ssh_connection]
pipelining = True
```

## scenario 5: Optimizing Facts Gathering Process

- By default Ansible will try to gather as much as possible facts for each host it connects to. This is pretty heavy operation and most of the time you don’t need most of the facts it will gather for you. 

**Fully disable facts gathering**
- That’s the most ‘brute-force’ solution, which should give you huge speed boost, but you won’t be able to rely on any facts for the hosts. 

```
- hosts: whatever
  gather_facts: no
  
```

```
vim httpd.yaml
```

```
---
- name: install httpd
  hosts: all
  gather_facts: False
  tasks:
   - name: install httpd
     yum:
      name: httpd
      state: latest
   - name: restart web service
     service:
      name: httpd
      state: restarted
```


- If you need only network and virtual facts groups (+ the minimal facts set), you can try something like this: 

```
---
- name: install httpd
  hosts: all
  gather_facts: False
  pre_tasks:
   - setup:
      gather_subset:
       - '!all'
       - '!any'
       - 'network'
       - 'virtual'
  tasks:
   - name: install httpd
     yum:
      name: httpd
      state: latest
   - name: restart web service
     service:
      name: httpd
      state: restarted
```

## scenario 5: Enable facts caching mechanism 

**If you still need some of the facts groups, but at the same time the gathering process is still slow for you, you could try use fact caching.**

** Caching enables Ansible to cache the facts for a given host in some kind of backend. **

- Currently the caching plugin supports the following cache backend:

- **memcache**
- **redis**
- **yaml**
- **json** 
- **mongodb**
- **memory**

- This is an example configuration of facts caching in json files

```
[defaults]
gathering = smart
fact_caching_connection = /tmp/facts_cache
fact_caching = jsonfile


# The timeout is defined in seconds 
# This is 2 hours 
fact_caching_timeout = 7200

```



## scenario 6: Playing with the Fork parameter

- By default Ansible has “fork” set to 5. Which means that in any given point of time, **Ansible could run as much as 5 parallel executions**. 

- If you are executing a playbook on more than 5 hosts, **this means you will actually execute on a portions of 5 hosts in parallel**. This can easily become a bottleneck when working with tens, hundreds or thousands of hosts. 

- Luckily this option could be manually controlled from your ansible.cfg file and looks like this: 
```

[defaults]
forks = 50

```

- By adding the line above, you will increase the forks to 50. 

- Keep in mind that playing with forks has its cost. By using more forks, you will be able to speedup the process of playing, but in the same time **this will require more CPU on your Ansible Host machine. So..tune with care**. 