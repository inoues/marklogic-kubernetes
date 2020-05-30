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
openssl base64 -in C:/Users/ユーザー名/.docker/config.json -out C:/Users/ユーザー名/.docker/config_base64.txt
```
registry-secret.yaml  
copy the content inside the config_base64.txt file and paste it as the value for .dockerconfigjson

## MarkLogic Service