
# Utilize Service Mesh with Istio
**[Istio](https://istio.io/)** is an Open Source implementation of **[Service Mesh](https://www.nginx.com/blog/what-is-a-service-mesh/)**, a configurable and low latency infrastructure layer. It behaves like DI/CDI in Java development.
<br />
<br />
<br />



# Setup

## 1. [Helm](https://helm.sh/docs/using_helm/)
[Helm](https://github.com/helm/helm) is a package management system for Kubernetes.
<br />
Helm consists of two parts, `helm` (client) and `tiller`(server).
<br />
In this case, we use only `helm`.

### 1.1. [Play with Kubernetes](https://labs.play-with-k8s.com/)
1. Run `curl https://kubernetes-helm.storage.googleapis.com/helm-v2.13.1-linux-amd64.tar.gz -o helm-v2.13.1-linux-amd64.tar.gz`
2. Run `tar xvf helm-v2.13.1-linux-amd64.tar.gz`
3. Run `export PATH=$PATH:/root/linux-amd64`
4. Run `helm init --history-max 200`

### 1.2. Docker for Mac
1. Run `brew install kubernetes-helm` for installing `helm`.
2. Run `helm init --history-max 200` for initializing `helm`.
<br />
<br />


## 2. [Istio](https://istio.io/docs/setup/kubernetes/install/helm/)

### 2.1. [Download Istio](https://istio.io/docs/setup/kubernetes/install/kubernetes/)
1. Run `curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.2.2 sh -`
    - `istio-1.2.2` directory (**=$ISTIO_HOME**) wiil be created.
2. Run `export PATH=$PATH:$ISTIO_HOME/bin`
3. Run `istioctl verify-install`

### 2.2. Install Istio
1. Run `helm repo add istio.io https://storage.googleapis.com/istio-release/releases/1.2.2/charts/`
    ```bash
    $ helm repo list
    stable  	https://kubernetes-charts.storage.googleapis.com
    local   	http://127.0.0.1:8879/charts
    istio.io	https://storage.googleapis.com/istio-release/releases/1.2.2/charts/
    ```

2. Run `kubectl create namespace istio-system`
    ```bash
    $ kubectl get ns
    NAME              STATUS   AGE
    default           Active   103s
    istio-system      Active   4s
    kube-node-lease   Active   107s
    kube-public       Active   107s
    kube-system       Active   107s
    ```
3. cd $ISTIO_HOME

4. Run `helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -`

5. Install all the Istio [Custom Resource Definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions).
    1. Run `for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done`
    2. Run `kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l` and check the result is **28**.

6. Run `helm template install/kubernetes/helm/istio --name istio --namespace istio-system --values install/kubernetes/helm/istio/values-istio-demo.yaml | kubectl apply -f -`
    1. kubectl get svc -n istio-system
    2. kubectl get deploy -n istio-system
    3. kubectl get po -n istio-system


    If Pod becomes `Pending` due to `1 node(s) had taints that the pod didn't tolerate`, please try the following steps:
    1. Run `kubectl get node` to get the node name.
    2. Run `kubectl describe node <node-name>` and check `Taints: node-role.kubernetes.io/master:NoSchedule`.
    3. Run `kubectl taint nodes <node-name> <value-of-tains>-` and check `Taints : <none>`

    Links:
    - https://qiita.com/nykym/items/dcc572c21885543d94c8
    - https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
<br />
<br />


### 2.3. Uninstall Istio (if necessary)
1. Run `helm template install/kubernetes/helm/istio --name istio --namespace istio-system \ --values install/kubernetes/helm/istio/values-istio-demo.yaml | kubectl delete -f -`
2. Run `kubectl delete namespace istio-system`
3. Run `kubectl delete -f install/kubernetes/helm/istio-init/files`
<br />
<br />


## 3. Grafana (Optional)
1. Run `helm install stable/grafana --name grafana --namespace default` and memorize the name of `v1/Secret`.
2. Run `kubectl get secret grafana --namespace default -o yaml` to confirm the secret for Grafana.
3. Decode password by running `echo "xxx" | base64 --decode`
4. export POD_NAME=$(kubectl get pods --namespace default -l "app=grafana,release=grafana" -o jsonpath="{.items[0].metadata.name}")
5. Run `kubectl --namespace default port-forward $POD_NAME 3000` -> Forwarding from 127.0.0.1:3000 -> 3000
6. Access to http://localhost:3000 and login with the above information
<br />
<br />
<br />



# Tasks

## 1. [Sidecar Injection](https://istio.io/docs/setup/kubernetes/additional-setup/sidecar-injection/)
We can add `Istio Sidecar` to the Pods/Deployments manually or automatically.

### 1.1. Manual Sidecar Injection
Use `istioctl kube-inject`.

### 1.2. Automatic Sidecar Injection
Use `Istio sidecar injector` and [MutatingWebhookConfiguration](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#mutatingwebhookconfiguration-v1beta1-admissionregistration-k8s-io).

1. Enable Automatic Sidecar Injection.
    ```bash
    $ kubectl label namespace default istio-injection=enabled
    $ kubectl get ns -L istio-injection
    NAME              STATUS   AGE     ISTIO-INJECTION
    default           Active   4d      enabled
    docker            Active   4d
    istio-system      Active   3d12h
    kube-node-lease   Active   4d
    kube-public       Active   4d
    kube-system       Active   4d
    ```

2. We use `$ISTIO_HOME/samples/sleep/sleep.yaml` for explanation.
    1. Run `kubectl apply -f $ISTIO_HOME/samples/sleep/sleep.yaml`
    2. Run `kubectl describe pod <pod-name>` and check `istio-proxy` exists.
<br />
<br />


## 2. [Circuit Breaker](https://istio.io/docs/tasks/traffic-management/circuit-breaking/)
We can utilize `Circuit Breaker` by using `Sidecar Injection` and [Destination Rule](https://istio.io/docs/reference/config/networking/v1alpha3/destination-rule/).

The workflow of this example is as follows:

[fortio](https://github.com/fortio/fortio) (client) -> Circuit Breaker (Destination Rule) ->[httpbin](https://github.com/istio/istio/tree/release-1.2/samples/httpbin) (backend)
<br />
<br />


## 3. [Request Timeouts](https://istio.io/docs/tasks/traffic-management/request-timeouts/)
⭐️ TODO
-  Set up [Bookinfo Application (Sample Application)](https://istio.io/docs/examples/bookinfo/) at first
    - Pod作成中に `Unable to connect to the server: EOF` エラー発生（2019/08/04）
<br />
<br />


## Others
- [Request Routing](https://istio.io/docs/tasks/traffic-management/request-routing/)
- [Fault Injection](https://istio.io/docs/tasks/traffic-management/fault-injection/)
- [Traffic Shifting](https://istio.io/docs/tasks/traffic-management/traffic-shifting/)
- [Control Ingress Traffic](https://istio.io/docs/tasks/traffic-management/ingress/)
<br />
<br />
<br />



# Links
- [Service Mesh by Terada-san](https://github.com/yoshioterada/k8s-Azure-Container-Service-AKS--on-Azure/blob/master/Kubernetes-Workshop6.md)
- [How to Monitor k8s](https://qiita.com/FY0323/items/72616d6e280ec7f2fdaf)
- [Istio wiki](https://github.com/istio/istio/wiki)
  - This wiki includes a lot of techniques and approaches we can refer.
<br />
<br />
<br />



# Misc.
```bash
$ kubectl config get-contexts
CURRENT   NAME                 CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop       docker-desktop   docker-desktop
          docker-for-desktop   docker-desktop   docker-desktop
```
```bash
$ kubectl config use-context docker-for-desktop
Switched to context "docker-for-desktop".
```
```bash
$ kubectl config current-context
docker-for-desktop
```

- Envoy Proxy
  - kubectl apply --record -f <(istioctl kube-inject -f ./xxx.yml)
  - istioctl kube-inject -f xxx
- [Traffic Management](https://istio.io/docs/concepts/traffic-management/)
  - Health Check
  - Routing
    - Round Robin
- [Security](https://istio.io/docs/concepts/security/)
- [Control and Observe](https://istio.io/docs/concepts/policies-and-telemetry/)
  - Transparency
- [Smart Endpoints and Dump Pipes](https://www.martinfowler.com/microservices/)
