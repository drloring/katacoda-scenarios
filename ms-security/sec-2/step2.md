<h2>Install Argo</h2>

`helm repo add argo https://argoproj.github.io/argo-helm`{{execute}}
`helm install wf argo/argo-workflows --set server.serviceType=NodePort --set server.serviceNodePort=30123`{{execute}}  Need to expose the nodeport and export it

`export ARGO_SERVER=127.0.0.1:30123`{{execute}}
`export ARGO_SECURE=false`{{execute}}
`export  ARGO_INSECURE_SKIP_VERIFY=true`{{execute}}

`kubectl create role jenkins --verb=list,update --resource=workflows.argoproj.io`{{execute}}
`kubectl create sa jenkins`{{execute}}
`kubectl create rolebinding jenkins --role=jenkins --serviceaccount=argo:jenkins`{{execute}}
`export SECRET=$(kubectl get sa jenkins -o=jsonpath='{.secrets[0].name}')`{{execute}}
`export ARGO_TOKEN="Bearer $(kubectl get secret $SECRET -o=jsonpath='{.data.token}' | base64 --decode)"`{{execute}}
`argo submit https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml --watch`{{execute}}
`argo list`{{execute}}

`helm install ev argo/argo-events`{{execute}}

`kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/master/examples/rbac/sensor-rbac.yaml`{{execute}}

`kubectl apply  -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/sensors/webhook.yaml`{{execute}}
`kubectl apply  -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/event-sources/webhook.yaml`{{execute}}

`kubectl port-forward events-eventsource-controller-79664595bb-46x8s 12000:12000 &`{{execute}}
`curl -d '{"message":"ok"}' -H "Content-Type: application/json" -X POST http://127.0.0.1:12000/example`{{execute}}

curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v0.0.0-dev-kc-7/argo-linux-amd64.gz && gunzip argo-linux-amd64.gz && chmod +x argo-linux-amd64 && mv ./argo-linux-amd64 /usr/local/bin/argo && argo version



