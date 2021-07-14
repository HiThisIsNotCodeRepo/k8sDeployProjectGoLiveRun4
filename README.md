# kubernetes Note
## What kubernetes can solve?
1. Handle server down.
2. Handle program hang.
3. Scale
## kubernete cluster set up
3 Master, 2 nodes
### IP address list


Token:
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


