---

title: "Configurer ses images cloud avec Glance"
date: 2025-08-20T10:00:00-05:00
author: "Kevin Allioli"
tags: ["openstack", "glance", "images", "virtio", "cloud"]
categories: ["cloud"]
thumbnail: "assets/img/headers/glance-cheatsheet.webp"
url: openstack-glance-image-cheatsheet
abstract: "Apprenez à utiliser efficacement les propriétés d’image dans OpenStack Glance. Ce guide couvre Linux et Windows, avec un focus sur les performances via VirtIO et la compatibilité Secure Boot."
description: "Un guide pratique pour comprendre et utiliser les propriétés d'image dans OpenStack Glance, avec des exemples concrets Linux, Windows et VirtIO."
draft: false

---

Quand on ajoute une image dans OpenStack, lancer la première instance peut vite tourner au casse-tête : écran noir, carte réseau absente, images mal organisées dans Horizon… Autant de petits soucis très courants dont on se passerait bien. C’est souvent à ce moment-là qu’on réalise qu’une image cloud n’est pas qu’un disque : c’est aussi un **ensemble de métadonnées**. Dans OpenStack, ces métadonnées — les *image properties* — sont gérées par le service **Glance**. Bien définies, elles pilotent le comportement de **Nova**, **Ironic** et **Cinder** pour que vos images fonctionnent comme prévu.

Ce guide vous montre comment poser les bonnes propriétés (OS, version, archi, Secure Boot), tirer parti de **VirtIO** pour les perfs, et éviter les pièges classiques **Linux vs Windows**. On y glisse aussi la bonne habitude d’utiliser la nomenclature **libosinfo** pour garder un catalogue clean et scriptable. On y va ?


## 🎯 Objectif

* Comprendre les propriétés d’image Glance **les plus utiles**
* Voir les différences clés **Linux vs Windows**
* Optimiser pour **la perf (VirtIO)** et **la compatibilité (Secure Boot, IDE/e1000)**
* Disposer d’un **cheat sheet prêt à l’emploi** (*sans remplacer la doc officielle* 😉)


## 🖥️ Propriétés système & OS

| Propriété        | Exemple             | Utilité                 |
| ---------------- | ------------------- | ----------------------- |
| `architecture`   | `x86_64`            | Architecture CPU        |
| `os_distro`      | `ubuntu`, `windows` | Nom de l’OS (libosinfo) |
| `os_version`     | `24.04`, `2019`     | Version de l’OS         |
| `os_type`        | `linux`, `windows`  | Famille d’OS            |
| `os_secure_boot` | `required`          | Active Secure Boot      |

> 💡 **Astuce pratique**
> Renseignez `os_distro` et `os_version` avec des **valeurs normalisées** (voir *libosinfo* ci-dessous).  
> Ça évite `Ubuntu` vs `ubuntu` (ou `Win2019` vs `windows`), simplifie les scripts, et garde vos listes propres dans Horizon.

### ❓ libosinfo, c’est quoi et à quoi ça sert ?

**libosinfo** est une base de connaissances des systèmes d’exploitation (projet open source) utilisée par de nombreux outils de virtualisation.
Elle fournit des **noms et versions normalisés** ainsi que des infos utiles (drivers, compatibilité, paramètres recommandés).

Dans notre contexte, l’utiliser pour **nommer `os_distro`/`os_version`** apporte :

* de la **cohérence** (scripts, CI/CD, filtres, tagging),
* une **meilleure lisibilité** dans les UIs,
* moins d’erreurs dues aux variations de casse ou d’orthographe.

**Exemples conformes :**

```
os_distro=ubuntu    os_version=24.04
os_distro=debian    os_version=12
os_distro=rocky     os_version=9
os_distro=windows   os_version=2019
```

> Ce n’est pas obligatoire pour Glance, mais c’est une bonne pratique.


## ⚙️ Propriétés hyperviseur

| Propriété          | Exemple    | Description                |
| ------------------ | ---------- | -------------------------- |
| `hypervisor_type`  | `kvm`      | Hyperviseur cible          |
| `img_config_drive` | `optional` | Config-drive requis ou non |
| `instance_uuid`    | `<UUID>`   | UUID source si snapshot    |

## 🌀 Propriétés VirtIO (performance)

| Propriété             | Exemple           | Description          |
| --------------------- | ----------------- | -------------------- |
| `hw_disk_bus`         | `virtio`, `ide`   | Contrôleur disque    |
| `hw_scsi_model`       | `virtio-scsi`     | Contrôleur SCSI perf |
| `hw_vif_model`        | `virtio`, `e1000` | Carte réseau         |
| `hw_rng_model`        | `virtio`          | RNG paraphrérique    |
| `hw_qemu_guest_agent` | `yes`             | QEMU guest agent     |

