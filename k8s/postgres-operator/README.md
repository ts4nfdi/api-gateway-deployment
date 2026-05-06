
# Postgres-operator
## Install namespaced postgres-operator (beside default postgres-operator):

1. Install secret from postgres-wal-s3-secret.yaml
2. Install operator:
```shell
helm install ts4nfdi-postgres-operator `
    --namespace='ts4nfdi' `
    --set configKubernetes.watched_namespace="ts4nfdi" `
    --set controllerID.create="true" `
    --set controllerID.name="ts4nfdi-operator" `
    --set podServiceAccount.name="ts4nfdi-postgres-pod" `
    -f postgres-operator-values.yaml `
postgres-operator-charts/postgres-operator
```
`podServiceAccount` if not configured, a default name will be used that will interfere with other postgres-operator instances  
`controllerID` must be set in the postgresql instances  
`watched_namespace` optional, controllerID is sufficient  


# Backup

Both, logical backup and WAL-G (for PITR) are enabled for the gateway.

## Restore from WAL
Create a clone from prod-clone.yaml: 
```
kubectl apply -f prod-clone.yaml
```
A new postgresql cluster will be cloned from S3, using the latest backup before the timestamp.

TODO: How to switch clusters (modify app-backend.yaml to use a value for the db instance)

## Do a manual logical backup

```
KUBE_CONTEXT="garden-419592--km-bonn-qa-external"
NAMESPACE="ts4nfdi"
CLUSTER_NAME="ts4nfdi-api-gateway-postgres"

TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
TMP_FILE="${CLUSTER_NAME}_${TIMESTAMP}.sql.gz"

POD=$(kubectl get pods --context "$KUBE_CONTEXT" -n "$NAMESPACE" \
-l application=spilo,cluster-name="$CLUSTER_NAME" \
-o jsonpath="{.items[0].metadata.name}")
```

### Run pg_dump inside the pod and compress
Restoration is simpler if backing up the gateway database, rather than the entire postgresql cluster.
```
kubectl exec --context "$KUBE_CONTEXT" -n "$NAMESPACE" "$POD" -- \
bash -c "pg_dump -U gateway -d gateway > /tmp/backup.sql.gz"
```
### Copy compressed backup from pod to local file
```
kubectl cp --context "$KUBE_CONTEXT" "$NAMESPACE/$POD:/tmp/backup.sql.gz" "$TMP_FILE"
```

### Clean up temporary file on the pod
```
kubectl exec --context "$KUBE_CONTEXT" -n "$NAMESPACE" "$POD" -- rm -f /tmp/backup.sql.gz
```

## Restore from logical backup

### Copy backup to pod and decompress
```
kubectl cp ts4nfdi-api-gateway-postgres_20260504_120303.sql.gz --context "$KUBE_CONTEXT" "$NAMESPACE/$POD:/tmp/backup.sql" 
kubectl exec --context "$KUBE_CONTEXT" -n "$NAMESPACE" "$POD" -- gzip -d /tmp/backup.sql.gz
```

### If versions don't match: remove line from dump (e.g. if dump was created with pg_dump 17, but target server is PostgreSQL 14)
```
kubectl exec -n "$NAMESPACE" "$POD" -- \
sed -i '/transaction_timeout/d' /tmp/backup.sql
```
### Recreate 'public' schema
```
kubectl exec -n "$NAMESPACE" "$POD" -- \
psql -U gateway -d gateway -c "
DROP SCHEMA public CASCADE;
DROP SCHEMA IF EXISTS metric_helpers CASCADE;
DROP SCHEMA IF EXISTS user_management CASCADE;
CREATE SCHEMA public;
"
```

### Restore db dump using psql into 'gateway' DB
```
kubectl exec -n "$NAMESPACE" "$POD" -- \
psql -U gateway -d gateway -v ON_ERROR_STOP=1 -f /tmp/backup.sql
```