# Étape 1 : Image de base Java 21 (correspond à votre config Spring Initializr)
FROM eclipse-temurin:21-jdk-jammy

# Étape 2 : Répertoire de travail dans le conteneur
WORKDIR /app

# Étape 3 : Copier le JAR généré (nom adapté à votre artifact "demo")
COPY target/demo-0.0.1-SNAPSHOT.jar app.jar

# Étape 4 : Exposer le port de l'application (votre port configuré)
EXPOSE 8087

# Étape 5 : Lancer l'application Spring Boot
ENTRYPOINT ["java", "-jar", "app.jar"]