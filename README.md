# csi-secret-store-mysql

## Install helm

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Install Vault 
```
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault \
    --set "server.dev.enabled=true" \
    --set "injector.enabled=false" \
    --set "csi.enabled=true"
```
## exec into the pod
```
kubectl exec -it vault-0 -- /bin/sh
```

## Create the KV store and secret from the Vault UI or CLI 
```
vault kv put secret/mysql-credentials username="root" password="password123"
vault kv get secret/mysql-credentials
```

## Enable Kubernetes Authentication
```
vault auth enable kubernetes
vault write auth/kubernetes/config kubernetes_host="https://172.30.1.2:6443" // this is for Killercoda

```
## Create a policy
```
vault policy write mysql-read-write - <<EOF
path "secret/data/mysql-credentials" {
  capabilities = ["read", "write"]
}
EOF
```

## Binding policy with Kubernetes SA
```
vault write auth/kubernetes/role/mysql-secret-role \
    bound_service_account_names=secret-sa \
    bound_service_account_namespaces=default \
    policies=mysql-read-only \
    ttl=40m
```

## Create service account 
```
kubectl create serviceaccount secret-sa

```

## Install csi-secret-store
```
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver --namespace kube-system
```

## Create the secret provider class
kubectl apply -f secretproviderclass

## create the pod 
kubectl apply -f pod.yaml


## [Customize installation](https://github.com/kubernetes-sigs/secrets-store-csi-driver/tree/main/charts/secrets-store-csi-driver#configuration)
```
enableSecretRotation	Enable secret rotation feature [alpha]	false
yncSecret.enabled	Enable rbac roles and bindings required for syncing to Kubernetes native secrets	false
```
