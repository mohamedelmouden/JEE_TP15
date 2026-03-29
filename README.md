#  TP15 – Déploiement Spring Boot avec Docker & Docker Compose

> **Étudiant :** Mohamed EL MOUDEN  
> **Groupe :** fst.tp15elmouden  
> **Date :** 29 Mars 2026  
> **Technologies :** Spring Boot 3.5 · Java 21 · Docker · MySQL 8 · Docker Compose · phpMyAdmin

---

## 📋 Objectifs

- ✅ Construire une image Docker à partir d'un projet Spring Boot
- ✅ Exécuter et gérer un conteneur d'application
- ✅ Configurer les variables d'environnement pour le conteneur
- ✅ Déployer une base de données MySQL dans un second conteneur
- ✅ Établir la communication via Docker Compose
- ✅ Ajouter phpMyAdmin pour la gestion visuelle de la base de données

---

## 🏗️ Architecture du projet

```
JEE_TP15/
└── demo/
    ├── src/
    │   └── main/
    │       ├── java/fst/tp15elmouden/demo/
    │       │   └── DemoApplication.java
    │       └── resources/
    │           └── application.properties
    ├── target/
    │   └── demo-0.0.1-SNAPSHOT.jar
    ├── Dockerfile
    ├── docker-compose.yml
    └── pom.xml
```

---

## ⚙️ Configuration du projet

### `application.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/jeeTp15?createDatabaseIfNotExist=true
spring.datasource.username=root
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=update
server.port=8087
```

### Dépendances Spring Boot

| Dépendance | Rôle |
|---|---|
| Spring Web | API REST avec Tomcat embarqué |
| Spring Data JPA | Persistance des données |
| MySQL Driver | Connexion à la base MySQL |
| Lombok | Réduction du code boilerplate |

---

##  Étape 1 – Génération du JAR

Avant de construire l'image Docker, générez le fichier JAR :

```powershell
# Configurer le JDK 21 (si nécessaire sous PowerShell)
$env:JAVA_HOME = "C:\Program Files\Java\jdk-21"
$env:PATH = "$env:JAVA_HOME\bin;$env:PATH"

# Générer le JAR
mvn clean package -DskipTests
```

**Résultat attendu :** `BUILD SUCCESS` ✅  
**JAR généré :** `target/demo-0.0.1-SNAPSHOT.jar`

---

##  Étape 2 – Dockerfile

```dockerfile
# Image de base Java 21
FROM eclipse-temurin:21-jdk-jammy

# Répertoire de travail dans le conteneur
WORKDIR /app

# Copier le JAR généré
COPY target/demo-0.0.1-SNAPSHOT.jar app.jar

# Exposer le port de l'application
EXPOSE 8087

# Lancer l'application Spring Boot
ENTRYPOINT ["java", "-jar", "app.jar"]
```

| Instruction | Description |
|---|---|
| `FROM` | Image de base Java 21 (Ubuntu Jammy) |
| `WORKDIR` | Répertoire de travail dans le conteneur |
| `COPY` | Copie le JAR dans le conteneur |
| `EXPOSE` | Informe Docker du port utilisé |
| `ENTRYPOINT` | Commande de démarrage du conteneur |

---

##  Étape 3 – Construction et exécution de l'image

### Construction de l'image

```powershell
docker build -t fst/tp15elmouden:1.0 .
```

**Résultat :** `Building 59.5s (9/9) FINISHED` ✅

### Exécution du conteneur

```powershell
docker run -d -p 8087:8087 --name tp15-elmouden-app fst/tp15elmouden:1.0
```

### Vérification des logs

```powershell
docker logs -f tp15-elmouden-app
```

### Arrêt et suppression

```powershell
docker stop tp15-elmouden-app
docker rm tp15-elmouden-app
```

> ⚠️ **Remarque importante :** À cette étape, les logs affichent une erreur de connexion MySQL (`Communications link failure`). C'est **normal et attendu** car le conteneur Spring Boot cherche MySQL sur `localhost:3306` à l'intérieur du conteneur, où il n'existe pas encore. La solution est apportée à l'étape suivante via **Docker Compose**, qui crée un réseau partagé entre les conteneurs Spring Boot et MySQL.

---

##  Étape 4 – Docker Compose (Multi-conteneurs)

### `docker-compose.yml`

```yaml
services:

  mysql-tp15elmouden:
    image: mysql:8.0
    container_name: mysql-tp15elmouden
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root1234
      MYSQL_DATABASE: jeeTp15
    ports:
      - "3307:3306"
    volumes:
      - mysql-data-tp15elmouden:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-proot1234"]
      interval: 10s
      timeout: 5s
      retries: 5

  springboot-tp15elmouden:
    image: fst/tp15elmouden:1.0
    container_name: springboot-tp15elmouden
    depends_on:
      mysql-tp15elmouden:
        condition: service_healthy
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql-tp15elmouden:3306/jeeTp15?createDatabaseIfNotExist=true
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=root1234
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update
      - SERVER_PORT=8087
    ports:
      - "8087:8087"

  phpmyadmin-tp15elmouden:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin-tp15elmouden
    depends_on:
      mysql-tp15elmouden:
        condition: service_healthy
    environment:
      PMA_HOST: mysql-tp15elmouden
      PMA_PORT: 3306
      MYSQL_ROOT_PASSWORD: root1234
    ports:
      - "8088:80"

