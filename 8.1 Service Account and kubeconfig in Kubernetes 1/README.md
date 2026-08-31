### Create a Kubeconfig 
1. Generate a private key and certificate
```bash
openssl genrsa -out test.key 2048
```
2. Create a CSR file with subject equal to username
```bash
openssl req -new -key test.key -out test.csr -subj "/CN=test"
```
3. Sign the CSR file with Cluser certificates
```bash
openssl x509 -req -in test.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out test.crt -days 365 
```
4. Set the cluster info for kubeconfig file (the IP is masternode:APIServer_port)
```bash
kubectl config set-cluster kubernetes --server=https://192.168.1.10:6443 --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true --kubeconfig=test.kubeconfig
```
5. Add the credentials for the kubeconfig
```bash
kubectl config set-credentials test --client-certificate=test.crt --client-key=test.key --embed-certs=true --kubeconfig=test.kubeconfig
```
6. Create a new context based on new kubeconfig for cluster
```bash
kubectl config set-context test-kubernetes-context --cluster=kubernetes --user=test --namespace=nginx --kubeconfig=test.kubeconfig
```
7. Set to use the kubeconfig
```bash
kubectl config use-context test-kubernetes-context --kubeconfig=test.kubeconfig
```

