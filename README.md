
## 1-1 Why should we run the container with a flag -e to give the environment variables?

- **Sécurisation des informations sensibles** : en utilisant le flag `-e` pour définir les variables d'environnement au moment du lancement du conteneur, on évite de stocker les informations sensibles (comme le mot de passe de la base de données) en texte brut dans le `Dockerfile`. Cela permet d'utiliser des fichiers `.env` pour mieux gérer les configurations sensibles.

- **Réutilisabilité de l’image** : cela rend l’image Docker plus générique et réutilisable, car les paramètres d’environnement peuvent être définis de manière dynamique au lancement du conteneur sans avoir besoin de modifier l'image elle-même.

---

## 1-2 Why do we need a volume to be attached to our postgres container?

Un conteneur Docker est **temporaire**. Si le conteneur est supprimé ou recréé, les données qu'il contient seront perdues.  
En attachant un **volume** à notre conteneur, les données restent stockées, même après la suppression ou le redémarrage du conteneur. Cela garantit que la base de données conserve ses données, même si le conteneur PostgreSQL est arrêté, supprimé, ou redémarré.

---

## 1-3 Document your database container essentials: commands and Dockerfile.

### Commandes essentielles

```bash
# Création du réseau
docker network create app-network

# Construction de l’image PostgreSQL
docker build -t my-postgres-db .

# Lancement du conteneur PostgreSQL
docker run -d --name postgres-container --net=app-network \
    -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd \
    -v postgres-data:/var/lib/postgresql/data my-postgres-db

# Connexion à PostgreSQL
docker exec -it postgres-container psql -U usr -d db

# Vérification des tables et données
docker exec -it postgres-container psql -U usr -d db -c "SELECT * FROM students;"

# Lancement d’Adminer
docker run -p 8090:8080 --net=app-network --name=adminer -d adminer
```

### Dockerfile

```Dockerfile
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY init-scripts/ /docker-entrypoint-initdb.d/
```

---

## 1-4 Why do we need a multistage build? And explain each step of this dockerfile.

- **Pourquoi le multistage ?**
Un multistage build permet de séparer les étapes de construction et d'exécution de l'image Docker. 
Cela permet de réduire la taille de l'image en ne gardant que l'exécutable nécessaire (JRE), tout en construisant le code dans une image plus lourde contenant le JDK. Cela permet d'amélirer lla gestion des dépendances et d'optimisé l'image.

- **Les étapes du dockerfile**

```bash
# Build Stage
# Utilise l'image Maven avec Amazon Corretto JDK 21 pour construire l'application
FROM maven:3.9.9-amazoncorretto-21 AS myapp-build

# Définit une variable d'environnement pour le répertoire d'installation de l'application
ENV MYAPP_HOME=/opt/myapp 

# Définit le répertoire de travail dans le conteneur où l'application sera construite
WORKDIR $MYAPP_HOME

# Copie le fichier pom.xml dans le conteneur
COPY pom.xml .

# Copie le répertoire src dans le conteneur
COPY src ./src

# Exécute Maven pour compiler l'application. -DskipTests permet de sauter les tests.
RUN mvn package -DskipTests


# Run Stage
# Utilise l'image Amazon Corretto JDK 21 pour exécuter l'application
FROM amazoncorretto:21

# Redéfinit la variable d'environnement MYAPP_HOME dans le conteneur d'exécution
ENV MYAPP_HOME=/opt/myapp 

# Définit à nouveau le répertoire de travail pour l'exécution de l'application
WORKDIR $MYAPP_HOME

# Copie le fichier JAR généré dans le répertoire de la phase de construction dans le conteneur d'exécution
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

# Définit la commande par défaut à exécuter quand le conteneur démarre : exécuter l'application Java
ENTRYPOINT ["java", "-jar", "myapp.jar"]

```

## 1-5 Why do we need a reverse proxy?

- **sécurité** : SSL
- **haute disponibilité** : répartition de la charge
- **authentification/autorisation centralisée** : un serveur pour toutes les applications

---

## 1-6 Why is docker-compose so important?

**Docker Compose** est un outil qui facilite la gestion de plusieurs conteneurs Docker. Voici pourquoi il est important :

