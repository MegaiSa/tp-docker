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

Question 1-5 

A reverse proxy is a single entry point in front of the backend: clients only talk to it, and it forwards requests to the right service. This means the backend doesn’t need to be exposed directly, and the proxy can handle shared concerns like SSL/HTTPS, load balancing between several backend instances, or serving a front-end.

Question 1-6 

Docker compose lets us define the whole multi-container application in a single file and manage it with one command, instead of running each container manually with long commands. It handles networks, volumes, environment variables and startup order, making the setup reproducible and easy to share.

Question 1-7

docker compose up -d --build: build images and start all services in the background
docker compose down: stop and remove containers and networks (-v also removes volumes)
docker compose ps: list the services and their status
docker compose logs -f <service>: follow a service’s logs
docker compose restart <service>: restart a service
docker compose build: rebuild the images

Question 1-8

database: built from ./database, credentials loaded from a .env file, data persisted in the db-data volume, with a healthcheck (pg_isready).
backend: built from ./backend, connects to the database via its service name; starts only once the database is healthy.
httpd: reverse proxy, the only service exposing a port (80).

Two networks segregate the services: frontend-network (httpd ↔ backend) and backend-network (backend ↔ database), so httpd can’t reach the database. All services use restart: unless-stopped.

Question 1-9

docker login
docker tag tp01-database <user>/tp-database:1.0
docker tag tp01-backend <user>/tp-backend:1.0
docker tag tp01-httpd <user>/tp-httpd:1.0
docker push <user>/tp-database:1.0
docker push <user>/tp-backend:1.0
docker push <user>/tp-httpd:1.0

Question 1-10

Images are stored locally by default. Pushing them to an online registry makes them available to other team members and other machines (servers, CI/CD pipelines), which can pull and run them without rebuilding. Versioned tags also keep a history of each release.
