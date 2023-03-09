---
layout: post
title:  "Ansible Starter-Guide: #008 - Conditionals 1" 
date:   2021-03-11 15:57:42 +0100
categories: Ansible
---

Halli Hallo! Nach längerer Pause geht es nun mit Teil 8 des Ansible Starter-Guides weiter. Ich möchte euch in diesem Post zeigen, was Conditionals sind
und wie wir diese in Playbooks nutzen können. Ich habe mich dazu entschieden, dieses Thema in zwei Artikel aufzuteilen, da der Blog-Eintrag sonst zu lang werden würde. Daher versuche ich euch hier die prinzipielle Funktion von Conditionals zu zeigen, in Teil 2 gehen wir dann zusammen praktische Beispiele durch.

Solltet ihr einzelne Punkte nicht verstehen, oder ich mich unklar ausgedrückt haben, schreibt mir gerne eine Mail. Ich versuche dann das aufzuklären :)

### Was sind Conditionals?

Conditionals sind Bedingungen, an die wir die Ausführung von Tasks knüpfen können. Zum Beispiel können wir einen Task nur dann ausführen lassen, wenn der
Ziel-Host ein bestimmtes Betriebssystem installiert hat. Wenn wir Conditionals nutzen, greifen wir bei deren Definition auf Variablen und Facts zurück.

### Ein erstes Beispiel

Lasst uns mit einem sehr einfachen Beispiel beginnen. 

##### simple-conditional.yml

```yaml
- hosts: ansible-guide-1
  vars:
    my_number: 6
  tasks:
    - name: task mit condition
      debug:
        msg: "Variable ist größer als 5!!"
      when: "my_number > 5"
```

<!-- excerpt-end -->

Conditionals werden im Parameter "when" definiert. Dieser ist ein Parameter auf Task-Ebene. Unter "when" kann dann eine, oder auch eine Liste mit Conditionals stehen. Die Conditions werden vor der Ausführung des Tasks geprüft. Sollten sie zutreffen, wird der Task ausgeführt, falls nicht wird er übersprungen und erhält den Status "skipped".

Was heißt das für das obige Beispiel-Playbook? Wir haben oben eine Variable "my_number" definiert, die den Wert 6 hat. Dann haben wir einen Task definiert, der die folgende Condition hat:

```yaml
...
when: "my_number > 5"
---
```

Wir wollen den Task also nur dann ausführen, wenn der Wert der Variable "my_number" größer ist als 5. In diesem Fall sollte der Task also ausgeführt werden. Dann lasst uns das Playbook doch einmal starten:

```bash
ansible-playbook -i inventory.txt simple-conditional.yml
```
```
PLAY [ansible-guide-1] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [task mit condition] ******************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => {
    "msg": "Variable ist größer als 5!!"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Wie wir sehen, wurde der Task ganz normal ausgeführt. Nun lasst uns einen weiteren Task definieren:

###### simple-conditional.yml
```yaml
- hosts: ansible-guide-1
  vars:
    my_number: 6
  tasks:
    - name: task mit condition
      debug:
        msg: "Variable ist größer als 5!!"
      when: "my_number > 5"

    - name: weiterer task mit condition
      debug:
        msg: "Variable ist 8!!"
      when: "my_number == 8"
```

```bash
ansible-playbook -i inventory.txt simple-conditional.yml
```
```
PLAY [ansible-guide-1] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [task mit condition] ******************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => {
    "msg": "Variable ist größer als 5!!"
}

TASK [weiterer task mit condition] *********************************************************************************************************************************************************************************************************************************************
skipping: [ansible-guide-1]

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Was ist nun passiert? Ansible hat unseren ersten Task ausgeführt, da dessen Bedingung zutrifft. Beim zweiten Task hat es die Bedingung geprüft und festgestellt, dass der Wert der Variable "my_number" nicht 8 ist, die Bedingung also nicht zutrifft und den Task übersprungen. Das sehen wir an der Ausgabe "skipping: [ansible-guide-1]"

