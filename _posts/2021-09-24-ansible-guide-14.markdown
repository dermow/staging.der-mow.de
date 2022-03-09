---
layout: post
title:  "Ansible: Listen dynamisch erstellen und erweitern"
date:   2022-03-09 10:00:42 +0100
categories: Ansible
---

Moin,

nachdem ich in der Firma nun häufiger das Problem hatte, dass ich mir Ansible-Listen dynamisch erstellen und erweitern muss, 
hier ein kurzes Code-Snippet:
`
```yaml
- hosts: all
  vars:
    my_list: [ "Hello", "World", "this", "is", "just", "a", "list", "of", "strings ]
  tasks:
    - name: build list with all items an 'i' in it
      set_fact:
        my_dynamic_list: "{{ my_dynamic_list|default([]) + [ item ] }}"
      loop: "{{ my_list }}
      when: 'i' in my_list
      
    - name: print results
      debug:
        msg: "{{ my_dynamic_list }}"
```
