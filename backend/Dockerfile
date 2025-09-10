# Stage 1: Build the application
FROM maven:3.8.6-openjdk-17-slim AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src/ /app/src/
RUN mvn clean package -DskipTests

# Stage 2: Create the runtime image
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

# Create a non-root user and switch to it
RUN useradd -m myuser
USER myuser

# Set environment variables with defaults
ENV SPRING_PROFILES_ACTIVE=production \
    JWT_SECRET=your-secret-key-here \
    SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/chattingo_db \
    SPRING_DATASOURCE_USERNAME=root \
    SPRING_DATASOURCE_PASSWORD=test@123 \
    CORS_ALLOWED_ORIGINS="http://localhost:3000,http://localhost:80"

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