- **Orchestration facile** : Il permet de gérer facilement plusieurs conteneurs, de leur démarrage à leur arrêt, en utilisant un seul fichier de configuration (`docker-compose.yml`). Cela simplifie grandement le déploiement de projets multi-conteneurs.
  
- **Gestion de configurations** : Docker Compose permet de définir les configurations de chaque service (base de données, API, frontend, etc.) dans un même fichier, avec des variables d'environnement et des volumes. Cela rend les déploiements cohérents, reproductibles et faciles à configurer.
  
- **Automatisation** : En combinant plusieurs services dans un fichier `docker-compose.yml`, on peut automatiser des tâches comme la construction des images, le démarrage des conteneurs, le nettoyage des volumes, etc., avec une seule commande.
  
- **Simplicité** : Grâce à Compose, il est plus facile de travailler en équipe sur des projets complexes qui impliquent plusieurs conteneurs. Chaque membre peut travailler avec la même configuration sans avoir à gérer des commandes Docker complexes.

---

## 1-7 Document docker-compose most important commands.

### Commandes Docker Compose les plus importantes :

- **`docker-compose up`** : Cette commande démarre tous les services définis dans le fichier `docker-compose.yml`. Elle crée et démarre les conteneurs en fonction de la configuration du fichier. L'option `-d` permet de démarrer les conteneurs en arrière-plan (mode détaché).
  
  ```bash
  docker-compose up -d
  ```

- **`docker-compose down`** : Cette commande arrête et supprime tous les conteneurs, réseaux et volumes associés au projet. Cela permet de nettoyer l'environnement de travail.
  
  ```bash
  docker-compose down
  ```

- **`docker-compose build`** : Permet de reconstruire les images des services définis dans le fichier `docker-compose.yml`. Cela est utile lorsque les Dockerfiles ou les fichiers associés ont été modifiés.
  
  ```bash
  docker-compose build
  ```

- **`docker-compose logs`** : Affiche les logs de tous les conteneurs ou d'un conteneur spécifique dans le projet Docker Compose.
  
  ```bash
  docker-compose logs
  ```

- **`docker-compose exec <service> <command>`** : Exécute une commande dans un conteneur en cours d'exécution d'un service spécifié.
  
  ```bash
  docker-compose exec backend bash
  ```

- **`docker-compose ps`** : Affiche l'état des conteneurs gérés par Docker Compose, y compris leurs ID et leurs ports associés.
  
  ```bash
  docker-compose ps
  ```

- **`docker-compose restart`** : Redémarre les services définis dans le fichier `docker-compose.yml`.
  
  ```bash
  docker-compose restart
  ```

---

## 1-8 Document your docker-compose file.

### Description de ton fichier `docker-compose.yml` :

Le fichier `docker-compose.yml` ci-dessous définit et configure plusieurs services pour ton application :

```yaml
# Définition des services
services:
  
  # Service backend : responsable de l'application backend
  backtp1:
    # Spécifie le répertoire contenant le Dockerfile pour construire l'image du backend
    build: "backend"
    # Nom du conteneur pour ce service
    container_name: backtp1
    # Variables d'environnement pour configurer l'application backend
    environment:
      - URL=${URL} # URL de connexion à la base de données
      - POSTGRES_USER=${POSTGRES_USER} # Utilisateur de la base de données
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD} # Mot de passe de la base de données
    # Réseau sur lequel le service va être connecté
    networks:
      - my-network
    # Dépendance du backend au service tp1db (la base de données)
    depends_on:
      - tp1db

  # Service base de données PostgreSQL : responsable de la gestion des données
  tp1db:
    # Spécifie le répertoire contenant le Dockerfile pour construire l'image de la base de données
    build: "db"
    # Nom du conteneur pour ce service
    container_name: tp1db
    # Variables d'environnement pour configurer PostgreSQL
    environment:
      - POSTGRES_DB=${POSTGRES_DB} # Nom de la base de données à créer
      - POSTGRES_USER=${POSTGRES_USER} # Utilisateur de la base de données
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD} # Mot de passe de l'utilisateur
    # Réseau sur lequel le service va être connecté
    networks:
      - my-network
    # Volume pour persister les données de la base de données
    volumes:
      - ./db/data:/var/lib/postgresql/data # Persistance des données dans ./db/data

  # Service frontend : responsable de l'interface utilisateur de l'application
  httptp1:
    # Spécifie le répertoire contenant le Dockerfile pour construire l'image du frontend
    build: "http"
    # Nom du conteneur pour ce service
    container_name: httptp1
    # Mapping des ports : le conteneur expose le port 80, accessible sur l'hôte via le port 80
    ports:
      - "80:80"
    # Réseau sur lequel le service va être connecté
    networks:
      - my-network
    # Dépendance du frontend au service backtp1 (le backend)
    depends_on:
      - backtp1

# Définition des réseaux
networks:
  # Définition du réseau my-network utilisé par les services
  my-network:
    # Utilisation du driver bridge pour permettre la communication entre les services
    driver: bridge
```

