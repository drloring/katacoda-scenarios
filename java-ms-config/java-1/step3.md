In this scenario, we're going to see how we can pass configurations in from the docker file and docker runtime environment.

First, let's collect the Dockerfile we created in the Learning Java Microservices scenario `curl -o Dockerfile https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}.

One way we could change the configuration of the service is to modify the Dockerfile to supply the changes like we did from the command line in the previous scenario.  <pre class="file" data-filename="gs-rest-service/complete/Dockerfile" data-target="insert" data-marker="CMD java -jar svc.jar">CMD java -jar svc.jar --server.port=9999</pre>.  And remember, we have to change the port we're exposing too <pre class="file" data-filename="gs-rest-service/complete/Dockerfile" data-target="insert" data-marker="EXPOSE 8080">EXPOSE 9999</pre> and finally <pre class="file" data-filename="gs-rest-service/complete/Dockerfile" data-target="insert" data-marker="COPY rest-service.jar svc.jar">COPY build/libs/rest-service-0.0.1-SNAPSHOT.jar svc.jar</pre>

First we have to build the executable jar file with `./gradlew bootJar`{{execute}}.

Now, run `docker build -t java-ws .`{{execute}}, then `docker run --name jws --rm -d -p80:9999 java-ws`{{execute}} and finally `curl http://localhost:80/greeting?name=User`{{execute}}

That is a pretty inflexible way since the configuration is baked into the image.  There may be scenarios where you want individual docker images for various environments, but in general, that approach is too restrictive.

Another way we can change configuration options at runtime is to set environment variables when we run the container and let the Docker runtime pass those along to the running service. 

First we have to stop our currently running container `docker container stop $(docker container ls -q --filter name=jws)`{{execute}}

Next, we'll update the Dockerfile to remove the command line parameters from the CMD <pre class="file" data-filename="gs-rest-service/complete/Dockerfile" data-target="insert" data-marker="CMD java -jar svc.jar --server.port=9999">CMD java -jar svc.jar</pre>.  Now that we've removed that parameter, the default behavior would be to publish to port 9090, so let's update the port we're exposing accordingly <pre class="file" data-filename="gs-rest-service/complete/Dockerfile" data-target="insert" data-marker="EXPOSE 9999>EXPOSE 9090</pre>

Every time we change our Dockerfile, we need to rebuild our image `docker build -t java-ws .`{{execute}}.

Now, we'll add the --env or -e parameters to the command line `docker run --name jws --rm -d -p80:9999 -e server.port=9999 java-ws`{{execute}} and  `curl http://localhost:80/greeting?name=User`{{execute}} to see the effects.  Notice the difference between exposing ports and port mapping.  Even though our Dockerfile exposes port 9090, we can override that with the -p option from the command line.

In the next scenario, we going to see how we can use the container environment overrides to map to kubernetes configmaps.


