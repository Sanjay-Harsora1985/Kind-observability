
# Terminal Command History for K8s Kind Voting App

## 1. Creating and Managing Kubernetes Cluster with Kind

- Clear terminal:
  ```bash
  clear
  ```

- Create a 3-node Kubernetes cluster using Kind:
  ```bash
  kind create cluster --config=config.yml
  ```

- Check cluster information:
  ```bash
  kubectl cluster-info --context kind-kind
  kubectl get nodes
  kind get clusters
  ```

---

## 2. Installing kubectl

- Download `kubectl` for managing Kubernetes clusters:
  ```bash
  curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
  chmod +x ./kubectl
  sudo mv ./kubectl /usr/local/bin
  kubectl version --short --client
  ```

---

## 3. Managing Docker and Kubernetes Pods

- Check Docker containers running:
  ```bash
  docker ps
  ```

- List all Kubernetes pods in all namespaces:
  ```bash
  kubectl get pods -A
  ```

---

## 4. Cloning and Running the Example Voting App

- Clone the voting app repository:
  ```bash
  git clone https://github.com/Sanjay-Harsora1985/Kind-observability.git
  cd Kind-observability/
  ```

- Apply Kubernetes YAML specifications for the voting app:
  ```bash
  kubectl apply -f k8s-specifications/
  ```

- List all Kubernetes resources:
  ```bash
  kubectl get all
  ```

- Forward local ports for accessing the voting and result apps:
  ```bash
  kubectl port-forward service/vote 5000:5000 --address=0.0.0.0 &
  kubectl port-forward service/result 5001:5001 --address=0.0.0.0 &
  ```

---

## 5. Managing Files in Example Voting App

- Navigate and view files:
  ```bash
  cd ..
  cd seed-data/
  ls
  cat Dockerfile
  cat generate-votes.sh
  ```

---

## 6. Installing Argo CD

- Create a namespace for Argo CD:
  ```bash
  kubectl create namespace argocd
  ```

- Apply the Argo CD manifest:
  ```bash
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```

- Check services in Argo CD namespace:
  ```bash
  kubectl get svc -n argocd
  ```

- Expose Argo CD server using NodePort:
  ```bash
  kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
  ```

- Forward ports to access Argo CD server:
  ```bash
  kubectl port-forward -n argocd service/argocd-server 8443:443 &
  ```

---

## 7. Argo CD Initial Admin Password
- Username: admin
- Retrieve Argo CD admin password:
  ```bash
  kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
  ```


---

## 8. Installing Kubernetes Dashboard

- Deploy Kubernetes dashboard:
  ```bash
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
  ```
- Create service Account for "admin-user"
    ```bash
  cd Kind-observability/kind-cluster
  kubectl apply -f dashboard-adminuser.yml
  ```
- Get service running in the namespace kubernetes-dashboard
  ```bash
  kubectl get svc -n kubernetes-dashboard
  ```
- Port-forward service name"kubernetes-dashboard" in the namespace kubernetes-dashboard
  ```bash
  kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8080:443 &
  ```
- Create a token for dashboard access:
  ```bash
  kubectl -n kubernetes-dashboard create token admin-user
  ```

---

## 9. Install HELM

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

---

## 10. Install Kube Prometheus Stack

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
kubectl create namespace monitoring
helm install kind-prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --set prometheus.service.nodePort=30000 --set prometheus.service.type=NodePort --set grafana.service.nodePort=31000 --set grafana.service.type=NodePort --set alertmanager.service.nodePort=32000 --set alertmanager.service.type=NodePort --set prometheus-node-exporter.service.nodePort=32001 --set prometheus-node-exporter.service.type=NodePort
kubectl get svc -n monitoring
kubectl get namespace
```

---

```bash
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0 &
```


---

## 11. Prometheus Queries

```bash
sum (rate (container_cpu_usage_seconds_total{namespace="default"}[1m])) / sum (machine_cpu_cores) * 100

sum (container_memory_usage_bytes{namespace="default"}) by (pod)


sum(rate(container_network_receive_bytes_total{namespace="default"}[5m])) by (pod)
sum(rate(container_network_transmit_bytes_total{namespace="default"}[5m])) by (pod)

```


---

## 12. Deleting Kubernetes Cluster

- Delete the Kind cluster:
  ```bash
  kind delete cluster --name=kind
  ```

---


