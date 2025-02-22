### Web Service
#### manifest.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    name: web
    tier: FE
    owner: Naveen
spec:
  containers:
  - name: web
    imagePullPolicy: Always
    image: naveen2809/web:2.0.1
    ports:
      - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: LoadBalancer
  selector:
    name: web
    tier: FE
    owner: Naveen
  ports:
  - port: 8081
    targetPort: 80
```
```
kubectl apply -f manifest.yml
```
#### Tunneling Service
```
minikube tunnel
```
![image](minikubeTunnel.png)
#### Pods & Services
![image](Pods-Svc.png)
#### HomePage
![image](HomePage.png)
```
kubectl delete -f manifest.yml
```