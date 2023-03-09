---
layout: post
title:  "Ansible Starter-Guide: #009 - Conditionals 2" 
date:   2021-04-27 12:45:00 +0100
categories: Ansible
---


Servus zusammen! Wie versprochen möchte ich euch in diesem Teil nun einige Beispiele zeigen, wie wir Conditionals in Ansible verwenden können. 
Mein Test-Setup habe ich mir zu diesem Zweck um einen weiteren Host - diesmal mit einem CentOS Betriebssystem erweitert:

#### ansible-guide-4
* OS: CentOS 8
* IP: 192.168.0.14

Unser neues Inventory sieht dann so aus:

##### ~/ansible-guide/inventory.txt
```
[webservers]
ansible-guide-1 ansible_ssh_user=ansible ansible_host=192.168.0.11
ansible-guide-2 ansible_ssh_user=ansible ansible_host=192.168.0.12

[db]
ansible-guide-3 ansible_ssh_user=ansible ansible_host=192.168.0.13
ansible-guide-4 ansible_ssh_user=ansible ansible_host=192.168.0.14

```

<!-- excerpt-end -->

Wir möchten nun also 4 Hosts mit Ansible verwalten. Drei davon auf Ubuntu-Basis und eines mit einem CentOS. Unsere beiden Webserver sind dabei identisch. Bei
den Datenbank-Servern dagegen setzen wir 2 unterschiedliche Betriebssysteme ein. 

## Beispiel 1: Paketinstallation auf unterschiedlichen Distributionen

Nehmen wir an, wir möchten nun auf beiden Datenbankservern den PostgreSQL-Server installieren. In den vorherigen Beispielen haben wir dazu das Modul "apt" genutzt. Bei unserem Server mit dem Ubuntu wird das auch weiterhin funktionieren. CentOS nutzt zum Verwalten von Paketen allerdings ein Tool mit dem Namen "yum". Glücklicherweise bringt Ansible auch hier bereits ein Modul von Haus aus mit:

[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html)

Doch wie bringen wir Ansible nun dazu, für jeden Host das richtige Modul zu nutzen. Das machen wir mit (... Trommelwirbel ...) Conditionals!
Besser gesagt, nutzen wir hier zwei Werkzeuge, die wir bereits kennen. Zum einen benötigen wir Facts, denn diese enthalten die Informationen über das Host-Betriebssystem. Zum Anderen Conditionals, um die Tasks von den Facts abhängig zu machen.

##### setup-postgres.yml
```yaml
- hosts: db
  gather_facts: true
  tasks:
    - name: install postgresql on debian based systems (Ubuntu)
      become: true
      apt:
        name: postgresql
        update_cache: true
      when: "ansible_os_family == 'Debian'"
      
    - name: install postgresql on Redhat based systems (CentOS)
      become: true
      yum:
        name: postgresql
      when: "ansible_os_family == 'RedHat'"
```

Wir nutzen hier den Fact 'ansible_os_family', der den Basis-Typen der eingesetzten Distribution beinhaltet. Ubuntu basiert auf einem Debian, wogegen CentOS in der Basis ein RedHat-Betriebssystem ist. Weitere mögliche Werte wären hier z.B. "Suse" oder auch "Gentoo".

Dann führen wir unser Playbook mal aus:
```bash
ansible-playbook -i inventory.txt setup-postgres.yml
```

Die Ausgabe sollte dann in etwa so aussehen:
```
PLAY [db] ********************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************
ok: [ansible-guide-3]
ok: [ansible-guide-4]

TASK [install postgresql on debian based systems (Ubuntu)] **************************************************************************************************************
changed: [ansible-guide-3]
skipping: [ansible-guide-4]

TASK [install postgresql on Redhat based systems (CentOS)] **************************************************************************************************************
skipping: [ansible-guide-3]
changed: [ansible-guide-4]

PLAY RECAP **************************************************************************************************************************************************************
ansible-guide-3                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0 
ansible-guide-4                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0 
```

