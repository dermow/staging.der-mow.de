---
layout: post
title:  "Ansible Starter-Guide: #005 - Variablen" 
date:   2021-02-21 12:20:42 +0100
categories: 
  - Ansible
---

Heyho und willkommen zurück zur Ansible Starter-Guide. In diesem fünften Teil der Reihe möchte ich euch zeigen, wie Variablen funktionieren und wie wir
mit ihnen mit Unterschieden zwischen verschiedenen Systemen umgehen können.

Die simpelste Anwendung von Variablen ist hierbei, mehrfach vorkommende Werte (z.B. Dateipfade) in einer Variable zu speichern. 

Lasst uns direkt mit einem Beispiel beginnen. Nehmen wir an, wir möchten eine index.html und eine style.css an unseren Webserver ausliefern. Zur besseren Übersicht 
packe ich das Ganze in ein eigenes Playbook. Ihr könnt aber auch einfach das webservers.yml aus dem letzten Teil erweitern.

##### ~/ansible-guide/playbook-without-vars.yml

```yaml
- name: Play ohne Variablen
  hosts: webservers
  tasks:
    - name: copy index.html
      copy: 
        src: files/index.hmlt
        dest: /var/www/html/index.html
      become: true
      
   - name: copy style.css
     copy:
       src: files/style.css
       dest: /var/www/html/style.css
     become: true
```

<!-- excerpt-end -->

Je weiter wir unser Playbook stricken, desto häufiger werden wir den Pfad "/var/www/html" verwenden müssen. Es wäre doch super, wenn wir diesen einfach in ein Variable packen können. Dann machen wir das doch:

##### ~/ansible-guide/playbook-with-vars.yml

```yaml
- name: Play mit Variable
  hosts: webservers
  vars:
    my_docroot: /var/www/html   # <-- Definition der Variable
  tasks:
    - name: copy index.html
      copy: 
        src: files/index.hmlt
        dest: {% raw %}"{{ my_docroot }}/index.html" # <-- Nutzung der Variable {% endraw %}
      become: true
      
   - name: copy style.css
     copy:
       src: files/style.css
       dest: {% raw %}"{{ my_docroot }}/style.css" {% endraw %}
     become: true
```

Dies ist die simpelste Definition von Variablen. Wir haben die Variable "my_docroot" ganz einfach im Playbook definiert und können auf sie nun im gesamten Play "Play mit Variable" zugreifen. Um Variablen zu verwenden, nutzen wir folgendes Format {% raw %}"{{ variablen_name }}"{% endraw %}. Wichtig ist hier, dass wir den gesamten String dafür in Quotes (") setzen müssen.

Variablen können auch auf Taskebene definiert werden. Das sähe dann so aus:

##### ~/ansible-guide/playbook-with-task-vars.yml

``` yaml
- name: Play mit Variablen auf Taskebene
  hosts: webservers
  tasks:
    - name: copy index.html
      copy: 
        src: files/index.hmlt
        dest: {% raw %}"{{ my_docroot }}/index.html" # <-- Nutzung der Variable {% endraw %}
      become: true
      vars:
        my_docroot: /var/www/html
      
   - name: copy style.css
     copy:
       src: files/style.css
       dest: {% raw %}"{{ my_docroot }}/style.css" {% endraw %}
     become: true
     vars:     
       my_docroot: /var/www/html
```

Das macht aber nur in ganz speziellen Fällen Sinn, wenn tatsächlich nur ein bestimmter Task diese Variable benötigt.

Im obigen Beispielen nutzen wir eine Variable des Typs "string", also eine einfache Zeichenkette. Es gibt aber noch weitere Variablen-Typen, auf die wir im Verlauf der Starter-Guide noch genauer eingehen werden.

### Variablen-Typen

#### Strings: Zeichenketten

```yaml
my_string: Hello World!
````

#### Numbers: Ganzzahlen

Wie der Name schon sagt, ist dieser Typ für numerische Werte zuständig. 

```yaml
number_of_files: 8
```

#### Float: Gleitkommazahlen
Diesen Datentyp nutzen wir immer dann, wenn es um Gleitkommazahlen geht:

``` yaml
my_float: 2.5
```

#### List
Weiter kann man in Ansible auch einfache Listen definieren
```yaml
files_to_copy:
  - file1.txt
  - file2.txt
  - file3.txt
```

#### Dictionary:
Oder etwas komplexere "Listen"
```yaml
files_to_copy:
  - filename: file2.txt
    file_src: /opt/file2.txt
    file_dest: /home/me/file.txt
```

#### Boolean:
Schlussendlich noch der Boolean-Datentyp. Diesen nutzen wir immer dann, wenn etwas wahr oder falsch sein kann.
```yaml
copy_files: true
overwrite_files: false
```

## Host- und Groupvars

Die im ersten Beispiel definierte Variable gilt nun für alle Hosts mit demselben Wert. Was machen wir nun aber, wenn wir Werte definieren wollen, die sich für jeden Host unterscheiden? Gehen wir zurück zu unserem Beispiel-Szenario. Nehmen wir an, der Inhalt der index.html unserer beiden Webserver soll sich pro Host unterscheiden, sodass der erste "Ich bin webserver1" und der zweite "Ich bin webserver2" ausgibt.

Dazu nutzen wir sogenannte Host-Variablen (host_vars). Um diese zu nutzen, legen wir uns ein neues Verzeichnis in unserem Beispiel-Setup an:

```bash
cd ~/ansible-guide
mkdir -p host_vars/ansible-guide-1
mkdir -p host_vars/ansible-guide-2
```
In jedem Verzeichnis erstellen wir jetzt noch ein File "main.yml"

##### ~/ansible-guide/host_vars/ansible-guide-1/main.yml
```yaml
my_welcome_text: Ich bin webserver1!
```

##### ~/ansible-guide/host_vars/ansible-guide-2/main.yml
```yaml
my_welcome_text: Ich bin webserver2!
```

Wenn wir unser Playbook starten, wird Ansible automatisch alle Dateien unter "host_vars" lesen und die Variablen-Werte den passenden Hosts zuordnen. Wichtig ist hierbei, dass die Namen der Verzeichnisse mit den Namen der Hosts in unserem Inventory übereinstimmen.

Um das nun zu testen, müssen wir noch eine kleine Anpassung an unserem Webserver-Playbook vornehmen. Mit dem Modul "copy" können wir nämlich anstelle einer Quell-Datei auch direkt den gewünschten Inhalt der Zieldatei definieren. Das sieht dann so aus:

#####  ~/ansible-guide/webservers.yml
``` yaml
- name: webserver setup
  hosts: webservers
  tasks:
    - name: Apache2 Setup
      apt:
        name: apache2
        state: present
        update_cache: true
      become: true

    - name: start and enable apache2
      systemd:
        name: apache2
        state: started
        enabled: true
      become: true

    - name: copy index.html
      copy:
        dest: /var/www/html/index.html
        content: {% raw %}"{{ my_welcome_text }}"{% endraw %}
      become: true

    - name: copy apache2.conf
      copy:
        src: files/apache2.conf
        dest: /etc/apache2/apache2.conf
      become: true
```

Beachtet hier die Änderung am Task "copy index.html". Statt eine Quelldatei für das Copy-Modul zu definieren, geben wir direkt den gewünschten Inhalt der
Zieldatei ein und nutzen hierfür unsere Variable "my_welcome_text".

Dann führen wir das Playbook doch mal aus:
```bash
cd ~/ansible-guide
ansible-playbook -i inventory.txt webservers.yml
```
```
PLAY [webserver setup] *******************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [Apache2 Setup] *********************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [start and enable apache2] **********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [copy index.html] *******************************************************************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1]
changed: [ansible-guide-2]

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1          : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-guide-2          : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Der Task "copy index.html" hat nun für beide Hosts den Status "changed" und somit den neuen Inhalt für die index.html übernommen. Ob wir den gewünschten Effekt erzielt haben können wir nun wieder mit curl, oder dem Browser prüfen:
```bash
curl http://192.168.0.11
<h1>Ich bin webserver1!</h1>

curl http://192.168.0.12
<h1>Ich bin webserver2!</h1>
```

