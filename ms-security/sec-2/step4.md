In this section, we're going to look at how we can protect the web services that we develop against unauthorized access, similar to RBAC.  For this example, we'll be using our previously developed spring boot application.

`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh && ./get_helm.sh && alias helm=/usr/local/bin/helm`{{execute}}

Let's pull down a sample project with OPA `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/sec-ms-opa.zip && unzip sec-ms-opa.zip && cd sec-ms`{{execute}}.

Install OPA from our tailored OPA helm charts with `helm install opa opa --set authz.enabled=false`{{execute}}.  Then export the node's IP Address with `export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")`{{execute}} and finally `curl $NODE_IP:30123`{{execute}} to verify it's running.

We'll need to compile the web service, but first we`export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64`{{execute}}, then we run `mvn clean install`{{execute}}.

Let's run it in the background with `java -jar target/rest-service-0.0.1-SNAPSHOT.jar --opa.url=http://$NODE_IP:30123/v1/data/http/authz/allow &`{{execute}}.

Now if we call our web service with `curl localhost:8080/greeting`{{execute}}, we'll see an authorization failure.

In the zip file, there's a file that allows all access called `allow-all.rego`. It looks like this:
<pre>
package http.authz

allow = true
</pre>

We'll add that file to our OPA server via `curl --location --request PUT $NODE_IP:30123/v1/policies/http/authz --header 'Content-Type: text/plain' --data-binary @allow-all.rego`{{execute}}
 
Now if we call our web service again with `curl localhost:8080/greeting`{{execute}}, we'll see we get the greeting response.

So, that's an example of how you can externalize your security policies in Kubernetes with Open Policy Agent.
