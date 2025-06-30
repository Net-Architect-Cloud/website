---
title: "Déployer sa première instance OpenStack !"
date: 2021-12-07T08:00:00-05:00
author: "Kevin Allioli"
tags: ["openstack", "cloud"]
categories: ["cloud"]
url: premiere-instance-openstack
thumbnail: "assets/img/headers/first-instance.webp"
abstract: "Découvrez comment déployer votre première instance OpenStack étape par étape : création, configuration réseau, sécurité, accès SSH et gestion. Un guide clair pour bien débuter sur n’importe quelle plateforme OpenStack."
description: "Découvrez comment déployer votre première instance OpenStack étape par étape : création, configuration réseau, sécurité, accès SSH et gestion. Un guide clair pour bien débuter sur n’importe quelle plateforme OpenStack."
draft: false
---

------------

Dans ce tutoriel, je vais vous montrer comment créer votre première instance sur OpenStack en utilisant le cloud public Infomaniak. Bien que j'utilise la plateforme Infomaniak pour cette démonstration, les étapes sont similaires sur n'importe quelle plateforme OpenStack.

{{< youtube DsJk0-g4nnI >}}

📺 [Voir la vidéo sur YouTube](https://www.youtube.com/watch?v=DsJk0-g4nnI)


## 🛠️ Prérequis

Pour suivre ce tutoriel, vous aurez besoin :
- D'un compte Infomaniak **(offre de 300 CHF valable 3 mois pour tester)**
- Ou d'un accès à une plateforme OpenStack


## 🌐 Connexion à la plateforme

1. Connectez-vous au portail Horizon d'Infomaniak avec vos identifiants
2. Une fois connecté, vous accédez au dashboard qui affiche vos ressources actuelles


## 💻 Création de l'instance

### Configuration de base

1. Dans le menu de gauche, allez dans la section **"Compute"** puis **"Instances"**
2. Cliquez sur **"Launch Instance"**
3. Donnez un nom à votre instance (ex: `my-first-instance`)

### Choix de l'image et des ressources

1. Dans l'onglet **"Source"**, sélectionnez une image Ubuntu 20.04 (ou le système d'exploitation de votre choix)
2. Dans l'onglet **"Flavor"**, choisissez la taille de votre instance :
   - vCPU
   - RAM
   - Disque
   - Performance disque (Perf 1 pour un usage standard)


## 🛜 Configuration réseau

1. Dans l'onglet **"Network"**, sélectionnez :
   - `ext-net1` pour avoir une IPv4 et IPv6
   - `ext-v6only1` pour uniquement IPv6


## 🔐 Sécurité et accès

1. Dans **"Security Groups"**, créez un nouveau groupe :
   - Nom : `my-perso-sg`
   - Ajoutez une règle pour autoriser SSH (port 22)
2. Dans **"Key Pair"**, créez une nouvelle paire de clés SSH :
   - Téléchargez la clé privée
   - Sauvegardez-la en tant que fichier `.pem`


## 🚀 Se connecter à l'instance

Une fois l'instance créée, connectez-vous en SSH :

```bash
ssh -i mykey.pem ubuntu@<IP_ADDRESS>
```

⚠️  Remplacez :
- `mykey.pem` par le chemin vers votre clé privée
- `<IP_ADDRESS>` par l'adresse IP publique de votre instance


## ⚙️  Gestion de l'instance

Quelques commandes utiles une fois connecté :

```bash
# Mise à jour des paquets
sudo apt update
sudo apt dist-upgrade

# Éteindre l'instance
sudo poweroff
```


## ⭐️ Points importants

- Les *Security Groups* sont **stateful** : une règle entrante crée automatiquement une règle sortante
- Supprimez vos instances inutilisées pour éviter des frais
- Le nom d'utilisateur par défaut dépend de l'image :
  - Ubuntu : `ubuntu`
  - Debian : `debian`
  - CentOS : `centos`


## 📚 Documentation supplémentaire

- [Documentation OpenStack](https://docs.openstack.org/)
- [Documentation Infomaniak](https://docs.infomaniak.cloud/)


## 🐦 On en parle ici

{{< x user="linit_io" id="1468252426524409861" >}}

## 🔗 Autres tutoriels

⚙️  Retrouvez tous nos tutoriels sur [https://netarchitect.cloud](https://netarchitect.cloud)
