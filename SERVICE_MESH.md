
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
- It requires `istio` and `istio-sidecar-injector` ConfigMaps in `istio-system` namespace.
- We use `$ISTIO_HOME/samples/sleep/sleep.yaml` for explanation.


### 1.1. Manual Sidecar Injection
Use `istioctl kube-inject` command to inject sidecar.

#### 1) Using In-cluster Configuration
1. Run `istioctl kube-inject -f samples/sleep/sleep.yaml | kubectl apply -f -`
  - It creates `serviceaccount`, `service`, `deployment`, and `pods`.
2. Run `kubectl get pods -l app=sleep`

#### 2) Using Local Copies of the Configuration
  ```bash
  $ cd $ISTIO_HOME/samples/sleep
  $ kubectl get configmap istio-sidecar-injector -n istio-system -o=jsonpath='{.data.config}' > inject-config.yaml
  $ kubectl get configmap istio -n istio-system -o=jsonpath='{.data.mesh}' > mesh-config.yaml
  $ istioctl kube-inject \
      --injectConfigFile inject-config.yaml \
      --meshConfigFile mesh-config.yaml \
      --filename sleep.yaml \
      --output sleep-injected.yaml
  $ kubectl apply -f sleep-injected.yaml
  $ kubectl get pods -l app=sleep
  ```
⭐️ sleep-injected.yamlを、この方法で作成済
  ⭐️　実際にapplyすると、Podだけ生成できない
  - ⭐️ istio関連のresourcesが全て消失 -> 作り直し & sleep-injected.yamlが古くなってしまっているため、生成できていない
  ```bash
  $ kubectl get deploy
  NAME    READY   UP-TO-DATE   AVAILABLE   AGE
  sleep   0/1     0            0           4m12s
  ```

  Play with k8sでの出力
  ```bash
  [node1 sleep]$ kubectl get pods -l app=sleep
  NAME                     READY   STATUS    RESTARTS   AGE
  sleep-57d87fc557-pnj8n   0/2     Pending   0          21s
  ```

### 1.2. Automatic Sidecar Injection
⭐️ TODO
Use `Istio sidecar injector`.
kubectl api-versions | grep admissionregistration
-> admissionregistration.k8s.io/v1beta1

Check [Admission Controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
- MutatingAdmissionWebhook
- ValidatingAdmissionWebhook

$ kubectl label namespace default istio-injection=enabled
$ kubectl get ns -L istio-injection
NAME              STATUS   AGE     ISTIO-INJECTION
default           Active   9m8s    enabled
kube-node-lease   Active   9m11s
kube-public       Active   9m11s
kube-system       Active   9m11s
<br />
<br />
<br />



# TODO

## 0. Set up [Bookinfo Application (Sample Application)](https://istio.io/docs/examples/bookinfo/)
1. `cd $ISTIO_HOME`
2. `kubectl label namespace default istio-injection=enabled`
3. `kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml`
  - `kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)`
4. `kubectl get svc`
5. `kubectl get po`
  - (i) `Play with Kubernetes` cannot create pods...
  - (i) Some pods aren't running
6. `kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"`
  - curl: (7) Failed to connect to productpage port 9080: Connection refused
7. `kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml`
8. `kubectl get gateway`
9. `export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT`
  INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
  export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
  export GATEWAY_URL=localhost:31380


`curl -s http://${GATEWAY_URL}/productpage | grep -o "<title>.*</title>"`

Cleanup
`samples/bookinfo/platform/kube/cleanup.sh`
<br />
<br />
<br />



## ?. [Control Ingress Traffic](https://istio.io/docs/tasks/traffic-management/ingress/)


## ?. [Request Timeouts](https://istio.io/docs/tasks/traffic-management/request-timeouts/)


## 3. [Circuit Breaker](https://istio.io/docs/tasks/traffic-management/circuit-breaking/)
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutiveErrors: 1
      interval: 1s
      baseEjectionTime: 3m
      maxEjectionPercent: 100
EOF

- Envoy Proxy
  - kubectl apply --record -f <(istioctl kube-inject -f ./xxx.yml)
  - istioctl kube-inject -f xxx
- [Traffic Management](https://istio.io/docs/concepts/traffic-management/)
  - Health Check
  - Routing
    - Round Robin
  - Timeout
- [Security](https://istio.io/docs/concepts/security/)
- [Control and Observe](https://istio.io/docs/concepts/policies-and-telemetry/)
  - Transparency
- [Smart Endpoints and Dump Pipes](https://www.martinfowler.com/microservices/)
<br />
<br />
<br />



# Links
- [Service Mesh by Terada-san](https://github.com/yoshioterada/k8s-Azure-Container-Service-AKS--on-Azure/blob/master/Kubernetes-Workshop6.md)
- [How to Monitor k8s](https://qiita.com/FY0323/items/72616d6e280ec7f2fdaf)
- [Istio wiki](https://github.com/istio/istio/wiki)
  - This wiki includes a lot of techniques and approaches we can refer.



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
