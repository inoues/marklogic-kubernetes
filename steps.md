## Assumptions
Windows 10 pro 2004  
Git  
Docker hub account  
Docker Desktop 2.3.0.3  
- Enable Kubernetes  

## Download MarkLogic
MarkLogic-10.0-4.x86_64.rpm  
https://developer.marklogic.com/download

## create docker hub registry
`registry/your image name`

## built image, push it to your registry.
`docker login`  
### build image
`docker build -t <your registry/your image name>:<image tag> .`  
like   
`docker build -t psychobmx/marklogic:v1 .`

check image
```
docker images
REPOSITORY                                  TAG                                              IMAGE ID            CREATED             SIZE
psychobmx/marklogic                         v1                                               e9bde9ecea99        24 seconds ago      4.23GB
```

### push image
`docker push <your registry/your image name>:<image tag>`  
like  
`docker push psychobmx/marklogic:v1`  

## update statefulset
ml-statefulset.yaml L21  
image  
`<your registry/your image name>:<image tag>`  
like  
`psychobmx/marklogic:v1`  

```
    spec:
      terminationGracePeriodSeconds: 30
      containers:
        - name: marklogic
          image: psychobmx/marklogic:v1
          imagePullPolicy: Always
```

## Creating the Registry Authentication Secret
Use Git bash
```
openssl base64 -in C:/Users/"username"/.docker/config.json -out C:/Users/"username"/.docker/config_base64.txt
```
registry-secret.yaml  
copy the content inside the config_base64.txt file and paste it as the value for .dockerconfigjson

## NGINX
1. load balancing between the nodes in the cluster from outside the Kubernetes internal network
2. Allows us to access each node individually
  
```
cd nginx
docker build -t <your registry/your image name>:<your image tag> .
docker push <your registry/your image name>:<your image tag>
```
like  
```
docker build -t psychobmx/nginx-ml:v1 .
docker push psychobmx/nginx-ml:v1
````

### NGINX Replication Controller
nginx-ingress.rc.yml  
image  
`<your registry/your image name>:<image tag>`  
like 
```
    spec:
      containers:
      - image: psychobmx/nginx-ml:v1
        imagePullPolicy: Always
        name: nginx-ingress
```

## Starting up the Cluster
### Create PV
`kubectl apply -f .\persistent-volumes\local-PersistantVolume.yaml`  
`kubectl apply -f .\persistent-volumes\local-PersistantVolume2.yaml`  
`kubectl apply -f .\persistent-volumes\local-PersistantVolume3.yaml`  
`kubectl get pv`  
```
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                         STORAGECLASS    REASON   AGE
dst-local-pv    20Gi       RWO            Retain           Bound       default/ml-data-marklogic-0   local-storage            31s
dst-local-pv2   20Gi       RWO            Retain           Available                                 local-storage            22s
dst-local-pv3   20Gi       RWO            Retain           Available                                 local-storage            16s
```
`kubectl apply -f .\persistent-volumes\local-StorageClass.yaml`  
`kubectl get sc`
```
NAME                 PROVISIONER                    AGE
hostpath (default)   docker.io/hostpath             11h
local-storage        kubernetes.io/no-provisioner   83s
```

`kubectl create namespace marklogic`  
`kubectl create -f .\registry-secret.yaml`    
`kubectl create -f .\marklogic-service.yaml`  
`kubectl create -f .\ml-statefulset.yaml`    
`kubectl create -f .\nginx\nginx-ingress.rc.yaml`    

? erro  

`kubectl get pods --all-namespaces`

```
NAMESPACE     NAME                                     READY   STATUS             RESTARTS   AGE
default       marklogic-0                              0/1     Pending            0          126m
default       nginx-ingress-rc-8bqgb                   0/1     CrashLoopBackOff   9          123m
```
`kubectl describe pods marklogic-0`  
```
Name:           marklogic-0
Namespace:      default
Priority:       0
Node:           <none>
Labels:         app=marklogic
                controller-revision-hash=marklogic-5f699f9865
                statefulset.kubernetes.io/pod-name=marklogic-0
Annotations:    <none>
Status:         Pending
IP:
IPs:            <none>
Controlled By:  StatefulSet/marklogic
Containers:
  marklogic:
    Image:       psychobmx/marklogic:v1
    Ports:       7997/TCP, 7998/TCP, 7999/TCP, 8000/TCP, 8001/TCP, 8002/TCP
    Host Ports:  0/TCP, 0/TCP, 0/TCP, 0/TCP, 0/TCP, 0/TCP
    Command:
      /opt/entry-point.sh
    Requests:
      cpu:        1
      memory:     1Gi
    Environment:  <none>
    Mounts:
      /var/opt/MarkLogic from ml-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-gwjx4 (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  ml-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  ml-data-marklogic-0
    ReadOnly:   false
  default-token-gwjx4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-gwjx4
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  3m3s (x88 over 11h)  default-scheduler  pod has unbound immediate PersistentVolumeClaims
```
 