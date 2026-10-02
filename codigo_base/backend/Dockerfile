# =============================================================================
# Taller Evaluativo 2 - Fase 2: Empaquetamiento Inmutable Multi-Stage
# Backend: studytrack-api
#
# Complete los TODOs siguiendo las buenas prácticas vistas en el curso:
# reducción de peso, cache de dependencias, usuario no root y ausencia
# total del SDK/compilador en la imagen final de runtime.
# =============================================================================

# ---------- Stage 1: Builder ----------
# TODO
FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder
WORKDIR /app
COPY pom.xml .

RUN mvn dependency:go-offline
COPY src ./src

RUN mvn package -DskipTests

# ---------- Stage 2: Runtime ----------
# TODO
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

COPY --from=builder /app/target/*.jar app.jar
# Expone puerto
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]