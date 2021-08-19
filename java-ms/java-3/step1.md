In this step, we're going to take the web service that we dockerized in the previous lesson and deploy it to a Kubernetes cluster.

First, we're going to collect the executable jar we made in the first lesson `curl -o rest-service.jar https://raw.githubusercontent.com/drloring/katacoda-resources/main/rest-service-0.0.1-SNAPSHOT.jar`{{execute}}.

Run `ls -lha`{{execute}} to confirm the jar file transferred.  

NExt, we're going to collect the Dockerfile we made in the last lesson `curl -o Dockerfile https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}.

Open the `Dockerfile`{{open}} in the editor 

We're going to build our Docker image now and tag it with the `java-ws` tag using `docker build -t java-ws .`{{execute}}.  

Verify the docker image was build by running `docker images | grep java-ws`{{execute}}. 

Install helm3:

`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3`{{execute}}

`chmod +x get_helm.sh`{{execute}}

`./get_helm.sh`{{execute}

To deconflict the locally installed helm from the new Helm 3, make an alias called h3 to replace the helm commands.
`alias h3=/usr/local/bin/helm`{{execute}}

`h3 version`{{execute}} should confirm version 3.6.3 is installed.

`h3 create`{{execute}} will create a simple helm chart

`h3 install ws ws --dry-run`{{execute}}

remove liveness and readiness from deployment.yml

`h3 install ws ws`{{execute}}





