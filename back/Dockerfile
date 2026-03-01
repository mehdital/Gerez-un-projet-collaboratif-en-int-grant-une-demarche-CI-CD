FROM maven:3.9-eclipse-temurin-11 AS build
RUN mkdir -p /workspace
WORKDIR /workspace
COPY pom.xml /workspace
COPY src /workspace/src
RUN mvn -B -f pom.xml clean package -DskipTests
FROM eclipse-temurin:11-jre
WORKDIR /app
COPY --from=build /workspace/target/*.jar /app/app.jar
RUN useradd -u 10001 appuser && chown appuser:appuser /app/app.jar
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
