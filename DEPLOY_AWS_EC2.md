# Guide de Déploiement de JunimoServer sur AWS EC2

Ce document vous guide pas à pas pour déployer le serveur dédié Stardew Valley (JunimoServer) sur une instance Amazon EC2, en optimisant les coûts.

## 1. Préparation de l'environnement AWS

Avant de lancer l'instance, vous devez configurer quelques éléments essentiels pour la sécurité et l'accès.

### a. Créer un Groupe de Sécurité (Security Group)
Le groupe de sécurité agit comme un pare-feu pour votre instance.
1.  Allez dans la console AWS > **EC2 > Network & Security > Security Groups**.
2.  Cliquez sur **Create security group**.
3.  Nommez-le (ex: `junimo-server-sg`) et donnez-lui une description.
4.  Dans la section **Inbound rules**, ajoutez les règles suivantes :
    *   **SSH (Port 22)** : Pour vous connecter à l'instance.
        *   Type: `SSH`
        *   Source: `My IP` (pour restreindre l'accès à votre adresse IP actuelle).
    *   **Stardew Valley Game Port (Port 24643)** : Pour que les joueurs puissent se connecter.
        *   Type: `Custom UDP`
        *   Port range: `24643`
        *   Source: `Anywhere` (0.0.0.0/0) ou restreignez aux IPs de vos amis.
    *   **VNC Web Interface (Port 8090)** : Pour l'administration web.
        *   Type: `Custom TCP`
        *   Port range: `8090`
        *   Source: `My IP` (fortement recommandé pour la sécurité).
5.  Cliquez sur **Create security group**.

### b. Créer une Paire de Clés (Key Pair)
Cette clé est nécessaire pour vous connecter en SSH à votre instance.
1.  Allez dans **EC2 > Network & Security > Key Pairs**.
2.  Cliquez sur **Create key pair**.
3.  Nommez-la (ex: `junimo-key`), choisissez le format **.pem**, et cliquez sur **Create**.
4.  **Téléchargez et conservez ce fichier en lieu sûr.** Vous ne pourrez plus le télécharger à nouveau.

## 2. Lancement de l'instance EC2

1.  Allez dans le **EC2 Dashboard** et cliquez sur **Launch instance**.
2.  **Nom**: Donnez un nom à votre instance (ex: `StardewValley-Server`).
3.  **Application and OS Images (AMI)**: Choisissez `Amazon Linux 2023 AMI`. C'est une option gratuite et bien maintenue.
4.  **Instance type**: Pour un faible coût, choisissez `t3a.small` ou `t2.micro`. Le `t2.micro` est éligible au Free Tier d'AWS, mais le `t3a.small` offre un meilleur rapport performance/prix si vous avez plusieurs joueurs.
5.  **Key pair (login)**: Sélectionnez la clé que vous avez créée (`junimo-key`).
6.  **Network settings**:
    *   Cliquez sur **Edit**.
    *   **Firewall (security groups)**: Choisissez **Select existing security group** et sélectionnez `junimo-server-sg`.
7.  **Storage (volumes)**: 30 Go de stockage `gp3` est un bon point de départ et offre un bon équilibre coût/performance.
8.  Cliquez sur **Launch instance**.

## 3. Connexion et Configuration de l'Instance

Une fois l'instance en cours d'exécution (`Running`), sélectionnez-la et copiez son **Public IPv4 address**.

### a. Se connecter en SSH
Ouvrez un terminal et utilisez la commande suivante (remplacez les valeurs) :
```bash
# Sur Windows, vous devrez peut-être ajuster les permissions du fichier .pem
# chmod 400 /chemin/vers/votre-cle.pem

ssh -i /chemin/vers/votre-cle.pem ec2-user@<VOTRE_IP_PUBLIQUE>
```

### b. Installer les dépendances
Une fois connecté, exécutez les commandes suivantes pour mettre à jour le système et installer Docker, Git et Docker Compose.

```bash
# Mettre à jour le système
sudo dnf update -y

# Installer Docker
sudo dnf install -y docker
sudo systemctl enable --now docker
sudo usermod -aG docker ec2-user
newgrp docker # Applique les permissions de groupe sans se déconnecter

# Installer Docker Compose
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p "$DOCKER_CONFIG/cli-plugins"
curl -SL https://github.com/docker/compose/releases/download/v2.24.7/docker-compose-linux-x86_64 -o "$DOCKER_CONFIG/cli-plugins/docker-compose"
chmod +x "$DOCKER_CONFIG/cli-plugins/docker-compose"

# Vérifier l'installation
docker compose version

# Installer Git
sudo dnf install -y git
```

## 4. Déploiement de JunimoServer

### a. Cloner le projet
Clonez le dépôt du serveur dans le répertoire de l'utilisateur `ec2-user`.
```bash
cd /home/ec2-user
git clone https://github.com/stardew-valley-dedicated-server/server.git
cd server
```

### b. Configurer le serveur
Copiez le fichier d'exemple `.env.example` et modifiez-le pour y mettre vos informations.
```bash
cp .env.example .env
nano .env
```
Modifiez les valeurs suivantes dans l'éditeur `nano` :
*   `STEAM_USER`: Votre nom d'utilisateur Steam.
*   `STEAM_PASS`: Votre mot de passe Steam.
*   `VNC_PASSWORD`: Un mot de passe pour l'interface web (au moins 6 caractères).

**Note sur Steam Guard** : Laissez `STEAM_GUARD_CODE` vide au premier lancement. Si la console Docker vous demande un code, vous devrez arrêter le serveur, éditer le fichier `.env` pour ajouter le code, puis relancer.

Sauvegardez (`Ctrl+O`) et quittez `nano` (`Ctrl+X`).

### c. Lancer le serveur
Utilisez Docker Compose pour télécharger l'image et démarrer le serveur en arrière-plan.
```bash
docker compose up -d
```
Le premier démarrage peut être long, car Docker télécharge l'image du serveur et le serveur lui-même télécharge les fichiers du jeu Stardew Valley via Steam.

Pour suivre les logs :
```bash
docker compose logs -f
```
Appuyez sur `Ctrl+C` pour quitter les logs sans arrêter le serveur.

## 5. Accès et Maintenance

### a. Se connecter au jeu
*   Dans Stardew Valley, allez dans **Co-op > Join > Join LAN game**.
*   Entrez l'**adresse IP publique** de votre instance EC2.
*   Le port par défaut est `24643`.

### b. Accéder à l'interface VNC
*   Ouvrez votre navigateur et allez sur `http://<VOTRE_IP_PUBLIQUE>:8090`.
*   Utilisez le mot de passe `VNC_PASSWORD` que vous avez défini.

### c. Redémarrer ou arrêter le serveur
Pour arrêter le serveur :
```bash
cd /home/ec2-user/server
docker compose down
```

Pour redémarrer le serveur :
```bash
cd /home/ec2-user/server
docker compose down && docker compose up -d
```

### d. Rendre le serveur persistant (démarrage automatique)
Pour que le serveur redémarre automatiquement si l'instance EC2 est redémarrée, créez un service `systemd`.
```bash
sudo tee /etc/systemd/system/junimoserver.service >/dev/null <<'EOF'
[Unit]
Description=JunimoServer via Docker Compose
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
WorkingDirectory=/home/ec2-user/server
ExecStart=/home/ec2-user/.docker/cli-plugins/docker-compose up -d
ExecStop=/home/ec2-user/.docker/cli-plugins/docker-compose down
RemainAfterExit=yes
User=ec2-user

[Install]
WantedBy=multi-user.target
EOF

# Activer et démarrer le service
sudo systemctl daemon-reload
sudo systemctl enable --now junimoserver
```

### e. Mettre à jour le serveur
Pour mettre à jour JunimoServer avec la dernière version :
```bash
cd /home/ec2-user/server
git pull # Met à jour le code source (si nécessaire)
docker compose pull # Télécharge la dernière image Docker
docker compose up -d --force-recreate # Redémarre le serveur avec la nouvelle image
docker image prune -f # Supprime les anciennes images inutilisées
```

### f. Sauvegardes
Les données du jeu sont stockées dans des volumes Docker sur le disque de l'instance. La méthode la plus simple et la plus sûre pour les sauvegarder est de créer des **snapshots EBS** de votre volume via la console AWS. Vous pouvez automatiser cela avec **AWS Backup**.

## 6. Dépannage

*   **Impossible de se connecter ?**
    *   Vérifiez que votre groupe de sécurité autorise bien le trafic sur les ports UDP 24643 et TCP 8090.
    *   Assurez-vous que l'adresse IP que vous utilisez est bien l'IP publique de l'instance.
*   **Le serveur ne démarre pas (logs Docker) ?**
    *   Vérifiez vos identifiants Steam dans le fichier `.env`.
    *   Si Steam Guard est activé, remplissez le champ `STEAM_GUARD_CODE`.
*   **Optimisation des coûts**
    *   Pensez à **arrêter** votre instance EC2 lorsque vous ne l'utilisez pas pour éviter les frais.
    *   Utilisez les alarmes **CloudWatch** pour surveiller l'utilisation du CPU et être alerté en cas de surcharge.
    *   Envisagez une **IP Élastique** (Elastic IP) si vous arrêtez/redémarrez souvent l'instance, pour garder une adresse IP fixe (gratuite tant qu'elle est attachée à une instance en cours d'exécution).

Ce guide devrait vous permettre de mettre en place votre serveur Stardew Valley de manière fonctionnelle et économique. Bon jeu !