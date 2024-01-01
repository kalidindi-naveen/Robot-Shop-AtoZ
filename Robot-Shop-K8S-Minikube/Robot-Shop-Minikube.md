### Create Minikube Cluster
```
minikube start
```
```
kubectl apply -f namespace.yml
namespace/robot-shop created

kubectl apply -f storage-class.yml
storageclass.storage.k8s.io/local-storage created

kubectl get sc
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
local-storage        k8s.io/minikube-hostpath   Delete          Immediate           false                  30s
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  84m
```
### MongoDB
```
kubectl apply -f manifest.yml
statefulset.apps/mongodb created
service/mongodb created

kubectl get pods -n robot-shop
NAME        READY   STATUS    RESTARTS   AGE
mongodb-0   1/1     Running   0          37s

kubectl get svc -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
mongodb     ClusterIP   None             <none>        27017/TCP   100s

kubectl get pv -n robot-shop
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                 STORAGECLASS    REASON   AGE
pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0   local-storage            6m6s

kubectl get pvc -n robot-shop
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0   Bound    pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            local-storage   5m48s
```
### Catalogue
```
kubectl apply -f manifest.yml
configmap/catalogue-config created
deployment.apps/catalogue created
service/catalogue created

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
catalogue-55c5d85dc7-m5xn8   1/1     Running   0          5m57s
mongodb-0                    1/1     Running   0          7m21s

kubectl get svc -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
catalogue   ClusterIP   10.100.171.157   <none>        8080/TCP    6m14s
mongodb     ClusterIP   None             <none>        27017/TCP   7m38s

kubectl logs -f catalogue-55c5d85dc7-m5xn8 -n robot-shop
{"level":"info","time":1703684893974,"pid":1,"hostname":"catalogue-55c5d85dc7-m5xn8","msg":"MongoDB connected","v":1}
```
### Redis
```
kubectl apply -f manifest.yml
statefulset.apps/redis created
service/redis created

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
catalogue-55c5d85dc7-m5xn8   1/1     Running   0          10m
mongodb-0                    1/1     Running   0          11m
redis-0                      1/1     Running   0          16s

kubectl get svc -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
catalogue   ClusterIP   10.100.171.157   <none>        8080/TCP    10m
mongodb     ClusterIP   None             <none>        27017/TCP   11m
redis       ClusterIP   None             <none>        6379/TCP    31s

kubectl get pv -n robot-shop
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM
         STORAGECLASS    REASON   AGE
pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0   local-storage            12m
pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            Delete           Bound    robot-shop/redis-volume-redis-0       local-storage            53s

kubectl get pvc -n robot-shop
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0   Bound    pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            local-storage   12m
redis-volume-redis-0       Bound    pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            local-storage   70s
```
### MySql
```
kubectl apply -f manifest.yml
configmap/mysql-config created
secret/mysql-secret created
statefulset.apps/mysql created
service/mysql created

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
catalogue-55c5d85dc7-m5xn8   1/1     Running   0          12m
mongodb-0                    1/1     Running   0          14m
mysql-0                      1/1     Running   0          25s
redis-0                      1/1     Running   0          3m2s

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE     SELECTOR
catalogue   ClusterIP   10.100.171.157   <none>        8080/TCP    13m     name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP   14m     name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP    49s     name=mysql,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP    3m26s   name=redis,owner=Naveen,tier=DB

kubectl get pv -n robot-shop
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM
         STORAGECLASS    REASON   AGE
pvc-5278ba50-5551-4fb2-8847-cb24262803b3   1Gi        RWO            Delete           Bound    robot-shop/mysql-volume-mysql-0       local-storage            87s
pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0   local-storage            15m
pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            Delete           Bound    robot-shop/redis-volume-redis-0       local-storage            4m4s

kubectl get pvc -n robot-shop
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0   Bound    pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            local-storage   15m
mysql-volume-mysql-0       Bound    pvc-5278ba50-5551-4fb2-8847-cb24262803b3   1Gi        RWO            local-storage   106s
redis-volume-redis-0       Bound    pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            local-storage   4m23s
```
### RabbitMQ
```
kubectl apply -f manifest.yml 
service/rabbitmq created
statefulset.apps/rabbitmq created

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
catalogue-55c5d85dc7-m5xn8   1/1     Running   0          18m
mongodb-0                    1/1     Running   0          19m
mysql-0                      1/1     Running   0          6m3s
rabbitmq-0                   1/1     Running   0          44s
redis-0                      1/1     Running   0          8m40s

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE     SELECTOR
catalogue   ClusterIP   10.100.171.157   <none>        8080/TCP             19m     name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            20m     name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             6m30s   name=mysql,owner=Naveen,tier=DB
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   71s     name=rabbitmq,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP             9m7s    name=redis,owner=Naveen,tier=DB

kubectl get pv -n robot-shop
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM
           STORAGECLASS    REASON   AGE
pvc-147c9d33-5a92-487e-b96c-f04a3bd1ed9a   1Gi        RWO            Delete           Bound    robot-shop/rabbitmq-volume-rabbitmq-0   local-storage            92s
pvc-5278ba50-5551-4fb2-8847-cb24262803b3   1Gi        RWO            Delete           Bound    robot-shop/mysql-volume-mysql-0         local-storage            6m51s
pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0     local-storage            20m
pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            Delete           Bound    robot-shop/redis-volume-redis-0         local-storage            9m28s

kubectl get pvc -n robot-shop
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0     Bound    pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            local-storage   20m
mysql-volume-mysql-0         Bound    pvc-5278ba50-5551-4fb2-8847-cb24262803b3   1Gi        RWO            local-storage   6m56s
rabbitmq-volume-rabbitmq-0   Bound    pvc-147c9d33-5a92-487e-b96c-f04a3bd1ed9a   1Gi        RWO            local-storage   97s
redis-volume-redis-0         Bound    pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            local-storage   9m33s
```
### User
```
kubectl apply -f .\manifest.yml
configmap/user-config created
deployment.apps/user created
service/user created

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
catalogue-55c5d85dc7-m5xn8   1/1     Running   0          40m
mongodb-0                    1/1     Running   0          41m
mysql-0                      1/1     Running   0          27m
rabbitmq-0                   1/1     Running   0          22m
redis-0                      1/1     Running   0          30m
user-6bdd466b69-x9bp4        1/1     Running   0          3m4s

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE     SELECTOR
catalogue   ClusterIP   10.100.171.157   <none>        8080/TCP             40m     name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            41m     name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             27m     name=mysql,owner=Naveen,tier=DB
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   22m     name=rabbitmq,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP             30m     name=redis,owner=Naveen,tier=DB
user        ClusterIP   10.108.83.158    <none>        8080/TCP             3m26s   name=user,owner=Naveen,tier=App
```
### Cart
```
kubectl apply -f manifest.yml
configmap/cart-config created
deployment.apps/cart created
service/cart created

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
cart-59f979596c-wxl74        1/1     Running   0          28s
catalogue-55c5d85dc7-m5xn8   1/1     Running   0          82m
mongodb-0                    1/1     Running   0          83m
mysql-0                      1/1     Running   0          69m
rabbitmq-0                   1/1     Running   0          64m
redis-0                      1/1     Running   0          72m
user-6bdd466b69-x9bp4        1/1     Running   0          45m

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE   SELECTOR
cart        ClusterIP   10.106.232.202   <none>        8080/TCP             50s   name=cart,owner=Naveen,tier=App
catalogue   ClusterIP   10.100.171.157   <none>        8080/TCP             82m   name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            84m   name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             70m   name=mysql,owner=Naveen,tier=DB
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   64m   name=rabbitmq,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP             72m   name=redis,owner=Naveen,tier=DB
user        ClusterIP   10.108.83.158    <none>        8080/TCP             45m   name=user,owner=Naveen,tier=App
```
### Shipping
```
kubectl apply -f manifest.yml
configmap/shipping-config created
deployment.apps/shipping created
service/shipping created

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
cart-59f979596c-wxl74        1/1     Running   0          4m58s
catalogue-55c5d85dc7-m5xn8   1/1     Running   0          86m
mongodb-0                    1/1     Running   0          88m
mysql-0                      1/1     Running   0          74m
rabbitmq-0                   1/1     Running   0          68m
redis-0                      1/1     Running   0          76m
shipping-66b78d4fc4-hvrhw    1/1     Running   0          54s
user-6bdd466b69-x9bp4        1/1     Running   0          49m

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE     SELECTOR
cart        ClusterIP   10.106.232.202   <none>        8080/TCP             5m17s   name=cart,owner=Naveen,tier=App
catalogue   ClusterIP   10.100.171.157   <none>        8080/TCP             87m     name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            88m     name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             74m     name=mysql,owner=Naveen,tier=DB
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   69m     name=rabbitmq,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP             77m     name=redis,owner=Naveen,tier=DB
shipping    ClusterIP   10.100.249.127   <none>        8080/TCP             73s     name=shipping,owner=Naveen,tier=App
user        ClusterIP   10.108.83.158    <none>        8080/TCP             50m     name=user,owner=Naveen,tier=App
```
### Payment
```
kubectl apply -f manifest.yml
configmap/payment-config created
deployment.apps/payment created
service/payment created

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
cart-59f979596c-wxl74        1/1     Running   0          39m
catalogue-55c5d85dc7-m5xn8   1/1     Running   0          120m
mongodb-0                    1/1     Running   0          122m
mysql-0                      1/1     Running   0          108m
payment-6659f94c87-6272c     1/1     Running   0          2m16s
rabbitmq-0                   1/1     Running   0          103m
redis-0                      1/1     Running   0          111m
shipping-66b78d4fc4-hvrhw    1/1     Running   0          35m
user-6bdd466b69-x9bp4        1/1     Running   0          83m

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE     SELECTOR
cart        ClusterIP   10.106.232.202   <none>        8080/TCP             39m     name=cart,owner=Naveen,tier=App
catalogue   ClusterIP   10.100.171.157   <none>        8080/TCP             121m    name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            122m    name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             108m    name=mysql,owner=Naveen,tier=DB
payment     ClusterIP   10.98.179.192    <none>        8080/TCP             2m33s   name=payment,owner=Naveen,tier=App
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   103m    name=rabbitmq,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP             111m    name=redis,owner=Naveen,tier=DB
shipping    ClusterIP   10.100.249.127   <none>        8080/TCP             35m     name=shipping,owner=Naveen,tier=App
user        ClusterIP   10.108.83.158    <none>        8080/TCP             84m     name=user,owner=Naveen,tier=App
```
### Ratings
```
kubectl apply -f manifest.yml 
deployment.apps/ratings created
service/ratings created

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
cart-59f979596c-wxl74        1/1     Running   0          47m
catalogue-55c5d85dc7-m5xn8   1/1     Running   0          129m
mongodb-0                    1/1     Running   0          131m
mysql-0                      1/1     Running   0          117m
payment-6659f94c87-2h7pm     1/1     Running   0          26s
rabbitmq-0                   1/1     Running   0          111m
ratings-68bd5586b7-llqx6     1/1     Running   0          2m8s
redis-0                      1/1     Running   0          119m
shipping-66b78d4fc4-hvrhw    1/1     Running   0          43m
user-6bdd466b69-x9bp4        1/1     Running   0          92m

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE    SELECTOR
cart        ClusterIP   10.106.232.202   <none>        8080/TCP             47m    name=cart,owner=Naveen,tier=App
catalogue   ClusterIP   10.100.171.157   <none>        8080/TCP             129m   name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            131m   name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             117m   name=mysql,owner=Naveen,tier=DB
payment     ClusterIP   10.101.137.157   <none>        8080/TCP             22s    name=payment,owner=Naveen,tier=App
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   111m   name=rabbitmq,owner=Naveen,tier=DB
ratings     ClusterIP   10.108.222.140   <none>        80/TCP               2m4s   name=ratings,owner=Naveen,tier=App
redis       ClusterIP   None             <none>        6379/TCP             119m   name=redis,owner=Naveen,tier=DB
shipping    ClusterIP   10.100.249.127   <none>        8080/TCP             43m    name=shipping,owner=Naveen,tier=App
user        ClusterIP   10.108.83.158    <none>        8080/TCP             92m    name=user,owner=Naveen,tier=App
```
### Web
```
kubectl apply -f .\manifest.yml
configmap/nginx-config created
deployment.apps/web created
service/web created
horizontalpodautoscaler.autoscaling/web-hpa created

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
cart-59f979596c-wxl74        1/1     Running   0          58m
catalogue-55c5d85dc7-m5xn8   1/1     Running   0          140m
mongodb-0                    1/1     Running   0          141m
mysql-0                      1/1     Running   0          127m
payment-6659f94c87-2h7pm     1/1     Running   0          11m
rabbitmq-0                   1/1     Running   0          122m
ratings-68bd5586b7-llqx6     1/1     Running   0          12m
redis-0                      1/1     Running   0          130m
shipping-66b78d4fc4-hvrhw    1/1     Running   0          54m
user-6bdd466b69-x9bp4        1/1     Running   0          103m
web-58598748d4-28sqx         1/1     Running   0          91s

kubectl get svc -o wide -n robot-shop
NAME        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE     SELECTOR
cart        ClusterIP      10.106.232.202   <none>        8080/TCP             58m     name=cart,owner=Naveen,tier=App
catalogue   ClusterIP      10.100.171.157   <none>        8080/TCP             140m    name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP      None             <none>        27017/TCP            141m    name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP      None             <none>        3306/TCP             127m    name=mysql,owner=Naveen,tier=DB
payment     ClusterIP      10.101.137.157   <none>        8080/TCP             11m     name=payment,owner=Naveen,tier=App
rabbitmq    ClusterIP      None             <none>        5672/TCP,15672/TCP   122m    name=rabbitmq,owner=Naveen,tier=DB
ratings     ClusterIP      10.108.222.140   <none>        80/TCP               12m     name=ratings,owner=Naveen,tier=App
redis       ClusterIP      None             <none>        6379/TCP             130m    name=redis,owner=Naveen,tier=DB
shipping    ClusterIP      10.100.249.127   <none>        8080/TCP             54m     name=shipping,owner=Naveen,tier=App
user        ClusterIP      10.108.83.158    <none>        8080/TCP             103m    name=user,owner=Naveen,tier=App
web         LoadBalancer   10.105.152.54    127.0.0.1     8081:30760/TCP       2m16s   name=web,owner=Naveen,tier=FE
```
### Apply Load on Web
```
Install Mertrics server
https://github.com/kubernetes-sigs/metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

Install Apache Beanch on aws linux
https://ubiq.co/tech-blog/how-to-use-apache-bench-for-load-testing/

ab -k -n 1000000 -c 10000 <LB Link> 

kubectl top pod

we can see Autoscale happening
```