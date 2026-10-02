# Imagen de Jenkins con Maven y el CLI de Docker, para que el controlador
# (montando /var/run/docker.sock) pueda compilar, construir y publicar
# los artefactos del backend y del frontend durante el pipeline.
FROM jenkins/jenkins:lts-jdk17

USER root

RUN apt-get update \
 && apt-get install -y --no-install-recommends maven docker.io curl \
 && rm -rf /var/lib/apt/lists/*

# Plugin de Docker Compose v2 (permite usar "docker compose")
ARG COMPOSE_VERSION=v2.29.7
RUN mkdir -p /usr/local/lib/docker/cli-plugins \
 && curl -fsSL "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-linux-$(uname -m)" \
      -o /usr/local/lib/docker/cli-plugins/docker-compose \
 && chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt

USER jenkins
