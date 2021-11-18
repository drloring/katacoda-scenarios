First we have to exit out of the anchore-cli pod `exit`{{execute}}, now let's get the web service we created in past courses `curl -o rest-service.jar https://raw.githubusercontent.com/drloring/katacoda-resources/main/rest-service-0.0.1-SNAPSHOT.jar`{{execute}} and the Dockerfile `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}

Now, we'll build the docker image with `docker build -t java-ws .`{{execute}}

`kubectl get svc`{{execute}} to see the list of services, take note of the NodePort address allocated, then  `curl -s https://ci-tools.anchore.io/inline_scan-latest | bash -s -- analyze -u admin -p $ANCHORE_CLI_PASS -r http://localhost:<NodePort> -g java-ws:latest`{{execute}}


