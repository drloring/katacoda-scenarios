In this demonstration, we're going to discover a couple different uses for the [Open Policy Agent](https://www.openpolicyagent.org/).  First, we'll look at how we can use it to ensure that our kubernetes deployments comply with our system's policies using OPA Gatekeeper, then we'll look at how we can leverage it for access control to our services with kube-mgmt.

There is a great tutorial [here](https://katacoda.com/austinheiman/scenarios/open-policy-agent-gatekeeper-editor) that shows you how to configure OPA Gatekeeper for kubernetes enforcement.  I"m going to borrow from that for a simple explanation of Gatekeeper, for more in-depth knowledge, you are highly encouraged to take that Katacoda course.

To get started, first we have to check if Minikube is Ready with `kubectl get nodes`{{execute}}.

Once it's up and running, we'll install Gatekeeper with`kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.7/deploy/gatekeeper.yaml`{{execute}}

Now, get the `template.yml` and `constraint.yml` we'll use for this demonstration `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/template.yml && wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/constraint.yml`{{execute}}.  If we open `template.yml`{{open}}, we'll notice the section of rego language which essentially returns a fail (1) if the count of the missing labels is > 0.
<pre>
provided := {label | input.review.object.metadata.labels[label]}
required := {label | label := input.parameters.labels[_]}
missing := required - provided
count(missing) > 0
msg := sprintf("you must provide labels for object %v : %v", [input.review.object.kind ,missing])
</pre>
Also, notice that this is applied to the kubernetes admission controller, which is responsible to allowing admission to the cluster.  
<pre>
- target: admission.k8s.gatekeeper.sh
</pre>
So, when the rule is violated, the resource is not allowed to be added to the cluster

Apply this file to your cluster `kubectl apply -f template.yml`{{execute}}

If we open `constraint.yml`{{open}} we'll see that the CRD that we previously created will be applied to Pods and Deployments.  The parameters at the bottom of the file declare that the owner label must exist. 
<pre>
  parameters:
    labels:
      - owner
</pre>   
And we'll also see that this is applied to all Pods and Deployments
<pre>
spec:
  match:
    namespace: ["default"]
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
    - apiGroups: ["apps"]
      kinds: ["Deployment"]

</pre>

Apply that yaml with `kubectl apply -f constraint.yml --validate=false`{{execute}}

And now we can test it with another file `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/fail.yml`{{execute}}, which simple creates a busybox pod with no labels.  Apply with `kubectl apply -f fail.yml`{{execute}} and notice the message that we saw in the rego expression is displayed.

Edit `fail.yml`{{open}} to get the deployment to pass by adding any label to it.  For example:
<pre>
  labels:
    owner: your-name-here
</pre>

In order to proceed, we'll have to delete the `RequiredLabels` CRD that we created previously.  To get the name of the label run `kubectl get requiredlabels`{{execute}} to see the required labels we created and `kubectl delete requiredlabels resources-must-have-owner`{{execute}}
