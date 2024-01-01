### Create EKS Cluster
```
Attach EC2 Full Access, AmazonElasticFileSystemFullAccess Policy to Node IAM Role

For EBS
-------
To Use StorageClass we need to Install EBS Driver
https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/install.md
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.26"

For EFS
-------
To Use StorageClass we need to Install EFS Driver
https://github.com/kubernetes-sigs/aws-efs-csi-driver via Helm

Create EFS in AWS Console
- Create SG in EKS VPC with Name AllowAllRobot-Shop
- Select SG (AllowAllRobot-Shop) while creating EFS
```
### EKS Admin Tasks
```
Create NameSpace
Install EBS Drivers
Create StorageClasses
Create Custom EKS-Read-List Policy (Modify ARN)
Create IAM User(robot-shop-admin) with Custom EKS-Read-List Policy For Authorization
For Authentication we mention in the rbac.yml

In EKS Admin Folder:-
1.created namespace.yml
2.Created storage-class.yml
3.Created rbac.yml
4.Created aws-auth.yml
```
### EKS Admin Tasks Done
```
kubectl apply -f namespace.yml
namespace/robot-shop created

kubectl apply -f rbac.yml
role.rbac.authorization.k8s.io/robot-shop-admin-role created
rolebinding.rbac.authorization.k8s.io/robot-shop-admin-rolebinding created
clusterrole.rbac.authorization.k8s.io/cluster-reader created
clusterrolebinding.rbac.authorization.k8s.io/cluster-reader-role-binding created

kubectl apply -f aws-auth.yml
configmap/aws-auth configured

kubectl apply -f storage-class.yml
storageclass.storage.k8s.io/efs-sc created
```
#### New Instance with robot-shop-admin
```
Installation
------------
[ec2-user@Instance ~]$ curl https://raw.githubusercontent.com/techworldwithsiva/k8-install/master/eks-client.sh | sudo bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1483  100  1483    0     0  15185      0 --:--:-- --:--:-- --:--:-- 15132
Downloaded AWS CLI V2 ...  SUCCESS 
bash: line 19: [: too many arguments
 ...  SUCCESS 
Updated AWS CLI V2 ...  SUCCESS 
Downloaded eksctl command ...  SUCCESS 
Added execute permissions to eksctl ...  SUCCESS 
moved eksctl to bin folder ...  SUCCESS 
Downloaded kubectl 1.24 version ...  SUCCESS 
Added execute permissions to kubectl ...  SUCCESS 
moved kubectl to bin folder ...  SUCCESS 

Check AWS Version
-----------------
[ec2-user@Instance ~]$ aws --version
aws-cli/2.15.4 Python/3.11.6 Linux/5.10.201-191.748.amzn2.x86_64 exe/x86_64.amzn.2 prompt/off

Check Kubectl Version
---------------------
[ec2-user@Instance ~]$ kubectl version

[ec2-user@Instance ~]$ aws configure
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
Default region name [None]: 
Default output format [None]:

[ec2-user@Instance ~]$ ls -al
total 12
drwx------ 5 ec2-user ec2-user 112 Dec 26 08:30 .
drwxr-xr-x 3 root     root      22 Dec 26 06:40 ..
drwxrwxr-x 2 ec2-user ec2-user  39 Dec 26 08:30 .aws
-rw-r--r-- 1 ec2-user ec2-user  18 Jul 15  2020 .bash_logout
-rw-r--r-- 1 ec2-user ec2-user 193 Jul 15  2020 .bash_profile
-rw-r--r-- 1 ec2-user ec2-user 231 Jul 15  2020 .bashrc
drwxr-xr-x 3 root     root      67 Dec 26 07:35 eks-client-install
drwx------ 2 ec2-user ec2-user  29 Dec 26 06:40 .ssh
```
### create kubeconfig
```
https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html

[ec2-user@Instance ~]$ aws sts get-caller-identity
{
    "UserId": "",
    "Account": "",
    "Arn": "arn:aws:iam::752692907119:user/robot-shop-admin"
}

[ec2-user@Instance ~]$ aws eks update-kubeconfig --region us-east-1 --name spot-cluster
Added new context arn:aws:eks:us-east-1:752692907119:cluster/spot-cluster to /home/ec2-user/.kube/config

[ec2-user@Instance ~]$ ls -al
total 12
drwx------ 6 ec2-user ec2-user 125 Dec 26 08:32 .
drwxr-xr-x 3 root     root      22 Dec 26 06:40 ..
drwxrwxr-x 2 ec2-user ec2-user  39 Dec 26 08:30 .aws
-rw-r--r-- 1 ec2-user ec2-user  18 Jul 15  2020 .bash_logout
-rw-r--r-- 1 ec2-user ec2-user 193 Jul 15  2020 .bash_profile
-rw-r--r-- 1 ec2-user ec2-user 231 Jul 15  2020 .bashrc
drwxr-xr-x 3 root     root      67 Dec 26 07:35 eks-client-install
drwxrwxr-x 2 ec2-user ec2-user  20 Dec 26 08:32 .kube
drwx------ 2 ec2-user ec2-user  29 Dec 26 06:40 .ssh
```
```
[ec2-user@Instance ~]$ sudo yum install -y docker 

[ec2-user@Instance ~]$ docker login
Username: naveen2809
Password: 
Login Succeeded
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
pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0   ebs-sc            6m6s

kubectl get pvc -n robot-shop
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0   Bound    pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            ebs-sc   5m48s
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
pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0   ebs-sc            12m
pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            Delete           Bound    robot-shop/redis-volume-redis-0       ebs-sc            53s

kubectl get pvc -n robot-shop
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0   Bound    pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            ebs-sc   12m
redis-volume-redis-0       Bound    pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            ebs-sc   70s
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
pvc-5278ba50-5551-4fb2-8847-cb24262803b3   1Gi        RWO            Delete           Bound    robot-shop/mysql-volume-mysql-0       ebs-sc            87s
pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0   ebs-sc            15m
pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            Delete           Bound    robot-shop/redis-volume-redis-0       ebs-sc            4m4s

kubectl get pvc -n robot-shop
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0   Bound    pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            ebs-sc   15m
mysql-volume-mysql-0       Bound    pvc-5278ba50-5551-4fb2-8847-cb24262803b3   1Gi        RWO            ebs-sc   106s
redis-volume-redis-0       Bound    pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            ebs-sc   4m23s
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
pvc-147c9d33-5a92-487e-b96c-f04a3bd1ed9a   1Gi        RWO            Delete           Bound    robot-shop/rabbitmq-volume-rabbitmq-0   ebs-sc            92s
pvc-5278ba50-5551-4fb2-8847-cb24262803b3   1Gi        RWO            Delete           Bound    robot-shop/mysql-volume-mysql-0         ebs-sc            6m51s
pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            Delete           Bound    robot-shop/mongodb-volume-mongodb-0     ebs-sc            20m
pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            Delete           Bound    robot-shop/redis-volume-redis-0         ebs-sc            9m28s

kubectl get pvc -n robot-shop
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
mongodb-volume-mongodb-0     Bound    pvc-79c04f14-f66e-4299-89c5-000f59a3d951   1Gi        RWO            ebs-sc   20m
mysql-volume-mysql-0         Bound    pvc-5278ba50-5551-4fb2-8847-cb24262803b3   1Gi        RWO            ebs-sc   6m56s
rabbitmq-volume-rabbitmq-0   Bound    pvc-147c9d33-5a92-487e-b96c-f04a3bd1ed9a   1Gi        RWO            ebs-sc   97s
redis-volume-redis-0         Bound    pvc-c487e49a-f3d4-4352-bf60-b361af14c0ba   1Gi        RWO            ebs-sc   9m33s
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