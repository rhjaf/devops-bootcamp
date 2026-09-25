

1. First if you have a cluster initalized before, you can remove it entirely (on control plane)
```bash
sudo kubeadm reset -f; sudo rm -rf /etc/cni/net.d; sudo rm -rf /var/lib/cni; sudo rm -rf $HOME/.kube
sudo rm -rf /etc/kubernetes/; sudo rm -rf /var/lib/etcd/*; sudo rm -rf /var/lib/kubelet/*
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo ipvsadm --clear 2>/dev/null || true
```
2. Now on worker nodes
```bash
sudo kubeadm reset -f
sudo rm -rf /etc/cni/net.d
sudo rm -rf /var/lib/cni
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo ipvsadm --clear 2>/dev/null || true
```
3. Now you can install the kubernetes with **Calico** as CNI using this [GUIDE](). Just remember to apply the following steps additonally:
-- Comment swap memory mount in `/etc/fstab`.
-- Use the explicitly advertised switch when initalizing cluster: 
```bash
sudo kubeadm init --control-plane-endpoint=192.168.1.10:6443 --apiserver-advertise-address=192.168.1.10 --pod-network-cidr=10.20.0.0/16
```
-- If you have problem to fetch and apply calico manifest from their repo, you can get them manually
```bash
git clone --branch v3.28.1 --depth 1 https://github.com/projectcalico/calico.git
```
-- When you are setting up calico network configurations (files: `custom-resources.yaml` and `tigera-operator`), you may need to download files manually.
-- You can also set the containerd endpoint in file `/var/lib/kubelet/kubeadm-flags.env`
```bash
KUBELET_KUBEADM_ARGS="--container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9 --node-ip=192.168.1.12"
```
Then run restart the kubelet
```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

-- If you have problems with Calico pods refer to [this - day 54](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main)

-- Please note that if you have problem with Calico pods (especially **Felix** which is Calico's per-node agent), deleting the pod with `kubectl delete pod` could be a savior. 

4. You can join any number of control-plane nodes by copying certificate authorities and service account keys on each node and then running the following as root:
```bash
kubeadm join 192.168.1.10:6443 --token 8t6jwh.wopb1v2a0dlpoiu6 --discovery-token-ca-cert-hash sha256:d899d7ddd5001186c7b8d111c101efd91a9bccd15d16d0bcde3fa33829071a02 --control-plane
```
5. To join as a worker node
```bash
kubeadm join 192.168.1.10:6443 --token 8t6jwh.wopb1v2a0dlpoiu6 --discovery-token-ca-cert-hash sha256:d899d7ddd5001186c7b8d111c101efd91a9bccd15d16d0bcde3fa33829071a02
```

#### Part 1 - Install Traefik
```bash
helm repo add traefik https://traefik.github.io/charts
helm repo update
kubectl create namespace traefik
helm install traefik traefik/traefik --namespace traefik --values values-new.yaml
```

#### Part 2 - Installing Rancher
1. **Rancher** uses TLS to commiunicate. We install **cert-manager** as a certificate manager for our cluster. Rancher uses cert-manager to generate certificates
```bash
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --set installCRDs=true
```


to verify
```bash
kubectl get pods --namespace cert-manager

```
2. We install stable version of Rancher with a helm chart
```bash
helm install rancher rancher-stable/rancher --namespace cattle-system --set hostname=kuber.local --set bootstrapPassword=admin
```




#### Extra notes - Unistalling services from your cluster
Uninstall Rancher, Traefik
```bash
helm uninstall rancher -n cattle-system; kubectl delete namespace cattle-system
helm uninstall traefik -n traefik
```
