# api-gateway-deployment
Kubernetes deployment scripts of the API Gateway

## Installation
```shell
helm install test \
--set-json='ingress.dns="test.ts4nfdi"'  \
--set-json='images.frontend="ghcr.io/ts4nfdi/api-gateway:latest"'  \
/path-to/api-gateway-deployment/k8s/api-gateway/
```

## Notes
Allow external access on Minikube: https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/

Download deployment scripts on a cluster with Helm: 
```shell
helm repo add api-gateway-deployment https://ts4nfdi.github.io/api-gateway-deployment/
```
