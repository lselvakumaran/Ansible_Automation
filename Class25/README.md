# Ansible Vault

## 1: What is ansible vault
-  **Vault:** Ansible Vault is an Ansible feature that helps you encrypt confidential information without compromising security.Ansible provides a feature named Ansible Vault that prevents this data from being exposed. It keeps passwords and other sensitive data in an encrypted file rather than in plain text files.

## 2: Is ansible vault secure?
 
- **Ansible Vault** is a feature that allows you to keep all your secrets safe. It can encrypt entire files, entire YAML playbooks or even a few variables.Vault is implemented with file-level granularity where the files are either entirely encrypted or entirely unencrypted.
 
## 3:When should I use ansible vault?
- ** Ansible Vault** is a feature that allows users to encrypt values and data structures within Ansible projects. This provides the ability to secure any sensitive data that is necessary to successfully run Ansible plays but should not be publicly visible, like passwords or private keys.
 
## 4: How do you use the ansible vault in a playbook?
- ** Ansible vault ** will prompt you for the password and later require you to confirm it. Next, type the string value that you want to encrypt. Finally, press ctrl + d . Thereafter, you can begin assigning the encrypted value in a playbook.

## 5: Where are Ansible vault files stored?
- ** Ansible vault ** is the answer to this. Ansible Vault can encrypt anything inside of a YAML file, using a password of your choice.

- Files within the group_vars directory.
- A role's defaults/main. yml file.
- A role's vars/main. yml file.
- Any other file used to store variables.

## 4: Prerequisites

1.	**server.cnl.com** – 1 CPU – 1GB RAM (Python 2.7) - **Ansible Server**
2.	**node1.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 1**
3.	**node2.cnl.com** – 1 CPU – 1GB RAM ( python 2.6 and above) - **Ansible Client 2**

from ansible server login as an ansible user as per class 4.From ansible user execute below command

ansible all -m ping

this above ping command should return with ping / pong green color.


## scenario 1: Create new playbook


```
vim httpd.yml
```

- create normal playbook

```
---
- name: install httpd
  hosts: all
  tasks:
   - name: install httpd
     yum:
	  name: httpd
	  state: latest
```
- save the file

- To execute the file.

```
ansible-playbook httpd.yml
```

- To view the file.The file inside contents will be displayed.

```
cat httpd.yml
```

## scenario 2: Encrypt the existing playbook

- To see ansible vault help and sub options


```
ansible-vault --help
```

- Encrypt existing normal playbook

```
ansible-vault encrypt httpd.yml
```


- To execute the file.

```
ansible-playbook httpd.yml
```

- To view the file.The file inside contents will be displayed.

```
cat httpd.yml
```

## scenario 3: Encrypt the new playbook

- Encrypt and create new playbook

```
ansible-vault create httpd1.yaml
```

```
---
- name: install httpd
  hosts: all
  tasks:
   - name: install httpd
     yum:
	  name: httpd
	  state: latest
```


- To execute the file.

```
ansible-playbook httpd1.yml
```

- To view the file.The file inside contents will be displayed.

```
cat httpd1.yml
```

## scenario 4: Decrypt the existing encrypted playbook

- Encrypt and create new playbook

```
ansible-vault decrypt httpd1.yaml
```

- To execute the file.

```
ansible-playbook httpd1.yml
```

- To view the file.The file inside contents will be displayed.

```
cat httpd1.yml
```

## scenario 5: View the existing encrypted playbook

- Encrypt and create new playbook

```
ansible-vault view httpd.yaml
```

- To execute the file.

```
ansible-playbook httpd.yml
```

- To view the file.The file inside contents will be displayed.

```
cat httpd.yml
```

## scenario 6: View the existing encrypted playbook

- create the zip file with password - cnl12345

```
vault.zip file passowrd is cnl12345
```

- To create file and keep the password inside this file .Encrypt this file.

```
ansible-vault create crypt.yml
```
- while creating the above crypt.yml file -- give the password - "password"
```
password: cnl12345
```

- store password in file

```
vim vaultpasswd
```

```
password
```

- Create new playbook

```
vim unarchive.yml
```
---
- name: unarchive yml
  hosts: node1,node2
  vars_files:
   - crypt.yml
  tasks:
   - name: install unzip
     yum:
      name: unzip
      state: latest
   - name: create directory
     file:
      path: /var/tmp/vault
      state: directory
      mode: 0644
   - name: download vault.zip
     get_url:
      url: https://raw.githubusercontent.com/cloudnloud/Ansible_Automation/main/Class25/vault.zip
      dest: /var/tmp/vault/vault.zip
      mode: 0644
   - name: unzip 
     command: unzip -o -P {{ password }} /var/tmp/vault/vault.zip
     args:
      chdir: /var/tmp/vault
```

