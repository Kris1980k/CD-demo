# Requirements

### Update your system

`sudo dnf update -y`

### Install Docker

<pre>sudo dnf install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER && newgrp docker </pre>

### Install KubeCTL

<pre>curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client -o json</pre>

### Install K3s (lightweight kubernetes)

<pre>curl -sfL https://get.k3s.io | sh - 
sudo k3s kubectl get node 
alias kubectl='sudo k3s kubectl'
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
sudo chmod +r /etc/rancher/k3s/k3s.yaml
</pre>

# Install ArgoCD 

### Create namespace

`kubectl create namespace argocd`

### Install ArgoCD

<pre>kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get all --namespace=argocd</pre>

### Install ArgoCD CLI

<pre>```yaml curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64 ```</pre>

### Create a nodeport via YAML to connect to the service
<pre>
apiVersion: v1
kind: Service
metadata:
  name: argocd-server-nodeport
  namespace: argocd
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30007 # Specify the nodePort you want to use, or let Kubernetes allocate one for you.
  selector:
    app.kubernetes.io/name: argocd-server
</pre>

<pre>
kubectl apply -f argocd-server-nodeport.yaml
</pre>

<pre>

  kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl get svc -n argocd

</pre>
# Install Helm

<pre>curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh</pre>


# Install Crossplane

`helm repo add crossplane-stable https://charts.crossplane.io/stable`

`helm repo update`

<pre>helm install crossplane \
  --namespace crossplane-system \
  --create-namespace \
  crossplane-stable/crossplane \
  --kubeconfig $KUBECONFIG</pre>



# Create secret

<pre>kubectl create secret generic aws-secret \
  --namespace=crossplane-system \
  --from-file=creds=./aws-credentials.ini</pre>



