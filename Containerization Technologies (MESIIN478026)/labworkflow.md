# Workspace GitHub + Docker dans VSCode

**DevOps Foundations with Docker - ESILV 2026/2027**  
**Support - TD1 / GitHub / Evidence**

Objectif :

```text
workspace local
      ↓
Dockerfile + fichiers
      ↓
build + run
      ↓
evidence/
      ↓
Git commit
      ↓
GitHub PRIVATE
```

> Terminal : Bash / Git Bash / WSL / Linux / macOS.

---

# 0. Machine not ready? - Git + GitHub + VS Code + Docker

## Comprendre les 4 éléments

```text
Git       = versionner les fichiers sur la machine
GitHub    = compte + repository dans le cloud
VS Code   = éditeur / workspace
Docker    = construire et exécuter les containers
```

> GitHub n'est pas un logiciel à installer.  
> Il faut créer un compte GitHub et installer Git sur la machine.

---

## 0.1 Diagnostic rapide

### Linux / macOS

```bash
git --version
code --version
docker version
```

Si une commande affiche :

```text
command not found
```

installer uniquement l'outil manquant.

### Windows PowerShell

```powershell
git --version
code --version
docker version
```

---

# 0.2 Ubuntu 24.04 - installation recommandée

## A. Installer Git

```bash
sudo apt update
sudo apt install -y git
```

Vérifier :

```bash
git --version
```

---

## B. Installer VS Code

Méthode simple :

```bash
sudo snap install --classic code
```

Vérifier :

```bash
code --version
```

Ouvrir VS Code :

```bash
code
```

---

## C. Installer Docker Engine

```bash
sudo apt update
sudo apt install -y ca-certificates curl
```

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
```

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Ajouter le repository Docker :

```bash
sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Installer Docker :

```bash
sudo apt update
```

```bash
sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

Tester :

```bash
sudo docker run --rm hello-world
```

Autoriser Docker sans `sudo` :

```bash
sudo usermod -aG docker "$USER"
```

Puis :

```bash
newgrp docker
```

Retester :

```bash
docker version
docker compose version
docker run --rm hello-world
```

Attendu :

```text
Hello from Docker!
```

---

# 0.3 Windows - installation rapide

Ouvrir **PowerShell**.

## A. Git

```powershell
winget install --id Git.Git -e
```

## B. VS Code

```powershell
winget install --id Microsoft.VisualStudioCode -e
```

## C. Docker Desktop

```powershell
winget install --id Docker.DockerDesktop -e
```

Fermer puis rouvrir le terminal.

Démarrer :

```text
Docker Desktop
```

Si Docker demande un backend :

```text
Use WSL 2
```

Vérifier WSL :

```powershell
wsl --status
```

Si WSL n'est pas installé :

```powershell
wsl --install
```

Redémarrer Windows si demandé.

Puis vérifier :

```powershell
git --version
code --version
docker version
docker compose version
```

Tester :

```powershell
docker run --rm hello-world
```

---

# 0.4 macOS - installation rapide

Si Homebrew est déjà installé :

```bash
brew install git
brew install --cask visual-studio-code
brew install --cask docker
```

Démarrer Docker Desktop :

```bash
open -a Docker
```

Puis :

```bash
git --version
code --version
docker version
docker compose version
```

Tester :

```bash
docker run --rm hello-world
```

Si Homebrew n'est pas installé, utiliser les installateurs officiels de VS Code et Docker Desktop.

---

# 0.5 Créer le compte GitHub

Dans le navigateur :

```text
https://github.com/signup
```

Créer le compte.

Puis vérifier l'adresse email GitHub.

Aucun repository n'est encore nécessaire.

---

# 0.6 Configurer Git une seule fois

Dans le terminal :

```bash
git config --global user.name "Prenom NOM"
```

```bash
git config --global user.email "email-utilise-pour-github@example.com"
```

Vérifier :

```bash
git config --global user.name
git config --global user.email
```

---

# 0.7 Vérification finale avant le TD

```bash
git --version
code --version
docker version
docker compose version
```

Puis :

```bash
docker run --rm hello-world
```

Checklist :

```text
[ ] Git fonctionne
[ ] VS Code fonctionne
[ ] Docker fonctionne
[ ] Docker Compose fonctionne
[ ] hello-world fonctionne
[ ] Compte GitHub créé
[ ] Email GitHub vérifié
```

Si tout est OK :

```text
START WORKSPACE
```

---

## 1. Créer le workspace

```bash
mkdir docker-td1
cd docker-td1

