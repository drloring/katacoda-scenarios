In this section, we're going to look at how we can protect the web services that we develop against unauthorized access, similar to RBAC.  For this example, we'll be using our a modified version of our previously developed spring boot application.

`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh && ./get_helm.sh && alias helm=/usr/local/bin/helm`{{execute}}

Pull down a sample project with OPA `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/sec-ms-opa.zip && unzip sec-ms-opa.zip && cd sec-ms`{{execute}}.

Let's explore some of the changes we've added to the project to incorporate OPA policy decisions into Spring Boot.  First, you'll see a new file `sec-ms/src/main/java/com/example/restservice/WebSecurityConfig.java`{{open}}.  If you open that, you can see that `AccessDecisionManager` has been delegated to `OPAVoter`.
<pre>
List<AccessDecisionVoter<? extends Object>> decisionVoters = Arrays.asList(new OPAVoter(opaUrl));
</pre>
Also, notice that we're passing in the URL to the OPA decision engine as a gonfigurable value
<pre>
@Value("${opa.url}")
private String opaUrl;
</pre>
Recall that this allows us to modify the value based on runtime environments, e.g. dev vs production.

Now that we see how we're delegating authority to OPA, it's time to install OPA from our tailored OPA helm charts with `helm install opa opa --set authz.enabled=false`{{execute}}.  Then export the node's IP Address with `export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")`{{execute}} and finally `curl $NODE_IP:30123`{{execute}} to verify it's running.

Let's see what policies are loaded initially with `curl $NODE_IP:30123/v1/policies`{{execute}} and see that our policies are empty.

In order to test our web service with it, we'll need to compile it, but first we `export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64`{{execute}}, then we run `mvn clean install`{{execute}}.

Let's run it in the background with `java -jar target/rest-service-0.0.1-SNAPSHOT.jar --opa.url=http://$NODE_IP:30123/v1/data/http/authz/allow &`{{execute}}.

Now if we call our web service with `curl localhost:8080/greeting`{{execute}}, we'll see an authorization failure `"status":403,"error":"Forbidden"`.  The default behavior is to fail authorization if no policies are loaded.

Next, we're going to add a policy.  In the zip file, there's a file that allows all access called `sec-ms/allow-all.rego`{{open}}. The policy is pretty simple and simply allows all access.  Recall, the `WebSecurityConfig` class is calling `http://$NODE_IP:30123/v1/data/http/authz/allow` to see whether this call is authorized.  By loading the rego file that allows all access, this should result in the resumption of our web service's business logic, which is to display Hello World. 

We'll add that file to our OPA server via `curl --location --request PUT $NODE_IP:30123/v1/policies/http/authz --header 'Content-Type: text/plain' --data-binary @allow-all.rego`{{execute}}

Then we'll verify it took by looking at the policies in OPA again `curl $NODE_IP:30123/v1/policies`{{execute}} and see that our policies was added.
 
Now if we call our web service again with `curl localhost:8080/greeting`{{execute}}, we'll see we get the greeting response which means it passed the lenient `allow = true` policy in OPA.  If you see `Hello World`, the policy allowed the request to our web service.

If you're interested, modify the allow-all.rego file to `allow = false` and reapply with `curl --location --request PUT $NODE_IP:30123/v1/policies/http/authz --header 'Content-Type: text/plain' --data-binary @allow-all.rego`{{execute}} and verify that the we get a 403 error accessing the web service.

So, that's an example of how you can externalize your security policies in Kubernetes with Open Policy Agent.
