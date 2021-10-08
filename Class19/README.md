# Ansible Task Control

## Step-1: What is loop
-  **Loop:** Ansible offers the loop, with_<lookup>, and until keywords to execute a task multiple times. Examples of commonly-used loops include changing ownership on several files and/or directories with the file module, creating multiple users with the user module, and repeating a polling step until a certain result is reached.

## Step-2: item
 - **item:** item is not a command, but a variable automatically created and populated by Ansible in tasks which use loops. In the following example: - debug: msg: "{{ item }}" with_items: - first - second. the task will be run twice: first time with the variable item set to first , the second time with second.
 
 - **Until:** In this short post I'll introduce you to lesser known type of Ansible loop: "until" loop. This loop is used for retrying task until certain condition is met. ... retry - specifies how many times we want to run the task before Ansible gives up. delay - delay, in seconds, between retries.
 
To start this handson lab,you need following resources.



## Step-3: Prerequisites

1.	**server.cnl.com** – 1 CPU – 1GB RAM (Python 2.7) - **Ansible Server**
2.	**node1.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 1**
3.	**node2.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 2**

from ansible server login as an ansible user as per class 4.From ansible user execute below command

ansible all -m ping

this above ping command should return with ping / pong green color.



## scenario 1: manual way of user creation

- create file called tasks.yml [vim tasks.yml].
```
---
- name: Repeated tasks can be written as standard loops
  hosts: all
  become: true
  tasks:
  - name: Add user testuser1
    ansible.builtin.user:
     name: "testuser1"
     state: present
     groups: "wheel"

  - name: Add user testuser2
    ansible.builtin.user:
     name: "testuser2"
     state: present
     groups: "wheel"
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```

- Verify the file from ansible client machines

```
ansible all -m shell -a "cat /etc/passwd | grep -i testuser1"
ansible all -m shell -a "cat /etc/passwd | grep -i testuser2"
ansible all -m shell -a "cat /etc/group | grep -i testuser1"
ansible all -m shell -a "cat /etc/group | grep -i testuser2"
```

## scenario 2: Repeated tasks can be written as standard loops

- create file called tasks.yml [vim tasks.yml].
```
---
- name: Repeated tasks can be written as standard loops
  hosts: all
  become: true
  tasks:
  - name: Add several users
    user:
     name: "{{ item }}"
     state: present
     groups: "wheel"
    loop:
     - testuser1
     - testuser2
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```

- Verify the file from ansible client machines

```
ansible all -m shell -a "cat /etc/passwd | grep -i testuser1"
ansible all -m shell -a "cat /etc/passwd | grep -i testuser2"
ansible all -m shell -a "cat /etc/group | grep -i testuser1"
ansible all -m shell -a "cat /etc/group | grep -i testuser2"
```

## scenario 4: Iterating over a list of hashes

- If you have a list of hashes, you can reference subkeys in a loop. 
- create file called tasks.yml [vim tasks.yml].

```
---
- name: If you have a list of hashes, you can reference subkeys in a loop
  hosts: all
  become: true
  tasks:
  - name: Add several users
    user:
     name: "{{ item.name }}"
     state: present
     groups: "{{ item.groups }}"
    loop:
     - { name: 'testuser1', groups: 'wheel' }
     - { name: 'testuser2', groups: 'root' }
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```

- Verify the package installation status from ansible client machines

```
ansible all -m shell -a "cat /etc/passwd | grep -i testuser1"
ansible all -m shell -a "cat /etc/passwd | grep -i testuser2"
ansible all -m shell -a "cat /etc/group | grep -i testuser1"
ansible all -m shell -a "cat /etc/group | grep -i testuser2"
```

## scenario 5: Retrying a task until a condition is met

- You can use the until keyword to retry a task until a certain condition is met. 
- create file called tasks.yml [vim tasks.yml].

```
---
- name: Retrying a task until a condition is met
  hosts: all
  become: true
  tasks:
  - name: Retry a task until a certain condition is met
    shell: /usr/bin/foo
    register: result
    until: result.stdout.find("all systems go") != -1
    retries: 5
    delay: 10
```

- save this tasks.yml file


- This task runs up to 5 times with a delay of 10 seconds between each attempt. If the result of any attempt has “all systems go” in its stdout, the task succeeds. The default value for “retries” is 3 and “delay” is 5.

- To see the results of individual retries, run the play with -vv.

- When you run a task with until and register the result as a variable, the registered variable will include a key called “attempts”, which records the number of the retries for the task.



- Execute the ansible playbook

```
ansible-playbook tasks.yml
```

## scenario 8: with_list is directly replaced by loop.


