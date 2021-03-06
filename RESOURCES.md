# 1. How to Use Resources with kubectl
`kubectl` controls the Kubernetes cluster manager.

## 1) Create Pods
1. Create a manifest file (yaml or json) for your Pods.
2. Run `kubectl create -f <manifest file>` to create Pods with defined status.

## 2) Investigate Pods
|Command                                                 |Description                                                             |
| ------------------------------------------------------ | ---------------------------------------------------------------------- |
|`kubectl get (po OR pod OR pods)`                             |List all status of Pods.                                                |
|`kubectl get (po OR pod OR pods) <pod-name>`                  |List the status of the specified Pod.                                   |
|`kubectl get (po OR pod OR pods) <pod-name> -o wide`          |List the status of the specified Pod including IP address and node name.|
|`kubectl get (po OR pod OR pods) <pod-name> -o (yaml OR json)`|List the status of the specified Pod with yaml or json.                 |
|`kubectl describe (po OR pod OR pods) <pod-name>`             |Show details of the specified Pod.                                      |

## 3) Update Pods
1. Modify your manifest file (yaml or json).
2. Run `kubectl apply -f <manifest file>`.

## 4) Login to Pods
Run `kubectl exec <pod-name> <commands> (--container <container-name>)`.
e.g.)
- kubectl exec <pod-name> ps aux
- kubectl exec -it <pod-name> sh

## 5) Stop Pods
Run `kubectl delete (po OR pod OR pods) <pod-name>`.

## TODO
`kubectl logs <pod-name>`


# 2. How to Expose Pods
We can NOT expose services only with Pod. We need additional configuration or resource.

## 1) Use `kubectl port-forward`
1. Create Pods.
2. Run `kubectl port-forward <pod-name> <from-port>:<to-port>`
  ```
  e.g.)
  $ kubectl port-forward web-service 8080:80
  > Forwarding from 127.0.0.1:8080 -> 80
  -> Can access to "web-service" Pod with http://localhost:8080
  ```
Link) https://medium.com/@lizrice/accessing-an-application-on-kubernetes-in-docker-1054d46b64b1

## 2) Use `Service` Resource
1. Create `Service`.
  - Configure the type of Service as `NodePort` or `LoadBalancer`.
  - We cannot use `ClusterIP` for exposing Pods. It's just for internal-cluster.
2. Run `kubectl get svc <service-name>` to check `EXTERNAL-IP`.
3. Specify `http://<EXTERNAL-IP>:<port>`.
  - e.g.) http://localhost:8080

Link) https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/


# 3. How to Update Pods

## 1) Change Pods and labels : Blue-Green Deployment OR Canary Release
1. Change the image and labels of Pods.
2. Apply Pods' manifest file.
3. Change labels of Service.
4. Apply Service's manifest file.

## 2) Rolling update and rollback with `Deployment`
1. Change Pods' information.
2. Rolling update  
  2.1. Run `kubectl apply -f <manifest-file-for-deployment> --record`. This command can record revision numbers (=history) of changing Deployment. We can use them for rollback.  
  2.2. Run `kubectl rollout status (deploy OR deployment) <deployment-name>` to check the status of rolling update.
3. Rollback  
  3.1. Run `kubectl rollout history (deploy OR deployment) <deployment-name>` to get revision numbers.  
  3.2. Run `kubectl rollout undo (deploy OR deployment) <deployment-name> --to-revision=<revision-number>`.  
  3.3. Run `kubectl rollout status (deploy OR deployment) <deployment-name>` to check the status of rollback.
