### Create Minikube Cluster
```
minikube start
```
### EKS-Admin
```
helm install eks-admin .
NAME: eks-admin
LAST DEPLOYED: Thu Dec 28 22:19:31 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
### MongoDB
```
helm install mongodb .
NAME: mongodb
LAST DEPLOYED: Thu Dec 28 22:22:26 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

kubectl get pods -n robot-shop
NAME        READY   STATUS    RESTARTS   AGE
mongodb-0   1/1     Running   0          38s

kubectl get svc -n robot-shop
NAME      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
mongodb   ClusterIP   None         <none>        27017/TCP   54s

kubectl get pv              
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                 STORAGECLASS    REASON   AGE
pvc-34fc58be-efd6-4a3c-ae46-79b0b7966bb7   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0   local-storage            81s

kubectl get pvc -n robot-shop
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0   Bound    pvc-34fc58be-efd6-4a3c-ae46-79b0b7966bb7   1Gi        RWO            local-storage   2m22s
```
### Catalogue
```
helm install catalogue .      
NAME: catalogue
LAST DEPLOYED: Thu Dec 28 22:27:52 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
catalogue-55c5d85dc7-rkg6x   1/1     Running   0          81s
mongodb-0                    1/1     Running   0          6m47s

kubectl get svc -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
catalogue   ClusterIP   10.103.253.255   <none>        8080/TCP    86s
mongodb     ClusterIP   None             <none>        27017/TCP   6m52s

kubectl logs -f catalogue-55c5d85dc7-rkg6x -n robot-shop
{"level":"info","time":1703782682784,"pid":1,"hostname":"catalogue-55c5d85dc7-rkg6x","msg":"MongoDB connected","v":1}
```
### Redis
```
helm install redis .
NAME: redis
LAST DEPLOYED: Thu Dec 28 22:32:18 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
catalogue-55c5d85dc7-rkg6x   1/1     Running   0          4m39s
mongodb-0                    1/1     Running   0          10m
redis-0                      1/1     Running   0          13s

kubectl get svc -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
catalogue   ClusterIP   10.103.253.255   <none>        8080/TCP    4m43s
mongodb     ClusterIP   None             <none>        27017/TCP   10m
redis       ClusterIP   None             <none>        6379/TCP    17s

kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                 STORAGECLASS    REASON   AGE
pvc-2188983d-79a5-48e3-864b-141ab45594a2   30Gi       RWO            Delete           Bound    default/data-elasticsearch-data-0     standard                 46m
pvc-34fc58be-efd6-4a3c-ae46-79b0b7966bb7   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0   local-storage            10m
pvc-6cb2e171-749a-43c5-ba1c-db06fe3749e8   1Gi        RWO            Delete           Bound    robot-shop/redis-volume-redis-0       local-storage            22s
pvc-96e19b71-5c59-450a-97bb-f46b09814784   4Gi        RWO            Delete           Bound    default/data-elasticsearch-master-0   standard                 46m

kubectl get pvc -n robot-shop
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0   Bound    pvc-34fc58be-efd6-4a3c-ae46-79b0b7966bb7   1Gi        RWO            local-storage   10m
redis-volume-redis-0       Bound    pvc-6cb2e171-749a-43c5-ba1c-db06fe3749e8   1Gi        RWO            local-storage   26s
```
### MySql
```
helm install mysql .
NAME: mysql
LAST DEPLOYED: Thu Dec 28 22:37:05 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
catalogue-55c5d85dc7-rkg6x   1/1     Running   0          9m54s
mongodb-0                    1/1     Running   0          15m
mysql-0                      1/1     Running   0          41s
redis-0                      1/1     Running   0          5m28s

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE    SELECTOR
catalogue   ClusterIP   10.103.253.255   <none>        8080/TCP    10m    name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP   15m    name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP    76s    name=mysql,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP    6m3s   name=redis,owner=Naveen,tier=DB

kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                 STORAGECLASS    REASON   AGE
pvc-2188983d-79a5-48e3-864b-141ab45594a2   30Gi       RWO            Delete           Bound    default/data-elasticsearch-data-0     standard                 51m
pvc-34fc58be-efd6-4a3c-ae46-79b0b7966bb7   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0   local-storage            15m
pvc-6cb2e171-749a-43c5-ba1c-db06fe3749e8   1Gi        RWO            Delete           Bound    robot-shop/redis-volume-redis-0       local-storage            5m21s
pvc-96e19b71-5c59-450a-97bb-f46b09814784   4Gi        RWO            Delete           Bound    default/data-elasticsearch-master-0   standard                 51m
pvc-c435900f-1de3-4e61-9971-3acd7240132e   1Gi        RWO            Delete           Bound    robot-shop/mysql-volume-mysql-0       local-storage            34s

