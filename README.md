# Docker

## [Reference](https://spring.io/guides/topicals/spring-boot-docker/#a-better-dockerfile)

## Multistage builds

The container will not have demo.jar as the second stage only copies the necessary folders

```Dockerfile
#Use JDK for building project
FROM eclipse-temurin:17-jdk-alpine as builder
WORKDIR /application
ADD demo/build/libs/demo-*-SNAPSHOT.jar demo.jar
RUN java -Djarmode=layertools -jar demo.jar extract

#Second stage for running the app uses JRE
FROM eclipse-temurin:17-jre-alpine
WORKDIR /application/app
#Security aspect
RUN addgroup -S javauser && adduser -S javauser -G javauser
RUN chown -R javauser:javauser /application/app
USER javauser

#Multilayered build
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/application/ ./
ENV MY_PRIVATE_IP=${MY_PRIVATE_IP}
ENTRYPOINT [ "sh", "-c", "java -Dspring.profiles.active=${profile} org.springframework.boot.loader.JarLauncher demo" ]
```

## Build without Dockerfile

1. Buildpacks
2. bootBuildImage - available for Spring Boot 2.3+
3. Jib plugin from Google - Faster build