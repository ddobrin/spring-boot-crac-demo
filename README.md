# Build and Deploy a Spring Boot App using CRaC in Google Cloud Run 

Demo started by Sebastian Deleuze

Build and test a regular Jit then a Native Image:
```shell
./mvnw package
java -jar target/spring-boot-crac-demo-1.0.0-SNAPSHOT.jar

./mvnw native:compile -Pnative
./target/spring-boot-crac-demo

curl localhost:8080
http :8080
```

Build a regular JIT Docker image then a Native Image
```shell
./mvnw spring-boot:build-image
docker run --rm -p8080:8080 spring-boot-crac-demo:1.0.0-SNAPSHOT

./mvnw spring-boot:build-image -Pnative -DskipTests

curl localhost:8080
http :8080
```

To build a checkpointed image, run the script:
```shell
./checkpoint.sh
```

To compare the startup time for the image, run the script:
```shell
./restore.sh

# or run the image
docker run --rm -p 8080:8080 --name spring-boot-crac-demo sdeleuze/spring-boot-crac-demo:checkpoint
```
