# marklogic-kubernetes
Deploying a MarkLogic cluster in Kubernetes

Based on this guide by Bill Miller: https://www.marklogic.com/blog/docker-deploy-kubernetes/ 

### With the following changes:
1. Replaced the Replication Controller with a Deployment (both manifests are still included)
2. Uses a "marklogic" namespace instead of "default". Changes made to the following files to reflect this:
  * _nginx/upstream-defs.conf_
    - Replace all instances of `marklogic-0.ml-service.default.svc.cluster.local:7997;` with `marklogic-0.ml-service.marklogic.svc.cluster.local:7997;` etc.
  * _entry-point.sh_
    - Replace `./setup-child.sh "marklogic-0.ml-service.default.svc.cluster.local" $CONFIGURATION_FILE_LOCATION` with `./setup-child.sh "marklogic-0.ml-service.marklogic.svc.cluster.local" $CONFIGURATION_FILE_LOCATION`
