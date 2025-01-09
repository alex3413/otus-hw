FROM openjdk:22
ARG TARGET_PATH=./target
ARG JAR_FILE=${TARGET_PATH}/hw2-0.0.1-SNAPSHOT.jar
COPY ${JAR_FILE} /app.jar
USER root
RUN chmod +x ./app.jar
EXPOSE 8000
WORKDIR /
ENTRYPOINT ["java", "-jar", "app.jar"]