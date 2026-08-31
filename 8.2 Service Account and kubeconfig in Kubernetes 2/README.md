
#### Send a request to API server from a pod
1. Use the following command
```bash
kubectl exec -n test nginx-test -- \
  sh -c 'TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token); \
  CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt; \
  curl --cacert $CACERT \
  -H "Authorization: Bearer $TOKEN" \
  https://192.168.1.10:6443/api/v1/namespaces/nginx/pods'
``` 
2. You can create the token via
```bash
TOKEN=$(kubectl create token default -n test)
```
3. The certificate for the cluster can be fetched
```bash
kubectl config view --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 -d > ca.crt
```
#### to query Kube API directl
1. First you need to extract certificates and keys of your cluster
```bash
kubectl config view --raw --minify --flatten -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 -d > ca.crt
kubectl config view --raw --minify --flatten -o jsonpath='{.users[0].user.client-certificate-data}' | base64 -d > client.crt
kubectl config view --raw --minify --flatten -o jsonpath='{.users[0].user.client-key-data}' | base64 -d > client.key
```
2. Now you can query Kubelet API directly. For example to get pods on a node 
```bash
curl --cacert ca.crt --cert client.crt --key client.key https://<node-ip>:10250/pods | jq
```
Remember that you can use tools like [kubeletctl]() to query KubeAPI endpoints. For example to get pods on a node: `kubeletctl -s pods`