kubectl get pvc -n robot-shop
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0   Bound    pvc-34fc58be-efd6-4a3c-ae46-79b0b7966bb7   1Gi        RWO            local-storage   15m
mysql-volume-mysql-0       Bound    pvc-c435900f-1de3-4e61-9971-3acd7240132e   1Gi        RWO            local-storage   37s
redis-volume-redis-0       Bound    pvc-6cb2e171-749a-43c5-ba1c-db06fe3749e8   1Gi        RWO            local-storage   5m24s
```
### RabbitMQ
```
helm install rabbitmq .
NAME: rabbitmq
LAST DEPLOYED: Thu Dec 28 22:44:16 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
catalogue-55c5d85dc7-rkg6x   1/1     Running   0          18m
mongodb-0                    1/1     Running   0          23m
mysql-0                      1/1     Running   0          9m4s
rabbitmq-0                   1/1     Running   0          113s
redis-0                      1/1     Running   0          13m

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE     SELECTOR
catalogue   ClusterIP   10.103.253.255   <none>        8080/TCP             18m     name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            23m     name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             9m19s   name=mysql,owner=Naveen,tier=DB
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   2m8s    name=rabbitmq,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP             14m     name=redis,owner=Naveen,tier=DB

kubectl get pv              
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                   STORAGECLASS    REASON   AGE
pvc-2188983d-79a5-48e3-864b-141ab45594a2   30Gi       RWO            Delete           Bound    default/data-elasticsearch-data-0       standard                 60m
pvc-34fc58be-efd6-4a3c-ae46-79b0b7966bb7   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0     local-storage            24m
pvc-61694d81-1cf9-4082-a2d3-c554981a22d3   1Gi        RWO            Delete           Bound    robot-shop/rabbitmq-volume-rabbitmq-0   local-storage            2m27s
pvc-6cb2e171-749a-43c5-ba1c-db06fe3749e8   1Gi        RWO            Delete           Bound    robot-shop/redis-volume-redis-0         local-storage            14m
pvc-96e19b71-5c59-450a-97bb-f46b09814784   4Gi        RWO            Delete           Bound    default/data-elasticsearch-master-0     standard                 60m
pvc-c435900f-1de3-4e61-9971-3acd7240132e   1Gi        RWO            Delete           Bound    robot-shop/mysql-volume-mysql-0         local-storage            9m38s

kubectl get pvc -n robot-shop
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0     Bound    pvc-34fc58be-efd6-4a3c-ae46-79b0b7966bb7   1Gi        RWO            local-storage   24m
mysql-volume-mysql-0         Bound    pvc-c435900f-1de3-4e61-9971-3acd7240132e   1Gi        RWO            local-storage   10m
rabbitmq-volume-rabbitmq-0   Bound    pvc-61694d81-1cf9-4082-a2d3-c554981a22d3   1Gi        RWO            local-storage   2m53s
redis-volume-redis-0         Bound    pvc-6cb2e171-749a-43c5-ba1c-db06fe3749e8   1Gi        RWO            local-storage   14m
```
### User
```
helm install user .
NAME: user
LAST DEPLOYED: Thu Dec 28 22:49:23 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

kubectl get pods -n robot-shop       
NAME                         READY   STATUS    RESTARTS   AGE
catalogue-55c5d85dc7-rkg6x   1/1     Running   0          27m
mongodb-0                    1/1     Running   0          32m
mysql-0                      1/1     Running   0          17m
rabbitmq-0                   1/1     Running   0          10m
redis-0                      1/1     Running   0          22m
user-6bdd466b69-jdbrt        1/1     Running   0          5m29s

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE     SELECTOR
catalogue   ClusterIP   10.103.253.255   <none>        8080/TCP             30m     name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            35m     name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             20m     name=mysql,owner=Naveen,tier=DB
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   13m     name=rabbitmq,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP             25m     name=redis,owner=Naveen,tier=DB
user        ClusterIP   10.97.54.229     <none>        8080/TCP             8m36s   name=user,owner=Naveen,tier=App
```
### Cart
```
helm install cart .
NAME: cart
LAST DEPLOYED: Thu Dec 28 22:59:55 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

