In this section, we're going to look at how we can protect the web services that we develop against unauthorized access, similar to RBAC.  For this example, we'll be using our previously developed spring boot application.

`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh && ./get_helm.sh && alias helm=/usr/local/bin/helm`{{execute}}

Let's pull down a sample project with OPA `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/sec-ms-opa.zip && unzip sec-ms-opa.zip && cd sec-ms`{{execute}}.

Install OPA from our tailored OPA helm charts with `helm install opa opa --set authz.enabled=false`{{execute}}.  Then export the node's IP Address with `export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")`{{execute}} and finally `curl $NODE_IP:30123`{{execute}} to verify it's running.

Let's see what policies are loaded initially with `curl $NODE_IP:30123/v1/policies`{{execute}} and see that our policies are empty.

In order to test our web service with it, we'll need to compile it, but first we `export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64`{{execute}}, then we run `mvn clean install`{{execute}}.

Let's run it in the background with `java -jar target/rest-service-0.0.1-SNAPSHOT.jar --opa.url=http://$NODE_IP:30123/v1/data/http/authz/allow &`{{execute}}.

Now if we call our web service with `curl localhost:8080/greeting`{{execute}}, we'll see an authorization failure `"status":403,"error":"Forbidden"`.  The default behavior is to fail authorization if no policies are loaded.

Next, we're going to add a policy.  In the zip file, there's a file that allows all access called `allow-all.rego`. It looks like this:
<pre>
package http.authz

allow = true
</pre>

We'll add that file to our OPA server via `curl --location --request PUT $NODE_IP:30123/v1/policies/http/authz --header 'Content-Type: text/plain' --data-binary @allow-all.rego`{{execute}}

Then we'll verify it took by looking at the policies in OPA again `curl $NODE_IP:30123/v1/policies`{{execute}} and see that our policies was added.
 
Now if we call our web service again with `curl localhost:8080/greeting`{{execute}}, we'll see we get the greeting response which means it passed the lenient `allow = true` policy in OPA.

So, that's an example of how you can externalize your security policies in Kubernetes with Open Policy Agent.
