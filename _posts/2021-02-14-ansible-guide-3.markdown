---
layout: post
title:  "Ansible Starter-Guide: #003 - Erste Schritte"
date:   2021-02-14 16:01:42 +0100
categories: Ansible
---

Moinsn! Jetzt geht´s endlich zur Sache und wir versuchen uns mal an den ersten Schritten mit Ansible. Ich hoffe ihr habt viel Spaß dabei!

## Mein Beispiel-Setup

Um diesen Teil mit praktischen Beispielen untermalen zu können, hab ich mir ein kleines Test-Szenario aufgebaut. Dieses wird für den restlichen Guide im Grunde gleich bleiben, aber zum Teil um weitere Hosts erweitert werden. 

Wer Lust hat, kann sich gerne ein ähnliches Szenario nachbauen und die Beispiele einfach mit ausprobieren.

So fangen wir an:

* Mein Notebook mit installiertem Ubuntu Desktop 20.04 LTS und Ansible in der Version 2.9.6 dient als Ansible-Controller

* Drei VMs mit Ubuntu-Server 20.04 LTS dienen als Ziele:

  * ansible-guide-1 (192.168.0.11)
  * ansible-guide-2 (192.168.0.12)
  * ansible-guide-3 (192.168.0.13)

Der Ziel-User für alle drei Hosts ist "ansible". Dieser hat volle Sudo-Rechte, ist also selbst kein Root-User, kann aber über das Kommando 'sudo', kurz für "super user do" Befehle mit Root-Rechten ausführen. Diese Mechanik macht sich auch Ansible zu nutze. Dazu aber gleich mehr.

<!-- excerpt-end -->

## SSH-Publickey-Authentifizierung einrichten

Zwar kann Ansible in der Theorie auch eine Authentifizierung per Passwort durchführen, dies ist aber nicht wirklich sinnvoll und ab einer gewissen Anzahl Zielhosts auch sehr aufwändig. Die einfachste und gleichzeitig auch sicherste Variante ist die Nutzung von SSH-Keys. Wir erzeugen uns also ein Keypair, bestehend aus privatem und öffentlichem Schlüssel und hinterlegen den öffentlichen Teil (Public Key) auf den Zielsystemen.

Den privaten Schlüssel (Private Key) verwahren wir sicher auf unserem Client auf.

Ausgehend davon, dass aktuell noch keine SSH-Key-Authentifizierung eingerichtet ist, sondern diese per Passwort erfolgt sind folgende Schritte zu tun:

### Keypair auf dem lokalen Client erzeugen

``` bash
ssh-keygen -t rsa -b 4096

```

