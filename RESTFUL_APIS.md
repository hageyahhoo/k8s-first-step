**Automate managing Kubernetes clusters and resources by using RESTful APIs.**
- Not kubectl!


# [kubectl proxy](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/#using-kubectl-proxy)
Run `kubectl` in Proxy Mode
1. Run `kubectl proxy --port=<port> &`
2. Run `curl https://localhost:<port>/api` or access to **https://localhost:<port>/** via browser.


# v=8 : View API Calls by kubectl
e.g.) Run `kubectl --v=8 get pod`
```
Request Headers:
    Accept: application/json;as=Table;v=v1beta1;g=meta.k8s.io, application/json
    User-Agent: kubectl/v1.13.0 (darwin/amd64) kubernetes/ddf47ac
Response Status: 200 OK in 34 milliseconds
Response Headers:
    Content-Type: application/json
    Content-Length: 3399
    Date: Wed, 23 Jan 2019 07:30:05 GMT
Response Body: {"kind":"Table","apiVersion":"meta.k8s.io/v1beta1","metadata":{"selfLink":"/api/v1/namespaces/default/pods","resourceVersion":"182128"},"columnDefinitions":[{"name":"Name","type":"string","format":"name","description":"Name must be unique within a namespace. Is required when creating resources, although some resources may allow a client to request the generation of an appropriate name automatically. Name is primarily intended for creation idempotence and configuration definition. Cannot be updated. More info: http://kubernetes.io/docs/user-guide/identifiers#names","priority":0},{"name":"Ready","type":"string","format":"","description":"The aggregate readiness state of this pod for accepting traffic.","priority":0},{"name":"Status","type":"string","format":"","description":"The aggregate status of the containers in this pod.","priority":0},{"name":"Restarts","type":"integer","format":"","description":"The number of times the containers in this pod have been restarted.","priority":0},{"name":"Age","type":"strin [truncated 2375 chars]
no kind is registered for the type v1beta1.Table in scheme "k8s.io/kubernetes/pkg/api/legacyscheme/scheme.go:29"
NAME         READY   STATUS              RESTARTS   AGE
web-server   0/1     ContainerCreating   0          8s
```


# Links
- [Kubernetes API Overview](https://kubernetes.io/docs/reference/using-api/api-overview/)
- [API Overview](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/)
- [Access Clusters Using the Kubernetes API](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/)
- [Controlling Access to the Kubernetes API](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/)
- [Example](https://techbeacon.com/one-year-using-kubernetes-production-lessons-learned)
