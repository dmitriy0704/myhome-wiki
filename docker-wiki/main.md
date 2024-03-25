# Dockerfile instructions:

### ___**Gradle**___

Сборка внутри контейнера:
```shell
FROM gradle:8.5-jdk17-alpine as build
COPY --chown=gradle:gradle . /home/gradle/src
WORKDIR /home/gradle/src
RUN gradle build --no-daemon
FROM openjdk:17-jdk-slim
EXPOSE 8080
RUN mkdir /app
COPY --from=build /home/gradle/src/build/libs/*.jar /app/spring-boot-application.jar
ENTRYPOINT ["java", "-jar", "/app/spring-boot-application.jar"]
```
Локальная сборка:
```shell
FROM eclipse-temurin:17-jre
WORKDIR /opt/app
EXPOSE 8080
COPY build/libs/*.jar app.jar
ENTRYPOINT ["java","-jar", "app.jar"]
```

### ___**Maven**___

```shell
FROM maven:3.9.6-eclipse-temurin-17 as builder
WORKDIR /opt/app
COPY mvnw pom.xml ./
COPY ./src ./src
RUN mvn clean install -DskipTests

FROM eclipse-temurin:17-jre-jammy
WORKDIR /opt/app
EXPOSE 8080
COPY --from=builder /opt/app/target/*.jar /opt/app/*.jar
ENTRYPOINT ["java", "-jar", "/opt/app/*.jar" ]
```

__Создание образа и контейнера из образа__

```shell

docker build .  # docker build -t [NEWIMAGENAME] .
docker tag sha256:bbd78... docker-gradle-image
docker run -i -t -p 8080:8080 --name docker-gradle-cont docker-gradle-image
```
