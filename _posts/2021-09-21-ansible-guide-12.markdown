---
layout: post
title:  "Ansible Starter-Guide: #012 - Loops 2"
date:   2021-09-21 18:19:42 +0100
categories: Ansible
---

Heyho! Nach etwas längerer (na gut... sehr langer) Pause bin ich nun endlich mal wieder zum Bloggen gekommen. Wie versprochen also hier der nächste Teil des Ansible Starter-Guide,
der euch weitere Informationen zur Verwendung von Loops näher bringen wird.

## Dictionaries

Bevor wir aber zu unseren Loops zurückkehren möchte ich euch noch kurz Dictionaries vorstellen, die bisher nur in Teil 5 kurz erwähnt wurden. Dictionaries sind
einfach ausgedrückt Listen, deren Items mehrere Felder besitzen können. Ein einfaches Beispiel:

```yaml
people:
  - name: Ronald
    nachname: Weasley
    alter: 21
  - name: Gilderoy
    nachname: Lockhart
    alter: 40
  - name: Albus
    nachname: Dumbledore
    alter: 99

```

<!-- excerpt-end -->

Wie in einer simplen "list" markiert der Bindestrich den Beginn eines neuen Items. In diesem Beispiel besitzt jedes der beiden Items die Felder "name", "nachname" und "alter".

Auch Dictionaries können wir in Loops verwenden. Möchten wir zum Beispiel die in unserem Beispiel definierten Informationen ausgeben, sähe das Playbook so aus:

##### print_dictionary.yml 
``` yaml
{%raw%}
- hosts: localhost
  vars:
    people:
      - name: Gilderoy
        nachname: Lockhard
        alter: 43
      - name: Minerva
        nachname: McGonnagal
        alter: 67
      - name: Albus
        nachname: Dumbledore
        alter: 99
       
  tasks:
    - name: list people
      debug:
        msg: "Vorname: {{ item.name }}, Nachname: {{ item.nachname }}, Alter: {{ item.alter }}"
      loop: "{{ people }}"
{%endraw%}
```

Ausgabe:
```
PLAY [localhost] **************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [localhost]

TASK [list people] ************************************************************************************************************************************************************************************************
ok: [localhost] => (item={'name': 'Gilderoy', 'nachname': 'Lockhard', 'alter': 43}) => {
    "msg": "Vorname: Gilderoy, Nachname: Lockhard, Alter: 43"
}
ok: [localhost] => (item={'name': 'Minerva', 'nachname': 'McGonnagal', 'alter': 67}) => {
    "msg": "Vorname: Minerva, Nachname: McGonnagal, Alter: 67"
}
ok: [localhost] => (item={'name': 'Albus', 'nachname': 'Dumbledore', 'alter': 99}) => {
    "msg": "Vorname: Albus, Nachname: Dumbledore, Alter: 99"
}

PLAY RECAP ********************************************************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Wenn wir ein Dictionary als Loop-Variable verwenden, können wir innerhalb des Tasks also mit "{%raw%}{{ item.FELDNAME }}{%endraw%}" auf die einzelnen Felder
zugreifen.

## Loops und Conditionals

Wenn wir im selben Task einen Loop und eine Condition benutzen, so wird die Condition **für jedes Loop-Item** validiert.

Ein Beispiel mit dem obenstehenden Dictionary:

##### print_dictionary.yml 
``` yaml
{%raw%}
- hosts: localhost
  vars:
    people:
      - name: Gilderoy
        nachname: Lockhard
        alter: 43
      - name: Minerva
        nachname: McGonnagal
        alter: 67
      - name: Albus
        nachname: Dumbledore
        alter: 99
       
  tasks:
    - name: list people oder than 90
      debug:
        msg: "{{ item.name }} Nachname: is older than 90"
      loop: "{{ people }}"
      when: "item.alter > 90"
      
 {%endraw%}
```
```
PLAY [localhost] **************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [localhost]

TASK [list people oder than 90] ***********************************************************************************************************************************************************************************
skipping: [localhost] => (item={'name': 'Gilderoy', 'nachname': 'Lockhard', 'alter': 43}) 
skipping: [localhost] => (item={'name': 'Minerva', 'nachname': 'McGonnagal', 'alter': 67}) 
ok: [localhost] => (item={'name': 'Albus', 'nachname': 'Dumbledore', 'alter': 99}) => {
    "msg": "Albus Nachname: is older than 90"
}

PLAY RECAP ********************************************************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Wie wir sehen können, wurden zwei Items (Gilderoy, Minerva) übersprungen, da für diese die Condition (alter > 90) nicht erfüllt ist. Nur der gute alte Albus übertrifft
die 90 :)


## Loop Control

In manchen Fällen ist es notwendig, das Verhalten von Loops zu beeinflussen. Zum Beispiel kann man den Namen der Index-Variable (standardmäßig "item") selbst bestimmen. Dies ist zum Beispiel dann interessant, wenn wir einen Loop innerhalb eines anderen Loops nutzen wollen.


### Ändern des Namens der Index-Variable

Ein Loop in einem anderen Loop? Hä? Wer mach denn sowas? 
In manchen Fällen kann das sogar sinnvoll sein. Hier haben wir zum Beispiel ein Playbook, das für mehrere Items ein Task-File einbindet:

```yaml
{% raw %}
- hosts: all
  tasks:
    - name: include tasks for every list item
      include: my-tasks.yml
      loop:
        - "A"
        - "B"
        - "C"
{% endraw %}
```

Das File my-tasks.yml könnte in der simpelsten Form zum Beispiel so aussehen:


```yaml
{% raw %}
- name: print item
  debug:
    msg: "{{ item }}"
{% endraw %}
```

Bei der Ausführung wird also das File "my-tasks" für jedes im Playbook unter "loop" gelistete Item inkludiert. Im File my-tasks.yml wird der Inhalt der
Loop-Variable ausgegeben.

Möchten wir jetzt aber im File my-tasks.yml einen weiteren Task mit Loop verwenden haben wir ein kleines Problem: Die Variable "item" wird schon vom "äußeren" Loop verwendet. Dies lässt sich nun mit loop_control lösen:

```yaml
{% raw %}
- name: print item
  debug:
    msg: "{{ item }}"
    
 - name: weiterer loop
   debug:
     msg: "{{ item_2 }}"
   loop:
     - "D"
     - "E"
     - "F"
   loop_control:
     index_var: item_2
 {% endraw %}
```

### Pause zischen den einzelnen Items

Auch können wir über die Loop Control zum Beispiel definieren, wie viel Pause Ansible zwischen der Bearbeitung der einzelnen Items machen soll:

```yaml
{% raw %}
- hosts: localhost
  tasks:
    - name: include tasks for every list item
      debug: 
        msg: "{{ item }}"
      loop:
        - "A"
        - "B"
        - "C"
      loop_control:
        pause: 5 # <= 5 Sekunden Pause zwischen jedem Item
{% endraw %}
```
So das war's dann auch schon wieder. Teil 13 wird eine Zusammenfassung des bisher gelernten sein und hoffentlich nicht wieder so lange dauern :)

Bis dahin!

Mow


