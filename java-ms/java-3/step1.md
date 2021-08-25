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

Edit `ws/values.yaml`{{open}}, change the repository to java-ws <pre class="file" data-filename="ws/values.yaml" data-target="insert" data-marker="  repository: nginx">  repository: java-ws</pre>

Now, change the name to latest <pre class="file" data-filename="ws/values.yaml" data-target="insert" data-marker="  tag: """>  tag: "latest"</pre>

Recall that our container port is 8080, the default in Helm is 80, so change <pre class="file" data-filename="ws/values.yaml" data-target="insert" data-marker="              containerPort: 80">              containerPort: 8080</pre>

And change in deployment.yaml as well <pre class="file" data-filename="ws/templates/deployment.yaml" data-target="insert" data-marker="              containerPort: 80">              containerPort: 8080</pre>

Remove or comment out the liveness and readiness probes (lines 40 - 47) from `ws/templates/deployment.yaml`{{open}} for now, we'll add those back in on a later session.

Comments in YAML start with the # sign, so the block of yaml would look like the following

<pre>
#          livenessProbe:
#            httpGet:
#              path: /
#              port: http
#          readinessProbe:
#            httpGet:
#              path: /
#              port: http
</pre>






