kubectl get nodes

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3

ls

chmod +x get_helm.sh 

get_helm.sh

./get_helm.sh 

#alias h3=/usr/local/bin/helm

helm version

helm repo add anchore https://charts.anchore.io

helm install anchore anchore/anchore-engine

export ANCHORE_CLI_PASS=$(kubectl get secret --namespace default anchore-anchore-engine-admin-pass -o jsonpath="{.data.ANCHORE_ADMIN_PASSWORD}" | base64 --decode; echo)

kubectl run -i --tty anchore-cli --restart=Always --image anchore/engine-cli  --env ANCHORE_CLI_USER=admin --env ANCHORE_CLI_PASS=${ANCHORE_CLI_PASS} --env ANCHORE_CLI_URL=http://anchore-anchore-engine-api.default.svc.cluster.local:8228/v1/

kubectl get po

kubectl exec -it <<podname>> -- bash


anchore-cli system status
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

