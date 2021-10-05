---
layout: post
title:  "Ansible Starter-Guide: #013 - Ansible Vault"
date:   2021-09-24 15:48:42 +0100
categories: Ansible
---

Servus!

In diesem Teil des Ansible Guides möchte ich euch die Ansible Vault vorstellen. Diese können wir nutzen, um Daten in Ansible zu verschlüsseln. Für sehr viele Tasks werden Zugangsdaten benötigt, die wir nicht im Klartext als Variable speichern wollen.

Die Ansible Vault ist bei der Installation von Ansible mit dabei, muss also nicht extra installiert werden.

### Neue Vault erstellen

Zunächst müssen wir uns ein Passwort für die Vault generieren, ich nutze dazu gerne das Tool "pwgen". Das Passwort lasse ich mir direkt
in ein verstecktes Files in meinem Home-Verzeichnis ausgeben. 

``` bash
sudo apt install pwgen

# 1 Passwort mit 12 Zeichen erstellen
pwgen 12 1 > ~/.vaultpw
```

<!-- excerpt-end -->

Nun können wir uns folgendermaßen eine Vault erstellen. In diesem Beispiel speichern wir die Vault in den Hostvars:

``` bash
cd ~/ansible-guide
mkdir -p host_vars/ansible-guide-1

ansible-vault create host_vars/ansible-guide-1/vault.yml --vault-password-file ~/.vaultpw
```

Mit dem obigen Command haben wir uns nun eine neues File erstellt und diese mit dem Passwort verschlüsselt, welches wir vorher generiert haben.
Direkt nach dem Command sollte sich ein Editor öffnen. 

Der Inhalt dieses Files kann nun beliebig sein, im einfachsten Beispiel können wir hier einfach Variablen definieren. Nehmen wir an, wir möchten auf
unserem Host einen neuen User anlegen, und dessen Passwort in der Vault hinterlegen:

#### ~/ansible-guide/host_vars/ansible-guide-1/vault.yml
``` yaml
user_password: geheim1337
```
Anschließend speichern wir das File ab und haben erfolgreich ein erstes Vault secret erstellt :)

Wenn wir uns den Inhalt des Files nun ansehen, sollte das ungefähr so aussehen:

```
$ANSIBLE_VAULT;1.1;AES256
32363637353431323335313831306531653461353135303736343138336238393861663962396139
6637636264613234326461316539636231353537666538370a663266346333643430313538646332
30663139653637313935646130366530363361646565313436633430363935613532313966323536
6632313462383464640a346538656131353038383338313337346365396131343336666466326566
38393836636437333939363330323033613535376661333330613934663066303639
```

Wie wir sehen, ist der Inhalt der Vault kryptisch und nicht lesbar. Wir können das File also auch z. B. in eine Sourcecodeverwaltung laden, ohne Zugangsdaten im Klartext abzuspeichern.

Nutzen können wir das Secret in diesem Fall wie eine ganz normale Variable:
```
- hosts: ansible-guide-1
  tasks:
    - name: create user
      user:
        name: me
        password: "{%raw%}{{ user_password }}{%endraw%}"
        state: present
```

Versuchen wir nun, das Playbook wie gewohnt auszuführen, sollte das Resultat in etwa so aussehen:
```
PLAY [localhost] **************************************************************************************************************************************************************************************************
ERROR! Attempting to decrypt but no vault secrets found

```

Stimmt! Wir müssen ja noch das Vault-Passwort angeben:

```bash
ansible-playbook --vault-password-file ~/.vaultpw playbook.yml
```

Und schon funzt das Ganze. Aber wie genau? Ansible wird beim Start wie gewohnt die Variablen-Verzeichnisse (host_vars, group_vars) durchsuchen, dabei die verschlüsselte Vault finden und entschlüsseln.

Eine Ansible Vault muss nicht zwingend ein verschlüsseltes YAML-File mit Variablen sein. Im Prinzip können wir beliebige Files damit verschlüsseln.

### Beispiel: Zertifikat und Key in einer Vault speichern

Nehmen wir an, wir möchten ein SSL-Zertifikat inklusive Key an einen Webserver ausliefern.

Wir legen uns beide Files zum Beispiel in einen Unterordner "files":

