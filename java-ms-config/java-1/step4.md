Next we're going to see how we can pass these variables in through the Helm application manager for Kubernetes.  But first, we'll need Helm 3 again.

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


