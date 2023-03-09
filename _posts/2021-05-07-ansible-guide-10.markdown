---
layout: post
title:  "Ansible Starter-Guide: #010 - Loops 1"
date:   2021-05-07 10:10:42 +0100
categories: Ansible
---

Heyho! In diesem Teil des Ansible Tutorials beschäftigen wir uns mit Loops (Schleifen). Da es auch zu diesem Thema einiges zu erzählen gibt, werde ich den Themenblock wieder in (mindestens) zwei Teile splitten. In diesem Teil beginnen wir mit den Basics und der einfachen Verwendung von Loops, in den folgenden Teilen zeige ich euch dann noch einige weitere Details und Besonderheiten dazu.

Einen Loop setzten wir immer dann ein, wenn wir einen bestimmten Task mehrfach wiederholen möchten. Ein klassisches Beispiel wäre hier zum Beispiel das Setzen von Dateiberechtigungen auf eine Liste von Dateien und Verzeichnissen. Bei zehn verschiedenen Pfaden bräuchten wir dafür schon 10 verschiedene Tasks, also ein ganz schöner Overhead an Code. Mit der Hilfe von Loops können wir das in einer einzigen Task-Definition lösen. Auch ermöglichen uns Loops die Ausführung von Tasks auf Listen, die wir uns während der Laufzeit erstellen.

Schauen wir uns mal einen Task an, für den ein simpler Loop definiert ist:

```yaml
- hosts: all
  tasks:
    - name: task mit loop
      debug: 
        msg: "Ich bin: {%raw%}{{ item }}{%endraw%}"
      loop:
        - "Loop-Item 1"
        - "Loop-Item 2"
        - "Item 3"
        - "Noch ein Item"
```
<!-- excerpt-end -->

Wichtig sind hier vor allem zwei Dinge. Zum einen der neue Parameter auf Task-Ebene “loop”. Unter diesem können wir entweder (wie im Beispiel) direkt eine Liste mit Werten angeben, oder aber eine Variable nutzen. Dazu aber gleich mehr. Zum Anderen nutzen wir in der Message die Variable “{%raw%}{{ item }}{%endraw%}”. Diese ist immer dann automatisch verfügbar, wenn wir einen Loop für unseren Task definieren und enthält dann je Durchlauf den entsprechenden Wert aus der Liste.

Die Ausgabe des obigen Beispiels würde dann in etwa so aussehen:

```
PLAY [localhost] ********************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************
ok: [localhost]

TASK [task mit loop] ****************************************************************************************************************************************************
ok: [localhost] => (item=Loop-Item 1) => {
    "msg": "Ich bin: Loop-Item 1"
}
ok: [localhost] => (item=Loop-Item 2) => {
    "msg": "Ich bin: Loop-Item 2"
}
ok: [localhost] => (item=Item 3) => {
    "msg": "Ich bin: Item 3"
}
ok: [localhost] => (item=Noch ein Item) => {
    "msg": "Ich bin: Noch ein Item"
}

PLAY RECAP **************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

Wir haben also einen Task definiert, der pro Item unter "loop" einmal ausgeführt wird. Und zwar jedes Mal mit dem entsprechenden Wert aus dem Loop in der Variable "item".

Schauen wir uns das Ganze an einem praktischen Beispiel an. Nehmen wir an, wir möchten auf unseren Webservern mehrere Verzeichnisse anlegen: 

* /var/www/html/my_icons
* /var/www/html/my_documents
* /var/www/html/my_other_stuff

Die 3 Werte möchten wir in diesem Beispiel nicht direkt unter dem Parameter "loop" definieren, sondern vorher in einer Variable definieren. So können wir den Inhalt z.B über die Host- und Groupvars variieren. 

Wir legen uns also zunächst eine Variable in den Groupvars für "webservers" an:

##### ~/ansible-guide/group_vars/webservers/main.yml
```yaml
---
dirs_to_create: 
  - "/var/www/html/my_icons"
  - "/var/www/html/my_documents"
  - "/var/www/html/my_other_stuff"
```

Die Variable "dirs_to_create" hat den Typ "list" und ist nun für alle Hosts in der Gruppe "webservers" verfügbar. Nun erstellen wir uns noch ein passendes Playbook:

##### ~/ansible-guide/loops1.yml
``` yaml
---
- hosts: webservers
  tasks:
    - name: create directories
      file:
        path: "{%raw%}{{ item }}{%endraw%}"
        state: directory
      loop: "{%raw%}{{ dirs_to_create }}{%endraw%}"
      become: true
      
```

Wir erstellen also einen Task mit dem Modul “file”. Mit diesem können wir Dateien und Verzeichnisse verwalten. Unter “loop” geben wir in diesem Fall nicht die Auflistung direkt, sondern die Variable “dirs_to_create” an. Ansible erkennt, dass es sich hierbei um eine Liste handelt und wendet “loop” darauf an. Die Liste enthält alle Pfade, die wir anlegen möchten. Wir geben im Modul-Parameter “path” also die Loop-Variable “item” an und definieren mit “state: directory”, dass Ansible ein Verzeichnis anlegen soll.

Dann führen wir das Playbook mal aus:
```bash
ansible-playbook -i inventory.txt loops1.yml
```
```
PLAY [webservers] ********************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [create directories] ***********************************************************************************************************************************************
changed: [ansible-guide-1] => (item=/var/www/html/my_icons)
changed: [ansible-guide-1] => (item=/var/www/html/my_documents)
changed: [ansible-guide-1] => (item=/var/www/html/my_other_stuff)
changed: [ansible-guide-2] => (item=/var/www/html/my_icons)
changed: [ansible-guide-2] => (item=/var/www/html/my_documents)
changed: [ansible-guide-2] => (item=/var/www/html/my_other_stuff)

PLAY RECAP **************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-guide-2                 : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Der Vollständigkeit halber prüfen wir noch, ob die Verzeichnisse auf den Server auch tatsächlich angelegt wurden:

```bash
ssh ansible-guide-1
ls -rtl /var/www/html
```
```
sseibold@sseibold-t470p:~/blog$ ls -rtl /var/www/html/
insgesamt 16
-rw-r--r-- 1 root root   22 Feb 21 10:33 index.html
drwxr-xr-x 2 root root 4096 Mai  4 17:02 my_icons
drwxr-xr-x 2 root root 4096 Mai  4 17:02 my_documents
drwxr-xr-x 2 root root 4096 Mai  4 17:02 my_other_stuff
```

Sieht gut aus! Damit hättet ihr auch schon die Basis um weiter mit Loops zu arbeiten. Ihr könnt diese natürlich nicht nur im Modul "file" und "debug" nutzen, sondern mit so gut wie jedem Modul! 

Das soll es für diesen Teil auch schon wieder gewesen sein. Im nächsten Beitrag zeige ich euch dann noch einige Besonderheiten und fortgeschrittene Anwendungen für Loops.

Wie immer freue ich mich über jegliche Art von konstruktivem Feedback! 

Bis denne

Mow