### Explication des services :
1. **Backend** : 
   - Ce service construit une image à partir du Dockerfile dans le répertoire `./backend`.
   - Il expose le port `8080` pour que l'API soit accessible.
   - Il se connecte à la base de données `database` via l'URL fournie dans les variables d'environnement.

2. **Database** : 
   - Ce service construit une image à partir du Dockerfile dans le répertoire `./database` et configure PostgreSQL avec les variables d'environnement fournies.
   - Il initialise la base de données à partir des scripts SQL présents dans le dossier `./database/init-scripts`.

3. **Frontend** : 
   - Ce service construit une image à partir du Dockerfile dans le répertoire `./frontend`.
   - Il expose le port `80` et le mappe au port `8081` de l'hôte pour rendre l'application frontend accessible.

### Réseau :
- Tous les services sont connectés au même réseau Docker (`app-network`), ce qui permet aux services de communiquer entre eux à l'aide de leurs noms de service (`backend`, `database`, etc.).


## 1-9 Documenter les commandes de publication et les images publiées sur Docker Hub

#### Commandes pour publier l'image Docker

1. **Se connecter à Docker Hub :**

   Pour se connecter à Docker Hub et pouvoir publier des images, il faut d'abord exécuter la commande suivante dans le terminal :

   ```bash
   docker login
   ```

   Cette commande te demandera ton nom d'utilisateur et ton mot de passe Docker Hub pour t'authentifier.

2. **Taguer ton image avec des informations de version :**

   Une fois que ton image est construite, tu dois la taguer avec un nom et une version avant de la publier sur Docker Hub. Par exemple, si ton image s'appelle `my-database` et que tu veux la taguer avec la version `1.0`, tu exécutes cette commande :

   ```bash
   docker tag my-database USERNAME/my-database:1.0
   ```

   Remplace `USERNAME` par ton nom d'utilisateur Docker Hub (par exemple `sylvainchu`) et assure-toi que l'image que tu veux taguer existe localement (dans ce cas `my-database`).

3. **Pousser l'image sur Docker Hub :**

   Ensuite, pour pousser ton image vers Docker Hub, tu utilises la commande suivante :

   ```bash
   docker push USERNAME/my-database:1.0
   ```

   Cette commande va envoyer l'image taguée vers Docker Hub, sous le dépôt `USERNAME/my-database` avec le tag `1.0`.

#### Image publiée sur Docker Hub

Après avoir poussé l'image, elle sera disponible dans ton compte Docker Hub sous l'URL suivante (en remplaçant `USERNAME` par ton nom d'utilisateur et `my-database` par le nom de l'image) :

```
https://hub.docker.com/r/USERNAME/my-database/tags
```

---

## 1-10 Pourquoi met-on nos images dans un dépôt en ligne ?

Mettre nos images Docker dans un dépôt en ligne comme Docker Hub ou un dépôt privé présente plusieurs avantages :

1. **Partage facile avec les équipes :**
   - Les images publiées sont accessibles depuis n'importe quel endroit, ce qui permet à d'autres membres de l'équipe ou à des utilisateurs externes de récupérer rapidement l'image et de l'utiliser sans avoir à la construire eux-mêmes.

2. **Centralisation des images :**
   - Un dépôt en ligne centralise toutes les images Docker, ce qui permet de garder une trace des différentes versions et de gérer les mises à jour de manière cohérente.

