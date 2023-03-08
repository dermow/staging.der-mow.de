---
layout: post
title:  "Ansible Starter-Guide: #002 - Installation"
date:   2021-02-13 11:35:42 +0100
categories: Ansible
---

Da bin ich wieder! Wie versprochen gehen wir in diesem Teil auf die Installation von Ansible ein. Diese ist kein Hexenwerk, also wird das wohl einer der kürzeren Teile.


## Allgemeine Infos

Ich werde in dem gesamten Guide hauptsächlich auf die Nutzung unter Linux eingehen. Die Nutzung unter Windows weicht geringfügig davon ab, ich werde aber versuchen, an den entsprechenden Stellen darauf hinzuweisen.

Zum Erstellen dieses Guides nutze ich aktuell Ubuntu Desktop 20.04 LTS.


## Installation unter Linux 

### Ubuntu

```bash
# Installation
sudo apt update
sudo apt install -y ansible

# Test
ansible --version
```


### RedHat / CentOS

```bash
# Installation
sudo yum install ansible

# Test
ansible --version

```

### Suse

```bash
# Installation
sudo zypper in ansible

# Test
ansible --version
```

<!-- excerpt-end -->

## Installation unter Windows (Linux Subsystem)

Für die Nutzung von Ansible unter Windows empfehle ich das in Windows nutzbare Linux-Subsystem (WSL). Gleichzeitig möchte ich aber darauf hinweisen, dass es hier zu einigen Abweichungen in der Nutzung kommen kann (Windows-Berechtigungen usw.).

Eine Installationsanleitung für das Subsystem findet ihr hier:

[https://docs.microsoft.com/de-de/windows/wsl/install-win10](https://docs.microsoft.com/de-de/windows/wsl/install-win10)

Sobald das geschehen ist, könnt ihr einfach der Anleitung für Ubuntu (oder der Distribution eurer Wahl) folgen!


## So geht es weiter

Im nächsten Artikel möchte ich auf die ersten Schritte mit Ansible eingehen. In der Zwischenzeit könnt ihr euch gerne eine kleine Testumgebung mit mindestens zwei Zielsystemen aufsezten. 

z.B mit [Ubuntu-Server](https://ubuntu.com/download/server)


Bis dahin!

Der Mow
