**Utilize Service Mesh with Istio**


# How to Install Grafana
1. Run `helm install stable/grafana --name grafana --namespace default` and memorize the name of `v1/Secret`.
2. Run `kubectl get secret grafana --namespace default -o yaml` to onfirm the secret for Grafana.
3. Decode password by running `echo "xxx" | base64 --decode`
4. export POD_NAME=$(kubectl get pods --namespace default -l "app=grafana,release=grafana" -o jsonpath="{.items[0].metadata.name}")
5. Run `kubectl --namespace default port-forward $POD_NAME 3000` -> Forwarding from 127.0.0.1:3000 -> 3000
6. Access to http://localhost:3000 and login with the above information


## Links
- [Istio](https://istio.io/)
- [What Is a Service Mesh?](https://www.nginx.com/blog/what-is-a-service-mesh/)
- [Instructions](https://istio.io/docs/setup/kubernetes/install/helm/)
- https://github.com/yoshioterada/k8s-Azure-Container-Service-AKS--on-Azure/blob/master/Kubernetes-Workshop6.md
- https://github.com/yoshioterada/k8s-Azure-Container-Service-AKS--on-Azure/blob/master/Kubernetes-Workshop8.md
