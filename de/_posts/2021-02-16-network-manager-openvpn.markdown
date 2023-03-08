---
layout: post
title:  "Network Manager: Import per CLI"
date:   2021-02-16 09:15:42 +0100
categories: 
  - HowTo
  - Linux
  - NetworkManager
  - OpenVPN
---

Ich habe kürzlich mein Notebook neu installiert, da ich vom Default-Ubuntu 20.04 auf Ubuntu MATE in selbiger Version 
gewechselt bin. Dabei musste ich auch meine OpenVPN-Verbindung neu einrichten. Ich ziehe mir also die aktuellste Version des OpenVPN-Configfiles und versuche erfolglos die Import-Funktion in der GUI zu nutzen.

Dabei lässt sich das über die Network-Manager CLI (nmcli) viel einfacher lösen:

### Configfile sicherheitshalber in das Unix-Format umwandeln:
```bash
apt install -y dos2unix
dos2unix mow@meinefirma.ovpn
```

### Sicherstellen, dass das Plugin "openvpn" für den Network-Manager installiert ist:
```bash
apt install -y network-manager-openvpn
```

<!-- excerpt-end -->

### Konfigurationsdatei importieren:
```bash
nmcli connection import type openvpn file mow@meinefirma.ovpn
```

### Tada:
```
Verbindung »mow@meinefirma« (41c98d04-4458-4576-9fb4-c134348cbc51) erfolgreich hinzugefügt.
```




Bis bald!

Der Mow