mkdir -p rendu evidence

touch README.md
touch .gitignore
touch rendu/Dockerfile
touch rendu/index.html
touch rendu/.dockerignore
touch rendu/run.sh

code .
```

Structure :

```text
docker-td1/
├── README.md
├── .gitignore
├── rendu/
│   ├── Dockerfile
│   ├── index.html
│   ├── .dockerignore
│   └── run.sh
└── evidence/
```

---

## 3. `.gitignore`

```gitignore
.env
*.key
*.pem
.vscode/
```

---

## 4. `rendu/index.html`

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Docker TD1</title>
</head>
<body>
  <h1>Docker TD1 OK</h1>
  <p>Container running.</p>
</body>
</html>
```

---

## 5. `rendu/Dockerfile`

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

---

## 6. `rendu/.dockerignore`

```text
.git
.gitignore
evidence
README.md
```

---

## 7. `rendu/run.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

docker run --detach \
  --name td1-web \
  --label course.lab=TD01 \
  --publish 127.0.0.1:8080:80 \
  td1-web
```

Puis :

```bash
chmod +x rendu/run.sh
```

---

# 8. Build Docker

Depuis la racine `docker-td1/` :

```bash
docker build -t td1-web ./rendu
```

Vérifier :

```bash
docker images td1-web
```

---

# 9. Run Docker

```bash
bash rendu/run.sh
```

Vérifier :

```bash
docker ps --filter name=td1-web
```

Tester :

```bash
curl http://127.0.0.1:8080/
```

Navigateur :

```text
http://127.0.0.1:8080/
```

Attendu :

```text
Docker TD1 OK
```

---

# 10. Créer les premières evidence

```bash
docker version --format \
'client={{.Client.Version}} server={{.Server.Version}}' \
> evidence/docker-version.txt
```

```bash
docker ps --filter name=td1-web \
> evidence/container-running.txt
```

```bash
docker inspect --format \
'id={{.Id}}
image={{.Config.Image}}
status={{.State.Status}}
started={{.State.StartedAt}}' \
td1-web \
> evidence/container-state.txt
```

```bash
docker inspect --format '{{.Id}}' td1-web \
> evidence/container-id-before.txt
```

```bash
docker port td1-web \
> evidence/ports.txt
```

```bash
docker logs --timestamps td1-web \
> evidence/logs.txt 2>&1
```

```bash
curl --fail --silent --show-error --include \
http://127.0.0.1:8080/ \
> evidence/http.txt
```

Vérifier :

```bash
ls -lh evidence/
```

---

# 11. Cycle de vie du container

Stop :

```bash
docker stop td1-web
```

Observer :

```bash
docker ps -a --filter name=td1-web \
> evidence/container-stopped.txt
```

Supprimer :

```bash
docker rm td1-web
```

Vérifier :

```bash
docker ps -a --filter name=td1-web
```

Recréer :

```bash
bash rendu/run.sh
```

Nouvel ID :

```bash
docker inspect --format '{{.Id}}' td1-web \
> evidence/container-id-after.txt
```

Retester :

```bash
curl --fail --silent --show-error --include \
http://127.0.0.1:8080/ \
> evidence/http-after-recreate.txt
```

Comparer :

```bash
cat evidence/container-id-before.txt
cat evidence/container-id-after.txt
```

Les IDs doivent être différents.

---

# 12. Initialiser Git

Première utilisation uniquement :

```bash
git config --global user.name "Prenom NOM"
git config --global user.email "email-utilise-pour-github@example.com"
```

Initialiser :

```bash
git init
git branch -M main
```

Vérifier :

```bash
git status
```

Premier commit :

```bash
git add .
git commit -m "Initial Docker TD1 workspace"
```

---

# 13. Créer le repository GitHub PRIVATE depuis VS Code

Dans VS Code :

```text
Ctrl + Shift + P
```

Chercher :

```text
Publish to GitHub
```

Puis :

```text
Sign in to GitHub
```

Le navigateur s'ouvre.

```text
1. Se connecter au compte GitHub
2. Autoriser VS Code
3. Revenir dans VS Code
```

Choisir :

```text
docker-td1
```

Puis :

```text
Publish to GitHub private repository
```

Ne pas choisir `public`.

---

# 14. Vérifier le remote

```bash
git remote -v
```

Puis :

```bash
git status
```

Attendu :

```text
nothing to commit, working tree clean
```

---

# 15. Ajouter de nouvelles evidence

Après une modification :

```bash
git status
```

```bash
git add .
```

```bash
git commit -m "Add Docker TD1 evidence"
```

```bash
git push
```

---

# 16. Vérification finale

```text
GitHub repository : PRIVATE
```

```text
docker-td1/
├── README.md
├── .gitignore
├── rendu/
│   ├── Dockerfile
│   ├── index.html
│   ├── .dockerignore
│   └── run.sh
└── evidence/
    ├── docker-version.txt
    ├── container-running.txt
    ├── container-state.txt
    ├── container-id-before.txt
    ├── container-id-after.txt
    ├── container-stopped.txt
    ├── ports.txt
    ├── logs.txt
    ├── http.txt
    └── http-after-recreate.txt
