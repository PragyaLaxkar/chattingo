# Stage 1: Build the application
FROM maven:3.9.11-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn -B -ntp dependency:go-offline
COPY src/ /app/src/
RUN mvn -B -ntp clean package -DskipTests

# Stage 2: Create the runtime image
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

RUN useradd -m myuser
USER myuser

ENV SPRING_PROFILES_ACTIVE=production \
    JWT_SECRET=your-secret-key-here \
    SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/chattingo_db \
    SPRING_DATASOURCE_USERNAME=root \
    SPRING_DATASOURCE_PASSWORD=test@123 \
    CORS_ALLOWED_ORIGINS="http://localhost:3000,http://localhost:80"

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
