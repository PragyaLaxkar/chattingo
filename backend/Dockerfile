# Stage 1: Build the application
FROM maven:3.9.11-eclipse-temurin-17 AS build
WORKDIR /app

# Copy only the files needed for downloading dependencies
COPY pom.xml .
RUN mvn -B -ntp dependency:go-offline

# Copy source code and build
COPY src/ /app/src/
RUN mvn -B -ntp clean package -DskipTests

# Stage 2: Create the runtime image
FROM openjdk:17-jdk-slim
WORKDIR /app

# Copy the built JAR file
COPY --from=build /app/target/*.jar app.jar

# Create a non-root user and switch to it
RUN useradd -m myuser && \
    chown -R myuser:myuser /app
USER myuser

# Environment variables (these should be provided at runtime)
ENV SPRING_PROFILES_ACTIVE=production

# Expose the port the app runs on
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
