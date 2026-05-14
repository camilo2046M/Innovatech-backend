FROM maven:3.8.4-openjdk-17-slim AS build
WORKDIR /app

COPY pom.xml .

RUN mvn dependency:go-offline -B

COPY src ./src
RUN mvn clean package -DskipTests

FROM openjdk:17-jdk-slim
WORKDIR /app

RUN groupadd -r cittgroup && useradd -r -g cittgroup cittuser
USER cittuser

COPY --from=build /app/target/*.jar app.jar

ENV DB_PORT=3306 \
    DB_NAME=bd_despachos \
    PORT=8081

EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar"]