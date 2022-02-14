In this scenario, we're going to automate the deployment of the web service that we previously installed in Kubernetes manually.  We'll be using the Open Source Software [skaffold](https://skaffold.dev) for this.  There is a quick start guide that demonstrates how you automate the deployment of a Go app on their site, but we're going to automate our Spring Boot application with Maven for this demonstration.

To save time, I've saved the solution that was completed previously in a zip file.  We'll gather the solution with `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/java-ms-config-java-1-step-1.zip && unzip java-ms-config-java-1-step-1.zip && cd java-1-step-1`{{execute}}.

Next we have to install helm again:
`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh &&./get_helm.sh && alias helm=/usr/local/bin/helm`{{execute}}
  
And now we can install skaffold `curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 && \
sudo install skaffold /usr/local/bin/`{{execute}}

Run `skaffold version`{{execute}} to ensureit's been installed.

Now, we'll pull in a skaffold.yaml that's been created for this project `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/skaffold.yaml`{{execute}}.

Examining the `java-1-step-1/skaffold.yaml`{{open}} file a bit, we see a couple familiar items:
Skaffold will inspect whether you have a Dockerfile present and helm charts in the directory.  If it finds a dockerfile, it will use that to create the image by default
  build:
      artifacts:
        - image: java-ws 

To deploy, it will use this section:
  deploy:
    helm:
      releases:
      - name: ws
      chartPath: ws
        artifactOverrides:
          image: java-ws 
        imageStrategy:
          helm: {}

Recall that we need to export our JAVA_HOME environment variable to build`export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64`{{execute}}, then we can `mvn clean install`{{execute}}.

Normally, we would run `docker build` and `helm install`, but instead we're going to see how we hook in skaffold for that job.  First, let's check to ensure the cluster is running with `kubectl get nodes`{{execute}} and when you see `Ready` then run `skaffold build`{{execute}}, then `skaffold run`{{execute}}.

Notice that skaffold takes care of building your docker image as well as deploying you application with Helm commands.  Once that's complete, run `skaffold dev`{{execute}}.  `skaffold dev` monitors your files for changes, and when detected, it will either rebuild or redeploy those changes automatically.

Let's try this out by opening `java-1-step-1/ws/values.yaml`{{open}}.  Modify the `replicaCount` and watch skaffold scale your deployment to match the current replica count.  In a new terminal, run `kubectl get pods`{{execute}} to verify the deployment was scaled successfully.

Now open the `java-1-step1/Dockerfile`{{open}} and change the base layer to `openjdk:11`.  After that change, notice the docker image builder starts re-building the docker image and immediately re-deploys the helm charts for you with the new base image.

Next, we're going to see how we can hook in our Java code and Maven `pom.xml`...





