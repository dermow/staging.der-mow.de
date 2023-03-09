---
layout: post
title:  "Ansible Starter-Guide: #001 - Allgemeines"
date:   2021-02-12 14:04:42 +0100
categories: Ansible
---

So! Willkommen zur Ansible-Starter-Guide. Wie in der Ankündigung angedroht, quatsch ich euch jetzt ein bisschen mit den wichtigsten Aspekten von 
Ansible zu. Einerseits werd ich versuchen, alle Mechaniken möglichst ausführlich und gleichzeitig verständlich zu erklären, das ganze aber auch mit
praktischen Beispielen zu untermalen. Ich hoff mal das klappt! Feedback eurerseits ist auf jeden Fall immer mehr als willkommen! 


## Was ist Ansible?

Anfangen sollten wir ganz vorne. Also was ist dieses Ansible eigentlich von dem alle sprechen? Ganz kurz gesagt ist es ein IT-Automation-Tool.
Viele werden vor allem im Zusammenhang mit Config-Management davon gehört haben, aber Ansible kann viel, VIEL mehr. Ein paar kleine Beispiele:

* Paketinstallationen
* Bereitstellen kompletter virtueller Machinen oder Cloud-Instanzen (AWS EC2, Azure VMs)
* Konfigurieren der oben genannten
* Installation & Konfiguration von Anwendungen

<!-- excerpt-end -->

## Wie funktioniert Ansible?

Ansible arbeitet per SSH, sprich es wird keinerlei Agent-Software (außer natürlich dem ssh agent) auf den Zielsystemen benötigt. Vielmehr muss der Ansible-Controller (Kiste auf der Ansible ausgeführt wird) eine SSH-Verbindung zu den Ansible-Zielen aufbauen können. Zu erledigende Tasks werden dann als Python-Scripts vorbereitet, auf das Ziel kopiert und ausgeführt. Das Ergebnis erhält der Ansible-Controller dann wiederum zurück.

## Wichtig: Idempotenz und der deklarative Ansatz

Kommt eigentlich erst später, ist mir aber so wichtig, dass ich es nicht lassen kann euch damit jetzt schon auf die Nerven zu gehen. Wenn wir mit Ansible arbeiten deklarieren wir Dinge, wir führen keine Skripte aus. 

Einfach zusammengefasst bedeutet das folgendes:

