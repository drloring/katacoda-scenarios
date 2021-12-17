In this demonstration, we're going to discover a few different uses for the [Open Policy Agent](https://www.openpolicyagent.org/).  First, we'll look at how we can use it to ensure that our kubernetes deployments comply with our system's policies using OPA Gatekeeper, then we'll look at how we can leverage it for Role-Based Access Controls (RBAC) on our services, then we'll see how we can control security aspects of our services including network shares to ensure they meet our system's compliance policies.  And finally, we'll look at how we can enforce our services service contract to further enhance out service's security posture.

There is a great tutorial [here](https://katacoda.com/austinheiman/scenarios/open-policy-agent-gatekeeper-editor) that shows you how to configure OPA Gatekeeper for kubernetes enforcement.  I"m going to borrow from that for a simple explanation of Gatekeeper, for more in-depth knowledge, you are highly encouraged to take that Katacoda course.

First, we'll add the helm repo and gatekeeper chart with `helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts`{{execute}} and `helm install gatekeeper gatekeeper/gatekeeper`{{execute}}

Notice that the following yaml will reject any pod that attempts to deploy without a label.
`cat > template.yml <<EOF
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: requiredlabels
spec:
  crd:
    spec:
      names:
        kind: RequiredLabels
        listKind: RequiredLabelsList
        plural: requiredlabels
        singular: requiredlabels
      validation:
        # Schema for the `parameters` field in the constraint
        openAPIV3Schema:
          properties:
            labels:
              type: array
              items: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package requiredlabels
        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels for object %v : %v", [input.review.object.kind ,missing])
        }
EOF`{{execute}}

and `'kubectl apply -f template.yml`{{execute}}

Now we create the constraint to identify what kubernetes objects we want this to apply to
`cat > constraint.yml <<EOF
apiVersion: constraints.gatekeeper.sh/v1beta1
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
EOF'{{execute}}

And now we'll create a deployment that will fail because it doesn't meet the constraints
`cat > fail.yml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: busybox-invalid
  labels:
spec:
  containers:
  - name: busybox
    image: busybox
    args:
      - sleep
      - "1000000"
EOF`{{execute}}

Edit the file to get the deployment to pass by adding any label to it.

In this next section, we're going to pull in the web service that we've been using for these courses with `curl -o rest-service.jar https://raw.githubusercontent.com/drloring/katacoda-resources/main/rest-service-0.0.1-SNAPSHOT.jar`{{execute}} and the Dockerfile `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/Dockerfile`{{execute}}

Now, we'll build the docker image with `docker build -t java-ws .`{{execute}}

To save time, we'll pull down the helm charts for the service we've previously created `curl -o step2.zip https://raw.githubusercontent.com/drloring/katacoda-resources/main/java-ms-config-java-2-step-2.zip && unzip step-2.zip && cd step-2`{{execute}}
  
Now, we'll install the application with `helm install ws ws`{{execute}}.  Now, we're going to create a policy and modify our curl command to show how we can limit who can access certain operations on our web service.
 
