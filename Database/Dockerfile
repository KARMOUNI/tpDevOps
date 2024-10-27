# database/Dockerfile
FROM postgres:14.1-alpine

# Configuration des variables d'environnement pour PostgreSQL
ENV POSTGRES_USER=user
ENV POSTGRES_PASSWORD=password
ENV POSTGRES_DB=pg_database

# Copier les scripts SQL dans le répertoire d'initialisation de PostgreSQL
COPY scripts/ /docker-entrypoint-initdb.d/

# Exposer le port par défaut de PostgreSQL
EXPOSE 5432
