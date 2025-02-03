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

# Exécute Maven pour compiler et empaqueter l'application. L'option -DskipTests permet de sauter les tests.
RUN mvn package -DskipTests


# Run Stage
# Utilise l'image Amazon Corretto JDK 21 pour exécuter l'application
FROM amazoncorretto:21

# Redéfinit la variable d'environnement MYAPP_HOME dans le conteneur d'exécution
ENV MYAPP_HOME=/opt/myapp 

# Définit à nouveau le répertoire de travail pour l'exécution de l'application
WORKDIR $MYAPP_HOME

# Copie le fichier JAR généré dans le répertoire cible de la phase de construction dans le conteneur d'exécution
# Le JAR est copié depuis le répertoire /opt/myapp/target/ du conteneur myapp-build
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

# Définit la commande par défaut à exécuter quand le conteneur démarre : exécuter l'application Java
ENTRYPOINT ["java", "-jar", "myapp.jar"]