3. **Accès à des images publiques :**
   - Docker Hub est un dépôt public, ce qui signifie que des milliers d'images publiques sont disponibles, permettant de gagner du temps et d'éviter de reconstruire des images courantes.

4. **Sauvegarde et gestion des versions :**
   - Lorsque les images sont stockées sur Docker Hub, elles sont protégées par une sauvegarde en ligne, ce qui évite la perte des images locales. De plus, tu peux gérer les versions de tes images et publier des mises à jour sans perdre les anciennes versions.

5. **Facilite l'intégration continue :**
   - Dans des systèmes d'intégration continue (CI/CD), il est essentiel de pouvoir récupérer des images de manière automatisée depuis un dépôt en ligne pour les utiliser dans des pipelines de tests ou de déploiement.

6. **Accessibilité à distance :**
   - Les images sont accessibles depuis n'importe quel ordinateur ou serveur qui a Docker installé, ce qui permet de les utiliser sur différentes machines ou dans des environnements de production à distance.

En résumé, publier tes images Docker sur un dépôt en ligne comme Docker Hub permet une meilleure collaboration, une gestion simplifiée des versions, et une intégration plus facile avec les autres services ou équipes.

## **2-1 Qu'est-ce que Testcontainers ?**  

**Testcontainers** est une bibliothèque Java qui permet de créer des instances temporaires et jetables de bases de données, de courtiers de messages ou d'autres services via des conteneurs Docker, afin de faciliter les tests d'intégration.  

## **2-2 Document your Github Actions configurations**

```yml
name: CI DevOps 2025  # Nom du workflow

on:
  push:
    branches:
      - main  # Le workflow s'exécute à chaque push sur la branche "main"
  pull_request:  # Il s'exécute aussi à chaque pull request

jobs:
  test-backend:
    runs-on: ubuntu-22.04  # Utilisation d'une machine virtuelle Ubuntu 22.04 pour l'exécution

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4  # Récupération du code source depuis GitHub

      - name: Set up JDK 21
        uses: actions/setup-java@v3  # Installation de Java 21
        with:
          distribution: 'corretto'  # Utilisation de la distribution AWS Corretto
          java-version: '21'  # Version Java 21

      - name: Build and test with Maven
        run: |
          cd TD1/backend  # Aller dans le dossier backend
          mvn clean install  # Compiler et exécuter les tests Maven
```

## **2-3 For what purpose do we need to push docker images?**

Pousser des images Docker permet de partager et déployer facilement son application sur différents serveurs ou machines. Cela permet aussi de faire en sorte que tout le monde utilise la même version de l'application.


## **2-4 Document your quality gate configuration**

