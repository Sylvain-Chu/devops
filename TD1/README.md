
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
