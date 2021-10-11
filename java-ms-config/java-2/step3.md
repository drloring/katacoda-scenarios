Next, we're going to deploy a clustered redis database to kubernetes.  But first, we'll need Helm 3 again.

Follow these steps to install helm3:

`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3`{{execute}}

`chmod +x get_helm.sh`{{execute}}

`./get_helm.sh`{{execute}}

To deconflict the locally installed helm from the new Helm 3, make an alias called h3 to replace the helm commands.
`alias h3=/usr/local/bin/helm`{{execute}}

`h3 version`{{execute}} should confirm version 3.6.3 or later is installed.

Bitnami helm charts were created to make deploying a Redis cluster simple, but first we have to add the Bitnami repo `h3 repo add bitnami https://charts.bitnami.com/bitnami --set auth.enabled=false`{{execute}}

We want to keep our database separated from our service, so we'll create a namespace for it `kubectl create ns db`{{execute}}.

Now, we can run the redis helm chart `h3 install redis bitnami/redis -n db`{{execute}}.  Notice, we specified the `-n db` option to select the db namespace.

Following the instructions on the command line, run `export REDIS_PASSWORD=$(kubectl get secret --namespace db redis -o jsonpath="{.data.redis-password}" | base64 --decode)`{{execute}} followed by `echo $REDIS_PASSWORD`{{export}} to view the redis password.  We'll use that password when we deploy our application to access the redis database in kubernetes.

Just like the prior course, we'll collect the Dockerfile we created in the Learning Java Microservices scenario `curl -o Dockerfile https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}.

We'll have to modify the COPY command <pre class="file" data-filename="gs-rest-service/complete/Dockerfile" data-target="insert" data-marker="COPY rest-service.jar svc.jar">COPY build/libs/rest-service-0.0.1-SNAPSHOT.jar svc.jar</pre>

And now we'll build the boot jar again with `./gradlew bootJar`{{execute}}.

Now, run `docker build -t java-ws .`{{execute}} to create the docker image.

`h3 create ws`{{execute}} will create a simple helm chart called ws (for web service).

Now we have all the charts to install an application.  The default installation simply installs redis , so we're going to change that for our service.

Edit `gs-rest-service/complete/ws/values.yaml`{{open}}, change the repository to java-ws <pre class="file" data-filename="gs-rest-service/complete/ws/values.yaml" data-target="insert" data-marker="  repository: nginx">  repository: java-ws</pre>

Edit `gs-rest-service/complete/ws/values.yaml`{{open}}, change the repository to java-ws <pre class="file" data-filename="gs-rest-service/complete/ws/values.yaml" data-target="insert" data-marker="  repository: nginxreplicaCount: 2">replicaCount: 3</pre>
And this time, we're going to create multiple replicas so that we can confirm that we're not using local storage. 

Now, change the name to latest <pre class="file" data-filename="gs-rest-service/complete/ws/values.yaml" data-target="insert" data-marker="  tag: &#x22&#x22">  tag: "latest"</pre>

And change `gs-rest-service/complete/ws/templates/deployment.yaml`{{open}} as well <pre class="file" data-filename="gs-rest-service/complete/ws/templates/deployment.yaml" data-target="insert" data-marker="              containerPort: 80">              containerPort: 8080</pre>

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

And now we're going to add some application specific environment variables to pass into the application.  First, we're going to add them to `gs-rest-service/complete/ws/values.yaml`{{open}} <pre class="file" data-filename="gs-rest-service/complete/ws/values.yaml" data-target="append">redishost=redis-master.db.svc.cluster.local</pre>

Then, paste the following code immediately after `protocol: TCP` line in `gs-rest-service/complete/ws/templates/deployment.yaml`{{open}}
<pre>          env:
            - name: "spring.redis.host"
              value: "{{ .Values.redishost }}"</pre>

Now, we can dry-run the helm install `helm install --set serverport=9999 --dry-run ws ws`{{execute}}  and you will notice that the container is getting the server.port from the command line environment variable and the template from the values.yaml.

Before we install it, let's open up the NodePort again so we don't have to port-forward to get access.  Start, by opening the `gs-rest-service/complete/ws/templates/service.yaml`{{open}} file.  Then change the targetPort to <pre class="file" data-filename="gs-rest-service/complete/ws/templates/service.yaml" data-target="insert" data-marker="      targetPort: http">      targetPort: 8080</pre> to reflect our pod's exposed port. 

Then change the NodePort in values.yaml <pre class="file" data-filename="gs-rest-service/complete/ws/values.yaml" data-target="insert" data-marker="  type: ClusterIP">  type: NodePort</pre>

Now, we can the helm install `helm install ws ws`{{execute}}  and you will notice that the container is getting the redis host from the template from the values.yaml.

Follow the instructions from helm and export the NODE_PORT and NODE_IP, then `curl $NODE_IP:$NODE_PORT/greeting`{{execute}} repeatedly.  If you are seeing duplicate counters and non-incremental updates, that means your docker image was built with the local in-memory counter.  If you were to change the counter.get() to count in `GreetingController.java`, rebuild the jar and the docker container and scale the deployment with `kubectl scale deployments/ws --replicas 0` and `kubectl scale deployments/ws --replicas 3` you will see the count monotonically increasing.  

The bitnami redis cluster installs Persistent Volumes (PV) and Persistent Volume Claims (PVC) as well, so if you uninstall and reinstall helm without removing those PVCs, then you will not see the counter reset as the data is persisted to the PVC. 