volumes:
  mysql-data-tp15elmouden:
```

| Clé | Description |
|---|---|
| `services` | Définit les 3 conteneurs : MySQL, Spring Boot, phpMyAdmin |
| `depends_on` + `healthcheck` | Garantit que MySQL est **prêt** avant Spring Boot |
| `environment` | Injecte les variables de configuration dynamiquement |
| `volumes` | Persiste les données MySQL même après suppression du conteneur |

---

##  Étape 5 – Exécution avec Docker Compose

### Démarrer tous les conteneurs

```powershell
docker compose up -d
```

**Résultat attendu :**
```
✔ Network demo_default              Created
✔ Container mysql-tp15elmouden      Healthy
✔ Container springboot-tp15elmouden Started
✔ Container phpmyadmin-tp15elmouden Started
```

### Vérifier les conteneurs actifs

```powershell
docker ps
```

```
CONTAINER ID   IMAGE                  PORTS                     NAMES
01558f162e54   fst/tp15elmouden:1.0   0.0.0.0:8087->8087/tcp   springboot-tp15elmouden
7025757dd420   mysql:8.0              0.0.0.0:3307->3306/tcp   mysql-tp15elmouden (healthy)
xxxxxxxxxxxx   phpmyadmin/phpmyadmin  0.0.0.0:8088->80/tcp     phpmyadmin-tp15elmouden
```

### Afficher les logs

```powershell
# Tous les conteneurs
docker compose logs -f

# Spring Boot uniquement
docker compose logs -f springboot-tp15elmouden

# MySQL uniquement
docker compose logs -f mysql-tp15elmouden
```

### Arrêter l'environnement

```powershell
docker compose down
```

---

## 🌐 Accès aux interfaces

| Interface | URL | Identifiants |
|---|---|---|
| 🍃 Spring Boot | http://localhost:8087 | - |
| 🛢️ phpMyAdmin | http://localhost:8088 | root / root1234 |

> ℹ️ La page **Whitelabel Error Page** sur `localhost:8087` est **normale** : elle confirme que Spring Boot tourne correctement et est connecté à MySQL. Il n'y a simplement aucun endpoint REST défini dans cette version du projet.

---

## 🗄️ Base de données

La base de données `jeeTp15` est créée **automatiquement** par Spring Boot au démarrage grâce à `createDatabaseIfNotExist=true`.

Elle est visible dans phpMyAdmin à l'adresse `localhost:8088` :

| Base de données | Créée par |
|---|---|
| `jeeTp15` | ✅ Spring Boot (automatique) |
| `information_schema` | MySQL système |
| `mysql` | MySQL système |
| `performance_schema` | MySQL système |
| `sys` | MySQL système |

---

## 🔧 Résolution des problèmes rencontrés

| Problème | Cause | Solution |
|---|---|---|
| `No POM in this directory` | Mauvais dossier | `cd demo` puis relancer |
| `No compiler provided` | JRE au lieu de JDK | Configurer `JAVA_HOME` vers JDK 21 |
| Port 3306 déjà utilisé | MySQL local installé | Changer le port hôte en `3307:3306` |
| Spring Boot crash (sans Compose) | MySQL absent dans le conteneur | Utiliser Docker Compose avec healthcheck |
| Spring Boot démarre avant MySQL | `depends_on` insuffisant | Ajouter `healthcheck` + `condition: service_healthy` |

---

## 🏆 Synthèse des compétences acquises

| Compétence | Description |
|---|---|
| **Dockerfile** | Construction d'images Docker à partir d'un projet Spring Boot |
| **Commandes Docker** | Gestion des conteneurs, images et logs |
| **Variables d'environnement** | Configuration dynamique des services |
| **Docker Compose** | Déploiement multi-conteneurs orchestré |
| **Persistance des données** | Utilisation de volumes pour MySQL |
| **Healthcheck** | Attendre qu'un service soit prêt avant d'en démarrer un autre |
| **phpMyAdmin** | Interface graphique pour gérer la base de données MySQL |

---

## 📦 Stack technique

![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5.14-brightgreen?logo=springboot)
![Java](https://img.shields.io/badge/Java-21-orange?logo=java)
![Docker](https://img.shields.io/badge/Docker-Desktop-blue?logo=docker)
![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?logo=mysql)
![phpMyAdmin](https://img.shields.io/badge/phpMyAdmin-5.2.3-orange?logo=phpmyadmin)
![Maven](https://img.shields.io/badge/Maven-3.x-red?logo=apachemaven)

---


### localhost:8087 :
<img width="960" height="540" alt="Screenshot 2026-03-29 055816" src="https://github.com/user-attachments/assets/64b40cd1-c7b5-4b0c-b6ee-cb02732dcb45" />

### phpmyadmin / localhost:8088
<img width="960" height="540" alt="Screenshot 2026-03-29 060701" src="https://github.com/user-attachments/assets/c8277a8e-74e2-49fa-ac98-f19e13b1beef" />
<img width="960" height="540" alt="Screenshot 2026-03-29 060757" src="https://github.com/user-attachments/assets/8d5e662b-2b85-48c4-9aef-1ae631ca38f9" />

*TP15 – JEE | FST | Mohamed EL MOUDEN | 2026*
