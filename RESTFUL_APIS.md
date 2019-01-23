**Use RESTful APIs to automate managing Kubernetes clusters and resources, NOT with kubectl!**


# 1. [RBAC (Role-based access control)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
Before using RESTful APIs, we need to set up (Cluster)Role and (Cluster)RoleBinding.
- Role & RoleBinding: To grant access to resources within a single namespace.
- ClusterRole & ClusterRoleBinding: To grant access to cluster-wide resources.

Without this setting, you may face with the following error message.
```
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "pods is forbidden: User \"system:anonymous\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
```

Here are examples of [Role](./role.yml) and [RoleBinding](./roleBinding.yml).


# 2. Call RESTful APIs with Credentials
1. Get the URL of the API server
  - `APISERVER=$(kubectl cluster-info | grep master | cut -f 6- -d " ")``
    - kubectl cluster-info : Kubernetes master is the URL of the API Server.
  - `APISERVER=$(kubectl config view | grep server | cut -f 2- -d ":" | tr -d " ")`
    - kubectl config view : clusters.cluster.server is the URL of the API Server.
2. Get the default Credential by calling `kubectl describe (secret OR secrets) <secret-name>`
  ```
  $ TOKEN=$(kubectl describe secret $(kubectl get secrets | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t')
  ```
3. Run `curl $APISERVER/<API> --header "Authorization: Bearer $TOKEN" --insecure`.


# Links
- [Kubernetes API Overview](https://kubernetes.io/docs/reference/using-api/api-overview/)
- [API Overview](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/)
- [Access Clusters Using the Kubernetes API](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/)
- [Controlling Access to the Kubernetes API](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/)
- [Example](https://techbeacon.com/one-year-using-kubernetes-production-lessons-learned)
- [RESTful operation by Terada-san](https://github.com/yoshioterada/k8s-Azure-Container-Service-AKS--on-Azure/blob/master/Kubernetes-Workshop5.md)