Die Ausgabe sieht dann ungefähr so aus:

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/mow/.ssh/id_rsa): /home/mow/.ssh/id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/mow/.ssh/id_rsa
Your public key has been saved in /home/mow/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:uIRqKu6ySijEPaHeNrkNJrs/PmzTH6T+0+WB9F+GKek mow@ubuntu1
The key's randomart image is:
+---[RSA 4096]----+
|                 |
|                 |
|   .             |
|. o .. . .       |
| + o. o.S o . o  |
|+ ..o.o. . * o o |
|o++B..... + + o  |
|=o=**. ... E .   |
|X*=++ooo.        |
+----[SHA256]-----+
```
Was haben wir gemacht? Wir haben ein neues SSH-Keypair erzeugt. Dieses hat den Typ RSA (-b rsa) und eine Länge von 4096 Byte (-b 4096).

Wichtig: Bei Keys die im produktiven Umfeld eingesetzt werden, rate ich dringend zur Verwendung einer Passphrase, da ein potentieller Angreifer so mit dem privaten Schlüssel allein nichts anfangen kann. Für diesen Guide ist dies aber nicht zwingend notwendig.

Jetzt, nachdem wir stolze Besitzer eines druckfrischen SSH-Keypairs sind, wollen wir den öffentlichen Teil davon, also den Public Key auf den Zielsystemen hinterlegen. Dafür gibt es das nützliche Tool "ssh-copy-id".

Folgenden Befehl führen wir also auf unserem Ansible-Controller aus:

```
ssh-copy-id -i ~/.ssh/id_rsa ansible@ansible-guide-1
```

Anschließend werden wir nach dem Passwort für den Zieluser gefragt. Danach wird der Public Key auf dem Zielsystem hinterlegt. Das Ganze wiederholen wir natürlich noch für die restlichen Systeme.

Hat alles funktioniert, sollten wir uns also ohne Passwort auf die Zielsysteme verbinden können:

``` bash
ssh ansible@ansible-guide-1
ssh ansible@ansible-guide-2
ssh ansible@ansible-guide-3
```
Und schon sind wir bereit um richtig mit Ansible loszulegen!

## Projektverzeichnis und Inventory anlegen

Beginnen wir mit einem Projektverzeichnis und einem Inventory. Zwar könnten wir auch mit dem Default-Inventory (/etc/ansible/hosts) und einem beliebigen Verzeichnis beginnen, ich bin aber immer dafür, ein dediziertes Verzeichnis pro Ansible-Projekt zu nutzen.

Wir erstellen also zunächst das Projekt-Verzeichnis:
``` bash
# Verzeichnis im eigenen Home anlegen
mkdir ~/ansible-guide
```

Dann erstellen wir uns unter ~/ansible-guide/inventory.txt eine Datei mit dem folgenden Inhalt:

```ini
[guide]
ansible-guide-1 ansible_ssh_user=ansible ansible_host=192.168.0.11
ansible-guide-2 ansible_ssh_user=ansible ansible_host=192.168.0.12
ansible-guide-3 ansible_ssh_user=ansible ansible_host=192.168.0.13
```
Wir haben in diesem Fall ein Inventory mit einer Gruppe "guide" angelegt. In dieser befinden sich unsere 3 Test-Hosts. Mit "ansible_ssh_user=ansible" geben wir den User an, der für die SSH-Verbindung genutzt werden soll. 

Sind die Hostnamen nicht per DNS in IPs auflösbar, geben wir mit  ansible_host=***" noch die Ziel-IPs der Hosts an.

Das geht alles auch noch ein bisschen einfacher, soll für jetzt aber reichen. Schließlich wollen wir uns mit dem Thema Inventories ja noch im Detail beschäftigen!


## AdHoc Commands

Ein Begriff, den ich im ersten Artikel leider schändlicherweise vergessen habe zu nennen. Daher hole ich das hier schnell nach:

Mit AdHoc-Commands können auf Kommandozeilenebene einzelne Tasks mit Ansible ausgeführt werden. Sie sind sehr einfach zu nutzen und eigenen sich immer dann besonders gut, wenn es darum geht schnell etwas auf mehreren Hosts einmalig durchzuführen.

Auch wenn ich - bis auf in einigen Ausnahmen - immer empfehle, wiederverwendbare Playbooks auch für kleinere Aufgaben zu nutzen, sind AdHoc-Commands für erste Ansible-Tests doch sehr nützlich.

Also, dann schauen wir doch mal, ob wir unsere SSH-Verbindung korrekt konfiguriert haben. Ansible hat dafür das Modul "ping":

``` bash
# Syntax
# ansible <hosts> -i <inventory> -m <modul>
cd ~/ansible-guide

ansible guide -i inventory.txt -m ping
```

Hat alles geklappt, sollte die Ausgabe ungefähr so aussehen:

```
ansible-guide-1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
ansible-guide-2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
ansible-guide-3 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

```

Perfekt, wir sind nun also in der Lage unsere drei Systeme mit Ansible zu verwalten!

Mit einem weiteren AdHoc-Command, könnten wir uns jetzt z.B Infos zum Betriebssystem anzeigen lassen:

``` bash
# Syntax ansible <hosts> -i <inventory> -m <modul> -a <parameter>

ansible guide -i inventory.txt -m command -a "cat /etc/os-release"
```
Wir rufen also erneut Ansible mit unserer Gruppe "guide" auf, nutzen diesmal das Modul "command" und geben diesen den Parameter "cat /etc/os-release".

Die Ausgabe sollte dann ca. so aussehen:

```
ansible-guide-1 | CHANGED | rc=0 >>
NAME="Ubuntu"
VERSION="20.04.1 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.1 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal

