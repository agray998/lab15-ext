# CKA Lab 15 - Extension

## Concepts covered
* validating admission policies
* network policies
* Pod security standards

## Instructions
1. Create a new namespace, called _sec-ns_, with the label lab=15
```shell
kubectl create ns sec-ns
kubectl label ns sec-ns lab=15
```

2. Included in this lab is a validating admission policy, and corresponding binding. These are defined in the _ap.yaml_ file. Review the configuration. This policy will ensure that all deployments into namespaces covered by the policy retain no more than 5 prior revisions. Create the policy and binding:
```shell
kubectl apply -f ap.yaml
```

3. Also provided is a network policy (in _netpol.yaml_). Review this configuration. This will apply to all pods with the label app=nginx-server in the sec-ns namespace, and will restrict traffic allowing only ingress from sources in namespaces that share the lab=15 label. Note that network policies are namespaced. Create the policy:
```shell
kubectl apply -f netpol.yaml
```

4. To finish configuring the sec-ns namespace, we will add another label which specifies a _pod security standard_ to enforce at the namespace level:
```shell
kubectl label ns sec-ns pod-security.kubernetes.io/enforce=restricted
kubectl label ns sec-ns pod-security.kubernetes.io/enforce-version=latest
```

5. The _deploy.yaml_ defines a deployment and associated service, both configured to be deployed into the sec-ns namespace. Attempt to create the deployment:
```shell
kubectl apply -f deploy.yaml
```
The deployment should fail. Referring to the configuration for the policies we applied, attempt to resolve the policy violations so that the deployment can be successfully created. (A solution, _deploy-fixed.yaml_, has been provided if you need a reference point). Once you can successfully create the deployment and service, do so.

6. We will now test the effect of the network policy we created. Create a pod called _test_, in the default namespace, running the alpine linux image, and attempt to curl the service in the sec-ns namespace:
```shell
kubectl run test --image=alpine -- sleep infinity
kubectl exec test -- sh -c "apk update; apk add curl; curl nginx-server.sec-ns"
```
The curl request should fail, as the network policy is rejecting the traffic because its' source is not from a namespace with labels matching the ingress rules' namespaceSelector.

7. Label the default namespace so that it matches the namespaceSelector for the network policy:
```shell
kubectl label ns default lab=15
```
Attempt to use the test pod to execute the same curl command again - it should now succeed, and display the standard nginx response:
```shell
kubectl exec test -- sh -c "curl nginx-server.sec-ns"
```