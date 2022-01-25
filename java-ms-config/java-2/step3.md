Next, we're going to deploy a clustered redis database to kubernetes.  But first, we'll need Helm 3 again.

Follow these steps to install helm3:

`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh && ./get_helm.sh && aleas helm=/usr/local/bin/helm`{{execute}}

Bitnami helm charts were created to make deploying a Redis cluster simple, but first we have to add the Bitnami repo `helm repo add bitnami https://charts.bitnami.com/bitnami`{{execute}}

We want to keep our database separated from our service, so we'll create a namespace for it `kubectl create ns db`{{execute}}.

Now, we can run the redis helm chart `helm install redis bitnami/redis -n db --set auth.enabled=false`{{execute}}.  Notice, we specified the `-n db` option to select the db namespace.

Just like the prior course, we'll collect the Dockerfile we created in the Learning Java Microservices scenario `curl -o Dockerfile https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}.

Open the `step-2/Dockerfile`{{open}}, and we'll modify the COPY command <pre class="file" data-filename="step-2/Dockerfile" data-target="insert" data-marker="COPY rest-service.jar svc.jar">COPY target/rest-service-0.0.1-SNAPSHOT.jar svc.jar</pre>

And now we'll build the boot jar again with `mvn clean install`{{execute}}.

Now, run `docker build -t java-ws .`{{execute}} to create the docker image.

The download for step-2 includes the modified helm charts, but we'll review the changes.

Open `step-2/ws/values.yaml`{{open}}, to see we set the `repository: java-ws` and the `tag: "latest"` and, since we want to see the effects of multiple instances, we set `replicaCount: 3`.

We also added some application specific environment variables to pass into the application.  We appended `redishost=redis-master.db.svc.cluster.local` to the end of the file.

We also changed `step-2/ws/templates/deployment.yaml`{{open}} correcting the `containerPort: 8080` and commenting out liveness and readiness probes.

We also added the `spring.redis.host` to the environment variables passed into the container.
<pre>          env:
            - name: "spring.redis.host"
              value: "{{ .Values.redishost }}"
            - name: "spring.redis.password"
              value: "{{ .Values.redispass }}"
</pre>

We also set the service `type: NodePort and set `targetPort: 8080` in to `step-2/ws/templates/service.yaml`{{open}}

Now, we can dry-run the helm install `helm install --dry-run ws ws`{{execute}} to see the deployment.

And, then we can run helm install `helm install ws ws`{{execute}}  and you will notice that the container is getting the redis host from the template from the values.yaml.

Follow the instructions from helm `export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services ws) && export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")`{{execute}}, then `curl $NODE_IP:$NODE_PORT/greeting`{{execute}} repeatedly.  Notice that the Local count is different than the Cached count.  

The bitnami redis cluster installs Persistent Volumes (PV) and Persistent Volume Claims (PVC) as well, so if you uninstall and reinstall helm without removing those PVCs, then you will not see the counter reset as the data is persisted to the PVC. For more information about PV/PVCs, see [https://kubernetes.io/docs/concepts/storage/persistent-volumes/]