```
---
- name: with_list is directly replaced by loop.
  hosts: all
  become: true
  tasks:
   - name: with_list
     ansible.builtin.debug:
      msg: "{{ item }}"
     with_list:
      - one
      - two

   - name: with_list -> loop
     ansible.builtin.debug:
      msg: "{{ item }}"
     loop:
      - one
      - two
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```

## scenario 9: Loop - User Creation.


```
---
- name: Iterating over a list of Dictionaries
  hosts: all
  become: yes
  tasks:
    - name: Create new users
      user:
        name: '{{ item.name }}'
        uid: '{{ item.uid }}'
        state: present

      loop:
        - name: john
          uid: 1020
        - name: mike
          uid: 1030
        - name: andrew
          uid: 1040
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```


## scenario 10: Loop - User Creation.


```
---
- name: Iterating over a list of Dictionaries
  hosts: all
  become: yes
  tasks:
    - name: Create new users
      user:
        name: '{{ item.name }}'
        uid: '{{ item.uid }}'
        state: present

      loop:
        - { name: john , uid: 1020 }
        - { name: mike , uid: 1030 }
        - { name: andrew , uid: 1040}
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```

## scenario 11: Ansible Loops with indexes

- Occasionally, you may want to keep track of the index values within your array of items. For this, use the ‘with indexed_items‘ lookup. The index value begins from 0 whilst The loop index begins from item.0 and the value from item.1


```
---
- name: Ansible Loops with indexes
  hosts: all
  become: yes
  tasks:
  - name: Indexes with Ansible loop
    debug:
      msg: "The car at {{ item.0 }} is {{ item.1 }}"
    with_indexed_items:
      - "Nissan"
      - "Mercedes"
      - "Toyota"
      - "Mazda"
      - "BMW"
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```


## scenario 12: Conditionals in Ansible Loops

- In Ansible loops you can use the conditional statement when to control the looping based on the nature of variables or system facts. Consider the playbook below where we have a list of packages that need to be installed.

- We have specified an array called ‘packages‘ that contains a list of packages that need to be installed. Each of the items on the array contains the name of the package to be installed and the property called ‘required‘ which is set to ‘True‘ for 2 packages and ‘False‘ for one package.

```
---
- name: Install software
  become: yes
  hosts: all
  vars:
    packages:
      - name: httpd
        required: True

      - name: vsftpd
        required: True

      - name: zip
        required: False
  tasks:
    - name: Install "{{ item.name }}" on CentOS
      yum:
        name: "{{ item.name }}"
        state: present

      when:
        - item.required == True
        - ansible_facts['distribution'] =="CentOS"

      loop: "{{ packages }}"
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```

## scenario 13: Iterating over list of hashes

- Let's take a look at a basic example of a hash to get a better idea of how this unique but popular data structure works:

Key: Value
FName: Deepak
LName: Prasad
Location: India


- From this, table we can see that a key is simply an identifier, and the value that key represents could be any string or data piece stored in the value table that is associated with that specific key.

- Here I have created a playbook with some hash value which contains 3 key elements i.e. fname, lname and location which can be used with item to iterate over the loop.

```
---
- name: Working with loop module
  hosts: all
  gather_facts: false
  tasks:
     - name: Iterate over hashes
       debug:
         msg:
          - "Hello '{{ item.fname }}', nice to meet you"
          - "Your last name as per our record is '{{ item.lname }}'"
          - "Your country of residence is '{{ item.location }}'"

       loop:
         - { fname: 'deepak', lname: 'prasad', location: 'India' }
         - { fname: 'amit', lname: 'kumar', location: 'Argentina' }
         - { fname: 'rahul', lname: 'sharma', location: 'Canada' }
         - { fname: 'vivek', lname: 'mittal', location: 'Brazil' }
         - { fname: 'anurag', lname: 'keshav', location: 'Japan' }
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```


## scenario 13: Iterate over dictionary

- A dictionary is a collection which is unordered, changeable and indexed. In Python dictionaries are written with curly brackets, and they have keys and values.

- Below we take data in the simplest form such as key value pair in the YAML format where Key and value are separated by a colon. It is important that you give a whitespace followed y colon differentiating the Key and Value

Fruit: Apple
Vegetable: Carrot
Liquid: Water
Meat: Chicken


- Here Keys are Fruit, Vegetable, Liquid, Meat while the respective Values are Apple, Carrot, Water and Chicken

```
---
- name: Working with loop module
  hosts: all
  gather_facts: false
  tasks:
     - name: Iterate over dictionary
       debug:
         msg:
          - "The {{ item.key }} of your car is {{ item.value }}"

       loop: "{{ my_car | dict2items }}"
       vars:
         my_car:
           Color: Blue
           Model: Corvette
           Transition: Manual
           Price: $20,000
```

- save this tasks.yml file

- Execute the ansible playbook

```
ansible-playbook tasks.yml
```
