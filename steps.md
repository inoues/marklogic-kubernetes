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
`kubectl create namespace marklogic`  
`kubectl create -f .\registry-secret.yaml`    
`kubectl create -f .\marklogic-service.yaml`  
`kubectl create -f .\ml-statefulset.yaml`    
`kubectl create -f .\nginx\nginx-ingress.rc.yaml`    
