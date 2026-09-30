FROM docker:cli AS docker-cli

FROM maven:3.9-eclipse-temurin-21 AS maven-source

FROM node:24-bookworm-slim AS node-source

# GNU grep for the minimal Jenkins runtime
FROM maven-source AS utility-source

RUN mkdir -p /opt/grep/bin /opt/grep/lib \
    && cp /usr/bin/grep /opt/grep/bin/grep \
    && cp /lib/x86_64-linux-gnu/libpcre2-8.so.0 /opt/grep/lib/libpcre2-8.so.0

# Prepare Node.js 24 for the Jenkins image
FROM node-source AS node-prep

RUN mkdir -p /opt/node/bin /opt/node/lib \
    && cp -a /usr/local/lib/node_modules /opt/node/lib/ \
    && cp /usr/local/bin/node /opt/node/bin/node \
    && ln -s /opt/node/lib/node_modules/npm/bin/npm-cli.js /opt/node/bin/npm \
    && ln -s /opt/node/lib/node_modules/npm/bin/npx-cli.js /opt/node/bin/npx

FROM dhi.io/jenkins:latest

# GNU grep
COPY --from=utility-source /opt/grep/bin/grep /usr/bin/grep
COPY --from=utility-source /opt/grep/lib/libpcre2-8.so.0 /lib/x86_64-linux-gnu/libpcre2-8.so.0

# Docker CLI
COPY --from=docker-cli /usr/local/bin/docker /usr/local/bin/docker

# Docker Compose CLI plugin
COPY --from=docker-cli /usr/local/libexec/docker/cli-plugins/docker-compose /usr/local/libexec/docker/cli-plugins/docker-compose

# Maven
# Java JDK 21 + Maven
COPY --from=maven-source /opt/java/openjdk /opt/java/openjdk
COPY --from=maven-source /usr/share/maven /opt/maven

# Node.js 24
COPY --from=node-prep /opt/node /opt/node

# Node.js required library
COPY --from=node-source /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.30 /opt/node-libs/libstdc++.so.6.0.30
COPY --from=node-source /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /opt/node-libs/libstdc++.so.6

# SSL Certificate
COPY --from=maven-source /etc/ssl/certs/ca-certificates.crt /opt/ca-certificates.crt

ENV JAVA_HOME=/opt/java/openjdk
ENV MAVEN_HOME=/opt/maven

ENV LD_LIBRARY_PATH=/opt/node-libs
ENV GIT_SSL_CAINFO=/opt/ca-certificates.crt

ENV PATH=/opt/node/bin:/opt/maven/bin:/opt/java/openjdk/bin:/usr/local/bin:$PATH

USER jenkins