FROM maven:3.9.6-eclipse-temurin-21 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests -B

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
RUN addgroup -S innovatech && adduser -S innovatech -G innovatech

COPY --chown=innovatech:innovatech --from=builder /app/target/*.jar app.jar
COPY --chown=innovatech:innovatech entrypoint.sh entrypoint.sh
RUN chmod +x entrypoint.sh

USER innovatech
EXPOSE 8081

ENTRYPOINT ["sh", "entrypoint.sh"]