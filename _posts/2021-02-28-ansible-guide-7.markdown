---
layout: post
title:  "Ansible Starter-Guide: #007 - Rückgabewerte (Return Values)" 
date:   2021-02-28 16:53:42 +0100
categories: Ansible
---

Hallo! Weiter geht es mit dem Ansible Starter-Guide. Wir sind bereits bei Teil 7 angelangt, der sich mit Rückgabewerten von Tasks beschäftigen wird.

Jetzt fragt ihr euch sicher: Was zur Hölle sind Rückgabewerte und wozu brauche ich die?

Kurz zusammengefasst: Wenn ein Task ausgeführt wurde, erstellt er einen Rückgabewert, der Informationen zum Task-Ergebnis enthält. Wir brauchen Rückgabewerte immer dann, wenn wir in einem Task mit dem Ergebnis aus einem vorausgegangenen Tasks weiterarbeiten wollen. 

### Beispiel

Wie immer möchte ich euch das mit einem kleinen Beispiel verdeutlichen. Nehmen wir an, wir möchten überprüfen, ob eine bestimmte Datei auf dem Remote-System existiert. Dies können wir mit dem Modul "stat" überprüfen:

```yaml
- name: Beispiel mit Return Values
  hosts: webservers
  tasks:
    - name: check if hosts file is there
      stat:
        path: /etc/hosts

```

<!-- excerpt-end -->

Führen wir dieses Playbook nun aus, ist das Ergebnis erstmal sehr unspektakulär:

```
PLAY [Beispiel mit Return Values] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [check if file is there] **************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
ansible-guide-2                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Wie wir sehen, haben beide Tasks funktioniert und liefern den Status "ok". Und was haben wir nun davon? Bisher noch nichts. Das Modul "stat" sammelt nur Statistiken zu einem File (bzw. Pfad) und würde übrigens auch "ok" melden, sollte die Datei nicht existieren.

Wenn wir nun die gesammelten Informationen zu diesem File nutzen möchten, müssen wir den Rückgabewert des Tasks nutzen. Um diesen festzuhalten, gibt es den Task-Parameter "register":

```yaml
- name: Beispiel mit Return Values
  hosts: webservers
  tasks:
    - name: check if hosts file is there
      stat:
        path: /etc/hosts
      register: my_var_with_return_value

```

Damit teilen wir Ansible mit, dass es den Rückgabewert des Tasks "check if hosts file is there" in einer Variable mit dem Namen "my_var_with_return_value" speichern soll. Nun enthält der Return-Value jedes Moduls unterschiedliche Felder. Diese finden wir aber in der offiziellen Ansible-Dokumentation zu jedem Modul. Für das Modul "stat" ist das hier:

[https://docs.ansible.com/ansible/devel/collections/ansible/builtin/stat_module.html](https://docs.ansible.com/ansible/devel/collections/ansible/builtin/stat_module.html)

Der Rückgabewert ist in einem Dictionary gespeichert. Diese haben wir noch nicht im Detail behandelt, für diesen Fall soll es aber erstmal reichen. Um zu prüfen, ob unsere Datei existiert gehen wir so vor:

```yaml
- name: Beispiel mit Return Values
  hosts: webservers
  tasks:
    - name: check if hosts file is there
      stat:
        path: /etc/hosts
      register: my_var_with_return_value

    - name: debug output
      debug:
        msg: "File exists is: {% raw %}{{ my_var_with_return_value.stat.exists }}{% endraw %}"

```

Führen wir dieses Playbook nun aus, erhalten wir folgende Ausgabe:

```
PLAY [Beispiel mit Return Values] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [check if hosts file is there] ********************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]
ok: [ansible-guide-2]

TASK [debug output] ************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => {
    "msg": "File exists is: True"
}
ok: [ansible-guide-2] => {
    "msg": "File exists is: True"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
ansible-guide-1                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
ansible-guide-2                  : ok=3    changed=0    unreachable=0    failed=0   2skipped=0    rescued=0    ignored=0 
```

Die Datei "/etc/hosts" existiert also auf unseren beiden Test-Webservern. 

### Noch ein Beispiel

Genauso könnten wir uns die Ausgabe eines Shellcommands ausgeben lassen, welches wir über das Modul "command" aufrufen:

```yaml
- name: Beispiel mit Return Values
  hosts: ansible-guide-1
  tasks:
    - name: get os version
      command: "uname -a"
      register: my_var_with_return_value

    - name: debug output
      debug:
        msg: "{% raw %}{{ my_var_with_return_value.stdout }}{% endraw %}"

```
###### Ausgabe:
```
PLAY [Beispiel mit Return Values] ***************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1]

TASK [get os version] *****************************************************************************************************************************************************************************************************************************************************
changed: [ansible-guide-1]

TASK [debug output] ************************************************************************************************************************************************************************************************************************************************************
ok: [ansible-guide-1] => {
    "msg": ""msg": "NAME=\"Ubuntu\"\nVERSION=\"20.04.2 LTS (Focal Fossa)\"\nID=ubuntu\nID_LIKE=debian\nPRETTY_NAME=\"Ubuntu 20.04.2 LTS\"\nVERSION_ID=\"20.04\"\nHOME_URL=\"https://www.ubuntu.com/\"\nSUPPORT_URL=\"https://help.ubuntu.com/\"\nBUG_REPORT_URL=\"https://bugs.launchpad.net/ubuntu/\"\nPRIVACY_POLICY_URL=\"https://www.ubuntu.com/legal/terms-and-policies/privacy-policy\"\nVERSION_CODENAME=focal\nUBUNTU_CODENAME=focal"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Wir führen also wieder einen Task aus und speichern den Rückgabewert per "register" in eine Variable. Dann schauen wir uns den Aufbau des Rückgabewerts an:

[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html)

Unter "Return Values" sehen wir, dass es das Feld "stdout" gibt, also den standard output des aufgerufenen Shellcommands. Dieses Feld geben wir uns hier wieder mit dem Modul "debug" aus.

### Standard-Felder
Zusätzlich zu den modulspezifischen Feldern, existieren in jedem Rückgabewert-Dictionary einige Standard-Felder. Ein paar Beispiele:

**changed**: Enthält die Information als Boolean (true/false), ob der Task den Status "changed" zurückgeliefert hat

**skipped**: Enthält die Information, ob der Task übersprungen wurde

**failed**: Ist hoffentlich "false". Enthält die Information, ob der Task fehlgeschlagen ist.

Die vollständige Liste findet ihr hier:

[https://docs.ansible.com/ansible/2.5/reference_appendices/common_return_values.html](https://docs.ansible.com/ansible/2.5/reference_appendices/common_return_values.html)


### So geht's weiter 
Im nächsten Teil möchte ich euch Conditionals zeigen, das sind Bedingungen, an die wir die Ausführung unserer Tasks knüpfen können. 

Viele Grüße und bis dahin!

Der Mow
