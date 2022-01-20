In this demonstration, we're going to discover a few different uses for the [Open Policy Agent](https://www.openpolicyagent.org/).  First, we'll look at how we can use it to ensure that our kubernetes deployments comply with our system's policies using OPA Gatekeeper, then we'll look at how we can leverage it for Role-Based Access Controls (RBAC) on our services, then we'll see how we can control security aspects of our services including network shares to ensure they meet our system's compliance policies.  And finally, we'll look at how we can enforce our services service contract to further enhance out service's security posture.

There is a great tutorial [here](https://katacoda.com/austinheiman/scenarios/open-policy-agent-gatekeeper-editor) that shows you how to configure OPA Gatekeeper for kubernetes enforcement.  I"m going to borrow from that for a simple explanation of Gatekeeper, for more in-depth knowledge, you are highly encouraged to take that Katacoda course.

To get started, first we have to check if Minikube is Ready with `kubectl get nodes`{{execute}}.

Once it's up and running, we'll install Gatekeeper with`kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.7/deploy/gatekeeper.yaml`{{execute}}

Now, get the `template.yml` we'll use for this demonstration `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/template.yml`{{execute}}.  If you `cat template.yml`{{execute}}, you'll notice the section of rego language which essentially returns a fail (1) if the count of the missing labels is > 0.
<pre>
      rego: |
        package requiredlabels
        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels for object %v : %v", [input.review.object.kind ,missing])
</pre>

Apply this file to your cluster `kubectl apply -f template.yml`{{execute}}

Let's get the yaml file that will actually enforce this template `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/constraint.yml`{{execute}}.  If we look at `constraint.yml` with `cat constraint.yml`{{execute}} we'll see that the CRD that we previously created will be applied to Pods and Deployments.  The parameters at the bottom of the file declare that the owner label must exist on all Pods and Deployments.

<pre>
kind: RequiredLabels
metadata:
  name: resources-must-have-owner
spec:
  match:
    namespace: ["default"]
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
    - apiGroups: ["apps"]
      kinds: ["Deployment"]
  parameters:
    labels:
      - owner
</pre>   

Apply that yaml with `kubectl apply -f constraint.yml --validate=false`{{execute}}

And now we can test it with another file `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/fail.yml`{{execute}}, which simple creates a busybox pod with no labels.  Apply with `kubectl apply -f fail.yml`{{execute}} and notice the message that we saw in the rego expression is displayed.

Edit the file to get the deployment to pass by adding any label to it.  For example:
<pre>
  labels:
    owner: your-name-here
</pre>

In order to proceed, we'll have to delete the `RequiredLabels` CRD that we created previously.  To get the name of the label run `kubectl get requiredlabels`{{execute}} to see the required labels we created and `kubectl delete requiredlabels resources-must-have-owner`{{execute}}

In this next section, we're going to look at how we can protect the web services that we develop against unauthorized access, similar to RBAC.  For this example, we'll be using a standard nginx deployment for the web application.

`curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod +x get_helm.sh && ./get_helm.sh && alias helm=/usr/local/bin/helm`{{execute}}

Let's pull down a sample project with OPA `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/sec-ms-opa.zip && unzip sec-ms-opa.zip && cd sec-ms`{{execute}}.

Install OPA from our tailored OPA helm charts with `helm install opa opa --set authz.enabled=false`{{execute}}.  Then export the node's IP Address with `export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")`{{execute}} and finally `curl $NODE_IP:30123`{{execute}} to verify it's running.

We'll need to compile the web service, but first we`export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64`{{execute}}, then we run `mvn clean install`{{execute}}.

Let's run it in the background with `java -jar target/rest-service-0.0.1-SNAPSHOT.jar --opa.url=$NODE_IP:30123/v1/data/http/authz/allow &`{{execute}}.

Now if we call our web service with `curl localhost:8080/greeting`{{execute}}, we'll see an authorization failure.

Next, we'll create a policy and see the effect...
 
