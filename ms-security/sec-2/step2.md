<h2>Install Argo</h2>

`helm repo add argo https://argoproj.github.io/argo-helm`{{execute}}
`helm install wf argo/argo-workflows`{{execute}}  Need to expose the nodeport and export it
`helm install ev argo/argo-events`{{execute}}


`kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/master/examples/rbac/sensor-rbac.yaml`{{execute}}

`kubectl apply  -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/sensors/webhook.yaml`{{execute}}
`kubectl apply  -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/event-sources/webhook.yaml`{{execute}}

`kubectl port-forward events-eventsource-controller-79664595bb-46x8s 12000:12000 &`{{execute}}
`curl -d '{"message":"ok"}' -H "Content-Type: application/json" -X POST http://127.0.0.1:12000/example`{{execute}}

curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v0.0.0-dev-kc-7/argo-linux-amd64.gz && gunzip argo-linux-amd64.gz && chmod +x argo-linux-amd64 && mv ./argo-linux-amd64 /usr/local/bin/argo && argo version



