# Install API Gateway
Kubernetes deployment scripts of the [TS4NFDI API Gateway](https://github.com/ts4nfdi/api-gateway)

## Installation
Done automatically in GitHub CICD.

Install test on QA cluster:
```powershell
helm upgrade ts4nfdi-api-gateway `
    --namespace='ts4nfdi' `
    --set-string ingress.dns="tsag.qa.km.k8s.zbmed.de"  `
    --set-string images.backend="ghcr.io/ts4nfdi/api-gateway:latest"  `
    --set-string postgres.logical_backup="false"  `
    --set-string postgres.number_instances_postgres="1"  `
    --set-string postgres.volume_size="1Gi"  `
    --set-string number_instances_backend="1"  `
IdeaProjects/api-gateway-deployment/k8s/api-gateway/
```

Upgrade prod instance:
```powershell
helm upgrade ts4nfdi-api-gateway `
    --namespace='ts4nfdi' `
    --set-string ingress.dns="terminology.services.base4nfdi.de"  `
    --set-string ingress.redirectDns="ts.services.base4nfdi.de"  `
    --set-string images.backend="ghcr.io/ts4nfdi/api-gateway:f356d86a5924e55ea535ab8c9dbae074626b6cbc"  `
    --set-string postgres.logical_backup="true"  `
    --set-string postgres.number_instances_postgres="2"  `
    --set-string number_instances_backend="2"  `
    --set-string postgres.volume_size="10Gi"  `
    --set-string ingress.enableSSL="true"  `
    --set-string ingress.certIssuer="letsencrypt-prod"  `
IdeaProjects/api-gateway-deployment/k8s/api-gateway/
```

## Notes
Allow external access on Minikube: https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/

Download deployment scripts on a cluster with Helm: 
```shell
helm repo add api-gateway-deployment https://ts4nfdi.github.io/api-gateway-deployment/
```
