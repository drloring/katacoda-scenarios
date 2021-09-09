In this scenario, we're going to see how we can pass configurations in from the docker file and docker runtime environment.

First, let's collect the Dockerfile we created in the Learning Java Microservices scenario `curl -o Dockerfile https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}.

One way we could change the configuration of the service is to modify the Dockerfile to supply the changes like we did from the command line in the previous scenario.  <pre class="file" data-filename="gs-rest-service/complete/Dockerfile" data-target="insert" data-marker="CMD java -jar svc.jar">CMD java -jar svc.jar --server.port=9999</pre>.  And remember, we have to change the port we're exposing too <pre class="file" data-filename="gs-rest-service/complete/Dockerfile" data-target="insert" data-marker="EXPOSE 8080">EXPOSE 9999</pre> and finally <pre class="file" data-filename="gs-rest-service/complete/Dockerfile" data-target="insert" data-marker="COPY rest-service.jar svc.jar">COPY build/libs/rest-service-0.0.1-SNAPSHOT.jar svc.jar</pre>

First we have to build the executable jar file with `./gradlew bootJar`{{execute}}.

Now, run `docker build -t java-ws .`{{execute}}, then `docker run --name jws --rm -d -p80:9999 java-ws`{{execute}} and finally `curl http://localhost:80/greeting?name=User`{{execute}}

That is a pretty inflexible way since the configuration is baked into the container.  There may be scenarios where you want individual docker images for various environments, but in general, that approach is too restrictive.

Another way we can change configuration options at runtime is to set environment variables and let the Docker runtime pass those along to the running service. `Put Commands here`.

And finally, we want to use the -E option in Docker to pass along named variables into the web service.  Spring Boot configurations supports all of these options, so there's plenty of options to manage configurations.

The main point of this exercize is to demonstrate how we can use the same binary images and configuration defaults throughout the devops pipeline and supply overrides only when necessary in the environments that require modification from the default configuration.



