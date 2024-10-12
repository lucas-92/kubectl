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

## 3 - Create the Gateway and Service files:
3.1 - Create the file gateway.yaml and apply:
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
```
kubectl apply -f gateway.yaml
``` 

3.2 - Create the file gateway-service.yaml, check the port number and apply:
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
```
kubectl get svc -n nginx-gateway
```
```
kubectl apply -f gateway-service.yaml
kubectl get svc -n nginx-gateway
```

3.5 - Access the 404 Nginx page:
```
curl localhost:8080
```

Congrats, Nginx is working!

## 4 - Creating the Deployment of Apps files:
4.1 - Create the app1.yaml and apply:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 2
  selector
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
        - name: app1
          image: hashicorp/http-echo
          args:
            - "-text=Hello from APP1"
          ports:
            - containerPort: 5678
```
```
kubectl apply -f app1.yaml
```
4.2 - Create the app2.yaml and apply:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 2
  selector
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
        - name: app1
          image: hashicorp/http-echo
          args:
            - "-text=Hello from APP2"
          ports:
            - containerPort: 5678
```
```
kubectl apply -f app2.yaml
```
## 5 - Creating the Service of Apps files:
5.1 - Create the app1-service.yaml and apply:
```
apiVersion: v1
kind: Service
metadata:
  name: app1-service
spec:
  selector:
    app: app1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678
```
```
kubectl apply -f app1-service.yaml
```
5.2 - Create the app2-service.yaml and apply:
```
apiVersion: v1
kind: Service
metadata:
  name: app2-service
spec:
  selector:
    app: app2
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678
```
```
kubectl apply -f app2-service.yaml
```
## 6 - Create the HTTP Route files:
6.1 - Create app-http_route.yaml and apply:
```
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: app1-app2-route
spec:
  parentRefs:
    - name: my-gateway
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: "/app1"
      backendRefs:
        - name: app1-service
          port: 80
    - matches:
        - path:
            type: PathPrefix
            value: "/app2"
      backendRefs:
        - name: app2-service
          port: 80
```
```
kubectl apply -f app-http_route.yaml 
```

6.1 - Checking app1:
```
curl localhost:8080/app1
```

6.2 - Checking app1:
```
curl localhost:8080/app2
```