##### ./files/key.pem
```
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIJnDBOBgkqhkiG9w0BBQ0wQTApBgkqhkiG9w0BBQwwHAQI3mHrY1wSLzgCAggA
MAwGCCqGSIb3DQIJBQAwFAYIKoZIhvcNAwcECPZFWT3K2GKaBIIJSHVxKQ3lvbwa
kh+ENQhmFQrxau5IZSeiw+D9QVFUCNc02AquLg4Pp0eBUnNtMzyGHl1w/yncdZwB
mw4ycH6Rfs0JJtP23nlcDJzTCqTeyHMQExkeQbVLADuPteoQFsjdvGLPbA5Sq5+w
WNAflSac61x3Hk9ND8y+2twISqyokixdgLWmlx72QvJ2znBA1BwRTkPzdwuvKVrA
Up5Zl82lOWzcDhRmQNxqHoH/P0h4oUfudt2RCTELX/OS/97PqMGIvu1BtcGdW+UO
DW+K+TIr/1JlfuqrUP7D3VTi5FP5nY8CgiQc5JfIY2s6K74VYfDZ1EwoUj8FJ1FX
QToErFfTm54tJ1yBpIHQjqXGGQGs1s3S3OnGztsTEZWNNgT5PzQXZN+pL4Bl0Zg5
0OubP1XJRrTjQLLpb2r+cYFnyw9vSUZ6iopA0h49nY8GatkUu39QzSg32wg4HJDh
oACRVKthGIjLyJYYeYDKc96cgKgpSEsuC5bCUzt0AmZbjgvdsBdNRq1R3dN/mZL6
Vr35I3hqHZ/HcomFYrLCy8LEMChiHMUsZA0nuWYiJXpj17so8k7gGA65U9iGjK5M
Me+mOSRIprlwW1bTHZ9OTq+IlWBrswHF0l+YXWPEfV2B9R0mBrt5Cmvf29cbjl4G
xgMDH5daXW2c0y4kDcZJ33Vw7wk7Dv7efKQqUUFKhXlSoI8gZ4zCFiflL3ZunzCe
6Uz8jscifY07U9MwEeBDTh1SG6Hc8bJJ31509D+zDrb5bwBXQHr6QojSxyHd/2WT
a4QM8Qqtk7PfLbUXIP2S1h69e6oGLBFH77VSLzzAnPuPW0/uCILabp4ekaeOJ99x
Q9wk1JrcKZk8TEom6o2hKVZ8goM3fxXvRulstoowl1MmTIlNf67iUn5/j42jnIkO
Bmvsv5J2VERyxm4ZR7Qz/kpANUkZPYunSRE1VeHWzK/dvkajnoHfrA1u/wL+C4QS
bB4GwE8O8RFS9mkLwvfRl8dHKhvoFil7hml/5MFIGWHJ40PJ+4xRFMMOq74+gnqr
eOlhOF1NG8+4dNVQhrltHdNrgXX18GXtG8qbGONrCgPGRFniB1V8T/ZV84Y7S2yt
kfTCNWpTTr2JoFo36jsyNQ+DOAZh8CryVxgTlsyG9g0m7GZL/6Smyl1RB20TV0UU
ZQAPAy6WawGTPZmJ4RJu5ijtseCu9wmHZqTPd1ORTnadAIWS7HVTAcVtfL1aH4eo
LRxBz1utryvIiV1nNibmDDNzNbCpt7UfMAyWNDeGkDIb6BQXvvixjjWmOs5KzLkr
jcIMbECAJrKyk/hzGGQBDD5NHL3S+Q2QlMgnkfZ9Gjbr69XcNMieMi2LKslzYKmw
X/t91y6eKSgFw+BF3ZmhzG1jH/vAccXG2IBzweIuUXeLigDk+3Rp+TEKxHa88sVm
2nNpL8M7jdoyYdkaejx9UQ==
-----END ENCRYPTED PRIVATE KEY-----
```

