# BigFix Agent installation

## Prerequisites

1.	**server.cnl.com** – 1 CPU – 1GB RAM (Python 2.7) - **Ansible Server**
2.	**node1.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 1**
3.	**node2.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 2**

from ansible server login as an ansible user as per class 4.From ansible user execute below command

ansible all -m ping

this above ping command should return with ping / pong green color.





## scenario 1: Big FIX Agent Installation


```
vim agent.yml
```

```
---
- name: Install Bigfix Agent in your project
  hosts: node1,node2
  tasks:
    - block:
      - name: to print which OS is currently
        debug: msg="This is CENTOS enterprise os"
        when: ansible_distribution == "CentOS"
      - name: To check whether folder existis on server
        stat: path=/etc/opt/BESClient
        register: filestatus
      - name: To print Jason output of the filestatus
        debug: var=filestatus
      - name: to create directory if its dosn't exist
        file: path=/etc/opt/BESClient state=directory mode=0755 force=yes
        when: filestatus.changed == false
      - name: To check whether folder existis on server
        stat: path=/var/opt/BESClient
        register: filestatus1
      - name: To print Jason output of the filestatus
        debug: var=filestatus1
      - name: to create directory if its dosn't exist
        file: path=/var/opt/BESClient state=directory mode=0755 force=yes
        when: filestatus1.changed == false
      - name: check the besclient.config is there or not
        file: path=/var/opt/BESClient/besclient.config state=file
        ignore_errors: yes
        register: besclientfile
      - name: print the custom besclient file output
        debug: msg="the file is not available"
        when: besclientfile.path == "/var/opt/BESClient/besclient.config"
      - name: print the jason output
        debug: var=besclientfile
      - name: copy the configuration file to patch
        copy: src=/tmp/actionsite.afxm dest=/etc/opt/BESClient/actionsite.afxm mode=0600
      - name: to copy the template of the bigfix agent
        copy: src=/tmp/besclient.config dest=/var/opt/BESClient/besclient.config mode=0644
      - name: To install bigfix package using command module
        command: rpm -qa BESAgent*
        register: package_status
      - name: To print Jason output of package
        debug: var=package_status
      - name: Install the package if it dosn't exist
        command: yum install http://software.bigfix.com/download/bes/92/BESAgent-9.2.8.74-rhe5.x86_64.rpm -y
        when: package_status.stdout == "" and besclientfile.path != "/var/opt/BESClient/besclient.config"
      - name: To start the service and enable it in chkconfig
        service: name=besclient state=started enabled=yes
      - name: To print playbook status
        debug: msg="Bigfix agent has been successfully installed on the client server"
      rescue:
      - name: to print which OS is currently
        debug: msg="This is ubuntu os"
        when: facter_osfamily == Ubuntu
      - name: To check whether folder existis on server
        stat: path=/etc/opt/BESClient
        register: filestatus
      - name: To print Jason output of the filestatus
        debug: var=filestatus
      - name: to create directory if its dosn't exist
        file: path=/etc/opt/BESClient state=directory mode=0755 force=yes
        when: filestatus.changed == false
      - name: To check whether folder existis on server
        stat: path=/var/opt/BESClient
        register: filestatus1
      - name: To print Jason output of the filestatus
        debug: var=filestatus1
      - name: to create directory if its dosn't exist
        file: path=/var/opt/BESClient state=directory mode=0755 force=yes
        when: filestatus1.changed == false
      - name: check the besclient.config is there or not
        file: path=/var/opt/BESClient/besclient.config state=file
        ignore_errors: yes
        register: besclientfile
      - name: print the custom besclient file output
        debug: msg="the file is not available"
        when: besclientfile.path == "/var/opt/BESClient/besclient.config"
      - name: print the jason output
        debug: var=besclientfile
      - name: copy the configuration file to patch
        copy: src=/tmp/actionsite.afxm dest=/etc/opt/BESClient/actionsite.afxm mode=0600
      - name: to copy the template of the bigfix agent
        copy: src=/tmp/besclient.config dest=/var/opt/BESClient/besclient.config mode=0644
      - name: To install bigfix package using command module
        command: rpm -qa BESAgent*
        register: package_status
      - name: To print Jason output of package
        debug: var=package_status
      - name: Install the package if it dosn't exist
        command: yum install http://software.bigfix.com/download/bes/92/BESAgent-9.2.8.74-rhe5.x86_64.rpm -y
        when: package_status.stdout == "" and besclientfile.path != "/var/opt/BESClient/besclient.config"
      - name: To start the service and enable it in chkconfig
        service: name=besclient state=started enabled=yes
      - name: To print playbook status
        debug: msg="Bigfix agent has been successfully installed on the client server"
      always:
        - debug:
            msg: "Bigfix implementation has been completed sucuessfully"
```
- save the file

- To execute the file.

```
ansible-playbook agent.yml
```

