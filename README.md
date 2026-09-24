# Dealing with sanction errors
- If you are using v2ray, you can always use proxy for your pod in the `env` using these environment variables:
  ```bash
  HTTP_PROXY=http://127.0.0.1:10808
  HTTPS_PROXY=http://127.0.0.1:10808
  ALL_PROXY=socks5://127.0.0.1:10808
  ```
  You can set them as environment variables inside shell
  ```bash
  export HTTPS_PROXY=http://127.0.0.1:10808
  export HTTPS_PROXY=http://127.0.0.1:10808
  ```
  In general the syntax should be like `HTTP_PROXY=http://username:password@proxy.corp.example.com:8080`
- you can also tunnel all of your interface traffic through a proxy by `TUN`:
  ```bash
  tun2socks --device tun://wg1 --proxy socks5://127.0.0.1:1080
  ```
# Common Errors
- "server gave HTTP response to HTTPS client" when the resource is trying to pull image from a repo:
  check the docker runtime ( if it was containerd, not docker, follow the steps )
  ```bash
  sudo crictl info | grep -i runtime
  ```
  edit the following file:
  ```bash
  sudo mkdir -p /etc/containerd/certs.d/192.168.1.23:8083
  sudo vi /etc/containerd/certs.d/192.168.1.23:8083/hosts.toml
  ```
  put the following content:
  ```bash
  server = "http://192.168.1.23:8084"
  [host."http://192.168.1.23:8084"]
    capabilities = ["pull", "resolve"]
  ```
  also edit this file:
  ```bash
  sudo vi /etc/containerd/config.toml
  ```
  and add the following lines:
  ```bash
  [plugins.'io.containerd.cri.v1.images'.registry]
    config_path = '/etc/containerd/certs.d'
  ```
  Now restart your docker runtime and test:
  ```bash
  sudo systemctl restart containerd
  sudo crictl pull 192.168.1.23:8084/prom/node-exporter:latest
  ```
- Kubernetes pv not allocating to pvc (A Released PV cannot automatically bind to the newly created PVC because it still has the old claim reference in claimRef):
  ```bash
  kubectl patch pv <pv-name> -p '{"spec":{"claimRef":null}}'
  ```
- If you have problems with `CoreDNS`, you can not access services by hostnames or pods can not see each other via hostnames, you can fix that by executing the following command:
  ```bash
  kubectl get pod -n kube-system coredns-6cd5f799cb-nmfkr -o jsonpath='{.spec.serviceAccountName}{"\n"}{.spec.automountServiceAccountToken}{"\n"}'
  kubectl patch deployment coredns -n kube-system -p '{"spec":{"template":{"spec":{"automountServiceAccountToken":true}}}}'
  kubectl rollout status deployment/coredns -n kube-system
  ```
  verify that the DNS lookup is now working
  ```bash
  kubectl exec -n monitoring grafana-5b9544c5bb-7cf76 -- nslookup prometheus.monitoring.svc.cluster.local
  ```
- If you encounter this error `'overlayfs idmapped layers are not supported'` and your nodes are disconnecting from cluster:
  ```bash
  systemctl stop kubelet
  systemctl stop containerd
  mv /var/lib/containerd/io.containerd.metadata.v1.bolt/meta.db \
     /var/lib/containerd/io.containerd.metadata.v1.bolt/meta.db.corrupt
  systemctl start containerd
  systemctl status containerd --no-pager
  ```
  wait for 2 minutes and then
  ```bash
  systemctl start kubelet
  ```
# Usefull Kubernetes commands
- Node selector
  ```bash
  kubectl label node n10 workload=monitoring
  kubectl get nodes --show-labels
  ```
  now you can deploy your resources specificly to that node
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: prometheus
    namespace: monitoring
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: prometheus
    template:
      metadata:
        labels:
          app: prometheus
      spec:
        nodeSelector:
          workload: monitoring
        containers:
          - name: prometheus
            image: 192.168.1.23:8084/prom/prometheus:latest
  ```
- Change a manifest for a resource and update (rollout update):
  execute the following command and edit your resource
  ```bash
  kubectl edit deployment grafana --namespace=my-grafana
  ```
  verify the rollout was successfull
  ```bash
  kubectl rollout status deployment grafana --namespace=my-grafana
  ```
  you can also annotate the update
  ```bash
  kubectl annotate deployment grafana --namespace=my-grafana kubernetes.io/change-cause='using grafana-dev:12.2.0-17161637292 for testing'
  ```
  view the rollout update history
  ```bash
  kubectl rollout history deployment/grafana --namespace=my-grafana
  ```
- Get an IP:Port address for a pod under a NodePort service:
  ```bash
  export NODE_PORT=$(kubectl get --namespace demo -o jsonpath="{.spec.ports[0].nodePort}" services podinfo)
  export NODE_IP=$(kubectl get nodes --namespace demo -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
  ```
