# api-gateway-deployment
Kubernetes deployment scripts of the [TS4NFDI API Gateway](https://github.com/ts4nfdi/api-gateway)

## Installation
```shell
helm install test \
--namespace='ts4nfdi' \
--set-json='ingress.dns="ts4nfdi-api-gateway.prod.km.k8s.zbmed.de"'  \
--set-json='images.frontend="ghcr.io/ts4nfdi/api-gateway:latest"'  \
--set-json='ingress.enableSSL="true"'  \
--set-json='ingress.certIssuer="letsencrypt-prod"'  \
/path-to/api-gateway-deployment/k8s/api-gateway/
```

## Notes
Allow external access on Minikube: https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/

Download deployment scripts on a cluster with Helm: 
```shell
helm repo add api-gateway-deployment https://ts4nfdi.github.io/api-gateway-deployment/
```
