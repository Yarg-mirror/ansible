# Ansible

## Présentation

Voici un projet contenant la compilation de mes différents scripts [ansible](https://docs.ansible.com) pour installer, configurer, sauvegarder et restaurer différents services.

Le principe de ce projet est d'utiliser une machine (Physique, VM, Conteneur) définie comme "contrôleur" qui aura pour but de déployer, gérer et maintenir les différentes machines qu'il créera ou ajoutera à son réseau.

Le contrôleur ayant les pleins pouvoir sur la totalité des systèmes qu'il gèrera, il est important de prendre en compte sa sécurité et de limiter les possibilitées d'accès à celui-ci.

Cependant, il n'est pas obligatoire de le mettre en place et de l'utiliser, tous les roles et playbooks présents peuvent également être utilisé depuis n'importe quelle instance ansible.

## Utilisation du contrôleur

### Installation du système

Pour le choix du système, je vais partir sur la mise en place de [Debian](https://debian.org) en version 12 Stable à la date de rédaction de ce document.

Voici les différentes actions effectuées pendant l'installation:

- Définir le nom et domaine de la machine
- Définir le mot de passe de root
- Créer un utilisateur "ansible"
- Définir le mot de passe de l'utilisateur
- Configurer les disques
  - automatique et tout sur une partition est tout à fait acceptable
- Au moment de l'execution de **tasksel** déselectionner tout.

L'objectif étant d'avoir un système réellement minimal, la sécurité étant importante, chiffrer tout ou partie du disque peut être une option intéressante. C'est au choix de chaque personne.

### Premier démarrage en tant que root

#### Installation des paquets de base

Ces installations sont le minimum nécessaire pour pouvoir se connecter au serveur à distance, récupérer le projet et executer ansible.

```
apt-get install --no-install-recommends openssh-server python3 ansible git dbus dbus-user-session ca-certificates
chown -R ansible:ansible /opt
```

#### Préparation de l'écran de connexion

Supprimer le fichier **/etc/update-motd.d/10-uname**
Créer le fichier **/etc/update-motd.d/00-ansible**
```
#!/bin/sh

cat /opt/ansible/motd 2>/dev/null
```

### Préparation de la connexion au contrôleur

#### Création et envoi des clé ssh

Cette opération fonctionne autant sur un système Linux que Windows sous PowerShell ou WSL 2, penser simple à adopter les dossiers de destination.
```
ssh-keygen -t ed25519 -f /home/<utilisateur>/.ssh/controller_ed25519
ssh-copy-id -i /home/<utilisateur>/.ssh/controller_ed25519.pub ansible@<IP ou adresse DNS>
```

Connexion au contrôleur
```
ssh -i /home/<utilisateur>/.ssh/controller_ed25519 ansible@<IP ou adresse DNS>
```

Il est également possible de configurer le fichier **[~/.ssh/config](https://linuxize.com/post/using-the-ssh-config-file/)** pour simplifier la commande de connexion
```
Host controller
        Hostname <IP ou adresse DNS>
        User ansible
        IdentityFile ~/.ssh/controller_ed25519
```

### Initialisation du contrôleur

Une fois connecté en tant qu'utilisateur "ansible", pour l'execution du playbook d'auto-déploiement, ansible vous demandera le mot de passe de l'utilisateur root.

```
git clone https://github.com/Yarg-mirror/ansible.git /opt/ansible
ln /opt/ansible/ansible.cfg ~/
ansible-playbook playbooks/take-over.yml  -l controller -Kk
ansible-playbook playbooks/deploy.yml  -l controller
```
