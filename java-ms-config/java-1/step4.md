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

Edit `gs-rest-service/complete/ws/values.yaml`{{open}}, change the repository to java-ws <pre class="file" data-filename="gs-rest-service/complete/ws/values.yaml" data-target="insert" data-marker="  repository: nginx">  repository: java-ws</pre>

Now, change the name to latest <pre class="file" data-filename="gs-rest-service/complete/ws/values.yaml" data-target="insert" data-marker="  tag: &#x22&#x22">  tag: "latest"</pre>

And change `gs-rest-service/complete/ws/templates/deployment.yaml`{{open}} as well <pre class="file" data-filename="gs-rest-service/complete/ws/templates/deployment.yaml" data-target="insert" data-marker="              containerPort: 80">              containerPort: 9090</pre>

Remove or comment out the liveness and readiness probes (lines 40 - 47) from deployment.yaml for now, we'll add those back in on a later session.
<pre class="file" data-filename="gs-rest-service/complete/ws/templates/deployment.yaml" data-target="insert" data-marker="          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
">
#          livenessProbe:
#            httpGet:
#              path: /
#              port: http
#          readinessProbe:
#            httpGet:
#              path: /
#              port: http
</pre>

And now we're going to add some application specific environment variables to pass into the application.  First, we're going to add them to `gs-rest-service/complete/ws/values.yaml`{{open}} <pre class="file" data-filename="gs-rest-service/complete/ws/values.yaml" data-target="append">message: "HELLO, %s!!!"
serverport: 9090</pre>

Then, paste the following code immediately after `protocol: TCP` line in `gs-rest-service/complete/ws/templates/deployment.yaml`{{open}}
<pre>          env:
            - name: "template"
              value: "{{ .Values.message }}"
            - name: "server.port"
              value: "{{ .Values.serverport }}"</pre>

Now, we can dry-run the helm install `helm install --set serverport=9999 --dry-run ws ws`{{execute}}  and you will notice that the container is getting the server.port from the command line environment variable and the template from the values.yaml.

Before we install it, let's open up the NodePort again so we don't have to port-forward to get access.  Start, by opening the `gs-rest-service/complete/ws/templates/service.yaml`{{open}} file.  Then change the targetPort to <pre class="file" data-filename="gs-rest-service/complete/ws/templates/service.yaml" data-target="insert" data-marker="      targetPort: http">      targetPort: 9090</pre> to reflect our pod's exposed port. 

Then change the NodePort in values.yaml <pre class="file" data-filename="gs-rest-service/complete/ws/values.yaml" data-target="insert" data-marker="  type: ClusterIP">  type: NodePort</pre>

Now, we can the helm install `helm install --set serverport=9090 ws ws`{{execute}}  and you will notice that the container is getting the server.port from the command line and the template from the values.yaml.

Follow the instructions from helm and export the NODE_PORT and NODE_IP, then `curl $NODE_PORT:$NODE_IP/greetings`{{execute}}
