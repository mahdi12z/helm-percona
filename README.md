# helm-kubernetes


# Percona XtraDB Cluster Deployment on Kubernetes using Helm + Dynamic Provisioning

## 1: Enable Dynamic Volume Provisioning
 Install local-path-provisioner:
```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

Set it as the default StorageClass:
```bash
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

```

 Verify StorageClass:
 ```bash
kubectl get storageclass
```

Expected output:
```bash
NAME           PROVISIONER                DEFAULT
local-path     rancher.io/local-path      Yes
```

## 2: Deploy Percona XtraDB Cluster with Helm

 Add the Percona Helm repo:
 ```bash
helm repo add percona https://percona.github.io/percona-helm-charts/
helm repo update

```

Create a dedicated namespace:

```bash
kubectl create namespace percona-db
```

 Install the Percona Operator:

 ```bash
helm install my-op percona/pxc-operator --namespace percona-db
```

## 3:Install the Database using Helm:
```bash
helm install my-db percona/pxc-db --namespace percona-db
```
## 4: Verify the Deployment
```bash
kubectl get pods -n percona-db
kubectl get pxc -n percona-db
```

You should see pods like:

```perl
my-db-pxc-db-pxc-0         Running
my-db-pxc-db-haproxy-0     Running
```

The creation process may take some time. When the process is over your cluster obtains the ready status.
```perl
Expected output
NAME       ENDPOINT                   STATUS   PXC   PROXYSQL   HAPROXY   AGE
cluster1   cluster1-haproxy.default   ready    3                3         33d
```

## 5: Get MySQL Root Password

List the Secrets objects:
```bash 
kubectl get secrets -n percona-db
```

Retrieve the password for the root user. Replace the secret-name and namespace with your values in the following commands:
```bash
kubectl get secret my-db-pxc-db-secrets -n percona-db -o jsonpath="{.data.root}" | base64 -d && echo
```

## 6: Connect to MySQL Cluster:
for see cluster name
```bash
kubectl get svc -n percona-db
```

```bash
kubectl run -n percona-db -i --rm --tty percona-client --image=percona:8.0 --restart=Never -- bash -il

mysql -h <cluster_name>-haproxy -uroot -p'<root_password>'

```

or

```bash
kubectl exec -it my-db-pxc-db-pxc-0 -n percona-db -- mysql -uroot -p

```



Or

```bash

kubectl run percona-client \
  --image=percona:8.0 \
  --namespace=percona-db \
  --restart=Never \
  --command -- sleep infinity
```
```bash
kubectl exec -n percona-db -it percona-client -- bash -il

```
```bash
mysql -h my-db-pxc-db-haproxy -uroot -p'v8R.>8S1%$_ncbhiDEz'
```



link:https://docs.percona.com/percona-operator-for-mysql/pxc/helm.html#pre-requisites




