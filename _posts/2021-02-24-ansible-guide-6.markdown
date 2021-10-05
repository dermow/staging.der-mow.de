---
layout: post
title:  "Ansible Starter-Guide: #006 - Facts" 
date:   2021-02-24 22:52:42 +0100
categories: Ansible
---

Hallo zusammen. Hiermit kommen wir zu Teil 6 des Ansible Starter-Guides. In diesem Teil möchte ich mir mit euch zusammen die sogenannten Facts anschauen. Diese sind nicht schwer zu verstehen, können uns aber bei vielen Aufgaben mit Ansible das Leben erleichtern.

### Was sind Facts?

Vielleicht ist euch zu Beginn jedes Playbook-Durchlaufs schon folgender Part aufgefallen:

```
TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
```
Dies ist ein Task, der von Ansible standardmäßig ausgeführt wird, wenn das Sammeln von Facts aktiviert ist. Facts sind Variablen, die von Ansible zum Start eines Playbooks automatisch gesetzt werden. Sie enthalten alle möglichen Informationen zum aktuellen Play, dessen Zielsystemen und der aktuellen Umgebung. 

<!-- excerpt-end -->

Ein paar kleine Beispiele:

**ansible_hostname**: Enthält den von Ansible herausgefundenen Hostnamen des aktuellen Hosts

**inventory_hostname**: Enthält den im Inventory definierten Hostname des aktuellen Hosts

**ansible_default_ipv4.address**: Enthält die erste von Ansible gefundene, primäre IPv4-Adresse

**ansible_distribution**: Enthält den Namen der OS-Distribution des Zielhosts, z.B. "Ubuntu", "Debian" oder "Suse".

Wir können uns mit einem kleinen AdHoc-Command alle verfügbaren Facts zu einem System anzeigen lassen:

```bash
ansible ansible-guide-1 -i inventory.txt -m setup
```

Die Ausgabe ist dann ein sehr großer Block im JSON (JavaScript Object Notation) Format, den ich jetzt zwecks Übersichtlichkeit nicht hier einfüge. Probiert das am besten einfach selbst aus!

### Fact-Gathering aktivieren / deaktivieren
Per Default ist das Sammeln von Facts für jedes Playbook aktiviert. Allerdings kostet das auch Zeit und kann daher bei Bedarf deaktiviert werden:

```yaml
- hosts: all
  gather_facts: no   # <-- deaktiviert das Sammeln von Facts
  tasks:
    - name: task 1
      debug:
        msg: "fact gathering is off"

```

Meine Empfehlung ist, das Sammeln von Facts nur dann zu aktivieren, wenn wir auch wirklich auf diese zugreifen müssen. Dies kann die Laufzeit eines Playbooks deutlich verkürzen.

### Zugriff auf Facts in einem Playbook
Da Facts schlussendlich auch Variablen sind, ist der Zugriff auf diese identisch. So können wir uns z.B. die aktuelle Distribution und OS-Familie ausgeben lassen:

```yaml
- hosts: all
  tasks:
    - name: print current distribution
      debug:
        msg: "My distribution is {% raw %}{{ ansible_distribution }}{% endraw %}"

    - name: print current os family
      debug:
        msg: "My os family is {% raw %}{{ ansible_os_family }}{% endraw %}"

```

### Eigene Facts definieren
Zusätzlich zu den von Ansible bereitgestellten Facts können wir diese auch selbst definieren. Dies ist immer dann nützlich, wenn wir zur Laufzeit eigenen Variablen festlegen möchten.

Dazu nutzen wir das Modul "set_fact":

```yaml
- hosts: ansible-guide-1
  gather_facts: true
  tasks:
    - name: set facts
      set_fact:
        my_custom_fact_1: "Ich wurde zur Laufzeit definiert"
        my_custom_fact_2: "Ich wurde auch zur Laufzeit definiert"
        another_custom_fact: "Hallo Welt"

    - name: Ausgabe
      debug:
        msg: "{% raw %}{{ my_custom_fact_1 }}{% endraw %}"
```

Mit dem Modul "set_fact" können wir einen oder mehrere Facts manuell definieren. Bei der Definition könnten wir auch wieder auf Facts oder Variablen zugreifen:

```yaml
- hosts: ansible-guide-1
  gather_facts: true
  tasks:
    - name: set facts
      set_fact:
        my_custom_fact: "Meine Distirubtion ist: {% raw %}{{ ansible_distribution }}{% endraw %}"

    - name: Ausgabe
      debug:
        msg: "{% raw %}{{ my_custom_fact }}{% endraw %}"

```
###### Ausgabe:
```
PLAY [localhost] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [set facts] ***************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [Ausgabe] *****************************************************************************************************************************************************************************************************************************************************************
ok: [ok: [ansible-guide-1]] => {
    "msg": "Meine Distirubtion ist: Ubuntu"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

Die mit dem Modul "set_fact" definierten Facts sind erst NACH der Ausführung des Tasks verfügbar. Das heißt, wir können nicht auf Facts zugreifen, die im selben Task definiert werden:

```yaml
- hosts: ansible-guide-1
  tasks:
    - name: Das hier funktioniert leider NICHT!
      set_fact:
        fact_1: "ich bin fact_1"
        fact_2: "fact_1 lautet: {% raw %}{{ fact_1 }}{% endraw %}"
```
###### Ausgabe:
```
PLAY [ansible-guide-1] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [Das hier funktioniert leider NICHT!] ********************************************************************************************************************************************************************************************************************************************
fatal: [ansible-guide-1]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'fact_1' is undefined\n\nThe error appears to be in '/home/sseibold/blog/facts.yml': line 5, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n    - name: Das hier funktioniert NICHT!\n      ^ here\n"}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```

Der Task schlägt fehl, da die Variable "fact_1" noch nicht verfügbar ist. Dies ist erst nach dem Durchlauf des Tasks der Fall. Das Playbook müsste also so aussehen:

```yaml
- hosts: ansible-guide-1
  tasks:
    - name: Das hier funktioniert!
      set_fact:
        fact_1: "ich bin fact_1"

    - name: fact_2 in neuem Task definieren
      set_fact:
        fact_2: "fact_1 lautet: {% raw %}{{ fact_1 }}{% endraw %}"

    - name: Ausgabe
      debug:
        msg: "{% raw %}{{ fact_2 }}{% endraw %}"
```
###### Ausgabe:
```
PLAY [ansible-guide-1] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [Das hier funktioniert] ***************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [fact_2 in neuem Task definieren] *****************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [Ausgabe] *****************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => {
    "msg": "fact_1 lautet: ich bin fact_1"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

Da der Fact "fact_1" nun in einem vorherigen Task definiert wurde, können wir im nächsten Task darauf zugreifen.


### Zusammenfassung
Damit kommen wir schon zum Ende dieses kurzen Kapitels, in dem wir gelernt haben, was Facts sind und wie wir auf diese zugreifen können. Wir werden Facts im Laufe dieses Guides noch sehr häufig in praktischen Beispielen nutzen.

### Wie geht es weiter?
Im nächsten Teil dieser Reihe befassen wir uns mit Rückgabewerten (Return Values). Wie immer hoffe ich, dass euch der Guide bisher weiterhelfen konnte. 

Ich würde mich sehr über euer Feedback freuen. Hinterlasst doch einfach ein Kommentar hier oder schreibt mir eine Mail!

Bis bald!

Der Mow
