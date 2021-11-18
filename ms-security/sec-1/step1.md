In this course, we're going to look at the open source tool Anchore to see how it inspects and finds vulnerabilities in docker containers.

First, we have to make sure kubernetes is running.  
`kubectl get nodes`{{execute}}
Wait until the minikube node is in the `Ready` state.

Again, we need to get helm
`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh && ./get_helm.sh`{{execute}}

Check the version with `helm version`{{execute}}

Now, we're adding the Anchore repo and running the anchore-engine helm charts
`helm repo add anchore https://charts.anchore.io && helm install anchore anchore/anchore-engine`{{execute}}

When anchore-engine starts it takes some time for it's database to synchronize with the online database.  Run `kubectl get po`{{execute}} to ensure the pods are ready.

If you read the instructions provided by the helm charts, it may appear that the Anchore API is exposed to external clients, but runnning `kubectl get svc`{{execute}} shows that ClusterIP is being used on the anchore-api service and is not available to external clients.

However, we can run an anchore-cli pod and exec into that to explore the Anchore CLI a bit.

First, we export the password kept in the secret. `export ANCHORE_CLI_PASS=$(kubectl get secret --namespace default anchore-anchore-engine-admin-pass -o jsonpath="{.data.ANCHORE_ADMIN_PASSWORD}" | base64 --decode; echo)`{{execute}}

Now, we run the anchore-engine-cli pod `kubectl run -i --tty anchore-cli --restart=Always --image anchore/engine-cli  --env ANCHORE_CLI_USER=admin --env ANCHORE_CLI_PASS=${ANCHORE_CLI_PASS} --env ANCHORE_CLI_URL=http://anchore-anchore-engine-api.default.svc.cluster.local:8228/v1/`{{execute}}

If you don't exec into the pod with the prior command or it appears to hang, press `Ctrl+C` to bread and run the following commands.

Get the anchore-cli pod name with this command `kubectl get po`{{execute}}, then exec into the pod with the following command (replacing podname with the name of your pod `kubectl exec -it <<podname>> -- bash`{{execute}}

Once in the Anchore-cli pod, verify anchore is ready
`anchore-cli system status`{{execute}}

Now, we'll add a container image to the list to scan `anchore-cli image add redis`{{execute}}

Run `anchore-cli image list`{{execute}} to view the status of the scan.  

Once it's complete, we can run a vulnerability assessment of it with `anchore-cli image vuln redis`{{execute}}

anchore-cli image add docker.io/library/debian:latest
anchore-cli image list
anchore-cli evaluate check docker.io/library/debian:latest
anchore-cli image vuln docker.io/library/debian:latest os
anchore-cli image vuln docker.io/library/debian:latest
anchore-cli image content docker.io/library/debian:latest os
anchore-cli image content docker.io/library/debian:latest files
anchore-cli evaluate check docker.io/library/debian:latest --detail

apt-get update

apt-get install python-pip

install anchorecli --ignore-installed

export PATH="$HOME/.local/bin/:$PATH"

anchore-cli

export ANCHORE_CLI_USER=admin

export ANCHORE_CLI_PASS=$(kubectl get secret --namespace default anchore-anchore-engine-admin-pass -o jsonpath="{.data.ANCHORE_ADMIN_PASSWORD}" | base64 --decode; echo)

export ANCHORE_CLI_URL=http://anchore-anchore-engine-api.default.svc.cluster.local:8228/v1/



A quick primer on using the Anchore Engine CLI follows. For more info see: https://github.com/anchore/anchore-engine/wiki/Getting-Started

View system status:

    anchore-cli system status

Add an image to be analyzed:

    anchore-cli image add <imageref>

List images and see the analysis status (not_analyzed initially):

    anchore-cli image list

Once the image is analyzed you'll see status change to 'analyzed'. This may take some time on first execution with a new database because
the system must first do a CVE data sync which can take several minutes. Once complete, the image will transition to 'analyzing' state.

When the image reaches 'analyzed' state, you can view policy evaluation output with:

    anchore-cli evaluate check <imageref>

List CVEs found in the image with:

    anchore-cli image vuln <imageref> os

List OS packages found in the image with:
    anchore-cli image content <imageref> os

List files found in the image with:
    anchore-cli image content <imageref> files
	
	
	
	    8  anchore-cli
    9  anchore-cli system status
   10  anchore-cli image add docker.io/library/debian:latest
   11  anchore-cli image list
   12  anchore-cli evaluate check docker.io/library/debian:latest
   13  anchore-cli image vuln docker.io/library/debian:latest os
   14  anchore-cli image vuln docker.io/library/debian:latest
   15  anchore-cli image content docker.io/library/debian:latest os
   16  anchore-cli image content docker.io/library/debian:latest files
   17  anchore-cli evaluate check docker.io/library/debian:latest --detail

