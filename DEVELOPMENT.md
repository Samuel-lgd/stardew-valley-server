# Guide de Développement Local

Ce guide explique comment configurer votre environnement de développement local pour modifier le code du mod, construire une image Docker et la publier.

## 1. Prérequis

Avant de commencer, assurez-vous d'avoir installé les outils suivants :
*   **Git** : Pour la gestion du code source.
*   **Docker Desktop** : Pour construire et exécuter des conteneurs Docker. Assurez-vous qu'il est en cours d'exécution.
*   **.NET SDK** : Version 6.0 ou supérieure, pour construire le mod C#.
*   **Make** : (Optionnel mais recommandé) Pour utiliser les commandes simplifiées du `Makefile`. Si vous êtes sur Windows, vous pouvez l'installer avec [Chocolatey](https://chocolatey.org/install) (`choco install make`).

## 2. Configuration Initiale

### a. Cloner le Dépôt
Clonez le projet depuis GitHub :
```bash
git clone https://github.com/Samuel-lgd/stardew-valley-server.git
cd stardew-valley-server
```

### b. Authentification au Registre Docker (GHCR)
Pour pouvoir pousser l'image Docker sur le registre de conteneurs de GitHub (ghcr.io), vous devez vous authentifier.

1.  **Générez un Personal Access Token (PAT) sur GitHub** :
    *   Allez dans **Settings > Developer settings > Personal access tokens > Tokens (classic)**.
    *   Cliquez sur **Generate new token (classic)**.
    *   Donnez-lui un nom (ex: `docker-push`).
    *   Cochez les permissions (`scopes`) : `write:packages` et `read:packages`.
    *   Générez le token et **copiez-le immédiatement**.

2.  **Connectez-vous via le terminal** :
    Utilisez votre nom d'utilisateur GitHub et le PAT que vous venez de créer comme mot de passe.
    ```powershell
    # Remplacez <VOTRE_NOM_UTILISATEUR>
    docker login ghcr.io -u <VOTRE_NOM_UTILISATEUR>
    ```

## 3. Cycle de Développement

### a. Modifier le Code
Vous pouvez maintenant modifier le code source du mod qui se trouve dans le dossier `mod/JunimoServer/`.

### b. Construire et Exécuter en Local
Pour tester vos modifications localement, utilisez les commandes suivantes :

1.  **Construire le mod et l'image Docker** :
    Cette commande compile le code C#, copie les fichiers nécessaires et construit l'image Docker.
    ```bash
    make build-image
    ```
    *(Si vous n'avez pas `make`, exécutez la commande `docker build...` directement depuis le `Makefile`)*.

2.  **Démarrer le serveur localement** :
    Cette commande démarre le serveur en utilisant votre image nouvellement construite.
    ```bash
    make run
    ```

Le serveur est maintenant accessible localement.

## 4. Publier les Modifications

Une fois que vos modifications sont prêtes, suivez ces étapes pour les déployer.

### a. Pousser le Code Source
Commitez et poussez vos modifications de code sur GitHub :
```bash
git add .
git commit -m "Description de vos changements"
git push origin master
```

### b. Pousser l'Image Docker
Construisez, taguez et poussez votre image Docker vers le registre GitHub (GHCR) :
```bash
make push
```
Cette commande va automatiquement construire l'image avec les dernières modifications et la téléverser.

Vos modifications sont maintenant prêtes à être déployées sur le serveur de production.
