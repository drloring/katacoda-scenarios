In this demonstration, we're going to discover a few different uses for the [Open Policy Agent](https://www.openpolicyagent.org/).  First, we'll look at how we can use it to ensure that our kubernetes deployments comply with our system's policies using OPA Gatekeeper, then we'll look at how we can leverage it for Role-Based Access Controls (RBAC) on our services, then we'll see how we can control security aspects of our services including network shares to ensure they meet our system's compliance policies.  And finally, we'll look at how we can enforce our services service contract to further enhance out service's security posture.

There is a great tutorial [here](https://katacoda.com/austinheiman/scenarios/open-policy-agent-gatekeeper-editor) that shows you how to configure OPA Gatekeeper for kubernetes enforcement.  I"m going to borrow from that for a simple explanation of Gatekeeper, for more in-depth knowledge, you are highly encouraged to take that Katacoda course.

`kubectl create clusterrolebinding cluster-admin-binding \
    --clusterrole cluster-admin \
    --user <YOUR USER NAME>`{{execute}}

`kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.7/deploy/gatekeeper.yaml`{{execute}}

Now, get the `template.yml` we'll use for this demonstration `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/template.yml`{{execute}}.  If you 'cat template.yml`{{execute}}, you'll notice the section of rego language which essentially returns a fail (1) if the count of the missing labels is > 0.
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

Apply this file to your cluster `'kubectl apply -f template.yml`{{execute}}

Let's get the yaml file that will actually enforce this template `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/constraint.yml`{{execute}}.  If we look at `constraint.yml` with `cat constraint.yml`{{execute}} we'll see that the CRD that we previously created will be applied to Pods and Deployments.
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

Apply that yaml with `kubectl apply -f constraint.yml`{{execute}}

And now we can test it with another file `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/fail.yml`{{execute}}, which simple creates a busybox pod with no labels.  Apply with `kubectl apply -f fail.yml`{{execute}} and notice the message that we saw in the rego expression is displayed.

Edit the file to get the deployment to pass by adding any label to it.

In this next section, we're going to pull in the web service that we've been using for these courses with `curl -o rest-service.jar https://raw.githubusercontent.com/drloring/katacoda-resources/main/rest-service-0.0.1-SNAPSHOT.jar`{{execute}} and the Dockerfile `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}

Now, we'll build the docker image with `docker build -t java-ws .`{{execute}}

To save time, we'll pull down the helm charts for the service we've previously created `curl -o step2.zip https://raw.githubusercontent.com/drloring/katacoda-resources/main/java-ms-config-java-2-step-2.zip && unzip step-2.zip && cd step-2`{{execute}}
  
Now, we'll install the application with `helm install ws ws`{{execute}}.  Now, we're going to create a policy and modify our curl command to show how we can limit who can access certain operations on our web service.
 
