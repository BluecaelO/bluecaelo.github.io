---
title: Créer un environement de Dev éphémère (Nix-shell + Direnv)
date: 2025-08-23
description: Utilisation de Nix et de Direnv pour la création de environnement de développement éphémère
tags:
  - nix
  - direnv
---

![Image](Billboard.svg)
## Introduction

Je ne sais pas pour vous, mais j’aime avoir un environnement aussi propre que possible. Pour cela, j’utilise **Nix** et **direnv**.

* **Nix** me servira à :

  * installer des paquets de manière éphémère ;
  * éviter les problèmes de dépendances.

* **direnv**, quant à lui, m’aidera à :

  * lancer mes fichiers de configuration Nix en me déplaçant dans un répertoire spécifique.

Dans cet article, je vais vous montrer comment créer un environnement **reproductible** et **portable**, tout en limitant le nombre de commandes à taper 🦾 .

## Prérequis

* Avoir un système **Linux** ou **macOS**.
* Si vous êtes sous **Windows**, vous devrez utiliser **WSL** ou passer par une machine virtuelle via **VirtualBox** ou **VMware Workstation**.

## Installation

L’installation est relativement simple, mais dépend de votre système.
Par exemple, sous **Fedora**, vous pouvez installer `direnv` avec :

```bash
sudo dnf install direnv
```

Pour ce qui est de **Nix**, il vous suffit d’exécuter le script proposé sur le [site officiel](https://nixos.org/download.html).

## Configuration

### Configurer direnv

On peut faire énormément de choses avec **direnv**. Il est souvent utilisé pour ajouter automatiquement des variables d’environnement lorsqu’on entre dans un dossier spécifique.

Ici, je vais l’utiliser pour exécuter une commande qui lancera le fichier `shell.nix`, lequel contiendra tous les paquets que je souhaite installer.

Concrètement, avant chaque prompt, **direnv** vérifie la présence d’un fichier `.envrc` dans le dossier courant.
Si ce fichier est trouvé, il est "sourcé", ce qui signifie que :

* les variables d’environnement qu’il contient sont ajoutées au shell ;
* les commandes définies dans ce fichier sont exécutées.

Dans mon cas, je vais uniquement utiliser **direnv** pour appeler le fichier `shell.nix`.

Si vous devez déclarer des variables d’environnement, vous pouvez de le faire directement dans le fichier `shell.nix`.

Ajoutez la ligne suivante à votre fichier `~/.bashrc` (ou `~/.zshrc` si vous utilisez Zsh) :

```bash
eval "$(direnv hook zsh)"  # Remplacez zsh par bash si nécessaire
```

La ligne ci-dessus ajoute un *hook*.
C’est ce qui permet à **direnv** de rechercher le fichier `.envrc` avant que l’invite de commande n’apparaisse.

Vous pouvez maintenant créer un fichier `.envrc` dans le dossier de votre projet, et y ajouter la ligne suivante :

```bash
use nix
```

Cela indique à **direnv** d’utiliser le fichier `shell.nix` présent dans le dossier pour configurer l’environnement.
La première fois que vous créez un `.envrc`, vous devrez l’autoriser avec :

```bash
direnv allow
```

### Création du fichier `shell.nix`

Nix est un gestionnaire de paquets fonctionnel. Chaque paquet est stocké dans le Nix store, dans son propre répertoire, et ne sera jamais modifié.
Cela signifie également que si vous installez une autre version d’un paquet déjà présent, celle-ci aura son propre répertoire dans le Nix store.
Ce fonctionnement permet d’éviter les problèmes de dépendances.

Imaginons que je veuille installer de manière éphémère une suite de paquets Python, comme **pandas** et **flask**.
Dans mon fichier `shell.nix`, je vais devoir ajouter les lignes suivantes :

```nix
{ pkgs ? import <nixpkgs> {} }:  # Importation du module nixpkgs
pkgs.mkShell { # Création du shell
 packages = [ # Liste des packages à installer
      (pkgs.python312.withPackages (python-pkgs: [ # Installation de Python avec ses packages
        python-pkgs.pandas
        python-pkgs.flask
      ]))
  ];
}
```

> Je vous recommande d’être le plus explicite possible sur la version de Python utilisée.
> Cela permet d’éviter les désagréments liés à des différences de version.

Maintenant, pour lancer le script, il vous suffit de vous rendre dans le répertoire dans lequel il a été créé — et c’est globalement tout 😁.
Dans le cas où direnv ne serait pas installé, vous devriez exécuter la commande `nix-shell <chemin vers le fichier .nix>`.
Ça ne paraît peut-être pas grand-chose, mais cela simplifie vraiment le quotidien : ce sont quelques secondes gagnées à chaque fois... qui finiront par s’accumuler !


**Nix** ne s’arrête pas à Python. Vous pouvez installer n’importe quel paquet disponible dans **nixpkgs**.

Prenons un autre exemple : imaginons que je veuille installer de manière éphémère des outils comme **Ansible** et **OpenTofu** (une implémentation open-source de Terraform).
Pour ce faire, je dois simplement écrire le fichier `shell.nix` suivant :

```nix
{ pkgs ? import <nixpkgs> {} }:  # Importation du module nixpkgs
pkgs.mkShell { # Création du shell
  packages = [ # Liste des packages à installer
    pkgs.opentofu
    pkgs.ansible
  ];
}
```

Une fois que vous avez terminé d’utiliser votre environnement de développement, il vous suffit de quitter le répertoire du projet, et direnv se charge du reste.
Les paquets installés ne seront alors plus accessibles en dehors de cet espace de travail.


## Conclusion
Voilà qui conclut ce petit post. Ce n’est pas grand-chose : je n’ai exploré qu’une petite partie des véritables capacités de Nix.
(Je vous conseille vivement d’y jeter un œil, même si cela peut sembler un peu complexe au début.)

Nous avons donc vu :
- comment utiliser direnv pour exploiter un fichier Nix ;
- comment installer des paquets de manière éphémère avec un script shell.nix.

En espérant que cela vous aura plu 😊.

---

# Sources

- [Nix wiki](https://nixos.wiki/wiki/Main_Page)
- [Nix Direnv](https://github.com/nix-community/nix-direnv)

Bonnus:
- [Premier pas avec Nix](https://lafor.ge/nix-1/#la-philosophie-derriere-nix)