kubectl get pods -n robot-shop       
NAME                         READY   STATUS    RESTARTS   AGE
cart-59f979596c-khw8t        1/1     Running   0          5m21s
catalogue-55c5d85dc7-rkg6x   1/1     Running   0          37m
mongodb-0                    1/1     Running   0          42m
mysql-0                      1/1     Running   0          28m
rabbitmq-0                   1/1     Running   0          21m
redis-0                      1/1     Running   0          32m
user-6bdd466b69-jdbrt        1/1     Running   0          15m

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE     SELECTOR
cart        ClusterIP   10.103.107.69    <none>        8080/TCP             5m34s   name=cart,owner=Naveen,tier=App
catalogue   ClusterIP   10.103.253.255   <none>        8080/TCP             37m     name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            43m     name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             28m     name=mysql,owner=Naveen,tier=DB
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   21m     name=rabbitmq,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP             33m     name=redis,owner=Naveen,tier=DB
user        ClusterIP   10.97.54.229     <none>        8080/TCP             16m     name=user,owner=Naveen,tier=App
```
### Shipping
```
helm install shipping .
NAME: shipping
LAST DEPLOYED: Thu Dec 28 23:07:15 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

kubectl get pods -n robot-shop       
NAME                         READY   STATUS    RESTARTS   AGE
cart-59f979596c-khw8t        1/1     Running   0          8m41s
catalogue-55c5d85dc7-rkg6x   1/1     Running   0          40m
mongodb-0                    1/1     Running   0          46m
mysql-0                      1/1     Running   0          31m
rabbitmq-0                   1/1     Running   0          24m
redis-0                      1/1     Running   0          36m
shipping-66b78d4fc4-mhgkv    1/1     Running   0          81s
user-6bdd466b69-jdbrt        1/1     Running   0          19m

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE     SELECTOR
cart        ClusterIP   10.103.107.69    <none>        8080/TCP             8m44s   name=cart,owner=Naveen,tier=App
catalogue   ClusterIP   10.103.253.255   <none>        8080/TCP             40m     name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            46m     name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             31m     name=mysql,owner=Naveen,tier=DB
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   24m     name=rabbitmq,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP             36m     name=redis,owner=Naveen,tier=DB
shipping    ClusterIP   10.103.123.124   <none>        8080/TCP             84s     name=shipping,owner=Naveen,tier=App
user        ClusterIP   10.97.54.229     <none>        8080/TCP             19m     name=user,owner=Naveen,tier=App
```
### Payment
```
helm install payment .
NAME: payment
LAST DEPLOYED: Thu Dec 28 23:23:38 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

kubectl get pods -n robot-shop       
NAME                         READY   STATUS    RESTARTS   AGE
cart-59f979596c-khw8t        1/1     Running   0          25m
catalogue-55c5d85dc7-rkg6x   1/1     Running   0          57m
mongodb-0                    1/1     Running   0          63m
mysql-0                      1/1     Running   0          48m
payment-6659f94c87-fpmgn     1/1     Running   0          2m10s
rabbitmq-0                   1/1     Running   0          41m
redis-0                      1/1     Running   0          53m
shipping-66b78d4fc4-mhgkv    1/1     Running   0          18m
user-6bdd466b69-jdbrt        1/1     Running   0          36m

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE     SELECTOR
cart        ClusterIP   10.103.107.69    <none>        8080/TCP             25m     name=cart,owner=Naveen,tier=App
catalogue   ClusterIP   10.103.253.255   <none>        8080/TCP             57m     name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            63m     name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             48m     name=mysql,owner=Naveen,tier=DB
payment     ClusterIP   10.109.199.199   <none>        8080/TCP             2m12s   name=payment,owner=Naveen,tier=App
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   41m     name=rabbitmq,owner=Naveen,tier=DB
redis       ClusterIP   None             <none>        6379/TCP             53m     name=redis,owner=Naveen,tier=DB
shipping    ClusterIP   10.103.123.124   <none>        8080/TCP             18m     name=shipping,owner=Naveen,tier=App
user        ClusterIP   10.97.54.229     <none>        8080/TCP             36m     name=user,owner=Naveen,tier=App
```
### Ratings
```
helm install ratings .
NAME: ratings
LAST DEPLOYED: Thu Dec 28 23:37:43 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

