# test-prom
This is an example project about install a Standalone Prometheus over Openshift usin the imperative way. 

## Create a NameSpace.
We will start creating a NameSpace.

```bash
└─ $ ▶ oc create ns ns-test-prom
```
## Create a Service account.
### Role
It is important to know that roles and clusterRoles only allow access to consume resources, access to resources cannot be denied.

```bash
└─ $ ▶ oc create role reader-role-test-prom  --verb=get,list,watch --resource=pods,services,endpoints  -o yaml > 01_role.yml
```
### Service Account
```bash
└─ $ ▶ oc create user user-test-prom  --dry-run=true -o yaml > 02_user.yml
```
### RoleBinding
```bash
└─ $ ▶ oc create rolebinding reader-role-binding-test-prom --role=reader-role-test-prom --user=user-test-prom -o yaml --dry-run=true > 03_rolebinding.yml
```

## Create a PVC.
My apologies, there aren't imperative way to create a PVC and following the next [doc](https://docs.openshift.com/enterprise/3.1/install_config/persistent_storage/persistent_storage_nfs.html) I will create the PVC with copy/paste :-(.

## Create ConfigMap.
```bash
└─ $ ▶ oc create configmap configmap-test-prom --from-file=resources/prometheus.yml --dry-run=true -o yaml > 05_configMap.yml
```

## Create StatefulSet deployment.
My apologies, you have to copy/paste some example about the statefullSet because we are in the same situation like with the PVC, there aren't a imperative way to create a StatefullSet but... In this case I can provide you a tricky solution, the next bash command will generate a StatefullSet.
```bash
└─ $ ▶ oc create deployment deployment-test-prom --image=prom/prometheus:latest --dry-run=true -o yaml | sed s/Deployment/StatefullSet/g | sed s/replicas/"podManagementPolicy: Parallel\n  serviceAccountName: user-test-prom\n  serviceName: svc-test-prom\n  replcias"/g | sed s/"resources: {}"/"envFrom:\n         -configMapRef:\n             name:env-props-test-prom"/g
```

## Create Route & Service.
```bash
oc create service loadbalancer svc-test-prom  --tcp=80:9090 --dry-run=true -o yaml
```

```bash
oc create route edge --service=svc-test-prom  --dry-run=true -o yaml
```