⚡ **Linux** : passez **directement** en `virtio` (pris en charge nativement par le noyau).  
🪟 **Windows** : démarrez en `ide + e1000`, puis **basculez en `virtio`** après installation des pilotes VirtIO.

## 🔐 Propriétés sécurité

| Propriété                               | Exemple             | Description              |
| --------------------------------------- | ------------------- | ------------------------ |
| `cinder_encryption_key_id`              | `<UUID>`            | Clé KMS pour volumes     |
| `cinder_encryption_key_deletion_policy` | `on_image_deletion` | Politique de suppression |

## 🚀 Exemples pratiques

### Créer une image **Ubuntu 24.04** optimisée VirtIO

```bash
openstack image create \
  --file ubuntu-24.04.qcow2 \
  --disk-format qcow2 \
  --container-format bare \
  --property os_distro=ubuntu \
  --property os_version=24.04 \
  --property os_type=linux \
  --property architecture=x86_64 \
  --property hw_disk_bus=virtio \
  --property hw_scsi_model=virtio-scsi \
  --property hw_vif_model=virtio \
  --property hw_rng_model=virtio \
  --property hw_qemu_guest_agent=yes \
  "ubuntu-24.04-virtio"
```

### Créer une image **Windows Server 2019** (mode compat)

```bash
openstack image create \
  --file windows-server-2019.qcow2 \
  --disk-format qcow2 \
  --container-format bare \
  --property os_distro=windows \
  --property os_version=2019 \
  --property os_type=windows \
  --property architecture=x86_64 \
  --property hw_disk_bus=ide \
  --property hw_vif_model=e1000 \
  "windows-server-2019"
```

👉 Ensuite, dans la VM Windows :

* installez **les pilotes VirtIO**,
* installez **Cloudbase-Init**,
* puis migrez les propriétés :

```bash
openstack image set <WIN_IMAGE_ID> \
  --property hw_disk_bus=virtio \
  --property hw_scsi_model=virtio-scsi \
  --property hw_vif_model=virtio
```


## 🔄 Comparatif Linux vs Windows (valeurs recommandées)

| Propriété                  | Linux (Ubuntu, CentOS…)      | Windows (Server 2019/2022/2025)    |
| -------------------------- | ---------------------------- | ---------------------------------- |
| **os\_type**               | `linux`                      | `windows`                          |
| **os\_distro**             | `ubuntu`, `centos`, `rocky`… | `windows`                          |
| **os\_version**            | `24.04`, `9`…                | `2019`, `2022`, `2025`             |
| **architecture**           | `x86_64`                     | `x86_64`                           |
| **hw\_disk\_bus**          | `virtio` / `scsi`            | `ide` → `virtio` (après drivers)   |
| **hw\_scsi\_model**        | `virtio-scsi`                | `virtio-scsi` (après drivers)      |
| **hw\_vif\_model**         | `virtio`                     | `e1000` → `virtio` (après drivers) |
| **hw\_rng\_model**         | `virtio`                     | `virtio` (si driver installé)      |
| **hw\_qemu\_guest\_agent** | `yes`                        | `yes` (Cloudbase QGA)              |
| **img\_config\_drive**     | `optional`                   | `optional`                         |
| **os\_secure\_boot**       | `optional` / `required`      | `required` (UEFI + Secure Boot)    |

✅ **Linux** → VirtIO partout dès le départ  
✅ **Windows** → IDE/e1000 pour installer → VirtIO après drivers + Cloudbase-Init


## ✅ En résumé

* Les **image properties** pilotent concrètement **boot, compat, perf**.
* **VirtIO** = gain immédiat (IO & réseau).
* **Windows** : compat d’abord (IDE/e1000), **VirtIO ensuite**.
* Standardisez `os_distro/os_version` avec **libosinfo** pour rester propre et scriptable.
* Pensez au **QEMU guest agent** et à la **description** de l’image (catalogue lisible).


## 📚 Pour aller plus loin

* Glance – *Useful image properties* : [https://docs.openstack.org/glance/latest/admin/useful-image-properties.html](https://docs.openstack.org/glance/latest/admin/useful-image-properties.html)
* **libosinfo / osinfo-db** (noms & versions normalisés) : [https://libosinfo.org](https://libosinfo.org)
* Cloudbase-Init (Windows) : [https://cloudbase.it/cloudbase-init/](https://cloudbase.it/cloudbase-init/)
* Pilotes VirtIO (Windows) : [https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/)
