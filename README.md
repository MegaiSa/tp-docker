Question 1-1

It's better to run the container with a flag -e because everyone can access a value defined with ENV. Also, it means we can use the same image with different identities without rebuild.

Question 1-2

Data inside a container is lost when the container is removed, while data stored in a volume persists and is available to any container that mounts it. A database needs this so its data survives container recreation or updates.

Question 1-3

Dockerfile
FROM postgres:17.2-alpine
COPY *.sql /docker-entrypoint-initdb.d/

the scripts in /docker-entrypoint-initdb.d/ are executed in alphabetic order at the first start. 

docker network create app-network
docker build -t tp-database ./database
docker run -d --name tp-database --network app-network \
  -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd \
  -v tp-db-data:/var/lib/postgresql/data \
  tp-database
docker run -d --name adminer --network app-network -p 8090:8080 adminer

Question 1-4

1-4 A multistage build separates building from running. The first stage uses a JDK and Maven to compile the app into a .jar; the second stage starts from a lightweight JRE image and only copies that .jar. The final image is smaller and more secure, since it contains no build tools or source code.

Steps:

FROM eclipse-temurin:21-jdk-alpine AS myapp-build: build stage with the JDK.
ENV / WORKDIR: set and move into the app folder.
RUN apk add maven: install Maven.
COPY pom.xml + RUN mvn dependency:go-offline: download dependencies (cached as long as pom.xml doesn’t change).
COPY src + RUN mvn package -DskipTests: compile and build the .jar.
FROM eclipse-temurin:21-jre-alpine: run stage, with only the Java runtime.
COPY --from=myapp-build ...: copy the .jar from the build stage.
ENTRYPOINT ["java", "-jar", "myapp.jar"]: start the app when the container runs.