Wird ein Ansible-Playbook zum ersten mal ausgeführt, wird es sehr wahrscheinlich Dinge ändern um den im Playbook (Erklärung folgt) deklarierten Status herzustellen. Führe ich nun das selbe Playbook, ohne Veränderung der Parameter oder der Zielsysteme erneut aus, darf es keine Änderungen mehr geben. Das ganze nennt sich dann [Idempotenz](https://de.wikipedia.org/wiki/Idempotenz).

Erstmal müsst ihr euch darüber aber keine weiteren Gedanken machen. Lasst uns also erstmal die verschiedenen Begrifflichkeiten und Mechaniken näher betrachten:

## Inventory

Das Ansible Inventory ist die Definition aller Ziel-Maschinen, auf die wir mit Ansible zugreifen möchten. Ein Inventory kann in mehrere Gruppen unterteilt werden.

Beispiel-Inventory:

``` ini
[webservers]
web1.example-company.intra

[databases]
db1.example-comany.intra

```

## Module

Module sind die wichtigsten Werkzeuge in Ansible. Sie führen die eigentliche Arbeit aus. z.B gibt es ein Ansible-Modul um Pakete auf Debian zu installieren ([apt](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)), eine oder mehrere
Zeilen an eine Datei anzuhängen ([lineinfile](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html)) oder eine Datei aus einer Vorlage zu erstellen ([template](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)). Jedes Modul hat sein eigenes Set an Parametern.

Wer jetzt schonmal einen ersten Eindruck gewinnen möchte, wie viele Möglichkeiten Ansible bietet findet hier eine Liste mit allen integrierten Modulen:

[https://docs.ansible.com/ansible/2.8/modules/list_of_all_modules.html](https://docs.ansible.com/ansible/2.8/modules/list_of_all_modules.html)

Sind ein paar oder? :)

Aber nicht von der Menge verunsichern lassen. Uns reichen erstmal schon ein paar wenige - ganze einfache um echt coole Sachen damit umzusetzen!


## Tasks

Ein Task definiert eine Aufgabe, die pro Zielhost zu tun ist. Eine Task-Definition besteht aus dem Aufruf eines Moduls und der Definition der Modul-Parameter. Hier ein kurzes Beispiel:


```yaml

- name: Ich bin ein Task
  apt: # <<--- Name des Moduls
    name: apache2
    state: present

```

Der im Beispiel genannte Task soll dafür sorgen, dass auf dem entfernten Debian-System das Paket mit dem Namen "apache2" installiert ist.

## Playbooks

Ein Playbook ist eine Sammlung eines oder mehrerer Plays (Erklärung folgt). Ein Play ist wiederum die Ausführung eines, meist aber mehrerer Tasks.

### Beispiel: Apache-Installation

```yaml
- hosts: webservers
  tasks:
    - name: install apache2
      apt:
        name: apache2
        state: present

    - name: start apache2
      systemd: 
        name: apache2
        state: started

```

## Plays

Ein Play definiert ein Set an Tasks, welches auf bestimmten Hosts ausgeführt werden soll. Im Beispiel oben also, die Installation und Start des Apache-Webservers auf alle Hosts in der Gruppe "webservers".

### Beispiel: Mehrere Plays in einem Playbook

``` yaml
# Play 1
- name: webserver play
  hosts: webservers
  tasks:
    - name: install apache2
      apt:
        name: apache2
        state: installed
    
    - name: start apache2
      systemd:
        name: apache2
        state: started

# Play2
- name: database server play
  hosts: databases
  tasks:
    - name: install mysql
      apt:
        name: mysql-server
        state: installed

    - name: start mysql
      systemd:
        name: mysql
        state: started

```

## Handler

Handler sind im Grunde Tasks, die nur bei Bedarf aufgerufen werden. In Ansible nennt man das dann "to notify" also "benachrichtigen". z.B können wir unser Playbook so gestalten, 
dass ein Task eine Config-Datei kopiert und nur bei Änderungen an dieser, der zugehörige Dienst neu gestartet wird. 

### Beispiel:

```yaml
- hosts: webservers
  handlers:
    - name: restart-apache
      systemd:
        name: apache2
        state: restarted
  tasks:
    - name: install apache
      apt: 
        name: apache
        state: installed
    
    - name: template apache config
      template:
        src: apache-config-template
        dest: /etc/apache2/apache2.conf
      notify: restart-apache # <- Benachrichtigung des Handlers

```

Zu Handlern sind 2 Dinge noch sehr wichtig zu wissen:

* Sie werden nur aufgerufen, wenn der zugehörige Task auch den Status "changed" liefert. 
* Sie werden immer am Ende des Playbooks ausgeführt, NICHT direkt nach dem aufrufenden Task.

## Roles

Rollen sind ein relativ fortgeschrittener Mechanismus, auf den wir später genauer eingehen werden. Kurz gesagt lassen sich bestimmte Aufgaben aber in wiederverwendbare "Pakete" packen und in mehreren Playbooks nutzen.


## To be continued...

Wenn euch das alles jetzt noch etwas weit weg vorkommt - keine Sorge. Wir werden das alles noch bis ins Detail behandeln. Ich versuche auch zwischendrin direkt Anwendungsbeispiele
aufzuzeigen, so dass ihr wenn ihr wollt auch schon beginnen könnt mit Ansible zu arbeiten.

Allgemein finde ich, es gibt keine bessere Methode etwas zu lernen, als es einfach auszuprobieren.

Deshalb werden wir im  nächsten Teil auf die Installation von Ansible eingehen. Was bringt das viele Gerede, wenn man es am Ende nicht selbst ausprobieren kann. Richtig? Richtig!


## Fragen, Kritik, Verbesserungsvorschläge... ?

... dann lasst einfach einen Kommentar auf dieser Seite oder schreibt mir auch gerne eine E-Mail!




Bis dann!

Der Mow