FROM openjdk:17-jdk-slim
LABEL maintainer="prem4131@gmail.com"
EXPOSE 8081
COPY target/accounts-0.0.1-SNAPSHOT.jar accounts.jar
ENTRYPOINT ["java","-jar","/accounts.jar"]

