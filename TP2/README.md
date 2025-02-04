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