```yml
name: CI/CD DevOps 2025

# Trigger the workflow on push to the 'main' branch or pull requests
on:
  push:
    branches:
      - main
  pull_request:

jobs:
  # Job to test the backend
  test-backend:
    runs-on: ubuntu-22.04  # Specifies the environment to run the job on (Ubuntu 22.04)

    steps:
      - name: Checkout repository  # Checkout the code from the repository
        uses: actions/checkout@v4

      - name: Set up JDK 21  # Set up Java Development Kit (JDK) version 21 with 'corretto' distribution
        uses: actions/setup-java@v3
        with:
          distribution: 'corretto'
          java-version: '21'

      - name: Build and test with Maven  # Build and test the backend code using Maven
        run: |
          cd TD1/backend  # Navigate to the backend directory
          mvn clean install  # Clean the project and install dependencies, run tests

  # Job to build and push Docker images
  build-and-push-docker-images:
    needs: test-backend  # This job will only run if 'test-backend' job succeeds
    runs-on: ubuntu-22.04  # Specifies the environment to run the job on (Ubuntu 22.04)

    steps:
      - name: Checkout repository  # Checkout the code from the repository
        uses: actions/checkout@v4

      - name: Login to Docker Hub  # Log in to Docker Hub using credentials stored in secrets
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}  # Docker Hub username from secrets
          password: ${{ secrets.DOCKERHUB_TOKEN }}  # Docker Hub token from secrets

      # Build and push the backend Docker image
      - name: Build and push backend image
        uses: docker/build-push-action@v3
        with:
          context: ./TD1/backend  # Directory containing the backend code
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-backend:latest  # Docker image tag
          push: ${{ github.ref == 'refs/heads/main' }}  # Push image only if commit is on the 'main' branch

      # Build and push the database Docker image
      - name: Build and push database image
        uses: docker/build-push-action@v3
        with:
          context: ./TD1/db  # Directory containing the database code
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-db:latest  # Docker image tag
          push: ${{ github.ref == 'refs/heads/main' }}  # Push image only if commit is on the 'main' branch

      # Build and push the HTTP Docker image
      - name: Build and push httpd image
        uses: docker/build-push-action@v3
        with:
          context: ./TD1/http  # Directory containing the HTTP service code
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-http:latest  # Docker image tag
          push: ${{ github.ref == 'refs/heads/main' }}  # Push image only if commit is on the 'main' branch

  # Job to build the project and perform static analysis with SonarQube
  build:
    name: Build and analyze  # Job name
    runs-on: ubuntu-22.04  # Specifies the environment to run the job on (Ubuntu 22.04)
    
    steps:
      - uses: actions/checkout@v4  # Checkout the code from the repository
        with:
          fetch-depth: 0  # Ensure a full clone (no shallow clone) for better analysis

      - name: Set up JDK 21  # Set up JDK 21 with 'zulu' distribution for SonarQube analysis
        uses: actions/setup-java@v4
        with:
          java-version: 21
          distribution: 'corretto'  # Use 'corretto' JDK distribution

      - name: Cache SonarQube packages  # Cache SonarQube dependencies to speed up analysis
        uses: actions/cache@v4
        with:
          path: ~/.sonar/cache  # Path to cache SonarQube data
          key: ${{ runner.os }}-sonar  # Cache key to uniquely identify the SonarQube cache
          restore-keys: ${{ runner.os }}-sonar  # Restore the cache from previous builds if available

      - name: Cache Maven packages  # Cache Maven dependencies to avoid re-downloading them
        uses: actions/cache@v4
        with:
          path: ~/.m2  # Path to cache Maven dependencies
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}  # Use 'pom.xml' files to generate a unique cache key
          restore-keys: ${{ runner.os }}-m2  # Restore the cache from previous Maven builds

      - name: Build and analyze  # Run Maven build and SonarQube analysis
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}  # Use SonarQube token from GitHub secrets
        run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=Sylvain-Chu_devops -f TD1/backend/pom.xml  # Run SonarQube analysis on backend project
```


# 3-1 Document your inventory and base commands

```yaml
all:
  vars:
    # Définition de l'utilisateur SSH par défaut pour tous les hôtes
    ansible_user: admin

    # Spécifie la clé privée SSH utilisée pour se connecter aux hôtes
    ansible_ssh_private_key_file: id_rsa

  children:
    # Groupe d'hôtes pour l'environnement de production
    prod:
      hosts:
        # Nom de l'hôte de production 
        sylvain.churlet.takima.cloud:
          ansible_python_interpreter: /usr/bin/python3
```


# 3-2 Document your playbook

```yaml
# Install the necessary system packages for Docker
- name: Deploy application with Docker
  hosts: all
  become: true
# lance les rôles
  roles:
    # - install_docker
    - copy_env_file 
    - create_network
    - launch_database
    - launch_app
    - launch_proxy
    - launch_front
```

# 3-3 Document your docker_container tasks configuration.

```yaml
# Create and run the proxy container
- name: Run proxy
  docker_container:
    name: httptp1 # The name of the container
        image: sylvainchu/tp-devops-http:0.0.15 # Use the latest version of the proxy image
    pull: yes # Always pull the latest version of the image before running
    published_ports:
      - "80:80" # Map port 80 on the host to port 80 in the container
      - "8080:8081" # Map port 8080 on the host to port 8081 in the container
    networks:
      - name: proxy_app_network
```

```yaml
# Create and run the database container
- name: Run database
  docker_container:
    name: tp1db # The name of the database container
    image: sylvainchu/tp-devops-db:latest  # Use the latest database image
    pull: yes # Always pull the latest version of the image before running
    env_file: /.env # Load environment variables from the .env file
    networks:
      - name: app_database_network # Connect the container to the 'app_database_network' network
    volumes:
        - tp1db_data:/var/lib/postgresql/data   
```