##### ./files/certificate.pem
```
-----BEGIN CERTIFICATE-----
MIIFazCCA1OgAwIBAgIUFiGHe0Hh1w5uxMsAITAC0loQ044wDQYJKoZIhvcNAQEL
BQAwRTELMAkGA1UEBhMCQVUxEzARBgNVBAgMClNvbWUtU3RhdGUxITAfBgNVBAoM
GEludGVybmV0IFdpZGdpdHMgUHR5IEx0ZDAeFw0yMTA5MjQxMTA5MjFaFw0yMjA5
MjQxMTA5MjFaMEUxCzAJBgNVBAYTAkFVMRMwEQYDVQQIDApTb21lLVN0YXRlMSEw
HwYDVQQKDBhJbnRlcm5ldCBXaWRnaXRzIFB0eSBMdGQwggIiMA0GCSqGSIb3DQEB
AQUAA4ICDwAwggIKAoICAQDHOgqyCi80NikeXGX101mjWUeJfejBw87KS1VXxtOk
L1tIAMFi36FYF+dGZ95e0hWBRJiE7hbMxLN21rt+P3v3EMOXVgLKkED1ME0W5EbJ
wUOcJ1Dh/MV9dF1WUakgopNU8CgJLGgS/VBdncUwWnsUjHOxf0j6XZ7dqGaresrp
3xRy69NBeqzXwjQCh58OpXo0lv6zZVeHxGh6mHkBPUTkbrk0/2YuObvvxF6DsiRu
zpN/s3GRZ927iMxViEj5DDEVMKpXJze2ScFml5V99zpSJvw/mOb2ieKmvT7atAZW
vI/O/fM/lO9RkhZwAaoV3Wn+MXlfXFbP9sDV441ENgU0sjkRJv8eyclkeEIM9wg6
XSX8LraFfaszWCtSphTLB+tX01FOGrkOm3lpf8OGEMr4RXVnjEvB/k3JMNrmzpSf
R5LjtAqaoD4rg3wAVnyLlWEOOtEcAZFBoVSY5TMDA9yb1akGiNBvj1WNryO0bcUI
yQ4xENJ/q/yJ7pbtqbuccL1UIWhcvQrvI9OTuQCNNfD+KWBZat7aEP79c7hQuZqA
1GN9ZFZv7xBXDjmV6GZwRy+wjvFHAmW0qCzXwpS0+ibHtWD2qBbulpZD+K0WdfVu
22tEFUOP5Lni7mJVqqIjL77yLswnldxZoF3qXHkEbQexds8FgqiSPm19uY/mW4L6
IQIDAQABo1MwUTAdBgNVHQ4EFgQUPHQPwK0qKdiP5C78p2urUn0IGA8wHwYDVR0j
BBgwFoAUPHQPwK0qKdiP5C78p2urUn0IGA8wDwYDVR0TAQH/BAUwAwEB/zANBgkq
hkiG9w0BAQsFAAOCAgEAf1WDaODUcxatC10Aa/iG1BSQeFwwIibHsP8uoR4rhvOc
32FPivbk6q/Fv5WgeBkZvpjTKglMJBV8EtbDO8aAH3/7cEe8ycchSgOvX64RgzZY
e6a2HJwQ8KIPGlKcgU57CHu6aaEW3BFzvw6j0GdQHKTw8eiYKuYW8bO/4fKRxr+F
w/bV2Cg4yyor87CSgLLawS+4yPzMCFlIOr8AMswV9VIqIlem3mWyf13WS0JAEQwe
dohfa2KGeccqTudTS9NfEJRoSt7ufXHS937gtVgimCJIdt08p5eSfznvhtHVAntl
yF0Irp1DxC+IyalWielLTxwSVsyyEu8oL4zX46C6yGG4rfZv7T6rBJOonoYEmRyv
fyFfhOI8N3CaxOk8S0FTe06HsR9caAfmGE3Qz8yDRR7qgZvaufYPgO7PjGzzvPWV
c4cevlKuGuW2IzHkzq1kQThNF74lbrBTa1QAaC6JbTBuFFwyZYAE7r4+zHx+1yFr
DY5haeOZaikakxwHaaTzGwCeQMufbIJ4kfu1zxNDvOlZVtfA7DyGcnSd763/nS5E
Kl4fE8YzHWPvQJNIpx27REXE4dEs1/soMZFWokHsjg2HmYlL4M8fH9PU8sBl/MLb
8UwAg369islBzup+yn1RrLl5iRITSkHyfuHyRv8Sgs6P29MuDraV91o23J8JAiU=
-----END CERTIFICATE-----
```

Nun möchten wir beide Files nicht im Klartext herumliegen haben. Wir verschlüsseln diese also mit der Ansible Vault:

``` bash
ansible-vault encrypt files/*.pem --vault-password-file ~/.vaultpw
```
```
Encryption successful
```

In unserem Playbook können wir ganz normal das Copy-Modul nutzen, um beide Files zu kopieren. Da Ansible erkennt, dass es sich um Vaults handelt, werden beide Files währenddessen entschlüsselt.

##### copy-certs.yml
``` yaml
- hosts: webserver-1
  tasks:
    - name: copy certificate and key
      copy:
        src: "files/{%raw%}{{ item }}{%endraw%}"
        dest: /etc/ssl/certs
      loop:
        - cert.pem
        - key.pem
```

Playbook ausführen:
``` bash
ansible-playbook playbook.yml --vault-password-file ~/.vaultpw
```

Fertig! Auf dem Server sollten beide Files in entschlüsselter Form liegen :)

### Einzelne Strings verschlüsseln

Statt ganze Files zu verschlüsseln, können wir auch einfach nur Variablenwerte verschlüsseln:

```bash
ansible-vault encrypt_string "hello world" --vault-password-file ~/.vaultpw
```
```
!vault |
          $ANSIBLE_VAULT;1.1;AES256
          64613036333366373032623562333639646565303830653335366361373533663835666135343861
          3864316335646165633439656662666537653538623662320a346432353261636466353964633365
          61636463306537313137373630323633376465663938633130633637623866326639613235323262
          3665643535633739640a366162363664356337363062636535343463323263653934623434626664
          3233
Encryption successful
```

Innerhalb von Ansible können wir den String dann folgendermaßen als Variable definieren:


```yaml
my_encrypted_var: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          64613036333366373032623562333639646565303830653335366361373533663835666135343861
          3864316335646165633439656662666537653538623662320a346432353261636466353964633365
          61636463306537313137373630323633376465663938633130633637623866326639613235323262
          3665643535633739640a366162363664356337363062636535343463323263653934623434626664
          3233
```

Somit wären wir auch wieder am Ende des Artikels angelangt. Ich hoffe, ich konnte dem Einen oder Anderen etwas Neues beibringen :)

Wie immer würde ich mich über Feedback freuen!

Bis bald,

Mow
