In this step, we're going to persist our data so that it survives restarts in the pods.  But first, we have to exit out of the pod's shell with `Ctrl+D`.

First, we have to create a persistent volume.  We'll start by creating `touch redis/templates/redis-pv.yaml`{{execute}}, then open `redis/templates/redis-pv.yaml`{{open}} and paste the following:
<pre>
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-pv
spec:
  storageClassName: ""
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
</pre>

Next, we'll have to create a claim for the PV we just created by making `touch redis/templates/redis-pvc.yaml`{{execute}}, then open `redis/templates/redis-pvc.yaml`{{open}} and paste the following:
<pre>
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redisdb-pvc
spec:
  storageClassName: ""
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
</pre>

Now run `helm upgrade redis redis`{{execute}} applies those changes.

Now, if we `kubectl get pv`{{execute}} and `kubectl get pvc`{{execute}}, we'll see the PV and PVC we just created.  Notice that the PVC references the PV, so the pod just needs a reference to the PVC.  We'll do that by modifying `redis/templates/deployment.yaml`{{open}} and before the `containers:` tag add the following:
<pre>
      volumes:
        - name: redis-data
          persistentVolumeClaim:
            claimName: redisdb-pvc
</pre>

and before the `ports:` section add:
<pre>
          volumeMounts:
            - name: redis-data
              mountPath: /data
</pre>

and replace `args: ["--requirepass", "$(REDIS_PASS)"]` with `args: ["--requirepass", "$(REDIS_PASS)", "--save", "9", "1"]`.  The redis container assumes that when you override one variable, you want all default behaviors disabled, so we have to add persistence back as an argument.  This parameter will save 9 seconds after 1 key change.

Running `helm upgrade redis redis`{{execute}} again, applies those changes.

We have to export our new password again with `export REDIS_PASS=$(kubectl get secret redis-secret -o jsonpath="{..redis-secret}" | base64 --decode)`{{execute}}, then we can run `kubectl -it exec $(kubectl get po -o jsonpath="{.items[0].metadata.name}") -- redis-cli -a $REDIS_PASS`{{execute}} and `set this that`{{execute}} and exit the pod with `Ctrl+D`.

Now, if scale the deployment down with `kubectl scale --replicas=0 deployment/redis`{{execute}} and back up `kubectl scale --replicas=1 deployment/redis`{{execute}}.  When the pod starts we can run `kubectl -it exec $(kubectl get po -o jsonpath="{.items[0].metadata.name}") -- redis-cli -a $REDIS_PASS`{{execute}} and `get this`{{execute}} to see that the data was persisted.

Up to this point, we've been working with a single redis pod.  Redis was built to support clustering in various configurations.  Next we're going to look into how we can leverage redis clustering for our deployments, ensure to exit out of the pod with `Ctrl+D` before proceeding.

