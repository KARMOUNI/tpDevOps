# tpDevOps 3-Tier Web Application 
Ce dépôt montre la configuration d'une application web en architecture 3-tiers utilisant Docker, comprenant un serveur HTTP, une API backend et une base de données PostgreSQL. Chaque composant est conteneurisé et géré à l'aide de Docker et Docker Compose.

❓Question : Pourquoi devons-nous exécuter le conteneur avec le flag -e pour fournir des variables d'environnement ?

 Exécuter le conteneur avec le flag -e pour fournir des variables d'environnement est essentiel, car cela permet de transmettre de manière sécurisée et dynamique les données de configuration nécessaires, comme les informations de connexion à la base de données (par exemple, POSTGRES_DB, POSTGRES_USER et POSTGRES_PASSWORD), directement au conteneur au moment de l'exécution. Cela évite de coder en dur des informations sensibles dans le Dockerfile ou le code de l'application, ce qui améliore la sécurité et la flexibilité. En utilisant des variables d'environnement, nous pouvons facilement ajuster les configurations pour différents environnements (développement, test, production) sans modifier l'image du conteneur elle-même, ce qui rend la configuration plus portable et facile à gérer.
 
 ❓Question : Pourquoi avons-nous besoin d'attacher un volume à notre conteneur PostgreSQL ?

  Attacher un volume au conteneur PostgreSQL est essentiel pour la persistance des données. Lorsque vous exécutez un conteneur, toutes les données stockées dans le système de fichiers du conteneur seront perdues si celui-ci s'arrête, plante ou est supprimé. En attachant un volume, vous vous assurez que les données de la base de données sont stockées sur la machine hôte, plutôt que dans le stockage temporaire du conteneur. Ainsi, même si le conteneur est recréé ou arrêté, les données restent intactes et peuvent être accessibles lorsque le conteneur redémarre, ce qui est essentiel pour maintenir un stockage de données cohérent et fiable dans un environnement de base de données.
  
  ❓ 1-1 Documentation des éléments essentiels pour le conteneur de base de données : Commandes et Dockerfile
     Voici le Dockerfile pour configurer un conteneur PostgreSQL :

Base d’image : Nous utilisons postgres:14.1-alpine, une version légère de PostgreSQL.

Variables d’environnement : Nous définissons des variables d’environnement pour configurer le nom de la base de données, l’utilisateur et le mot de passe.

Scripts d’initialisation : Les scripts SQL dans /docker-entrypoint-initdb.d/ sont exécutés automatiquement lors du premier démarrage du conteneur, ce qui permet de configurer notre schéma et d'injecter des données initiales.

Commandes pour exécuter le conteneur PostgreSQL
Construire l’image Docker : docker build -t database .
Créer un réseau : docker network create app-network
Exécuter le conteneur PostgreSQL :docker run --name mydatabase -v /my/folder/datadir:/var/lib/postgresql/data --net=app-network -d database
Ces étapes permettront de lancer un conteneur PostgreSQL avec la configuration et la persistance des données.

❓Question : 1-2 Pourquoi avons-nous besoin d'une construction multi-étapes ? Et expliquez chaque étape de ce Dockerfile.
  Une construction multi-étapes est essentielle dans Docker pour optimiser la taille de l'image finale et rationaliser le processus de construction. En utilisant des constructions multi-étapes, nous pouvons inclure toutes les dépendances nécessaires pour construire l'application dans une première étape, puis copier uniquement les artefacts nécessaires dans une image finale plus légère. Cela réduit la taille de l'image et améliore la sécurité en excluant les outils de build et dépendances inutiles.

Dockerfile Explication
dockerfile
# Étape de construction
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

# Étape finale
FROM amazoncorretto:17
WORKDIR /app
COPY --from=myapp-build /app/target/*.jar myapp.jar
ENTRYPOINT ["java", "-jar", "myapp.jar"]
Explication de Chaque Étape
Première Étape (Étape de Construction) :
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build : Utilise une image Maven avec Amazon Corretto JDK 17, adaptée pour compiler des applications Java.
WORKDIR /app : Définit le répertoire de travail à l'intérieur du conteneur.
COPY . . : Copie tous les fichiers du répertoire actuel dans le répertoire /app du conteneur.
RUN mvn clean package -DskipTests : Construit l'application avec Maven sans exécuter les tests. Le fichier JAR résultant est créé dans le répertoire target.
Deuxième Étape (Étape Finale) :
FROM amazoncorretto:17 : Utilise une image de base plus légère avec seulement le JDK Amazon Corretto, sans outils de compilation comme Maven. Cela réduit la taille de l'image finale.
WORKDIR /app : Définit le répertoire de travail pour l'image finale.
COPY --from=myapp-build /app/target/*.jar myapp.jar : Copie uniquement le fichier JAR construit à partir de l'étape précédente (myapp-build) dans l'image finale.
ENTRYPOINT ["java", "-jar", "myapp.jar"] : Spécifie la commande pour exécuter l'application.

❓Question : Pourquoi avons-nous besoin d'un proxy inverse ?
Un proxy inverse est essentiel pour équilibrer la charge, améliorer la sécurité, simplifier l'accès des clients et améliorer les performances grâce au cache et à la compression. Dans Docker, il facilite la gestion des applications multi-conteneurs en simplifiant la communication et en centralisant l'accès.

❓Question : Pourquoi Docker Compose est-il si important ?
  Docker Compose est essentiel car il simplifie la gestion des applications multi-conteneurs. Il permet de définir tous les services (par exemple, base de données, backend, serveur HTTP) dans un seul fichier, facilitant ainsi la configuration, le réseau et les variables d'environnement. Vous pouvez tout démarrer avec une seule commande, gérer facilement les données persistantes et centraliser la configuration pour un développement, des tests et un déploiement plus fluides. Cela est particulièrement utile dans votre TP pour gérer efficacement les services interconnectés.

❓ 1-3 Commandes Docker Compose les Plus Importantes
  Voici quelques commandes essentielles de Docker Compose qui sont largement utilisées :

Démarrer les Services :docker-compose up
Arrêter les Services  :docker-compose down
Arrête et supprime tous les conteneurs créés par docker-compose up.
Reconstruire les Services :docker-compose up --build
Construit ou reconstruit les services avant de les démarrer.
Voir les Conteneurs en Cours d'Exécution :docker-compose ps
Liste les conteneurs gérés par Docker Compose.
Vérifier les Logs :docker-compose logs
Affiche les logs de tous les services. Utilisez docker-compose logs -f pour une sortie continue des logs.
Exécuter une Commande dans un Conteneur de Service :docker-compose exec <nom-du-service> <commande>

❓ Question : Pourquoi mettons-nous nos images dans un dépôt en ligne ?
  Mettre les images dans un dépôt en ligne permet un accès facile, un partage cohérent, le contrôle des versions, des déploiements automatisés et sert de sauvegarde fiable pour vos images Docker. C’est essentiel pour la collaboration, la cohérence entre les environnements et l'évolutivité.