Wie wir sehen können, wurde für beide Install-Tasks nun jeweils der Host geskippt, bei dem die Condition nicht zutreffend war. Wir können somit Tasks vom eingesetzten Betriebssystem abhängig machen. Besonders für Umgebungen mit verschiedenen Distributionen, macht uns dies das Leben sehr viel einfacher!

## Beispiel 2: Einen Task auf einem bestimmten Host einschränken

Ab und an kommt es vor, dass wir einen Task im Playbook nur auf einem bestimmten Host ausführen möchten. Nehmen wir an, der 2. Webserver soll unser Entwicklungs- und Testsystem werden und wir wollen daher noch einen User für den Entwickler anlegen.

Dafür springe ich etwas zurück und erweitere das Playbook aus Teil 4, in dem wir den Webserver installiert und gestartet haben:

##### webservers.yml
``` yaml
 - name: webserver install 
   hosts: webservers
   tasks: 
     - name: install apache2
       become: true
       apt:
         name: apache2
         state: present
         update_cache: yes
       become: true

    - name: enable and start apache2 systemd service
      become: true
      systemd:
        name: apache2
        enabled: true
        state: started
    
    - name: add dev user on second webserver
      become: true
      user: 
        name: herbert
        state: present
      when: "inventory_hostname == 'ansible-guide-2'"
```

Wir machen uns hier die Ansible Variable "inventory_hostname" zunutze, die in jedem Play (auch mit deaktiviertem Fact-Gathering) existiert und den jeweils
aktuellen Hostnamen enthält so wie er im Inventory definiert ist. Wichtig: Das muss nicht zwingend der tatsächliche Hostname des Systems sein. Diesen enthält der Fact (Gathering muss aktiviert sein) "ansible_hostname".

Im obigen Beispiel-Playbook wird der Task also nur ausgeführt, wenn der Name des aktuellen Hosts exakt "ansible-guide-2" ist.

## Beispiel 3: Task nur ausführen, wenn der Host Mitglied einer bestimmten Gruppe ist

Möchten wir einen Play auf für eine bestimmte Gruppe ausführen, so definieren wir das über den Parameter "hosts" im Playbook. Aber auch, wenn wir einen Play zum 
Beispiel auf alle Hosts ausführen, können wir die Ausführung einzelner Tasks bei Bedarf von der Gruppenzugehörigkeit abhängig machen:

##### all-hosts.yml

``` yaml
 - name: webserver install 
   hosts: all
   tasks: 
     - name: add default user to all systems
       become: true
       user:
         name: technik
         state: present
       become: true
    
    - name: add dbadmin user on db servers
      become: true
      user: 
        name: dbadmin
        state: present
      become: true
      when: "'db' in group_names"
```

In diesem Beispiel führen wir den Play auf alle Hosts in unserem Inventory aus. Auf allen Systemen wird der User 'technik' angelegt. Für den User 'dbadmin' haben wir einen Condition definiert, die prüft, ob der String 'db' in der von Ansible bereitgestellten Liste 'group_names' enthalten ist. Das ist eine Liste, die für jeden Host im aktuellen Play existiert und die Namen aller Gruppen enthält, welchen der Host zugehört.

# Zusammenfassung

Mit der Kombination aus Facts und Conditions, können wir unsere Playbooks sehr flexibel gestalten und so z.B. auch in heterogenen Infrastrukturen mit Ansible arbeiten. Bei Ansible gibt es sehr viele Wege, die zum selben Ziel führen, wichtig ist, hier ein Gefühl dafür zu bekommen, welche Wege am besten zu unserem jeweiligen Anwendungsfall passen. Nutze ich zum Beispiel ein Playbook und arbeite darin mit Conditions oder Teile ich mir die Tasks in separate Playbooks auf. 

Ich werde im weiteren Verlauf dieses Guides versuchen, wann immer möglich Best Practices und eigene Erfahrungen dazu mit einzubringen. 

Im nächsten Teil werden wir uns mit einer weiteren Kontrollstruktur in Ansible beschäftigen, den sogenannten Loops (Schleifen). Ich hoffe, die Pause zwischen den Beiträgen ist diesmal etwas kürzer. 

Bis dahin!

Der mow
