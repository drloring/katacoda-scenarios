Welcome to the stateless Java Web Services with backing service challenge!.  This is a challenge course of the previous katacoda course [here](https://www.katacoda.com/ng-dloring/courses/java-ms-config/java-2).  If you haven't reviewed that one yet, you may want to go through it now and return to this scenario afterwards.

First, we need to fetch the solution from github (https://raw.githubusercontent.com/drloring/katacoda-resources/main/java-ms-config-java-2-step-1.zip).

Once you have that downloaded and unzipped, update the pom file with the required dependencies for `spring-data-redis` and `lettuce-core`.  Fix  any problems with the GreetingController class to get it to compile.

Build the application with maven or gradle and run the `bitnami/redis:latest` docker container and verify that your service can connect to the redis container running in docker.


