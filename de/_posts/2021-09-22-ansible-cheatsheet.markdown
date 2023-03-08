---
layout: post
title:  "Ansible-Cheatsheet"
date:   2021-09-22 12:13:42 +0100
categories: Ansible
---

Heyho,

an dieser Stelle ein kleines Ansible Cheatsheet, welches ich gleich der Übersicht zum Guide ständig aktuell halten werde.

## Playbooks
``` bash
# Run
ansible-playbook playbook.yml

# Check-Mode
ansible-playbook --check playbook.yml

# Define inventory
ansible-playbook -i my-inventory.txt playbook.yml

# Extra vars
ansible-playbook -e "foo=bar, bar=foo" playbook.yml

# Extra vars from file
ansible-playbook -e "@extra-vars.json" playbook.yml
```

<!-- excerpt-end -->

## AdHoc
``` bash
# System informations
ansible all -i my_inventory -m setup

# Shell command
ansible all -i my_inventory -m command -a "uname -a"

# Install updates
ansible all -i my_inventory -m apt -a "upgrade=yes"

# Install package
ansible all -i my_inventory -m apt -a "name=apache2, state=present"

# Check system availability
ansible all -i my_inventory -m ping

```

## Vault
``` bash
# create new vault
ansible-vault create --vault-password-file ~/.myvaultpw vault.pw

# edit vault
ansible-vault edit --vault-password-file ~/.myvaultpw vault.pw

# Run playbook with vault
ansible-playbook --vault-password-file ~/.myvaultpw playbook_with_vault_secrets.yml
```

## Line in file
``` yaml
...
  # add etc hosts entry
  - name: enter line in /etc/hosts
    lineinfile:
      path: /etc/hosts
      line: "192.168.1.1 example.local"
      regexp: "^192.168.1.1"
..
```

## Create directory
``` yaml
...
 - name: create dir
   file:
     path: /opt/mydir
     state: directory
     owner: user1
     group: group1
...
```
