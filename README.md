# Multi-Stage Dockerfile for Java Web Application
Multi-Stage Dockerfile
This project uses a multi-stage Dockerfile to build and run a Java web application on a Tomcat server. The Dockerfile defines two stages: one for building the application using Maven, and another for deploying the application on Tomcat. By using a multi-stage build, we can optimize the size of the final image by excluding unnecessary build tools and dependencies from the runtime environment.

# Dockerfile Breakdown

# Stage 1: Build the Application

FROM openjdk:8 AS BUILD_IMAGE:
The first stage uses the openjdk:8 image, which includes the Java Development Kit (JDK) version 8. This is essential for compiling and building the Java application.

RUN apt update && apt install maven -y:
Maven is required for building the Java project. This command updates the package manager and installs Maven inside the build environment.

RUN git clone -b vp-docker https://github.com/vprofile-repo.git:
The source code for the application is cloned from a Git repository using the vp-docker branch.

RUN cd vprofile-repo && mvn install:
Maven is used to build the Java application and generate a vprofile-v2.war file in the target/ directory. This file is a Web Application Archive (WAR) that will be deployed in the next stage.

# Stage 2: Deploy to Tomcat

FROM tomcat:8-jre11:
The second stage uses the tomcat:8-jre11 image, which includes the Tomcat 8 application server with the Java Runtime Environment (JRE) 11. This environment is much lighter since we don't need build tools like Maven in the final runtime image.

RUN rm -rf /usr/local/tomcat/webapps/*:
This command removes any pre-existing web applications from the Tomcat server's webapps/ directory to ensure a clean deployment.

COPY --from=BUILD_IMAGE vprofile-repo/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war:
This copies the vprofile-v2.war file from the BUILD_IMAGE stage (the first stage) to the webapps/ROOT.war directory in the Tomcat container. By using --from=BUILD_IMAGE, we fetch the WAR file directly from the build stage, without having to install Maven or Git in the runtime image.

EXPOSE 8080:
Exposes port 8080, which is the default port that Tomcat listens to for incoming HTTP requests.

CMD ["catalina.sh", "run"]:
This is the command that runs when the container starts. It starts the Tomcat server using the catalina.sh script, making the web application accessible via the exposed port.

# Benefits of Multi-Stage Docker Build

Smaller Final Image:
Only the Tomcat server and the compiled .war file are included in the final image, which significantly reduces the image size by leaving out unnecessary build tools like Maven, Git, and the JDK.

Improved Security:
The final image contains fewer dependencies, reducing the attack surface since tools like Maven or Git are not needed in production.

Separation of Concerns:
Build and runtime environments are separated, making the final image lean and focused purely on running the application



