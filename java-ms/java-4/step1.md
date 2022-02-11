In this scenario, we're going to automate the deployment of the web service that we previously installed in Kubernetes manually.  We'll be using the Open Source Software [skaffold](https://skaffold.dev) for this.  There is a quick start guide that demonstrates how you automate the deployment of a Go app on their site, but we're going to automate our Spring Boot application with Maven for this demonstration.

To save time, I've saved the solution that was completed previously in a zip file.  We'll gather the solution with `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/java-ms-config-java-1-step-1.zip && unzip java-ms-config-java-1-step-1.zip && cd java-1-step-1`{{execute}}.

Next we have to install helm again:
`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh &&./get_helm.sh && alias helm=/usr/local/bin/helm`{{execute}}
  
And now we can install skaffold `curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 && \
sudo install skaffold /usr/local/bin/`{{execute}}

`skaffold`{{execute}} should now be available on your command line.

Recall that we need to export our JAVA_HOME environment variable to build`export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64`{{execute}}, then we can `mvn clean install`{{execute}}.
  
Now we can run `docker build -t java-ws .`{{execute}} to create our container.

Normally, we would run `helm install`, but instead we're going to see how we hook in skaffold for that job.





