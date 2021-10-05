---
layout: post
title:  "Ansible Starter-Guide: #014 - Rollen"
date:   2021-09-27 07:50:42 +0100
categories: Ansible
---

Moinsn!

wir nähern uns so langsam dem Abschluss der “Starter Guide”, also der Ansible-Grundlagen. Ein großes Thema haben wir allerdings noch offen: Rollen.

Bisher haben wir unseren Ansible-Code in Playbooks geschrieben und diese ausgeführt. Möchten wir nun Ansible-Code schreiben, den wir z.B. projektübergreifend nutzen - oder gar veröffentlichen wollen, stoßen wir mit Playbooks aber recht schnell an Grenzen. Dazu gibt es Ansible Roles. Sie bieten uns die Möglichkeit wiederverwendbaren und parametrisierbaren Ansible Content zu erstellen. Eine Rolle können wir dann wiederum ein - oder mehrere - playbooks einbinden.

Die Struktur einer Ansible Rolle ist in einer Verzeichnisstruktur abgebildet. Dabei sieht eine typische Rolle folgendermaßen aus:

```
# playbooks
webservers.yml
roles/
    my-webserver-role/
        tasks/
          - main.yml
        handlers/
          - main.yml
        files/
        templates/
        vars/
          - main.yml
        defaults/
          - main.yml
        meta/
          - main.yml
 ```

<!-- excerpt-end -->

In diesem Beispiel sehen wir eine Rolle "my-webserver-role". Die einzelnen Unterverzeichnisse möchte ich kurz erklären:

* **tasks** - Hier liegen Files in denen wir unsere Tasks definieren
* **handlers** - Hier liegen handler, falls unserer Rolle welche benötigt
* **files** - Hier legen wir Dateien ab, die wir dann z.B mit dem Copy-Modul nutzen möchten
* **templates** - Benötigen wir Jinja2-Templates, so legen wir diese hier ab
* **vars** - Hierhin können wir rollen-interne Variablendefinitionen verlagern
* **defaults** -  Hier können wir Standardwerte für Variablen definieren, die vom "Rollen-User" überschrieben werden können
* **meta** - Hier werden Informationen zur Rolle selbst abgelegt

## Beispiel

Ein Beispiel für eine ausführlichere Webserver-Rolle könnt ihr übrigens hier finden:

