<h2>Install Argo</h2>
In order to integrate our security scanning into a CI/CD pipeline, we'll need a pipeline up and running. Argo CD and Workflows provides a lightweight CI/CD pipeline, however there is a bit of setup required first.

We'll need to add the Argo repo and install Argo Workflows with `helm repo add argo https://argoproj.github.io/argo-helm`{{execute}} and `helm install wf argo/argo-workflows --set server.serviceType=NodePort --set server.serviceNodePort=30123`{{execute}}  Notice that we overrode the server type and node port address so that we can connect from the command line.

Next, we'll install the Argo CLI so that we can administer the server.  Run `curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v0.0.0-dev-kc-7/argo-linux-amd64.gz && gunzip argo-linux-amd64.gz && chmod +x argo-linux-amd64 && mv ./argo-linux-amd64 /usr/local/bin/argo`{{execute}} to install the CLI.


Argo requires access tokens to be created in order to securely access the workflow, for convenience, we'll reuse the Argo events SA that we'll use later `kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/master/examples/rbac/sensor-rbac.yaml`{{execute}}

Now that we've set up the service account, let's set some environment variables for the CLI with `export ARGO_SERVER=127.0.0.1:30123`{{execute}}, `export ARGO_SECURE=false`{{execute}}, `export  ARGO_INSECURE_SKIP_VERIFY=true`{{execute}}.

Now we export the `SECRET` and use it to create the `ARGO_TOKEN` with `export SECRET=$(kubectl get sa operate-workflow-sa -o=jsonpath='{.secrets[0].name}')`{{execute}} and `export ARGO_TOKEN="Bearer $(kubectl get secret $SECRET -o=jsonpath='{.data.token}' | base64 --decode)"`{{execute}}

Now that we're setup, let's verify our Argo CLI can connect to our server.  Run `argo submit https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml --watch`{{execute}}.

When that completes, you can see the job with `argo list`{{execute}} and you can view the output of the pods by getting the pod name with `kubectl get po | grep hello`{{execute}} and `kubectl logs <<PODNAME>> main`.

Now that workflows is working we're going to integrate Argo Events to and configure a webhook to simulate a CI/CD event in production

First, we install the helm charts for Argo Events with `helm install ev argo/argo-events`{{execute}}

Next, we'll install the webhook with `kubectl apply  -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/sensors/webhook.yaml`{{execute}}

Now, we'll add an Event Source and expose it so we can curl it from the command-line with `kubectl apply  -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/event-sources/webhook.yaml`{{execute}} and `kubectl port-forward events-eventsource-controller-79664595bb-46x8s 12000:12000 &`{{execute}}

Now, we can curl the Event Source to verify operations `curl -d '{"message":"ok"}' -H "Content-Type: application/json" -X POST http://127.0.0.1:12000/example`{{execute}}



<h3>JIC</h3>


so we'll create the role first `kubectl create role argo-cli --verb=list,update --resource=workflows.argoproj.io`{{execute}}, then we'll create the service account `kubectl create sa argo-cli`{{execute}}, next we'll make the rolebinding `kubectl create rolebinding jenkins --role=argo-cli --serviceaccount=argo:argo-cli`{{execute}}

