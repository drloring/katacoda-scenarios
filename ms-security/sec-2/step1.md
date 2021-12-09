In this course, we're going to integrate the open source tool Anchore into a DevSecOps pipeline and configure it to scan our microservices.

First, we have to make sure kubernetes is running.  
`kubectl get nodes`{{execute}}
Wait until both nodes are in the `Ready` state. NOTE: this may take several minutes.

For this course, we're using GitLab, the public CI/CD pipeline runner.  We're going to register our kubernetes cluster as a runner.  To do that, we have to pull down a yaml file to pass into helm `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/gitlab-values.yml`{{execute}}.  If we `cat gitlab-values.yml`{{execute}}, we see how we're registering our gitlab runner with gitlab
<pre>
gitlabUrl: https://gitlab.com
runnerRegistrationToken: zF3q34LuyaV8yF44nyz8
</pre>

Next, we'll pull in the gitlab-ci.yml file with `wget https://gitlab.com/drloring/katacoda-resources/-/raw/main/.gitlab-ci.yml`{{execute}}.  If we look at that `cat .gitlab-ci.yml`{{execute}}, we'll see the part of the script that performs the image scanning
<pre>
anchore-scan-job:
  stage: scan
  image:
    name: ubuntu

  script:
    - |
      apt update
      apt-get -y install curl
      
      curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
      curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin

      syft packages redis:latest -o json > out.json
      grype sbom:./out.json --fail-on critical
    - |
</pre>

As we saw in the prior course, Anchore is moving away from anchore-engine, towards the stand-alone tools syft and grype to support lightweight CI/CD pipeline tools like GitLab.

Once we've verified that both nodes are ready, we have to install the gitlab-runnerby adding their helm repo `helm repo add gitlab https://charts.gitlab.io`{{execute}}, then installing the chart `helm install gitlab-runner -f ./gitlab-values.yml gitlab/gitlab-runner`{{execute}}

Now that we have it installed, we can see the executor with `kubectl get po`{{execute}}, you'll see `gitlab-runner-gitlab-runner` pod.  The executor pod actually creates pods to execute the scan.  If we had installed anchore-engine, we could have used that to verify our internal docker images, but with syft and grype, there's no need to have that installed in your cluster.  

Once that pod is ready, we can trigger the pipeline with run `curl -X POST -F token=91c93dac274b0539a99e19a944f776 -F ref=main https://gitlab.com/api/v4/projects/31966501/trigger/pipeline`{{execute}} to trigger the build, then run `kubectl get po`{{execute}} repeatedly until you see a `runner---` pod in the cluster.  Once those pods are deleted, the job status is available at `https://gitlab.com/drloring/katacoda-resources/-/pipelines`{{execute}}

<h2>GitLab Setup</h2>
`helm repo add gitlab https://charts.gitlab.io/`{{execute}}
`helm install gitlab gitlab/gitlab --timeout 600s -f https://gitlab.com/gitlab-org/charts/gitlab/raw/master/examples/values-minikube-minimum.yaml`{{execute}}



<br>
<br>
<br>
<br>
<br>
<br>
<br>

For a minimal DevSecOps pipeline, we're going to use Argo-CD.

First, we'll add the argo repo to helm `helm repo add argo https://argoproj.github.io/argo-helm`{{execute}}
Next, we install just the argo-cd project `helm install cd argo/argo-cd`{{execute}}

Port-forward the API server so we can access it from the CLI with `kubectl port-forward service/cd-argocd-server -n default 8080:443 &`{{execute}}

Let's save the password in an environment variable `export ARGO_PASSWORD=$(kubectl -n default get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)`{{execute}}

Now, let's get the Argo-CD CLI `curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64 
chmod +x /usr/local/bin/argocd`{{execute}} 

`argocd login localhost:8080 --insecure --password $ARGO_PASSWORD`{{execute}} and when prompted, enter `admin`{{execute}} for the username.

helm repo add incubator https://charts.helm.sh/incubator
helm install incubator/gogs


curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb"

dpkg -i gitlab-runner_amd64.deb

`echo [[HOST_SUBDOMAIN]]-8080-[[KATACODA_HOST]].[[KATACODA_DOMAIN]]`{{execute}}







Again, we need to get helm
`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh && ./get_helm.sh`{{execute}}

Check the version with `helm version`{{execute}}

For this configuration, we'll have to create a PV.  Run `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/pv.yaml`{{execute}} and then apply it with `kubectl apply -f pv.yaml`{{execute}}

Now, we're adding the Anchore repo and running the anchore-engine helm charts
`helm repo add anchore https://charts.anchore.io && helm install anchore anchore/anchore-engine`{{execute}}

When anchore-engine starts it takes some time for it's database to synchronize with the online database.  Run `kubectl get po`{{execute}} to ensure the pods are ready.

If you read the instructions provided by the helm charts, it may appear that the Anchore API is exposed to external clients, but runnning `kubectl get svc`{{execute}} shows that ClusterIP is being used on the anchore-api service and is not available to external clients.

However, we can run an anchore-cli pod and exec into that to explore the Anchore CLI a bit.

First, we export the password kept in the secret. `export ANCHORE_CLI_PASS=$(kubectl get secret --namespace default anchore-anchore-engine-admin-pass -o jsonpath="{.data.ANCHORE_ADMIN_PASSWORD}" | base64 --decode; echo)`{{execute}}

Now, we run the anchore-engine-cli pod `kubectl run -it anchore-cli --restart=Always --image anchore/engine-cli  --env ANCHORE_CLI_USER=admin --env ANCHORE_CLI_PASS=${ANCHORE_CLI_PASS} --env ANCHORE_CLI_URL=http://anchore-anchore-engine-api.default.svc.cluster.local:8228/v1/`{{execute}}

It takes a while, but if you don't enter the anchore-cli command line, then `Ctrl+C` and wait until the pod is running `kubectl get po`{{execute}}, then exec into the pod with the following command `kubectl exec -it anchore-cli -- bash`{{execute}}

Once in the Anchore-cli pod, verify anchore is ready
`anchore-cli system status`{{execute}}

Now, we'll add a container image to the list to scan `anchore-cli image add redis`{{execute}}

Run `anchore-cli image list`{{execute}} to view the status of the scan.  When the images says `analyzed`, then it's ready for inspection.  

Once it's complete, we can see if it passed assessment by running `anchore-cli evaluate check redis`{{execute}}.  You'll notice that `Status: pass`.  To get more details, run `anchore-cli evaluate check redis --detail`{{execute}} to see if there were any warnings detected.

We can also run a vulnerability assessment with `anchore-cli image vuln redis`{{execute}} and we'll see that there are three types of vulnerability assessments available: os, non-os and all.  You can view the details of each by running `anchore-cli image vuln redis os `{{execute}} `anchore-cli image vuln redis non-os`{{execute}} or `anchore-cli image vuln redis all`{{execute}}

Next, we're going to see whether the microservice that was used in the past lessons passes vulnerabilty assessment.