BAM! Sieht gut aus! Wir haben also erfolgreich hostspezifische Variablen verwendet. 

Das Ganze funktioniert auch mit Gruppen. Dazu dienen die sogenannten group_vars. Auch dafür erstellen wir uns ein neues Playbook und nutzen dafür das nützliche Modul "debug" um die Variablen zu testen.

#### ~/ansible-guide/groupvars-test.yml
In Teil 4 dieses Guides haben wir ein Inventory mit zwei verschiedenen Gruppen angelegt "webservers" und "db". Wir erstellen also ein Playbook, welches genau diese beiden nutzt.

```yaml
- name: group_vars showcase
  hosts:
    - webservers
    - db
  tasks:
    - name: Debug Ausgabe
      debug:
        msg: {% raw %}"{{ test_text }}"{% endraw %}
```

Das Modul "debug" ist sehr hilfreich, wenn wir Playbooks erstellen und zwischendurch testen wollen. Wir können uns damit z.B. Werte von Variablen ausgeben lassen. In unserem Fall definieren wir für das Modul einen Parameter:

**msg**: Beliebiger String, der dann während dem Play ausgegeben wird

Jetzt müssen wir die Variable "test_text" nur noch definieren. Und zwar für jede Gruppe anders. Wir legen uns auf derselben Ebene wie "host_vars" ein weiteres Verzeichnis an:

```bash
mkdir ~/ansible-guide/group_vars
mkdir ~/ansible-guide/group_vars/webservers
mkdir ~/ansible-guide/group_vars/db
```

In beiden Verzeichnissen erstellen wir uns eine "main.yml":

##### ~/ansible-guide/group_vars/webservers/main.yml
```yaml
test_text: Ich bin ein Host in der Gruppe 'webservers'!
```

##### ~/ansible-guide/group_vars/db/main.yml
```yaml
test_text: Ich bin ein Host in der Gruppe 'db'!
```

Wir sind also genauso vorgegangen wie auch schon bei den host_vars, nur diesmal eben auf Gruppen-Ebene. Lasst uns nun das neue Playbook ausführen:

```bash
cd ~/ansible-guide
ansible-playbook -i inventory.txt groupvars-test.yml
```
```
ASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]
ok: [ansible-guide-3]

TASK [Debug Ausgabe] *********************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => {
    "msg": "Ich bin ein Host in der Gruppe 'webservers'!"
}
ok: [ansible-guide-2] => {
    "msg": "Ich bin ein Host in der Gruppe 'webservers'!"
}
ok: [ansible-guide-3] => {
    "msg": "Ich bin ein Host in der Gruppe 'db'!"
}
PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-guide-2                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-guide-3                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Wie wir oben sehen, zieht jeder der drei Hosts nun die gruppenspezifische Variable "test_text" und gibt diese entsprechend aus.


## Zusammenfassung

In diesem Kapitel hab ihr gelernt, was Variablen sind und wie wir diese an verschiedenen Stellen definieren können. Im nächsten Kapitel möchte ich euch 
dann Facts vorstellen. Diese sind den Variablen sehr ähnlich, es gibt aber einige Unterschiede!

Bis dahin!

Der Mow

