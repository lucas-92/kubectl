## 1 - Create the cluster:
1.1 - Run the command:
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
## 2 - Install CRDs of the GatewayAPI of nginx:
2.1 Applying the files:
```
kubectl kustomize "https://github.com/nginxinc/nginx-gateway-fabric/tree/main/config/crd/gateway-api/standard?ref=v1.4.0" | kubectl apply -f -
kubectl apply -f https://raw.githubusercontent.com/nginxinc/nginx-gateway-fabric/v1.4.0/deploy/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/nginxinc/nginx-gateway-fabric/v1.4.0/deploy/nodeport/deploy.yaml
```
2.2 - Check the created Pods:
```
kubectl get pods -n nginx-gateway
```

## 3 - Create and applying the yaml files:
3.1 - Create the file gateway.yaml:
```
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: my-gateway
spec:
  gatewayClassName: nginx
  listeners:
    - name: http
      protocol: HTTP
      port: 80
```
3.2 - Apply gateway.yaml:
```
kubectl apply -f gateway.yaml
``` 

3.3 - Create the file gateway-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-gateway
  namespace: nginx-gateway
  labels:
    app.kubernetes.io/name: nginx-gateway
    app.kubernetes.io/instance: nginx-gateway
    app.kubernetes.io/version: "1.4.0"
spec:
  type: NodePort
    selector:
      app.kubernetes.io/name: nginx-gateway
      app.kubernetes.io/instance: nginx-gateway
    ports:
      - name: http
        port: 80
        protocol: TCP
        targetPort: 80
        nodPort: 31437
      - name: https
        port: 443
        protocol: TCP
        targetPort: 443
        nodPort: 31438
```

3.4 - Apply gateway-service.yaml:
```
kubectl apply -f gateway-service.yaml
``` 
