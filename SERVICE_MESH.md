# Utilize Service Mesh with Istio!
[Istio](https://istio.io/) is an Open Source implementation of [Service Mesh](https://www.nginx.com/blog/what-is-a-service-mesh/), a configurable and low latency infrastructure layer. It behaves like DI/CDI in Java development.



# Setup

## 0. [Play with Kubernetes](https://labs.play-with-k8s.com/)

### 0.1. Master Node
1. Press "ADD NEW INSTANCE"
2. Run `kubeadm init --apiserver-advertise-address $(hostname -i)`
3. Run `kubectl apply -n kube-system -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 |tr -d '\n')"`

### 0.2. Slave Node
1. Press "ADD NEW INSTANCE"
2. Run `kubeadm join`


## 1. [Helm (Package Management system)](https://github.com/helm/helm)
1. curl https://kubernetes-helm.storage.googleapis.com/helm-v2.13.1-linux-amd64.tar.gz -o helm-v2.13.1-linux-amd64.tar.gz
2. tar xvf helm-v2.13.1-linux-amd64.tar.gz
3. export PATH=$PATH:/root/linux-amd64
4. helm
4. helm init
5. helm search

Link: https://github.com/docker/compose-on-kubernetes/issues/35


## 2. [Istio](https://istio.io/docs/setup/kubernetes/install/helm/)
1. Run `curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.1.4 sh -` on Master Node.
2. export PATH=$PATH:/root/istio-1.1.4/bin
3. istioctl
4. kubectl create namespace istio-system
5. cd $ISTIO_HOME
6. helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -
7. helm template install/kubernetes/helm/istio --name istio --namespace istio-system | kubectl apply -f -
8. kubectl get svc -n istio-system
9. kubectl get po -n istio-system
  - ⭐️ Istio関連のPodが生成できない
    https://qiita.com/megaman-go-go/items/3b709e90aa133d199459
    helm template install/kubernetes/helm/istio --name istio --namespace istio-system --set security.enabled=true | kubectl apply -f -

10. helm repo add istio.io https://storage.googleapis.com/istio-release/releases/1.1.4/charts/
11. for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
12. kubectl get crd


## 3. Grafana (Optional)
1. Run `helm install stable/grafana --name grafana --namespace default` and memorize the name of `v1/Secret`.
2. Run `kubectl get secret grafana --namespace default -o yaml` to confirm the secret for Grafana.
3. Decode password by running `echo "xxx" | base64 --decode`
4. export POD_NAME=$(kubectl get pods --namespace default -l "app=grafana,release=grafana" -o jsonpath="{.items[0].metadata.name}")
5. Run `kubectl --namespace default port-forward $POD_NAME 3000` -> Forwarding from 127.0.0.1:3000 -> 3000
6. Access to http://localhost:3000 and login with the above information


## Links
- [Service Mesh by Terada-san](https://github.com/yoshioterada/k8s-Azure-Container-Service-AKS--on-Azure/blob/master/Kubernetes-Workshop6.md)
- [How to Monitor k8s](https://qiita.com/FY0323/items/72616d6e280ec7f2fdaf)



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
      --filename samples/sleep/sleep.yaml \
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
