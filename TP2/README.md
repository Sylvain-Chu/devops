### **2-1 Qu'est-ce que Testcontainers ?**  

**Testcontainers** est une bibliothèque Java qui permet de créer des instances temporaires et jetables de bases de données, de courtiers de messages ou d'autres services via des conteneurs Docker, afin de faciliter les tests d'intégration.  

### **2-2 Document your Github Actions configurations**

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

### **2-2 Document your Github Actions configurations**

Pousser des images Docker permet de partager et déployer facilement son application sur différents serveurs ou machines. Cela permet aussi de faire en sorte que tout le monde utilise la même version de l'application.


### **2-4 Document your quality gate configuration**

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
          distribution: 'zulu'  # Use 'zulu' JDK distribution

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