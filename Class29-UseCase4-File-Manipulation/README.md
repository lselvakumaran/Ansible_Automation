# Configuration Files Manipulationn

- **Ansible lineinfile module** is used to insert a line, modify, remove, and replace an existing line. ... Ansible lineinfile provides many parameters to do the job quickly. You can also use the condition to match the line before modifying, removing using the regular expressions.


## What is Lineinfile?
- **lineinfile** - Ensure a particular line is in a file, or replace an existing line using a back-referenced regular expression.


## Prerequisites

1.	**server.cnl.com** – 1 CPU – 1GB RAM (Python 2.7) - **Ansible Server**
2.	**node1.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 1**
3.	**node2.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 2**

from ansible server login as an ansible user as per class 4.From ansible user execute below command

ansible all -m ping

this above ping command should return with ping / pong green color.





## scenario 1: File Manipulation


```
vim file.yml
```

```
---
- name: yml script to check out file manipulations
  hosts: all
  tasks:
   - name: Create a File
     file:
      path: /tmp/testFile.txt
      state: touch
      mode: u=rw,g=r,o=r
   - name: line insert at top of file
     lineinfile:
        path: /tmp/testFile.txt
        line: 'Added Line 1'
        insertbefore: BOF
   - name: Insert a line at the end of a file.
     lineinfile:
        path: /tmp/testFile.txt
        line: 'Added as last line'
   - name: Inserting a line after a pattern
     lineinfile:
        path: /tmp/testFile.txt
        line: alias ll='ls -lhA'
        insertafter: alias.*
   - name: Insert a line before example
     lineinfile:
        path: /tmp/testFile.txt
        line: 'New line added before alias'
        insertbefore: alias*
   - name: Insert multiple lines using blockinfile
     blockinfile:
      dest: /tmp/testFile.txt
      block: |
        These lines are added by blockinfile module
        Check out the marker lines
      backup: yes
```
- save the file

- To execute the file.

```
ansible-playbook file.yml --step
```



- While executing the above command in ansible server.Open new ssh putty window and login to any one of the client and run the below command.

- you will understand the file entry in each play/task execution in the above entire playbook
```
cat /tmp/testFile.txt
```
