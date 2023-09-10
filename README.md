# Build and Deploy a Spring Boot App using CRaC in Google Cloud Run 

This repo builds on the demo by Seb Deleuze (sdeleuze/spring-boot-crac-demo)

Please not that, while you can build the checkpointed image on both Intel and Apple M* series, in order to deploy to Cloud Run you have to build in a similar architecture as the one you would be running on. See the [Cloud Run Container Runtime Contract](https://cloud.google.com/run/docs/container-contract)

## Build JIT, Native and CRaC images

Build and test a regular JIT then a Native Image:
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

## Deploy CRaC Image
Let's build the checkpointed CRaC image and deploy it to Cloud Run. 
```shell
# first, build the image if you have not done so before
./checkpoint,sh

# tag the image
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
echo   $PROJECT_ID
docker tag sdeleuze/spring-boot-crac-demo:checkpoint gcr.io/${PROJECT_ID}/spring-crac

# push the image to GCR
docker push gcr.io/optimize-serverless-apps/spring-crac
```

Deploy the image to Cloud Run and test it
```shell
# get the PROJECT_ID
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
echo   $PROJECT_ID

gcloud beta run deploy spring-crac  \
    --image=gcr.io/${PROJECT_ID}/spring-crac  \
    --execution-environment=gen2  \
    --allow-unauthenticated \
    --region=us-central1 \
    --memory 2Gi --cpu 2 

# observe the deploy output
...
  ✓ Deploying... Done.                                                                                                                                
  ✓ Creating Revision...                                                                                                                            
  ✓ Routing traffic...                                                                                                                              
  ✓ Setting IAM Policy...                                                                                                                           
Done.                                                                                                                                               
Service [spring-crac] revision [spring-crac-00003-his] has been deployed and is serving 100 percent of traffic.
Service URL: https://spring-crac-....XYZ   
```

Run a test request:
```shell
# use your deployment URL
curl https://spring-crac-....XYZ

# observe
Greetings from Spring Boot!

# Note: if you run your service as authenticated, use 
curl -i  -H "Authorization: Bearer $(gcloud auth print-identity-token)" https://spring-crac-....XYZ
```

## Deploy Native Java Image
```shell
# tag native image
docker tag spring-boot-crac-demo:1.0.0-SNAPSHOT gcr.io/${PROJECT_ID}/spring-native

# push image to GCR
docker purh gcr.io/${PROJECT_ID}/spring-native

# deploy to Cloud Run 
gcloud beta run deploy spring-native  \
    --image=gcr.io/${PROJECT_ID}/spring-native  \
    --execution-environment=gen2  \
    --allow-unauthenticated \
    --region=us-central1 \
    --memory 2Gi --cpu 2 

curl https://spring-native-...XYZ   
```

## Deployed services
You can observe the deployed services in Cloud Run
```shell
✔  spring-crac               us-central1   https://spring-crac-...uc.a.run.app                 2023-09-10T13:10:18.822405Z
✔  spring-native             us-central1   https://spring-native-...-uc.a.run.app              2023-09-10T13:22:04.057953Z
```