[https://github.com/dermow/ansible-role-httpd](https://github.com/dermow/ansible-role-httpd)

Für unseren Guide möchte ich das Beispiel aber etwas simpler gestalten. Folgende Aufgabenstellung legen wir uns für die Rolle fest:

* Support auf Ubuntu begrenzt
* Installation des Apache-Webservers
* Ausliefern einer individuellen index.html
* Starten und aktivieren des Apache-Services

Vorab ist noch wichtig zu erwähnen, dass Ansible per Default an definierten Orten nach Rollen sucht. Unter anderem in "./roles" im jeweiligen Projektverzeichnis.
Beginnen wir damit uns die Ordnerstruktur anzulegen:

``` bash
mkdir ~/ansible-guide-roles
cd ~/ansible-guide-roles
mkdir -p ./roles/my-webserver-role
cd ./roles/my-webserver-role
mkdir tasks templates handlers vars defaults
```
Was uns fehlt, ist noch ein Inventory - ich nehme dafür nochmal unsere Testumgebung her. Schlau wie ich bin, hab ich mir natürlich vorher einen Snapshot erstellt, und musste diese natürlich nicht nochmal neu installieren (chrm chrm).

##### ~/ansible-guide-roles/inventory.txt
```
[webservers]
ansible-guide-1 ansible_ssh_user=ansible ansible_host=192.168.0.11
ansible-guide-2 ansible_ssh_user=ansible ansible_host=192.168.0.12
```


Im nächsten Schritt legen wir uns eine Datei für unsere Variablendefinitionen an:

##### ~/ansible-guide-roles/roles/my-webserver-role/vars/main.yml
```yaml
---
packages_to_install:
  - apache2
  - libapache2-mod-php
```

Wir haben uns eine Liste definiert, in der festgelegt wird, welche Pakete später installiert werden sollen. Im nächsten Schritt legen wir den passenden
Task dazu an:

##### ~/ansible-guide-roles/roles/my-webserver-role/tasks/main.yml
```yaml
---
- name: install packages
  become: true
  apt:
    name: "{%raw%}{{ item }}{%endraw%}"
    state: present
  loop: "{%raw%}{{ packages_to_install }}{%endraw%}"
```

So - damit hätten wir schonmal eine erste Version der Rolle, die wir testen können. Dazu legen wir uns ein Playbook mit folgendem Code an:

##### ~/ansible-guide-roles/playbook.yml
```yaml
---
- hosts: webservers
  roles:
    - my-webserver-role
```

Es gibt zwei Varianten, eine Rolle in ein Playbook einzubinden. Bei der obigen ist wichtig zu wissen, dass die dort aufgelisteten Rollen **zuerst** ausgeführt werden. Erst danach folgen die im Playbook evtl. definierten Tasks. Wollen wir die Stelle selbst definieren, an der unsere Rolle ausgeführt wird, können wir das Modul "include_role" benutzen:

##### ~/ansible-guide-roles/playbook.yml
```yaml
---
- hosts: webservers
  tasks:
    - name: inlclude my role
      include_role:
        name: my-webserver-role
```


Dann starten wir doch mal unser Playbook:

``` bash
ansible-playbook -i inventory.txt playbook.yml
```
```
PLAY [webservers] *****************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [my-webserver-role : install packages] **************************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1] => (item=apache2)
changed: [ansible-guide-1] => (item=libapache2-mod-php)
changed: [ansible-guide-2] => (item=apache2)
changed: [ansible-guide-2] => (item=libapache2-mod-php)

PLAY RECAP ***********************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
ansible-guide-2                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Top, hat funktioniert: Auf beiden Zielmaschinen  sollte nun der Apache-Webserver und das PHP-Modul installiert  worden sein. Als Nächstes stellen wir noch sicher, dass der Webserver-Dienst gestartet und aktiviert wird:

##### ~/ansible-guide-roles/roles/my-webserver-role/tasks/main.yml
```yaml
---
- name: install packages
  become: true
  apt:
    name: "{%raw%}{{ item }}{%endraw%}"
    state: present
  loop: "{%raw%}{{ packages_to_install }}{%endraw%}"
  
- name: start and enable apache
  become: true
  systemd:
    name: apache2
    state: started
    enabled: true
    
```

Nun möchten wir mit der Rolle noch unsere eigne Config ausliefern. Dazu hab ich mir einfach die "default.conf" aus der Standardinstallation geklaut und die
Kommentierungen entfernt:



```bash
cat /etc/apache2/sites-available/000-default.conf | grep -v "#"
<VirtualHost *:80>

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html


	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

```

Nehmen wir an wir möchten einfach nur den "ServerAdmin" konfigurierbar haben. Wir legen also das entsprechende Template an:

##### ~/ansible-guide-roles/roles/my-webserver-role/tepmlates/000-default.conf
```
<VirtualHost *:80>

	ServerAdmin {%raw%}{{ my_server_admin }}{%endraw%}
	DocumentRoot /var/www/html


	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

Jetzt müssen wir die Variable "my_server_admin" nur noch definieren. Da diese vom "User" überschrieben werden können soll, bietet es sich an das unter
"defaults" zu tun:


##### ~/ansible-guide-roles/roles/my-webserver-role/defaults/main.yml
``` yaml
my_server_admin: me@example.com
```

Und nun fehlt noch der entsprechende Tasks, um das Template auszurollen:

##### ~/ansible-guide-roles/roles/my-webserver-role/tasks/main.yml
```yaml
---
- name: install packages
  become: true
  apt:
    name: "{%raw%}{{ item }}{%endraw%}"
    state: present
  loop: "{%raw%}{{ packages_to_install }}{%endraw%}"
  
- name: start and enable apache
  become: true
  systemd:
    name: apache2
    state: started
    enabled: true
    
- name: template web config
  become: true
  template:
    src: 000-default.conf
    dest: /etc/apache2/sites-available/000-default.conf
    
```

Nun wäre es natürlich noch super, wenn der Webserver bei Änderungen an der Konfiguration automatisch neu gestartet wird. Ein passendes Werkzeug dazu hatten wir in Teil 4 bereits besprochen: Handler.

Wir legen also einen entsprechenden Handler an...

##### ~/ansible-guide-roles/roles/my-webserver-role/handlers/main.yml
```yaml
---
- name: restart-apache
  become: true
  systemd:
    name: "apache2"
    state: restarted
  
```

... Und passen unseren Task entsprechend an:

##### ~/ansible-guide-roles/roles/my-webserver-role/tasks/main.yml
```yaml
---
- name: install packages
  become: true
  apt:
    name: "{%raw%}{{ item }}{%endraw%}"
    state: present
  loop: "{%raw%}{{ packages_to_install }}{%endraw%}"
  
- name: start and enable apache
  become: true
  systemd:
    name: apache2
    state: started
    enabled: true
    
- name: template web config
  become: true
  template:
    src: 000-default.conf
    dest: /etc/apache2/sites-available/000-default.conf
  notify: restart-apache # <---- HANDLER
    
```

Und los:
```bash
ansible-playbook -i inventory.txt playbook.yml
```
```
PLAY [webservers] *****************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [my-webserver-role : install packages] **************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => (item=apache2)
ok: [ansible-guide-1] => (item=libapache2-mod-php)
ok: [ansible-guide-2] => (item=apache2)
ok: [ansible-guide-2] => (item=libapache2-mod-php)

TASK [my-webserver-role : start and enable apache] *******************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [my-webserver-role : template web config] ***********************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1]
changed: [ansible-guide-2]

RUNNING HANDLER [my-webserver-role : restart-apache] *****************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1]
changed: [ansible-guide-2]

PLAY RECAP ***********************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ansible-guide-2                  : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Das sieht doch gut aus :) Bleibt nur noch der letzte Teil unserer To-do-Liste: Eine eigene index.html ausliefern. Mit dem bisher Gelernten sollte dies kein Problem sein: Wir legen uns einfach wieder ein Template an:

##### ~/ansible-guide-roles/roles/my-webserver-role/tepmlates/index.html
```html
<html>
  <head>
    <title>{%raw%}{{ my_website_title }}{%endraw%}</title>
  </head>
  <body>
    {%raw%}{{ my_website_content }}{%endraw%}
  </body>
</html>
```

Wir haben also ein weiteres Template angelegt, in dem wir zwei neue Variablen nutzen wollen. Auch für diese legen wir wieder Standardwerte an:

##### ~/ansible-guide-roles/roles/my-webserver-role/defaults/main.yml
``` yaml
my_server_admin: me@example.com
my_website_title: "Ansible-Guide"
my_website_content: "Hello World"
```

Zuletzt fehlt dann noch der Task um die index.html auszuliefern:

##### ~/ansible-guide-roles/roles/my-webserver-role/tasks/main.yml
```yaml
---
- name: install packages
  become: true
  apt:
    name: "{%raw%}{{ item }}{%endraw%}"
    state: present
  loop: "{%raw%}{{ packages_to_install }}{%endraw%}"
  
- name: start and enable apache
  become: true
  systemd:
    name: apache2
    state: started
    enabled: true
    
- name: template web config
  become: true
  template:
    src: 000-default.conf
    dest: /etc/apache2/sites-available/000-default.conf
  notify: restart-apache # <---- HANDLER
  
- name: template index.html
  become: true
  template:
    src: index.html
    dest: /var/www/html/index.html
    
```

Nehmen wir aber an, wir möchten für unseren Titel nun einen eigenen Wert verwenden und nicht den Default aus der Rolle:

```bash
ansible-playbook -i inventory.txt -e "my_website_title='Mein krasser Titel'" playbook.yml
```
```
PLAY [webservers] *****************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [my-webserver-role : install packages] **************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => (item=apache2)
ok: [ansible-guide-1] => (item=libapache2-mod-php)
ok: [ansible-guide-2] => (item=apache2)
ok: [ansible-guide-2] => (item=libapache2-mod-php)

TASK [my-webserver-role : start and enable apache] *******************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [my-webserver-role : template web config] ***********************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1]
changed: [ansible-guide-2]

