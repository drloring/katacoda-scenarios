First we have to exit out of the anchore-cli pod `exit`{{execute}}, now let's get the web service we created in past courses `curl -o rest-service.jar https://raw.githubusercontent.com/drloring/katacoda-resources/main/rest-service-0.0.1-SNAPSHOT.jar`{{execute}} and the Dockerfile `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}

Now, we'll build the docker image with `docker build -t java-ws .`{{execute}}

Now we can exec back into the Anchore CLI pod `kubectl exec -it ancore-cli -- bash`{{execute}} and add the image `anchore-cli add image java-ms`{{execute}}

Once we see that the status is `analyzed` with `anchore-cli image list`{{execute}}.