```

Checklist :

```text
[ ] Docker build fonctionne
[ ] Container td1-web fonctionne
[ ] http://127.0.0.1:8080 fonctionne
[ ] evidence/ contient les preuves
[ ] container supprimé puis recréé
[ ] IDs avant/après différents
[ ] Git repository initialisé
[ ] Premier commit créé
[ ] Repository GitHub créé
[ ] Repository GitHub PRIVATE
[ ] Push réussi
[ ] Aucun mot de passe / token / clé privée dans Git
```

---

# Commandes Git à retenir

```bash
git status
git add .
git commit -m "message"
git push
```

Cycle simple :

```text
modifier
   ↓
git status
   ↓
git add .
   ↓
git commit
   ↓
git push
```

---

# Nettoyage Docker

Uniquement le container du TD :

```bash
docker stop td1-web
docker rm td1-web
```

Ne pas utiliser :

```bash
docker system prune -a
```

Ne pas publier avec :

```bash
-p 8080:80
```

Utiliser :

```bash
--publish 127.0.0.1:8080:80
```


---


---

# 17. Avant chaque nouveau commit ou push

Toujours commencer par :

```bash
git status
git branch --show-current
git log --oneline --decorate -5
```

Vérifier :

```text
[ ] je suis dans le bon dossier
[ ] je suis sur la bonne branche
[ ] je comprends les fichiers modifiés
```

Exemple attendu :

```text
main
```

Puis :

```bash
git add .
git status
git commit -m "Describe the change"
git push
```

Workflow recommandé :

```text
git status
    ↓
git branch --show-current
    ↓
modifier
    ↓
git add .
    ↓
git status
    ↓
git commit
    ↓