ansible-guide-2 | CHANGED | rc=0 >>
NAME="Ubuntu"
VERSION="20.04.1 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.1 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal

ansible-guide-3 | CHANGED | rc=0 >>
NAME="Ubuntu"
VERSION="20.04.1 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.1 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```

## Unser erstes Playbook - Yeah!

Jetzt ist es an der Zeit, unser erstes Playbook zu erstellen. Nehmen wir einfach an, wir wollen auf allen drei Systemen das Paket "htop" installieren. Ich werde hier nur ein paar kurze Worte zum Inhalt verlieren, da wir in den nächsten Artikeln genauer auf Playbooks eingehen werden.

Unser einfaches Playbook sieht so aus:

```yaml
# /home/mow/ansible-guide/playbook-htop.yml
- hosts: guide               # <-- Definition der Ziele
  tasks:                     # <-- Liste mit Tasks
    - name: install htop     # <-- Name für den Task (frei wählbar)
      become: true           # <-- Root-Rechte vor dem Ausführen
      apt:                   # <-- Name des Modules
        name: htop           # <-- Modul-Parameter: Name des Pakets
        state: present     # <-- Modul-Parameter: Status des Pakets, z.B "present, absent, latest"
```
Um das ganze nun auszuführen, brauchen wir ein weiteres Kommando:

```bash
ansible-playbook -i inventory.txt playbook-htop.yml
```
Und so sieht dann die Ausgabe aus:

```
PLAY [guide] **************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]
ok: [ansible-guide-3]

TASK [install htop] ***********************************************************************************************************************************************************************************************
changed: [ansible-guide-1]
changed: [ansible-guide-2]
changed: [ansible-guide-3]
]
PLAY RECAP ********************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=1    unreachable=0 failed=0    skipped=0    rescued=0    ignored=0
ansible-guide-2                  : ok=2    changed=1    unreachable=0 failed=0    skipped=0    rescued=0    ignored=0 
ansible-guide-3                  : ok=2    changed=1    unreachable=0  failed=0    skipped=0    rescued=0    ignored=0
```

## Task States

Zum Abschluss dieses Teils möchte ich noch kurz auf einige Stati eingehen, die ein Task annehmen kann. Vielleicht ist euch schon aufgefallen, dass bei einigen unserer Tasks ein "ok" bei anderen ein "changed" zurückgeliefert wird.

Den Status "ok" nimmt ein Task immer dann an, wenn Ansible keine Änderungen vornehmen musste, um den Zielzustand herzustellen. Das Gegenteil ist dann natürlich beim Status "changed" der Fall.

Aber halt Moment! Was erzählt der denn da für einen Quatsch? Wir haben doch im Beispiel ein "cat /etc/os-relase" ausgeführt, lassen uns also nur den Inhalt einer Datei ausgeben.  Warum sagt der Task dann "changed" ???

Das kommt daher, dass das Modul "command" nichts anderes tut, als stumpfsinnig ein Shellcommand auszuführen. Da Ansible nicht "wissen" kann, was dieses wiederum tut (könnte ja auch ein Skript sein oder sonstiges) geht es automatisch von einer Änderung aus. 

Daher auch direkt ein wichtiger Tipp, den ihr noch des Öfteren von mir hören werdet:

Benutzt das Modul command NUR WENN ES SICH NICHT VERMEIDEN LÄSST!! Wir sollten wann immer möglich versuchen, Module zu nutzen, bei denen Ansible in der Lage ist, Änderungen zu tracken. So bleibt für uns auch bei komplexeren Playbooks, der Zustand unserer Umgebung immer nachvollziehbar.

Neben "ok" und "changed" gibt es noch weitere States, auf diese gehen wir aber zu gegebener Zeit genauer ein. 

## Wie geht es weiter?
 
Im nächsten Teil werden wir zusammen ein etwas umfangreicheres Playbook erstellen und ich werde dabei mehr zur Funktionsweise von Ansible sagen.

Wie immer freue ich mich über Feedback und hoffe es hat euch gefallen.


Bis denne!

Der Mow