kubectl get pods -n robot-shop
NAME                         READY   STATUS    RESTARTS   AGE
cart-59f979596c-khw8t        1/1     Running   0          38m
catalogue-55c5d85dc7-rkg6x   1/1     Running   0          70m
mongodb-0                    1/1     Running   0          75m
mysql-0                      1/1     Running   0          61m
payment-6659f94c87-fpmgn     1/1     Running   0          14m
rabbitmq-0                   1/1     Running   0          53m
ratings-54cf5d68f7-77qx6     1/1     Running   0          29s
redis-0                      1/1     Running   0          65m
shipping-66b78d4fc4-mhgkv    1/1     Running   0          30m
user-6bdd466b69-jdbrt        1/1     Running   0          48m

kubectl get svc -o wide -n robot-shop
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE   SELECTOR
cart        ClusterIP   10.103.107.69    <none>        8080/TCP             38m   name=cart,owner=Naveen,tier=App
catalogue   ClusterIP   10.103.253.255   <none>        8080/TCP             70m   name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP   None             <none>        27017/TCP            75m   name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP   None             <none>        3306/TCP             61m   name=mysql,owner=Naveen,tier=DB
payment     ClusterIP   10.109.199.199   <none>        8080/TCP             14m   name=payment,owner=Naveen,tier=App
rabbitmq    ClusterIP   None             <none>        5672/TCP,15672/TCP   54m   name=rabbitmq,owner=Naveen,tier=DB
ratings     ClusterIP   10.106.254.57    <none>        80/TCP               36s   name=ratings,owner=Naveen,tier=App
redis       ClusterIP   None             <none>        6379/TCP             66m   name=redis,owner=Naveen,tier=DB
shipping    ClusterIP   10.103.123.124   <none>        8080/TCP             31m   name=shipping,owner=Naveen,tier=App
user        ClusterIP   10.97.54.229     <none>        8080/TCP             48m   name=user,owner=Naveen,tier=App
```
### Web
```
helm install web .
NAME: web
LAST DEPLOYED: Thu Dec 28 23:49:14 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

kubectl get pods -n robot-shop       
NAME                         READY   STATUS    RESTARTS   AGE
cart-59f979596c-khw8t        1/1     Running   0          51m
catalogue-55c5d85dc7-rkg6x   1/1     Running   0          83m
mongodb-0                    1/1     Running   0          88m
mysql-0                      1/1     Running   0          74m
payment-6659f94c87-fpmgn     1/1     Running   0          27m
rabbitmq-0                   1/1     Running   0          66m
ratings-54cf5d68f7-77qx6     1/1     Running   0          13m
redis-0                      1/1     Running   0          78m
shipping-66b78d4fc4-mhgkv    1/1     Running   0          43m
user-6bdd466b69-jdbrt        1/1     Running   0          61m
web-58598748d4-4nvpb         1/1     Running   0          115s

kubectl get svc -o wide -n robot-shop
NAME        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE    SELECTOR
cart        ClusterIP      10.103.107.69    <none>        8080/TCP             51m    name=cart,owner=Naveen,tier=App
catalogue   ClusterIP      10.103.253.255   <none>        8080/TCP             83m    name=catalogue,owner=Naveen,tier=App
mongodb     ClusterIP      None             <none>        27017/TCP            88m    name=mongodb,owner=Naveen,tier=DB
mysql       ClusterIP      None             <none>        3306/TCP             74m    name=mysql,owner=Naveen,tier=DB
payment     ClusterIP      10.109.199.199   <none>        8080/TCP             27m    name=payment,owner=Naveen,tier=App
rabbitmq    ClusterIP      None             <none>        5672/TCP,15672/TCP   66m    name=rabbitmq,owner=Naveen,tier=DB
ratings     ClusterIP      10.106.254.57    <none>        80/TCP               13m    name=ratings,owner=Naveen,tier=App
redis       ClusterIP      None             <none>        6379/TCP             78m    name=redis,owner=Naveen,tier=DB
shipping    ClusterIP      10.103.123.124   <none>        8080/TCP             43m    name=shipping,owner=Naveen,tier=App
user        ClusterIP      10.97.54.229     <none>        8080/TCP             61m    name=user,owner=Naveen,tier=App
web         LoadBalancer   10.102.200.164   127.0.0.1     8081:30248/TCP       118s   name=web,owner=Naveen,tier=FE
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