### Operators und Jinja2

Wir nutzen für beide Conditionals verschiedene Operators: '>' und '=='. Conditionals in Ansible basieren auf der Templating-Sprache Jinja2. Wir können also alle in dieser Sprache verfügbaren Operatoren nutzen. Eine Übersicht hierzu findet ihr hier:

[https://jinja.palletsprojects.com/en/3.0.x/templates/](https://jinja.palletsprojects.com/en/3.0.x/templates/)

Mit Jinja2 werden wir noch sehr viel arbeiten, wenn wir zu den fortgeschritteneren Teilen des Guides kommen. 

### Mehrere Conditionals

Wie weiter oben kurz erwähnt, kann unter dem Parameter "when" sowohl eine einzelne Bedingung stehen, als auch eine Liste. Hier ist es wichtig zu wissen, dass in Listen angegebene Bedingungen immer in einer "AND-Beziehung" zueinander stehen. Das heißt kurz gesagt, dass jedes Listen-Item unter "when" wahr sein muss, um die Taskausführung zu ermöglichen. 

Auch dazu möchte ich noch ein kurzes Beispiel einbringen:

##### multi-conditionals.yml

```yaml
- hosts: ansible-guide-1
  vars:
    my_number: 5
    my_text: Hallo Welt
    my_other_text: Blubb.
  tasks:
    - name: task mit condition
      debug:
        msg: "Dieser Task wird ausgeführt, weil alle 3 Bedingungen wahr sind"
      when:
        - "my_number == 5"
        - "'Hallo' in my_text"
        - "my_other_text == 'Blubb.'"
```

```bash
ansible-playbook -i inventory.txt multi-conditionals.yml
```
```
PLAY [ansible-guide-1] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [task mit condition] ******************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => {
    "msg": "Dieser Task wird ausgeführt, weil alle 3 Bedingungen wahr sind"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Wie wir sehen, sind alle drei Bedingungen wahr und der Task wird ausgeführt. Lasst die drei Bedingungen ganz kurz durchgehen:

```yaml
- "my_number == 5"
```
Dies ist dasselbe Prinzip, wie auch schon in den ersten Beispielen. Die Prüfung lautet "wenn my_number gleich 5 ist"

```yaml
- "'Hallo' in my_text"
```
Hier prüfen wir mit dem Operator "in", ob sich der Substring "Hallo" im String my_text befindet. Der Operator "in" prüft, ob sich eine Untermenge an Elementen in einer Liste befindet. Da ein String nichts anderes ist, als eine Liste aus Zeichen, können wir uns das hier zunutze machen.

```yaml
- "my_other_text == 'Blubb.'"
```
Der Operator "==" kann auch auf Strings angewendet werden. Wir prüfen also "Ist die Variable my_other_text gleich 'Blubb.'". Auch dies trifft zu.

### AND

Im obigen Beispiel definieren wir Conditinals in einer Liste, was eine AND-Beziehung dieser impliziert. Wir können AND und OR Operatoren aber auch in einer Zeile definieren. So könnten wir den Task aus "multi-conditionals.yml" auch so definieren:

##### and-oneline.yml
```yaml
- hosts: ansible-guide-1
  vars:
    my_number: 5
    my_text: Hallo Welt
    my_other_text: Blubb.
  tasks:
    - name: task mit condition
      debug:
        msg: "Dieser Task wird ausgeführt, weil alle 3 Bedingungen wahr sind"
      when:
        - "my_number == 5 and 'Hallo' in my_text and my_other_text == 'Blubb.'"
```

Wenn wir diesen nun ausführen sollten wir dasselbe Ergebnis erhalten:

```bash
ansible-playbook -i inventory.txt and-oneline.yml
```
```
PLAY [ansible-guide-1] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [task mit condition] ******************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => {
    "msg": "Dieser Task wird ausgeführt, weil alle 3 Bedingungen wahr sind"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

### OR

Möchten wir, dass der Task ausgeführt wird, sobald auch nur eine der Bedingungen wahr ist, nutzen wir den OR-Operator:

##### or-oneline.yml
```yaml
- hosts: ansible-guide-1
  vars:
    my_number: 5
    my_text: Bye Bye Welt
    my_other_text: Blubb.
  tasks:
    - name: task mit condition
      debug:
        msg: "Dieser Task wird ausgeführt, weil alle 3 Bedingungen wahr sind"
      when:
        - "my_number == 5 or 'Hallo' in my_text or my_other_text == 'Blubb.'"
```

```bash
ansible-playbook -i inventory.txt and-oneline.yml
```
```
PLAY [ansible-guide-1] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [task mit condition] ******************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => {
    "msg": "Dieser Task wird ausgeführt, weil alle 3 Bedingungen wahr sind"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Ich habe mir erlaubt, die Variable "my_text" ein wenig abzuändern, dabei aber die Bedingung selbst unten gleich zu lassen. Lediglich die Operatoren habe ich von "and" zu "or" geändert. Dennoch wird der Task ausgeführt, da die Bedingung ist: 

**Wenn my_number 5 ist ODER 'Hallo' in my_text enthalten ist ODER my_other_text "Blubb." ist.**

### AND und OR kombiniert

Etwas komplizierter wird es, wenn wir AND und OR kombinieren möchten:

##### and-or.yml
```yaml
- hosts: ansible-guide-1
  vars:
    my_number: 5
    my_text: Hallo Welt
    my_other_text: Blubb.
  tasks:
    - name: task mit condition
      debug:
        msg: "Dieser Task wird ausgeführt, weil alle 3 Bedingungen wahr sind"
      when:
        - "my_text == 'Hallo Welt'"
        - "my_number == 5 or my_number > 3"
```

Wir haben hier also zwei Conditionals in einer Liste definiert. Wie wir gelernt haben, müssen beide Listen-Items am Ende wahr sein, um den Task auszuführen. Das Beispiel oben umformuliert bedeutet also folgendes:

**Wenn my_text 'Hallo Welt' ist UND my_number 5 oder größer als 3 ist.**

Führen wir das Playbook nun aus, wird der Task ausgeführt:

``` bash
ansible-playbook -i inventory.txt and-or.yml
```
```
PLAY [ansible-guide-1] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansbible-guide-1]

TASK [task mit condition] ******************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => {
    "msg": "Dieser Task wird ausgeführt, weil alle 3 Bedingungen wahr sind"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Wie erwartet wird der Task ausgeführt. Nun lasst uns das aber noch gegenprüfen, indem wir den Wert von "my_number" auf 2 setzen:

##### and-or.yml
```yaml
- hosts: ansible-guide-1
  vars:
    my_number: 2
    my_text: Hallo Welt
    my_other_text: Blubb.
  tasks:
    - name: task mit condition
      debug:
        msg: "Dieser Task wird ausgeführt, weil alle 3 Bedingungen wahr sind"
      when:
        - "my_text == 'Hallo Welt'"
        - "my_number == 5 or my_number > 3"
```

Was wird nun passieren? Die erste Bedingung in der Liste ist noch immer wahr. Der zweite Teil allerdings nicht mehr, da die Zahl 2 weder gleich 5, noch größer als 3 ist:
``` bash
ansible-playbook -i inventory.txt and-or.yml
```
```
PLAY [ansible-guide-1] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [task mit condition] ******************************************************************************************************************************************************************************************************************************************************
skipping: [ansible-guide-1]

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=1    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

### Ausprobieren!
Am besten probiert ihr das einfach selbst mal aus und variiert die Variablen-Werte beliebig. 

### So geht es weiter

Nachdem wir uns in diesem Teil die Funktionsweise von Conditionals angeschaut haben, möchte ich im nächsten Teil einige paraktische Anwendungsfälle mit euch durchgehen. 

Ich hoffe, euch hat es auch diesmal wieder gefallen und bin gespannt auf euer Feedback!

Viele Grüße

Der Mow
