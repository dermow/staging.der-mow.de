---
layout: post
title:  "Kubespray - Kubernetes Cluster im Easy-Mode" 
date:   2021-04-27 14:45:42 +0100
categories: 
  - Ansilbe
  - Kubernetes
  - k8s
---

Tag zusammen! Seid ihr schonmal in den Genuss gekommen, ein Kubernetes Cluster mit mehreren Nodes per Hand aufzusetzen? Zwar ist auch das mittlerweile 
unkomplizierter als noch vor einigen Jahren - dennoch kann das noch immer sehr kniffelig werden.

Vor einiger Zeit wurde ich durch einen Kollegen auf das Projekt "kubespray" aufmerksam gemacht. Es handelt sich hier um ein Projekt, mit dem man per Ansible oder Vagrant produktionsreife Kubernetes-Cluster extrem einfach ausrollen kann. 

Das Projekt inklusive Doku findet sich auf GitHub:

[https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)

Ich möchte euch noch kurz an einem Beispiel zeigen, wie einfach wir uns damit ein Kubernetes Cluster bauen können. In diesem Szenario haben wir 3 VMs:

* k8s-node1 (192.168.0.1) - Control Plane
* k8s-node2 (192.168.0.2) - Erste Node
* k8s-node3 (192.168.0.3) - Zweite Node

Für dieses Beispiel werden wir Ansible nutzen um das Cluster bereitzustellen. Dazu muss sichergestellt sein, dass unser Host auf die drei Nodes per SSH zugreifen kann.

<!-- excerpt-end -->

## Schritt 1: kubespray Repository klonen
``` bash
cd ~
git clone git@github.com:kubernetes-sigs/kubespray.git
cd kubespray
```

## Schritt 2: Inventory erstellen

Pro Cluster wird in kubespray ein Inventory angelegt. Im Repository finden wir bereits ein Beispiel-Inventory, welches wir uns der Einfachheit halber kopieren:

```bash
cd ~/kubespray/inventory
cp -r sample mycluster
```

Interessant ist für unser Beispiel das File "hosts.yml", welches wir auf unsere Bedürfnisse anpassen. Der Aufbau ist praktisch selbsterklärend. Das File für
unser Beispiel sieht dann so aus:

##### ~/kubespray/inventory/mycluster/hosts.ini
```ini
[all]
k8s-node1 ansible_host=192.168.0.1
k8s-node2 ansible_host=192.168.0.2
k8s-node3 ansible_host=192.168.0.3

[kube_control_plane]
k8s-node1

[etcd]
k8s-node1

[kube-node]
k8s-node1
k8s-node2
k8s-node3

[calico-rr]

[k8s-cluster:children]
kube_control_plane
kube-node
calico-rr

```

Damit hätten wir das Basis-Setup für unser Cluster auch schon fertig. Also lassen wir das ganze dann mal auf unsere Server los!

## Schritt 3: Cluster-Playbook starten

```bash
cd ~/kubespray
ansible-playbook -i inventory/mycluster/hosts.ini cluster.yml --become
```

Das Playbook läuft nun los und dauert eine ganze Weile (bei mir so ca. 15 Minuten). Anschließend war das Cluster verfügbar:

``` bash
ssh k8s-node1
sudo kubectl get nodes

NAME        STATUS   ROLES    AGE     VERSION
k8s-node1   Ready    master   26h     v1.19.2
k8s-node2   Ready    <none>   26h     v1.19.2
k8s-node3   Ready    <none>   4h39m   v1.19.2

```

Huh? So einfach? Jep, ich war ebenfalls erstaunt - wie einfach und zuverlässig das ganze funktioniert.

## Fazit

Das Beispiel beschreibt eine sehr einfache Variante. Natürlich lässt sich per Kubespray das Cluster nach den eigenen Bedürfnissen anpassen. Ich habs bisher nur für meine k8s-Testumgebung genutzt, werde es aber für künftige Produktionscluster definitiv in Betracht ziehen!

Bis denne!

Mow
