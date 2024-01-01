### Volumes
```
Deployment 
          Replicaset
                    POD's
```
`Problem Statement1`
```
- Persisting Data
- POD consists of Independent container + volume
- POD Deleted/Restarted then data will also be deleted
```
`Problem Statement2`
```
- Sharing Data
- How Relicas share data
      Data not shared across replicas by default
```
### Store Data@Container Level
- if Container Deleted/Restarted then data will also be deleted 
#### deployment-mysql.yml
```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: bmF2ZWVuMTIz
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:latest
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
          ports:
            - containerPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: NodePort
```
```
kubectl apply -f deployment-mysql.yml
```
```
kubectl port-forward svc/mysql-service 3307:3306
```
#### Connect to MySQL Work Beanch
```
127.0.0.1:3307
uname: root, password: naveen123

create database db;
use db;
show databases;
```
```
kubectl get svc
    NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP          47m
    mysql-service   NodePort    10.105.8.230   <none>        3306:30094/TCP   22m

kubectl get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    mysql-deployment-9bdc77b69-4gn2p   1/1     Running   0          22m 
```
```
kubectl exec -it mysql-deployment-9bdc77b69-4gn2p -- /bin/bash

ps aux

kill <Process-ID>
```
```
kubectl get pods
    NAME                               READY   STATUS    RESTARTS       AGE
    mysql-deployment-9bdc77b69-4gn2p   1/1     Running   1 (38s ago)    22m 
```
#### Connect to MySQL Work Beanch
```
127.0.0.1:3307
uname: root, password: naveen123

show databases;
// we can't see db database
```
### Store Data@POD Level
- POD Deleted/Restarted then data will also be deleted
- Share Data between Containers in POD
#### deployment-mysql-emptydir.yml
```
# kubectl port-forward svc/mysql-service 3307:3306

apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: bmF2ZWVuMTIz
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      volumes:
        - name: mysql-volume
          emptyDir: {}
      containers:
        - name: mysql
          image: mysql:latest
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: mysql-volume
          ports:
            - containerPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: NodePort
```
```
kubectl get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    mysql-deployment-b456c545d-xcc69   1/1     Running   0          2m25s

kubectl get svc                                               
    NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    mysql-service   NodePort    10.106.44.225   <none>        3306:30776/TCP   2m30s
```
#### Connect to MySQL Work Beanch
```
127.0.0.1:3307
uname: root, password: naveen123

create database db;
use db;
show databases;
```
```
kubectl exec -it mysql-deployment-b456c545d-xcc69 -- bash

ps aux

kill <Process-ID>
```
```
kubectl get pods
    NAME                               READY   STATUS    RESTARTS     AGE
    mysql-deployment-b456c545d-xcc69   1/1     Running   1 (5s ago)   3m14s
```
#### Connect to MySQL Work Beanch
```
127.0.0.1:3307
uname: root, password: naveen123

show databases;
// we can see db database
```
### Login to Minikube
```
minikube ssh
    o/p:
    docker@minikube:~$ sudo ls /var/lib/kubelet/pods
        o/p:
        574f6dd6-3a97-4015-a511-b49397e735c3  bd495b7643dfc9d3194bd002e968bc3d
        7e6ece94cd0950fdbbf66ddae1e4c53b      bd8b8fe30652798a5ae3fc51e66ef681
        91c789ffd31943242ef457881ac1824a      c47f02b5-1cff-45ec-ace6-1d96651b838f
        9762f797-aa98-4dc7-8050-6193dce755c9  d4a95447-0d4d-4c3a-906c-2a624c873fdf
```
#### Get POD ID
```
kubectl get pod mysql-deployment-b456c545d-xcc69 -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2023-12-23T09:34:27Z"
  generateName: mysql-deployment-b456c545d-
  labels:
    app: mysql
    pod-template-hash: b456c545d
  name: mysql-deployment-b456c545d-xcc69
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: mysql-deployment-b456c545d
    uid: 6b0b61ec-47ff-4379-9051-443c062e235a
  resourceVersion: "7443"
  uid: 574f6dd6-3a97-4015-a511-b49397e735c3
  .
  .
  .
  - emptyDir: {}
    name: mysql-volume
  .
  .
```
```
docker@minikube:~$ sudo ls /var/lib/kubelet/pods/574f6dd6-3a97-4015-a511-b49397e735c3
    containers  etc-hosts  plugins  volumes

docker@minikube:~$ sudo ls /var/lib/kubelet/pods/574f6dd6-3a97-4015-a511-b49397e735c3/volumes
    kubernetes.io~empty-dir  kubernetes.io~projected

docker@minikube:~$ sudo ls /var/lib/kubelet/pods/574f6dd6-3a97-4015-a511-b49397e735c3/volumes/kubernetes.io~empty-dir
    mysql-volume

docker@minikube:~$ sudo ls /var/lib/kubelet/pods/574f6dd6-3a97-4015-a511-b49397e735c3/volumes/kubernetes.io~empty-dir/mysql-volume
    # sudo ls /var/lib/kubelet/pods/574f6dd6-3a97-4015-a511-b49397e735c3/volumes/kubernetes.io~empty-dir/mysql-volume
    '#ib_16384_0.dblwr'   auto.cnf      ca-key.pem        ib_buffer_pool mysql.ibd      public_key.pem    undo_001
    '#ib_16384_1.dblwr'   binlog.000001   ca.pem        ibdata1 mysql.sock      server-cert.pem   undo_002
    '#innodb_redo'      binlog.000002   client-cert.pem   ibtmp1 performance_schema   server-key.pem
    '#innodb_temp'      binlog.index    client-key.pem    mysql private_key.pem      sys
```
```
kubectl get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    mysql-deployment-b456c545d-xcc69   1/1     Running   0          20m

kubectl delete pod mysql-deployment-b456c545d-xcc69
    pod "mysql-deployment-b456c545d-xcc69" deleted

kubectl get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    mysql-deployment-b456c545d-mqlhp   1/1     Running   0          40s
```
### We are unable to get data of POD
```
docker@minikube:~$ sudo ls /var/lib/kubelet/pods/574f6dd6-3a97-4015-a511-b49397e735c3/volumes/kubernetes.io~empty-dir/mysql-volume
    ls: cannot access '/var/lib/kubelet/pods/574f6dd6-3a97-4015-a511-b49397e735c3/volumes/kubernetes.io~empty-dir/mysql-volume': No such file or directory
```
### Store Data@Node Level
### HostPath
- Node Deleted then Data will be deleted
- POD/Container Deleted/Restarted the Data will not be deleted
- Problem: Data between Node1 can't be shared to Node2
- DaemonSet is Example of HostPath
```
# kubectl port-forward svc/mysql-service 3307:3306

apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: bmF2ZWVuMTIz
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      volumes:
        - name: mysql-volume
          hostPath:
            path: /var/lib/mysql
      containers:
        - name: mysql
          image: mysql:latest
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: mysql-volume
          ports:
            - containerPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: NodePort
```
```
kubectl apply -f .\deployment-mysql-hostpath.yml
    secret/mysql-secret unchanged
    deployment.apps/mysql-deployment configured
    service/mysql-service unchanged

kubectl get pods
    NAME                                READY   STATUS    RESTARTS   AGE
    mysql-deployment-768c988db7-fvg99   1/1     Running   0          8s

kubectl exec -it mysql-deployment-768c988db7-fvg99 -- bash

    bash-4.4# cd /var/lib/mysql

    bash-4.4# ls
    '#ib_16384_0.dblwr'  '#innodb_temp'   binlog.000002   ca.pem            db               ibtmp1      mysql.sock           public_key.pem    sys
    '#ib_16384_1.dblwr'   auto.cnf        binlog.index    client-cert.pem   ib_buffer_pool   mysql       performance_schema   server-cert.pem   undo_001
    '#innodb_redo'        binlog.000001   ca-key.pem      client-key.pem    ibdata1          mysql.ibd   private_key.pem      server-key.pem    undo_002

    bash-4.4# cat > naveen.txt
    Hi THis File Belongs to NAveen

    bash-4.4# 
    bash-4.4# ls
    '#ib_16384_0.dblwr'  '#innodb_temp'   binlog.000002   ca.pem            db               ibtmp1      mysql.sock           private_key.pem   server-key.pem   undo_002
    '#ib_16384_1.dblwr'   auto.cnf        binlog.index    client-cert.pem   ib_buffer_pool   mysql       naveen.txt           public_key.pem    sys
    '#innodb_redo'        binlog.000001   ca-key.pem      client-key.pem    ibdata1          mysql.ibd   performance_schema   server-cert.pem   undo_001
```
### Checking HostVolume
```
minikube ssh
    # cd /var/lib/mysql
    # ls
    '#ib_16384_0.dblwr'   auto.cnf      ca-key.pem        db mysql      private_key.pem   sys
    '#ib_16384_1.dblwr'   binlog.000001   ca.pem        ib_buffer_pool mysql.ibd      public_key.pem    undo_001
    '#innodb_redo'      binlog.000002   client-cert.pem   ibdata1 mysql.sock      server-cert.pem   undo_002
    '#innodb_temp'      binlog.index    client-key.pem    ibtmp1 performance_schema   server-key.pem

// After Creating the file(naveen.txt) in POD
    # ls      
    '#ib_16384_0.dblwr'   auto.cnf      ca-key.pem        db mysql      performance_schema   server-key.pem
    '#ib_16384_1.dblwr'   binlog.000001   ca.pem        ib_buffer_pool mysql.ibd    private_key.pem   sys
    '#innodb_redo'      binlog.000002   client-cert.pem   ibdata1 mysql.sock   public_key.pem   undo_001
    '#innodb_temp'      binlog.index    client-key.pem    ibtmp1 naveen.txt   server-cert.pem   undo_002
```
#### Delete POD & Checking For Data(naveen.txt) in New POD
- Note:- if we delete minikube node then we can loose data
```
kubectl delete pod mysql-deployment-768c988db7-fvg99
    pod "mysql-deployment-768c988db7-fvg99" deleted

kubectl get pods
    NAME                                READY   STATUS    RESTARTS   AGE
    mysql-deployment-768c988db7-p55cv   1/1     Running   0          5s

kubectl exec -it mysql-deployment-768c988db7-p55cv -- bash
    bash-4.4# cd /var/lib/mysql
    bash-4.4# ls
    '#ib_16384_0.dblwr'  '#innodb_temp'   binlog.000002   ca-key.pem        client-key.pem   ibdata1   mysql.ibd    performance_schema   server-cert.pem   undo_001
    '#ib_16384_1.dblwr'   auto.cnf        binlog.000003   ca.pem            db               ibtmp1    mysql.sock   private_key.pem      server-key.pem    undo_002
    '#innodb_redo'        binlog.000001   binlog.index    client-cert.pem   ib_buffer_pool   mysql     naveen.txt   public_key.pem       sys
    bash-4.4#
```
### Persistent Volumes
- POD/NODE Deleted/Restart still data exists
- Note:- 3 Components of Persistent Volumes
```
- Persistent Volumes
- Persistent Volume Claims
- Storage Classes
```
### Persistent Volumes
https://kubernetes.io/docs/concepts/storage/persistent-volumes/
```
Mode:
  - ReadWriteMany:- if all pod's are running in different node's
  - ReadWriteOnce:- if all pod's are running in Single node
  - ReadOnlyOnce
  - ReadOnlyMany
  - ReadWriteOncePod:- Read,Write by single Pod.
```
```
# kubectl port-forward svc/mysql-service 3307:3306
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: demo-storage
provisioner: k8s.io/minikube-hostpath
volumeBindingMode: Immediate
reclaimPolicy: Delete
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  local:
    path: /storage/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - minikube
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc-sc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: "demo-storage"
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: bmF2ZWVuMTIz
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      volumes:
        - name: mysql-volume
          hostPath:
            path: /var/lib/mysql
      containers:
        - name: mysql
          image: mysql:latest
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: mysql-volume
          ports:
            - containerPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: NodePort
```
```
kubectl get pv,pvc,sc
    NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                  STORAGECLASS   REASON   AGE
    persistentvolume/mysql-pv                                   5Gi        RWX            Retain           Available                                                  14m
    persistentvolume/pvc-afa56a3c-93b3-48a5-a2de-698b6634fcfb   5Gi        RWX            Delete           Bound       default/mysql-pvc-sc   demo-storage            14m

    NAME                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    persistentvolumeclaim/mysql-pvc-sc   Bound    pvc-afa56a3c-93b3-48a5-a2de-698b6634fcfb   5Gi        RWX            demo-storage   14m

    NAME                                             PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
    storageclass.storage.k8s.io/demo-storage         k8s.io/minikube-hostpath   Delete          Immediate           false                  14m
    storageclass.storage.k8s.io/standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  7h3m
```
```
kubectl get pods     
    NAME                                READY   STATUS    RESTARTS   AGE
    mysql-deployment-768c988db7-dmlbd   1/1     Running   0          14m
```