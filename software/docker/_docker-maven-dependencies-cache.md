# Docker Maven Dependencies Cache

Edit `Dockerfile` of your project:

```dockerfile
#
# Build stage
#

# our base build image
FROM maven:3.5.3-jdk-8-alpine AS maven
# copy the project files
COPY ./pom.xml ./pom.xml
# build all dependencies
RUN mvn dependency:go-offline -B
# copy your other files
COPY ./src ./src
# build for release
RUN mvn clean
RUN mvn package spring-boot:repackage

#
# Package stage
#

# our final base image
FROM openjdk:8-jdk-alpine
# set deployment directory
WORKDIR /app/jenkins-playground
# copy over the built artifact from the maven image
COPY --from=maven target/*.jar ./app.jar
EXPOSE 8080
# set the startup command to run your binary
ENTRYPOINT ["java","-jar","./app.jar"]
```

Docker build

```shell
cd <your_project_dir>
docker build -t <your_specified_image_name> .
```

Docker run

```
docker run -d -p 8080:80 <your_image_name>
```

- -p: Create a port mapping rule like -p `ip:hostPort:containerPort`. `containerPort` is required. If no hostPort is specified, Docker will automatically allocate one.
- 80: your application listening port.
- 8080: client http connection port.

Visit web project url: `http://<your_server_ip>/8080`



## References

- dockerize maven project
  - [How to dockerize maven project? and how many ways to accomplish it? - Stack Overflow](https://stackoverflow.com/questions/27767264/how-to-dockerize-maven-project-and-how-many-ways-to-accomplish-it)
- Docker maven cache
  - [Optimizing Docker Images for Maven Projects](http://whitfin.io/speeding-up-maven-docker-builds/)
  - [Caching Maven dependencies in a Docker build](https://medium.com/@nieldw/caching-maven-dependencies-in-a-docker-build-dca6ca7ad612)