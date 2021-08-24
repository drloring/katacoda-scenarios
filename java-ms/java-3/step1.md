In this scenario, we're going to take the web service that we dockerized in the previous lesson and deploy it to a Kubernetes cluster.  This scenario starts a minimal kubernetes cluster on one node called minikube.  The initialization of the cluster takes some time initially, so it's generally better to wait until you see the command prompt before issuing any commands.

First, we're going to collect the executable jar we made in the first lesson `curl -o rest-service.jar https://raw.githubusercontent.com/drloring/katacoda-resources/main/rest-service-0.0.1-SNAPSHOT.jar`{{execute}}.

Run `ls -lha`{{execute}} to confirm the jar file transferred.  

Next, we're going to collect the Dockerfile we made in the last lesson `curl -o Dockerfile https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}.

Open the `Dockerfile`{{open}} in the editor 

We're going to build our Docker image now and tag it with the `java-ws` tag using `docker build -t java-ws .`{{execute}}.  

Verify the docker image was build by running `docker images | grep java-ws`{{execute}}. 

Helm is an application manager for Kubernetes.  We're going to use Helm to create an application configuration for our web service.

Follow these steps to install helm3:

`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3`{{execute}}

`chmod +x get_helm.sh`{{execute}}

`./get_helm.sh`{{execute}}

To deconflict the locally installed helm from the new Helm 3, make an alias called h3 to replace the helm commands.
`alias h3=/usr/local/bin/helm`{{execute}}

`h3 version`{{execute}} should confirm version 3.6.3 or later is installed.

`h3 create ws`{{execute}} will create a simple helm chart called ws (for web service).

Now we have all the charts to install an application.  The default installation simply installs redis, so we're going to change that for our service.

Edit `ws/Values.yml`{{open}}, change repository to java-ws
and name to latest
remove liveness and readiness from deployment.yml for now, we'll add those back in on a later session.

Dry run the install with the following command `h3 install ws ws --dry-run`{{execute}}.

This shows you what the final kubernetes yml files will look like prior to installing in the cluster.

When ready, run the installer again, this time with non --dry-run `h3 install ws ws`{{execute}}

Run `kubectl pet pods`{{execute}} and ensure that your pod is installed. 





