---
title: "Migrer de VMware vers OpenStack"
date: 2025-06-24T10:00:00-05:00
author: "Kevin Allioli"
tags:
  - openstack
  - vmware
  - migration
  - cloud
categories:
  - cloud
thumbnail: "assets/img/headers/vmware-openstack-migration.webp"
abstract: "Découvrez comment migrer efficacement vos machines virtuelles depuis VMware ESXi vers OpenStack. Une approche claire, reproductible, et indépendante du système d’exploitation."
description: "Découvrez comment migrer efficacement vos machines virtuelles depuis VMware ESXi vers OpenStack. Une approche claire, reproductible, et indépendante du système d’exploitation."
draft: false
---

Dans cette vidéo, je vous montre comment migrer une **machine virtuelle VMware (ESXi) vers OpenStack** en utilisant l’outil open source `virt-v2v`. Ce guide s’applique quelle que soit la VM à convertir : serveur applicatif, base de données, frontend ou autre, qu’il s’agisse de Linux, Windows ou tout autre OS.

> 📝 Une version enrichie de cet article est également disponible sur le blog d’Infomaniak :  
> [Lire l’article original](https://news.infomaniak.com/migrer-vmware-server-vers-openstack/)

{{< youtube WBKdnQdva8k >}}

📺 [Voir la vidéo sur YouTube](https://www.youtube.com/watch?v=WBKdnQdva8k)


## 🧭 Objectif

Migrer une machine virtuelle complète (stockage, configuration, compatibilité matérielle) depuis un hyperviseur **VMware ESXi** vers une infrastructure **OpenStack**, en préservant son intégrité et sa capacité à démarrer dans le nouvel environnement.


## 🛠️ Préparer l’environnement

Avant de commencer, il vous faudra :

- Une **instance Linux (de préférence base RHEL)** sur votre cloud OpenStack pour piloter la migration
- L’**accès SSH à votre hôte ESXi**
- Le **fichier OpenStack RC** de votre projet
- L’outil `virt-v2v` (disponible dans les dépots)

### Connexion à l’instance de migration

```bash
ssh -i key.pem rocky@<IP_INSTANCE>
```


## ⚙️ Installation des outils

```bash
sudo dnf install centos-release-openstack-caracal
sudo dnf install python3-openstackclient virt-v2v
```

Et rechargez si nécessaire :

```bash
sudo reboot
```


## 🔐 Authentification et prérequis

> ⚠️ Les actions suivantes sont à réalisées **depuis l'instance** précédement déployée.

1. **Accès ESXi via SSH** :

```bash
ssh root@esxi.domain.tld
```

2. **Connexion à OpenStack via OpenRC** :

```bash
source openrc.sh
openstack token issue
```

3. **Création d’un fichier mot de passe** :

```bash
echo "mot_de_passe_esxi" > passwordfile
```

4. **Récupération de l’ID de l’instance de migration** :

```bash
dmidecode -s system-serial-number
```


## 🚚 Lancer la migration avec virt-v2v

Commande type :

```bash
virt-v2v -i vmx \
  -it ssh \
  -ip passwordfile \
  -ic root@esxi.domain.tld \
  -os datastore/ma-vm/ma-vm.vmx \
  -of openstack \
  -oo id=<ID_INSTANCE_MIGRATION>
```

📦 Cette commande :
- Convertit la VM depuis son format VMware
- Crée un **volume OpenStack** contenant l’image convertie
- Installe automatiquement les pilotes nécessaires (virtio, QEMU agent…)


## 🚀 Démarrer la VM sur OpenStack

1. Allez dans **Volumes > Launch as Instance**
2. Sélectionnez une flavor avec **disque = 0** (boot from volume)
3. Configurez le réseau et les options de sécurité
4. Lancez l’instance depuis le volume migré


## ⚠️ À savoir en cas de blocage

Si la VM reste bloquée à l’état *Spawning* (ça arrive sur certaine plaformes Openstack):

1. Supprimez l’instance (gardez le volume)
2. Retirez cette propriété metadata :

```bash
openstack volume unset --property hw_machine_type <ID_VOLUME>
```

3. Relancez une instance depuis le volume corrigé


## ✅ Résumé des points clés

- `virt-v2v` prend en charge **tout type de VM**
- La migration repose sur **une conversion + un volume OpenStack**
- Il est crucial de **préparer correctement l’environnement SSH et OpenStack**
- Le processus est **automatisé**, reproductible, et **OS-agnostique**


## 📚 Pour aller plus loin

- [Documentation virt-v2v](https://access.redhat.com/documentation/en-us/red_hat_virtualization/4.4/html/migration_toolkit_for_virtualization_guide/)
- [Documentation OpenStack](https://docs.openstack.org/)
- [Article original Infomaniak](https://news.infomaniak.com/migrer-vmware-server-vers-openstack/)


## 🔗 Autres tutoriels

📚 Retrouvez tous nos guides sur [https://netarchitect.cloud](https://netarchitect.cloud)
