## 1. Create the cluster:
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
## 2. Install CRDs of the GatewayAPI of nginx:
```
kubectl kustomize "https://github.com/nginxinc/nginx-gateway-fabric/tree/main/config/crd/gateway-api/standard?ref=v1.4.0" | kubectl apply -f -
kubectl apply -f https://raw.githubusercontent.com/nginxinc/nginx-gateway-fabric/v1.4.0/deploy/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/nginxinc/nginx-gateway-fabric/v1.4.0/deploy/nodeport/deploy.yaml
```
Check the created Pods:
```
kubectl get pods -n nginx-gateway
```
