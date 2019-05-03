# Utilize Service Mesh with Istio!
[Istio](https://istio.io/) is an Open Source implementation of [Service Mesh](https://www.nginx.com/blog/what-is-a-service-mesh/), a configurable and low latency infrastructure layer. It behaves like DI/CDI in Java development.



# Setup

## 1. [Play with Kubernetes](https://labs.play-with-k8s.com/)

### 1.1. Master Node
1. Press "ADD NEW INSTANCE"
2. Run `kubeadm init --apiserver-advertise-address $(hostname -i)`
3. Run `kubectl apply -n kube-system -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 |tr -d '\n')"`

### 1.2. Slave Node
1. Press "ADD NEW INSTANCE"
2. Run `kubeadm join`


## 2. [Helm (Package Management system)](https://github.com/helm/helm)
1. curl https://kubernetes-helm.storage.googleapis.com/helm-v2.13.1-linux-amd64.tar.gz -o helm-v2.13.1-linux-amd64.tar.gz
2. tar xvf helm-v2.13.1-linux-amd64.tar.gz
3. export PATH=$PATH:/root/linux-amd64
4. helm
4. helm init
5. helm search

Link: https://github.com/docker/compose-on-kubernetes/issues/35


## 3. [Istio](https://istio.io/docs/setup/kubernetes/install/helm/)
1. Run `curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.1.4 sh -` on Master Node.
2. export PATH=$PATH:/root/istio-1.1.4/bin
3. istioctl
4. kubectl create namespace istio-system
5. cd $ISTIO_HOME
6. helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -
7. helm template install/kubernetes/helm/istio --name istio --namespace istio-system | kubectl apply -f -
8. kubectl get svc -n istio-system
9. kubectl get po -n istio-system
10. helm repo add istio.io https://storage.googleapis.com/istio-release/releases/1.1.4/charts/
11. for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
12. kubectl get crd


## 4. Grafana (Optional)
1. Run `helm install stable/grafana --name grafana --namespace default` and memorize the name of `v1/Secret`.
2. Run `kubectl get secret grafana --namespace default -o yaml` to onfirm the secret for Grafana.
3. Decode password by running `echo "xxx" | base64 --decode`
4. export POD_NAME=$(kubectl get pods --namespace default -l "app=grafana,release=grafana" -o jsonpath="{.items[0].metadata.name}")
5. Run `kubectl --namespace default port-forward $POD_NAME 3000` -> Forwarding from 127.0.0.1:3000 -> 3000
6. Access to http://localhost:3000 and login with the above information


## Links
- [Service Mesh by Terada-san](https://github.com/yoshioterada/k8s-Azure-Container-Service-AKS--on-Azure/blob/master/Kubernetes-Workshop6.md)
- [How to Monitor k8s](https://qiita.com/FY0323/items/72616d6e280ec7f2fdaf)



# Tasks (TODO)

## 0. Set up [Base Application](https://istio.io/docs/examples/bookinfo/)
1. cd $ISTIO_HOME
2. kubectl label namespace default istio-injection=enabled
3. kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
4. kubectl get svc
5. kubectl get po
  -> No resources found.
6. @@@@@
7. kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml


## 1. [Request Timeouts](https://istio.io/docs/tasks/traffic-management/request-timeouts/)


## 2. [Sidecar](https://istio.io/docs/setup/kubernetes/additional-setup/sidecar-injection/)
- Automatic Sidecar Injection
  kubectl api-versions | grep admissionregistration
  -> admissionregistration.k8s.io/v1beta1


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
