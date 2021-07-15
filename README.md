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
apiVersion: networking.k8s.io/v1beta1 
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
      - backend:
          serviceName: service-paotui-back-end
          servicePort: 80
        path: /api/v1
        pathType: Prefix
      - backend:
          serviceName: service-paotui-front-end
          servicePort: 80
        path: /

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
  replicas: 3
  template:
    metadata:
      labels:
        app: deployment-paotui-front-end
    spec:
      containers:
      - name: deployment-paotui-front-end
        image: magicpowerworld/paotui_front_end:20210714
        ports:
        - containerPort: 80
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
    replicas: 3
    template:
      metadata:
        labels:
          app: deployment-paotui-back-end
      spec:
        containers:
        - name: deployment-paotui-back-end
          image: magicpowerworld/paotui_back_end:20210714
          ports:
          - containerPort: 5000

```



