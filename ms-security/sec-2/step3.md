In this demonstration, we're going to discover a few different uses for the [Open Policy Agent](https://www.openpolicyagent.org/).  First, we'll look at how we can use it to ensure that our kubernetes deployments comply with our system's policies using OPA Gatekeeper, then we'll look at how we can leverage it for Role-Based Access Controls (RBAC) on our services, then we'll see how we can control security aspects of our services including network shares to ensure they meet our system's compliance policies.  And finally, we'll look at how we can enforce our services service contract to further enhance out service's security posture.

There is a great tutorial [here](https://katacoda.com/austinheiman/scenarios/open-policy-agent-gatekeeper-editor) that shows you how to configure OPA Gatekeeper for kubernetes enforcement.  I"m going to borrow from that for a simple explanation of Gatekeeper, for more in-depth knowledge, you are highly encouraged to take that Katacoda course.

`kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.7/deploy/gatekeeper.yaml`{{execute}}

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

Apply that yaml with `kubectl apply -f constraint.yml --validate=false`{{execute}}

And now we can test it with another file `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/fail.yml`{{execute}}, which simple creates a busybox pod with no labels.  Apply with `kubectl apply -f fail.yml`{{execute}} and notice the message that we saw in the rego expression is displayed.

Edit the file to get the deployment to pass by adding any label to it.

In this next section, we're going to look at how we can protect the web services that we develop against unauthorized access, similar to RBAC.  For this example, we'll be using a standard nginx deployment for the web application.

First, we need to install another OPA service called `kube-mgmt`.  Run the following:
`helm repo add opa https://open-policy-agent.github.io/kube-mgmt/charts`{{execute}}
`helm repo update`{{execute}}
`helm upgrade -i -n opa --create-namespace opa opa/opa`{{execute}}

Now that we have that installed, we can install a simple nginx web server using the Bitnami helm charts `helm repo add bitnami https://charts.bitnami.com/bitnami`{{execute}} and then `helm install ws bitnami/nginx`{{execute}}.  Once that's installed, verify you can access it by `curl HERE`.

Next, we'll create a policy and see the effect...
 
