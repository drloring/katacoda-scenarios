Welcome to the stateless Java Web Services with backing service challenge!.  This is a challenge course of the previous katacoda course [here](https://www.katacoda.com/ng-dloring/courses/java-ms-config/java-2).  If you haven't reviewed that one yet, you may want to go through it now and return to this scenario afterwards.

First, we need to fetch the solution from github (https://raw.githubusercontent.com/drloring/katacoda-resources/main/java-ms-config-java-2-step-2.zip).

Build the application with maven or gradle and run the `bitnami/redis:latest` docker container.  Recall that redis runs on port 6379 by default and make sure to pass in ALLOW_EMPTY_PASSWORD=yes into the environment. 

Once, that's running,  verify that your service can connect to the redis container running in docker with a curl command.


