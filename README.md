# Deploy ProjectGoLiveRun4 on kubernetes
## What kubernetes can solve?
1. Handle server down.
2. Handle program hang.
3. Scale
## Architecture
![](https://i.imgur.com/PMP2vT8.png)
## kubernete cluster
![](https://i.imgur.com/Z9GJfmE.png)
## Deployment
![](https://i.imgur.com/rWwiAEF.png)
## Ingress
![](https://i.imgur.com/ooz23FM.png)
## Service
![](https://i.imgur.com/FlJ4xgY.png)
## Dashboard
![](https://i.imgur.com/2sP1kQX.png)
## Create secret for https
```shell=
kubectl create secret tls paotui-ingress-secret --cert=paotui.crt --key=paotui.key
```
## Deployment file
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/cors-allow-methods: 'PUT, GET, POST, OPTIONS'
    nginx.ingress.kubernetes.io/cors-allow-origin: '*'
    nginx.ingress.kubernetes.io/enable-cors: 'true'
  name: paotui-nginx-ingress
spec:
  tls:
  - hosts:
    - paotui.sg
    secretName: paotui-ingress-secret
  rules:
  - host: paotui.sg
    http:
      paths:
        - path: /api/v1
          pathType: Prefix
          backend:
            service:
              name: service-paotui-back-end
              port:
                number: 80
        - path: /
          pathType: Prefix
          backend:
            service:
              name: service-paotui-front-end
              port:
                number: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: service-paotui-front-end
  name: service-paotui-front-end
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: deployment-paotui-front-end
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: service-paotui-back-end
  name: service-paotui-back-end
spec:
  ports:
  - port: 80
    targetPort: 5000
    protocol: TCP
    name: http
  selector:
    app: deployment-paotui-back-end
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: deployment-paotui-front-end
  name: deployment-paotui-front-end
  namespace: default
spec:
  selector:
    matchLabels:
      app: deployment-paotui-front-end
  replicas: 6
  template:
    metadata:
      labels:
        app: deployment-paotui-front-end
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - deployment-paotui-front-end
              topologyKey: kubernetes.io/hostname
      containers:
      - name: deployment-paotui-front-end
        image: magicpowerworld/paotui_front_end:20210718
        imagePullPolicy: Always
        ports:
        - containerPort: 80
      tolerations:
      - effect: NoExecute
        key: node.kubernetes.io/unreachable
        operator: Exists
        tolerationSeconds: 10
      - effect: NoExecute
        key: node.kubernetes.io/not-ready
        operator: Exists
        tolerationSeconds: 10
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: deployment-paotui-back-end
  name: deployment-paotui-back-end
  namespace: default
spec:
  selector:
    matchLabels:
      app: deployment-paotui-back-end
  replicas: 6
  template:
    metadata:
      labels:
        app: deployment-paotui-back-end
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - deployment-paotui-back-end
              topologyKey: kubernetes.io/hostname
      containers:
      - name: deployment-paotui-back-end
        image: magicpowerworld/paotui_back_end:20210719
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
        envFrom:
        - configMapRef:
            name: paotui-be-cm
      tolerations:
      - effect: NoExecute
        key: node.kubernetes.io/unreachable
        operator: Exists
        tolerationSeconds: 10
      - effect: NoExecute
        key: node.kubernetes.io/not-ready
        operator: Exists
        tolerationSeconds: 10

```
## Using busybox debug service
```shell=
# 1
cat<<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
EOF
# 2
kubectl exec -ti busybox -- sh
# 3
curl service-paotui-front-end
```
## Using configmap store config data
```yaml=
      - name: deployment-paotui-back-end
        image: magicpowerworld/paotui_back_end:20210718
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
        envFrom:
        - configMapRef:
            name: paotui-be-cm
```
## Manual overwrite pod eviction second
When node become notready state the pod will leave the node in 10sec. 
```yaml=
      tolerations:
      - effect: NoExecute
        key: node.kubernetes.io/unreachable
        operator: Exists
        tolerationSeconds: 10
      - effect: NoExecute
        key: node.kubernetes.io/not-ready
        operator: Exists
        tolerationSeconds: 10
```
## Ensure pods spread evenly across nodes
Affinity define how pods creation distributed across nodes, podAntiAffinity will ensure not all pods created on one single host. It enhances pods availability. 
```yaml=
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - deployment-paotui-front-end
              topologyKey: kubernetes.io/hostname
```
![](https://i.imgur.com/hnZbtNR.png)
## kube-prometheus
It enable use Prometheus to monitor Kubernetes and applications running on Kubernetes, [link](https://github.com/prometheus-operator/kube-prometheus)
![image](https://user-images.githubusercontent.com/43861132/126287492-e41a30a3-62fe-4875-959e-b7ec266e2da6.png)
### Installation Guide
```shell
# 1
git clone -b release-0.8 --single-branch https://github.com/coreos/kube-prometheus.git
# 2
kubectl create -f kube-prometheus/manifests/setup
# 3
kubectl create -f kube-prometheus/manifests
```
### Create Ingress
```yaml=
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "nginx"
  name: paotui-prom-ingress
  namespace: monitoring
spec:
  tls:
  - hosts:
    - paotui.sg
    secretName: paotui-ingress-secret
  rules:
  - host: paotui.sg
    http:
      paths:
        - path: /alert
          pathType: Prefix
          backend:
            service:
              name: alertmanager-main
              port:
                number: 9093
        - path: /grafana
          pathType: Prefix
          backend:
            service:
              name: grafana
              port:
                number: 3000
        - path: /prom
          pathType: Prefix
          backend:
            service:
              name: prometheus-k8s
              port:
                number: 9090
```
![image](https://user-images.githubusercontent.com/43861132/126285780-fe4fc510-8435-4d5a-8457-872b74edfa16.png)
![image](https://user-images.githubusercontent.com/43861132/126285793-faab14c8-2cb9-4ad2-a12a-b527893613de.png)
![image](https://user-images.githubusercontent.com/43861132/126285820-bf2fe0bf-798a-41cc-9dc2-f820a783ede3.png)

