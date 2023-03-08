---
layout: post
title:  "Ansible Starter-Guide: #011 - check & diff"
date:   2021-05-10 16:35:00 +0100
categories: Ansible
---

Servus liebe Leser! 

Ich hab gerade nochmal die bisherigen Beiträge der Ansible-Guide durchgelesen und dabei mit Schrecken festgestellt, dass ich vergessen habe, zwei wichtige Ansible-Features zu erwähnen. Es handelt sich zum einen um den Check-Mode, mit dem wir Playbooks in einem Testmodus starten können. Dabei werden etwaige Änderungen nur in der Ausgabe angezeigt, aber nicht tatsächlich durchgeführt. Zum Anderen gibt es noch den Diff-Mode, in welchem uns in der Ausgabe die Änderungen im “Vorher-Nachher Modus” angezeigt werden.

### Check-Mode

Um den Check-Mode zu nutzen, müssen wir einfach ein "--check" an unser Command anhängen. Hier am Beispiel unseres Playbooks setup-postgres.yml aus dem letzten
Teil:

```bash
ansible-playbook -i inventory.txt setup-postgres.yml --check
```
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

<!-- excerpt-end -->

Die Ausgabe ist identisch zu der ohne Check-Mode, jedoch wurden die Änderungen nicht auf den Hosts durchgeführt.

Das Ganze funktioniert im Prinzip mit jedem Playbook, es gibt jedoch einige Besonderheiten:

#### Abhängigkeiten
Hängt ein Task von einem vorherigen ab, zum Beispiel weil ein Paket installiert wird und im Anschluss der entsprechende Service gestartet, wird der folgende Task im Check-Mode vermutlich fehlschlagen, da das entsprechende Paket nicht wirklich installiert wurde.

#### Shell Commands
Die Module “command” bzw. “shell” werden zwar, wie auch im normalen Modus den Status “changed” zurückliefern, die tatsächliche Auswirkung, kann aber nicht geprüft werden, da das Ergebnis des Shell commands unabhängig von Ansible ist.

Für solche Fälle können wir einzelne Tasks im Check-Mode überspringen:
```yaml
 - name: Diesen Task im Checkmode überspringen
   lineinfile:
      line: "important config"
      dest: /path/to/myconfig.conf
      state: present
    check_mode: no
```

Mit dem Task-Parameter “check_mode: no” teilen wir Ansible mit, dass es den entsprechenden Task im Checkmode nicht ausführen soll.

### Diff-Mode
Der Diff-Mode ist unabhängig vom Check-Mode, kann aber mit diesem zusammen genutzt werden. Im Diff-Mode zeigt Ansible alle Änderungen im Vergleich zum Ursprungszustand an. Hier ein kleines Beispiel mit dem Modul “lineinfile”:

```yaml
- hosts: ansible-guide-1
  tasks:
    - name: add entry to /etc/hosts
      lineinfile:
         path: /etc/hosts
         line: "192.168.0.3  ansible-guide-3"
         regexp: "^192.168.0.3"
      become: true
```

Das Playbook starten wir nun im Check- und Diff-Modus:
```bash
ansible-playbook -i inventory.txt lineinfile_diff.yml --check --diff
```
Ausgabe:
```
PLAY [ansible-guide-1] ******************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************
cok: [ansible-guide-1]

TASK [add entry to /etc/hosts] ****************************************************************************************************
--- before: /etc/hosts (content)
+++ after: /etc/hosts (content)
@@ -8,3 +8,4 @@
 ff00::0 ip6-mcastprefix
 ff02::1 ip6-allnodes
 ff02::2 ip6-allrouters
+192.168.0.3  ansible-guide-3

changed: [ansible-guide-1]

PLAY RECAP ************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Wenn das Ergebnis für uns ok ist, können wir das Command erneut aufrufen und diesmal das "--check" weglassen.

Das wärs auch schon! Im nächsten Teil machen wir dann weiter mit Teil 2 der Loops!

Viele Grüße

Mow
