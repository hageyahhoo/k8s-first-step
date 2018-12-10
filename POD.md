# 1. How to Use Pods with kubectl
`kubectl` controls the Kubernetes cluster manager.

## Create Pods
1. Create a manifest file (yaml or json) for your Pods.
2. Run `kubectl create -f <manifest file>` to create Pods with defined status.

## Investigate Pods
|Command                                                 |Description                                                             |
| ------------------------------------------------------ | ---------------------------------------------------------------------- |
|`kubectl get (po OR pod OR pods)`                             |List all status of Pods.                                                |
|`kubectl get (po OR pod OR pods) <pod-name>`                  |List the status of the specified Pod.                                   |
|`kubectl get (po OR pod OR pods) <pod-name> -o wide`          |List the status of the specified Pod including IP address and node name.|
|`kubectl get (po OR pod OR pods) <pod-name> -o (yaml OR json)`|List the status of the specified Pod with yaml or json.                 |
|`kubectl describe (po OR pod OR pods) <pod-name>`             |Show details of the specified Pod.                                      |

## Update Pods
1. Modify your manifest file (yaml or json).
2. Run `kubectl apply -f <manifest file>`.

## Login to Pods
Run `kubectl exec <pod-name> <commands> (--container <container-name>)`.
e.g.)
- kubectl exec <pod-name> ps aux
- kubectl exec -it <pod-name> sh

## Stop Pods
Run `kubectl delete (po OR pod OR pods) <pod-name>`.

## TODO
`kubectl logs <pod-name>`


# 2. How to Expose Pods
We can NOT expose services only with Pod. We need to use `Service`.

## How to Configure
1. Create `Service`
  - Configure the type of Service as `LoadBalancer`.
  - We cannot use `ClusterIP` or `NodePort` for it.
3. Run `kubectl get svc <service-name>` to check `EXTERNAL-IP`.
4. Specify `http://<EXTERNAL-IP>:<port>`.

## Links
https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/


# 3. How to Update Pods

## 1) Change Pods and labels
1. Change the image and labels of Pods.
2. Apply Pods' manifest file.
3. Change labels of Service.
4. Apply Service's manifest file.

## 2) Rolling update with `Deployment`
1. Change Pods' information.
2. Run `kubectl apply -f <manifest-file-for-deployment>`
3. Run `kubectl rollout status deploy <deployment-name>` to check the status of rolling update.

## 3) Rollback with `Deployment`
**TODO**
