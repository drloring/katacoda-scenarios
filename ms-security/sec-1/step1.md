In this course, we're going to look at the open source tool Anchore to see how it inspects and finds vulnerabilities in docker containers.

First, we have to make sure kubernetes is running.  
`kubectl get nodes`{{execute}}
Wait until both nodes are in the `Ready` state.

Again, we need to get helm
`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh && ./get_helm.sh`{{execute}}

Check the version with `helm version`{{execute}}

Now, we're adding the Anchore repo and running the anchore-engine helm charts
`helm repo add anchore https://charts.anchore.io && helm install anchore anchore/anchore-engine`{{execute}}

For this configuration, we'll have to create a PV.  Run `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/pv.yaml`{{execute}} and then apply it with `kubectl apply -f pv.yaml`{{execute}}

When anchore-engine starts it takes some time for it's database to synchronize with the online database.  Run `kubectl get po`{{execute}} to ensure the pods are ready.

If you read the instructions provided by the helm charts, it may appear that the Anchore API is exposed to external clients, but runnning `kubectl get svc`{{execute}} shows that ClusterIP is being used on the anchore-api service and is not available to external clients.

However, we can run an anchore-cli pod and exec into that to explore the Anchore CLI a bit.

First, we export the password kept in the secret. `export ANCHORE_CLI_PASS=$(kubectl get secret --namespace default anchore-anchore-engine-admin-pass -o jsonpath="{.data.ANCHORE_ADMIN_PASSWORD}" | base64 --decode; echo)`{{execute}}

Now, we run the anchore-engine-cli pod `kubectl run -i --tty anchore-cli --restart=Always --image anchore/engine-cli  --env ANCHORE_CLI_USER=admin --env ANCHORE_CLI_PASS=${ANCHORE_CLI_PASS} --env ANCHORE_CLI_URL=http://anchore-anchore-engine-api.default.svc.cluster.local:8228/v1/`{{execute}}

If you don't exec into the pod with the prior command or it appears to hang, press `Ctrl+C` to break and run the following commands.

Get the anchore-cli pod name with this command `kubectl get po`{{execute}}, then exec into the pod with the following command (replacing podname with the name of your pod `kubectl exec -it anchore-cli -- bash`{{execute}}

Once in the Anchore-cli pod, verify anchore is ready
`anchore-cli system status`{{execute}}

Now, we'll add a container image to the list to scan `anchore-cli image add redis`{{execute}}

Run `anchore-cli image list`{{execute}} to view the status of the scan.  When the images says `analyzed`, then it's ready for inspection.  

Once it's complete, we can run a vulnerability assessment of it with `anchore-cli image vuln redis`{{execute}} and we'll see that there are three types of vulnerability assessments available: os, non-os and all.  You can view the details of each by running `anchore-cli image vuln redis os `{{execute}} `anchore-cli image vuln redis non-os`{{execute}} or `anchore-cli image vuln redis all`{{execute}}

We can see if it passed assessment by running `anchore-cli evaluate check redis`{{execute}}.  You'll notice that `Status: fail`.  To get more details, run `anchore-cli evaluate check redis --detail`{{execute}} to get the exact reasons for failure.

Also, notice the policy ID that was used.  To get a look at the policies that are being enforced, run `anchore-cli policy list`{{execute}} to see the matching policy id and `anchore-cli policy describe`{{execute}} to see the policies in effect.

To query all images for specific vulnerabilities, run `anchore-cli query images-by-vulnerability --vulnerability-id CVE-2021-43396`{{execute}}

To get more details on the files in the image run `anchore-cli image content redis os`{{execute}} and `anchore-cli image content redis files`{{execute}}

Next, we're going to see whether the microservice that was used in the past lessons passes vulnerabilty assessment.
