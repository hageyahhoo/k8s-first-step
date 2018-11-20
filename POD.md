# How to Use Pods with kubectl
`kubectl` controls the Kubernetes cluster manager.


## Create Pods
1. Create a manifest file (yaml or json) for your Pods.
2. Run `kubectl create -f <manifest file>` to create Pods with defined status.


## Investigate Pods
|Command                                                 |Description                                                             |
| ------------------------------------------------------ | ---------------------------------------------------------------------- |
|`kubectl get (pod OR pods)`                             |List all status of Pods.                                                |
|`kubectl get (pod OR pods) <pod-name>`                  |List the status of the specified Pod.                                   |
|`kubectl get (pod OR pods) <pod-name> -o wide`          |List the status of the specified Pod including IP address and node name.|
|`kubectl get (pod OR pods) <pod-name> -o (yaml OR json)`|List the status of the specified Pod with yaml or json.                 |
|`kubectl describe (pod OR pods) <pod-name>`             |Show details of the specified Pod.                                      |


## Update Pods
1. Modify your manifest file (yaml or json).
2. Run `kubectl apply -f <manifest file>`.


## Login to Pods
Run `kubectl exec <pod-name> <commands> (--container)`


## Stop Pods
Run `kubectl delete pods <pod-name>`


## TODO
`kubectl logs <pod-name>`
