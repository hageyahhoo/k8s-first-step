[Play with Kubernetes](https://labs.play-with-k8s.com/)

# 1. Set up k8s
## 1.1. Master Node
1. Press "ADD NEW INSTANCE"
2. Run `kubeadm init --apiserver-advertise-address $(hostname -i)`
3. Run `kubectl apply -n kube-system -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 |tr -d '\n')"`

## 1.2. Slave Node
1. Press "ADD NEW INSTANCE"
2. Run `kubeadm join`


# 2. Set up Istio
1. Run `curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.1.1 sh -` on Master Node.
