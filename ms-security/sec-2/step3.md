In this demonstration, we're going to discover a few different uses for the [Open Policy Agent | https://www.openpolicyagent.org/].  First, we'll look at how we can use it to ensure that our kubernetes deployments comply with our system's policies using OPA Gatekeeper, then we'll look at how we can leverage it for Role-Based Access Controls (RBAC) on our services, then we'll see how we can control security aspects of our services including network shares to ensure they meet our system's compliance policies.  And finally, we'll look at how we can enforce our services service contract to further enhance out service's security posture.

helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
helm install gatekeeper gatekeeper/gatekeeper 