git push
```

---

# 18. Revenir rapidement à la branche précédente

```bash
git switch -
```

Voir toutes les branches :

```bash
git branch
```

Voir les branches locales + distantes :

```bash
git branch -a
```

---

# 19. Je suis sur la mauvaise branche - aucun fichier modifié

Vérifier :

```bash
git status
```

Si le working tree est propre :

```bash
git switch main
```

Mettre à jour :

```bash
git pull --ff-only
```

Puis continuer.

---

# 20. Je suis sur la mauvaise branche avec des modifications NON commitées

Ne pas utiliser :

```bash
git reset --hard
```

Sauvegarder temporairement :

```bash
git stash push -u -m "move work to correct branch"
```

Changer de branche :

```bash
git switch main
git pull --ff-only
```

Récupérer les modifications :

```bash
git stash pop
```

Puis vérifier :

```bash
git status
```

---

# 21. J'ai déjà fait le commit sur la mauvaise branche

Ne pas supprimer le commit.

Récupérer son SHA :

```bash
git log -1 --oneline
```

Exemple :

```text
a1b2c3d Add Docker evidence
```

Changer vers la bonne branche :

```bash
git switch main
git pull --ff-only
```

Copier le commit :

```bash
git cherry-pick a1b2c3d
```

Puis :

```bash
git push
```

Vérifier :

```bash
git log --oneline --decorate -5
```

---

# 22. HEAD détachée - `git branch --show-current` ne retourne rien

Diagnostic :

```bash
git status
git branch --show-current
```

Exemple :

```text
HEAD detached at a1b2c3d
```

Ne pas continuer à travailler longtemps dans cet état.

Créer immédiatement une branche de récupération :

```bash
git switch -c recovery-my-work
```

Vérifier :

```bash
git branch --show-current
```

Puis :

```bash
git status
```

Si le travail doit finalement aller sur `main` :

```bash
git add .
git commit -m "Recover work"
```

Récupérer le SHA :

```bash
git log -1 --oneline
```

Puis :

```bash
git switch main
git pull --ff-only
git cherry-pick <SHA>
git push
```

---

# 23. J'ai perdu un commit / je ne retrouve plus mon HEAD

Git garde généralement une trace locale.

Afficher l'historique des mouvements de HEAD :

```bash
git reflog -10
```

Exemple :

```text
a1b2c3d HEAD@{0}: checkout: moving from ...
9f8e7d6 HEAD@{1}: commit: Add Docker evidence
```

Créer une branche de récupération depuis le bon SHA :

```bash
git switch -c recovery-lost-work 9f8e7d6
```

Vérifier :

```bash
git log --oneline --decorate -5
```

Ne pas utiliser `reset --hard` pour récupérer un commit étudiant.

---

# 24. `git push` est refusé

Exemple :

```text
rejected
fetch first
```

D'abord :

```bash
git status
git branch --show-current
```

Puis :

```bash
git pull --ff-only
```

Si cela fonctionne :

```bash
git push
```

Si `git pull --ff-only` refuse parce que les historiques divergent :

```text
STOP
```

Ne pas utiliser automatiquement :

```bash
git push --force
git reset --hard
```

Comparer :

```bash
git log --oneline --decorate --graph --all -10
```

Puis demander une vérification avant de modifier l'historique.

---

# 25. Mini diagnostic Git à connaître

```bash
pwd
git status
git branch --show-current
git remote -v
git log --oneline --decorate -5
```

Ces cinq commandes répondent à :

```text
Où suis-je ?
Qu'est-ce qui a changé ?
Sur quelle branche suis-je ?
Quel GitHub est connecté ?
Quel est mon HEAD actuel ?
```

---

# 26. Règle simple pour les étudiants

Avant chaque commit :

```bash
git status
git branch --show-current
```

Avant chaque push :

```bash
git status
git log -1 --oneline
git push
```

En cas de doute :

```text
STOP
NE PAS utiliser --force
NE PAS utiliser reset --hard
NE PAS supprimer une branche contenant du travail non vérifié
```


# Références officielles

```text
Docker Ubuntu:
https://docs.docker.com/engine/install/ubuntu/

Docker Windows:
https://docs.docker.com/desktop/setup/install/windows-install/

Docker macOS:
https://docs.docker.com/desktop/setup/install/mac-install/

VS Code Linux:
https://code.visualstudio.com/docs/setup/linux

VS Code + GitHub:
https://code.visualstudio.com/docs/sourcecontrol/repos-remotes

GitHub account:
https://docs.github.com/en/account-and-profile/how-tos/account-management/creating-an-account-on-github
```
