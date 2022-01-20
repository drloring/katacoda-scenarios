Next we're going to see how we can pass these variables in through the Helm application manager for Kubernetes.  But first, we'll need Helm 3 again.

Follow these steps to install helm3:

`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh &&./get_helm.sh && alias helm=/usr/local/bin/helm`{{execute}}

Now we have all the charts to install an application.  The default installation simply installs redis, so we're going to change that for our service.

If we look at the values in `ws/values.yaml`{{open}}, we see that the tag and repository are all set.
<pre>
repository: java-ws
...
tag: "latest"
</pre>

Also notice that we've added some additional values to `values.yaml`
<pre>
message: "HELLO, %s!!!"
serverport: 9090
</pre>

And, looking in `ws/templates/deployment.yaml`{{open}} we see the continer port has been set to 9090
<pre>
containerPort: 9090
</pre>

And that the values are being passed into the container's environment variables
<pre>
env:
- name: "template"
  value: "{{ .Values.message }}"
- name: "server.port"
  value: "{{ .Values.serverport }}"
</pre>

Now, we can dry-run the helm install `helm install --set serverport=9999 --dry-run ws ws`{{execute}}  and you will notice that the container is getting the server.port from the command line environment variable and the template from the values.yaml.

Now, we can the helm install `helm install --set serverport=9999 ws ws`{{execute}}  and you will notice that the container is getting the server.port from the command line and the template from the values.yaml.

Follow the instructions from helm and export the NODE_PORT and NODE_IP, then `curl $NODE_IP:$NODE_PORT/greeting`{{execute}}
