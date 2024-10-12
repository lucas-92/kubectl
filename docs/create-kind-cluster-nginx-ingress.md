Create a cluster with kind and nginx-ingress support:

```
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: cluster-kind-nginx-ingress-omega-strain
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
      hostPort: 8080      # Alterando a porta HTTP do host para 8080
      protocol: TCP
    - containerPort: 443  # Porta HTTPS dentro do container
      hostPort: 8443      # Alterando a porta HTTPS do host para 8443
      protocol: TCP
- role: worker
- role: worker
EOF
```

Mapeamento de portas:
É importante expor serviços Ingress diretamente no host, permitindo que você acesse os serviços através das portas padrão de HTTP e HTTPS sem precisar configurar portas alternativas. Para isso, mapeia as portas do host (sua máquina local) para as portas no container onde o nó control-plane está rodando. Essas portas são:
- 80: Porta HTTP.
- 443: Porta HTTPS.
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
