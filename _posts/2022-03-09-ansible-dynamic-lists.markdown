---
layout: post
title:  "Ansible: Listen dynamisch erstellen und erweitern"
date:   2022-03-09 10:00:42 +0100
categories: Ansible
---

Moin,

nachdem ich nun häufiger das Problem hatte, dass ich mir Ansible-Listen dynamisch erstellen und erweitern muss, 
hier ein kurzes Code-Snippet:

```yaml
- hosts: localhost
  connection: local
  vars:
    my_list: [ "Hello", "World", "this", "is", "just", "a", "list", "of", "strings" ]
  tasks:
    - name: build list with all items with an 'i' in it
      set_fact:
        my_dynamic_list: "{%raw%}{{ my_dynamic_list|default([]) + [ item ] }}{%endraw%}"
      loop: "{%raw%}{{ my_list }}{%endraw%}"
      when: "'i' in item"
      
    - name: print results
      debug:
        msg: "{%raw%}{{ my_dynamic_list }}{%endraw%}"
```

<!-- excerpt-end -->

Output:

```
PLAY [localhost] ****************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************
ok: [localhost]

TASK [build list with all items an 'i' in it] ***********************************************************************************************************************************************************************
skipping: [localhost] => (item=Hello) 
skipping: [localhost] => (item=World) 
ok: [localhost] => (item=this)
ok: [localhost] => (item=is)
skipping: [localhost] => (item=just) 
skipping: [localhost] => (item=a) 
ok: [localhost] => (item=list)
skipping: [localhost] => (item=of) 
ok: [localhost] => (item=strings)

TASK [print results] ************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": [
        "this",
        "is",
        "list",
        "strings"
    ]
}

PLAY RECAP **********************************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## Kurze Erklärung
Zunächst wurde eine List mit Beispielwerten "my_list" definiert. Im ersten Task wird dann durch alle Items aus "my_list" iteriert. Zusätzlich bekommt der Task die Bedingung 

```yaml
when: "'i' in item"
```

Sprich, der task wird nur ausgeführt, wenn das jeweilige Item den Buchstaben "i" enthält. Sollte die Bedingung zutreffen, wird das Item der Ziel-Liste angefügt. Der Jinja2-Filter 'default':

```yaml
my_dynamic_list: "{%raw%}{{ my_dynamic_list|default([]) + [ item ] }}{%endraw%}"
```

fängt im Prinzip den ersten Durchlauf ab, zu dem die Liste "my_dynamic_list" noch nicht existiert und liefert in diesem Fall eine leere Liste zurück. Da man an eine Liste nur eine weitere Liste - und keinen String - anfügen kann, definieren wir den String in einer Liste mit einem einzigen Item:

```yaml
 ... [ item ] ...
````

Vielleicht findet der ein oder Andere diesen Kleinen Codeschnipsel ja noch nützlich :)

Grüße!
