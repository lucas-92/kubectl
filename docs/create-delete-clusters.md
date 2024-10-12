## Create a k8s Cluster:
### 1. Kind:
#### 1.1 nginx-ingress
```
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: cluster-kind-nginx-ingress
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
    - containerPort: 80   # Porta HTTP dentro do container
      hostPort: 80      # Alterando a porta HTTP do host para 8080
      protocol: TCP
    - containerPort: 443  # Porta HTTPS dentro do container
      hostPort: 443      # Alterando a porta HTTPS do host para 8443
      protocol: TCP
- role: worker
- role: worker
EOF
```
Mapeamento de portas:
É importante expor serviços Ingress diretamente no host, permitindo que você acesse os serviços através das portas padrão de HTTP e HTTPS sem precisar configurar portas alternativas. Para isso, mapeia as portas do host (sua máquina local) para as portas no container onde o nó control-plane está rodando. Essas portas são: 80: Porta HTTP e 443: Porta HTTPS.
```
extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
```
ou
```
extraPortMappings:
  - containerPort: 80   # Porta HTTP dentro do container
    hostPort: 8080      # Alterando a porta HTTP do host para 8080
    protocol: TCP
  - containerPort: 443  # Porta HTTPS dentro do container
    hostPort: 8443      # Alterando a porta HTTPS do host para 8443
    protocol: TCP
```


Install the nginx-ingress:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Check the Pods of created nginx-ingress:
```
kubectl get pods -n ingress-nginx
kubectl get ingress -n ingress-nginx
```

#### 1.2 Control Plane Node Only Cluster
```
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: cluster-kind-control-plane-node-only
nodes:
  - role: control-plane
    extraPortMappings:
    - containerPort: 31437
      hostPort: 8080
      protocol: TCP
    - containerPort: 31438
      hostPort: 8443
      protocol: TCP
EOF
```

### 2. AWS:
Install eksctl:
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

Install AWS CLI:
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Get your AWS Credentials:
- AWS Access Key ID
- AWS Secret Access Key
- Default region name
- Default output format

Configure your AWS Credentials:
```
aws configure
```

Create the cluster:
```
eksctl create cluster --name=eks-cluster --version=1.24 --region=us-east-1 --nodegroup-name=eks-cluster-nodegroup --node-type=t3.medium --nodes=2 --nodes-min=1 --nodes-max=3 --managed
```
- The above command will create an EKS cluster with the name eks-cluster, in the us-east-1 region, with 2 nodes of type t3.medium, and with a minimum of 1 node and a maximum of 3 nodes. Additionally, the above command will create a nodegroup called eks-cluster-nodegroup. eksctl will take care of all the infrastructure necessary for the operation of our EKS cluster. The version of Kubernetes that will be installed on our cluster will be 1.24.

Install kubectl:
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

Config kubectl to use EKS Cluster:
```
aws eks --region us-east-1 update-kubeconfig --name eks-cluster
```
- Where us-east-1 is the region of our EKS cluster, and eks-cluster is the name of our EKS cluster. This command is necessary so that kubectl knows which cluster it should use, it will take the credentials from our EKS cluster and store it in the ~/.kube/config file.

Testing:
```
kubectl get nodes
```

## Delete the cluster:
### Kind:
```
kind delete cluster --name cluster-kind-nginx-ingress-omega-strain
```
ou (Para saber o ID-DO-CONTAINER: `docker container ls -a`)
```
docker container stop ID-DO-CONTAINER
docker contaier rm ID-DO-CONTAINER
```
### AWS:
```
eksctl delete cluster --name=eks-cluster -r us-east-1
```
