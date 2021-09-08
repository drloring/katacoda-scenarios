In this scenario, we're going to see how we can pass configurations in from the docker file and docker runtime environment.

First, let's collect the Dockerfile we created in the Learning Java Microservices scenario `curl -o Dockerfile https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}.

One way we could change the configuration of the service is to modify the Dockerfile to supply the changes.  `File commands here`.

That is a pretty inflexible way since the configuration is baked into the container.  There may be scenarios where you want individual docker images for various environments, but in general, that approach is too restrictive.

Another way we can change configuration options at runtime is to set environment variables and let the Docker runtime pass those along to the running service. `Put Commands here`.

And finally, we want to use the -E option in Docker to pass along named variables into the web service.  Spring Boot configurations supports all of these options, so there's plenty of options to manage configurations.

The main point of this exercize is to demonstrate how we can use the same binary images and configuration defaults throughout the devops pipeline and supply overrides only when necessary in the environments that require modification from the default configuration.