TASK [my-webserver-role : template index.html] ***********************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1]
changed: [ansible-guide-2]

RUNNING HANDLER [my-webserver-role : restart-apache] *****************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1]
changed: [ansible-guide-2]

PLAY RECAP ***********************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ansible-guide-2                  : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Fertig! Nun noch ein kurzer Test, ob das ganze auch funktioniert hat:

```bash
curl 192.168.0.11
```
```
<html>
  <head>
    <title>Mein krasser Titel</title>
  </head>
  <body>
    Hello World
  </body>
</html>
```
Das war's! Damit haben wir unsere erste Ansible Rolle erstellt. Ich hoffe ich konnte euch das Grundprinzip näher bringen, diese Rolle (natürlich noch weiter ausgebaut) könnt ihr nun in verschiedenen Projekten und Playbooks verwenden und die Input-Parameter entsprechend variieren.

## PRO TIP

Für sehr viele Anwendungszwecke gibt es bereits fertige Rollen. Schaut einfach mal hier:

[https://galaxy.ansible.com/](https://galaxy.ansible.com/)

## ENDE

Damit sind wir am Ende des Starter-Guides angelangt. Ihr solltet nun in der Lage sein, alleine weiterzulernen und einfach selbst damit in euren Umgebungen zu arbeiten. Ich weiß noch nicht genau, ob ich einen “Advanced Guide” im gleichen Stil starten werde, oder einfach bestimmte Themen einzeln behandle, die mir gerade so in den Kopf kommen.

Über Feedback würde ich mich natürlich sehr freuen. Bis bald

Mow
