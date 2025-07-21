+++
date = '2025-07-16T23:08:55+02:00'
draft = false
title = 'Secure Shell'
author = "Alexandre Trigueros"
+++

# 🔐 SSH - Secure Shell

SSH est un protocole de communication sécurisé permettant d'établir une connexion chiffrée entre deux machines, il est utilisé pour accéder à un serveur à distance ou exécuter des commandes. Il est également possible de transférer des fichiers de manière sécurisée d'une machine à l'autre avec `scp` ou `sftp` ou encore de faire des tunnels pour permettre à l'hote d'accéder à certains ports du client ou vice-versa.

## Connection et authentification

Se connecter à un serveur se fait avec la commande

```bash
ssh user@server
```

Par défaut, le port dédié au ssh est le port **22**, mais le paramètre -p permet de spécifier un autre port.

SSH va d'abord essayer d'authentifier l'utilisateur avec ses clés de chiffrement et si aucune ne correspond, un mot de passe lui est demandé.

---

Les étapes pour la créer et enregistrer la clé sur le serveur sont:

1. Générer une paire de clés (si ce n’est pas déjà fait) :

```bash
ssh-keygen -b 4096
```

(Certains serveurs n'acceptent que le RSA, l'argument `-t rsa` permet de spécifier l'algorithme)

2. Copier la clé publique sur le serveur :

```bash
ssh-copy-id user@serveur
```

---

## Cheat sheet

### CLI

| Action | Commande |
|--------|----------|
| Connexion simple | `ssh user@host` |
| Connexion en précisant la clé privée | `ssh -i /chemin/vers/id_rsa user@host` |
| Exécuter une commande sur le serveur distant | `ssh user@host 'commande'` |
| Expose un port sur serveur distant sur client (hote_cible est tel que vu depuis le serveur distant) | `sssh -L [port_local]:[hôte_cible]:[port_cible] utilisateur@serveur` |
| Expose un port local sur le serveur distant (hote_local est tel que vu depuis le client) | `ssh -R [port_cible]:[hote_local]:[port_local] user@host` |
| Transfert de fichiers (scp) | `scp fichier.txt user@host:/chemin` |
| SFTP interactif | `sftp user@host` |

---

### Fichier de configuration `~/.ssh/config`

Permet de simplifier les connexions et de passer des paramètres par défaut selon à quel server on se connecte.

Par exemple:

```bash
Host monserveur
  HostName 192.168.1.10
  User user
  IdentityFile ~/.ssh/id_rsa
```

---

## Bonnes pratiques de sécurité

- Utiliser des **clés SSH** au lieu de mots de passe
- Désactiver l'accès root distant (`PermitRootLogin no`)
- Désactiver la connection par mot de passe (`PasswordAuthentication no`)
- Limiter les IP autorisées / les ports exposés via `iptables` ou `ufw`
- Modifier le port par défaut (`Port 22`)
- Utiliser des outils comme `fail2ban` contre les tentatives de brute force
