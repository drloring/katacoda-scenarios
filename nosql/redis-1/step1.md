In this course, we're going to expand on the work that we previously did with Redis in Kubernetes [here](https://katacoda.com/ng-dloring/courses/java-ms-config/java-2).
Previously, we deployed our Redis cluster with `--noauth`, not requiring a password.  That's not really realistic in production environments, so we're going to see how to create and use secrets in kubernetes to manage our passwords.

In the previous course, we used the Bitnami helm charts to quickly get up to speed.  However, in this course, we're going to re-create some of the features used in the bitnami charts to learn the basics of deploying production charts.

First, though, we have to install `curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh && ./get_helm.sh && alias helm=/usr/local/bin/helm`{{execute}}.

Not that we have helm installed, let's create a basic chart with `helm create redis`{{execute}}.  You'll recall that the redis charts reside in the redis directory.  If we open `redis/values.yaml`{{open}}, and change the `repository` from `nginx` to `redis` and tag from `""` to `"latest"`, we'll have a chart that deploys a simple redis database.

We also have to remove the liveness and readiness probes (lines 40 - 47) in `redis/templates/deployment.yaml`{{open}}.

Ensure that minikube is Ready by running `kubectl get nodes`{{execute}}, then run `helm install redis redis`{{execute}} and `kubectl get all`{{execute}}, we'll see our service and pods running in kubernetes.  If we `kubectl -it exec $(kubectl get po -o jsonpath="{.items[0].metadata.name}") -- redis-cli`{{execute}} to connect to the local server.

Run `set this that`{{execute}} and `get this`{{execute}} to verify that we can save to the database.

Now that we've verified that our database is operational, exit out of the pod with `Ctrl+D`.

This is not really a secure way to deploy your database, however.  Next, we're going to see how we can generate and pass in a secret with the redis username and password.

First, we need to automatically generate a secret in kubernetes.  We'll start by creating `secret.yaml` with `touch redis/templates/secret.yaml`{{execute}}, then open the file `redis/templates/secret.yaml`{{open}} and paste the following text:

<pre>
apiVersion: v1
kind: Secret
metadata:
  name: "redis-secret"
type: Opaque
data:
  # generate 32 chars long random string, base64 encode it and then double-quote the result string.
  redis-secret: {{ randAlphaNum 8 | b64enc | quote }}
</pre>

You can see we're using the helm macro to generate an 8 byte secret `randAlphaNum 8` here.

Run `helm upgrade redis redis`{{execute}} to create the secret, then `kubectl get secret redis-secret -o jsonpath="{..redis-secret}" | base64 --decode`{{execute}} to see the randomly generated secret.  

Now we have to get that secret into the running pod, so we'll edit `redis/templates/deployment.yaml`{{open}} and add the `--requirepass` parameter to the startup of the container as below.  Replace lines 30 - 41 with the text below.
<pre>
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          env:
          - name: REDIS_PASSWORD
            valueFrom:
              secretKeyRef:
                name: redis-secret
                key: redis-secret
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
</pre>

And again, upgrade the app with `helm upgrade redis redis`{{execute}}.  Since we upgraded, we created a new secret, so we'll need to get the secret again.  This time, we'll set an environment variable to hold it for us with `export REDIS_PASS=$(kubectl get secret redis-secret -o jsonpath="{..redis-secret}" | base64 --decode)`{{execute}}.  There are ways to retain secrets between deployments, that information can be found in the kubernetes [documentation](https://kubernetes.io/docs/concepts/configuration/secret/).  

Now we'll verify that redis is enforcing the password by execing into the pod with `kubectl -it exec $(kubectl get po -o jsonpath="{.items[0].metadata.name}") -- redis-cli`{{execute}}.  But, if you attempt to `get this`{{execute}}, you'll see a `NOAUTH Authentication Required` message.

In order to get back into the database you'll have to pass that to redis-cli.  But first `Ctrl+D` to get out of the redis-cli, then run  `kubectl -it exec $(kubectl get po -o jsonpath="{.items[0].metadata.name}") -- redis-cli -a $REDIS_PASS`{{execute}}.  Once in, you can run `get this`{{execute}} to see the query succeed.

Notice that the previous value that we saved in redis was not retained.  When we upgraded the redis helm charts, we destoyed the pod that we previously had saved data in.  Next, we're going to look at how we can persist the data so that it can survive pod failures.
