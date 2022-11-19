# Fix MySQL Deployment

## Task

> One of the Nautilus DevOps team members was working on to update an existing Kubernetes template. Somehow, he made some mistakes in the template and it is failing while applying. We need to fix this as soon as possible, so take a look into it and make sure you are able to apply it without any issues. Also, do not remove any component from the template like pods/deployments/volumes etc.<br><br>`/home/thor/mysql_deployment.yml` is the template that needs to be fixed.<br><br>Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster

## Preliminary Steps

1. Check the architecture map at <https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0>
2. Check server details at <https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus>

## Research

* PV and PVC - <https://kubernetes.io/docs/concepts/storage/persistent-volumes/> and <https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/>
* Deployments - <https://kubernetes.io/docs/concepts/workloads/controllers/deployment/>
* Secerts - <https://kubernetes.io/docs/concepts/configuration/secret/>

## Steps

Follow the [k8s environment setup steps.](setup-k8s-env.md).

```bash
# Check cluster connection
k get no -o wide
```

```
NAME                      STATUS   ROLES                  AGE    VERSION                          INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION   CONTAINER-RUNTIME
kodekloud-control-plane   Ready    control-plane,master   145m   v1.20.5-rc.0.18+c4af4684437b37   172.17.0.2    <none>        Ubuntu 20.10   5.4.0-1092-gcp   containerd://1.5.0-beta.0-69-gb3f240206
```

We can connect fine from Thor jumpbox.

```bash
# Check deployment file.
k apply -f /home/thor/mysql_deployment.yml
```

```
unable to recognize "/home/thor/mysql_deployment.yml": no matches for kind "Persistentvolume" in version "apps/v1"
unable to recognize "/home/thor/mysql_deployment.yml": no matches for kind "Persistentvolumeclaim" in version "apps/v1"
error validating "/home/thor/mysql_deployment.yml": error validating data: ValidationError(Service.metadata): unknown field "app" in io.k8s.apimachinery.pkg.apis.meta.v1.ObjectMeta; if you choose to ignore these errors, turn validation off with --validate=false
```

There are the following syntax errors in this YAML file.

* `Persistentvolumeclaim` is wrong.
* `Persistentvolume` is wrong.
* `apps/v1` is wrong for PersistentVolume and PersistentVolumeClaim.
* `Service.metadata.app` has incorrect indentation.

```bash
# Take a backup
cp mysql_deployment.yml mysql_deployment.yml.bak

# Fix PV and PVC syntax errors.
sed -i -r 's/Persistentvolumeclaim/PersistentVolumeClaim/g' mysql_deployment.yml
sed -i -r 's/Persistentvolume/PersistentVolume/g' mysql_deployment.yml
sed -i -r 's|apps/v1|v1|g' mysql_deployment.yml

# Fix the Service syntax error.
vi mysql_deployment
```

```yaml
# Fix indentation
metadata:
  name: mysql         
  labels:             
  app: mysql-app 

metadata:
  name: mysql         
  labels:             
    app: mysql-app 
```

```bash
# Deploy the app.
k apply -f /home/thor/mysql_deployment.yml
```

```
error: error validating "/home/thor/mysql_deployment.yml": error validating data: [ValidationError(PersistentVolume.spec.accessModes): invalid type for io.k8s.api.core.v1.PersistentVolumeSpec.accessModes: got "string", expected "array", ValidationError(PersistentVolume.spec.persistentVolumeReclaimPolicy): invalid type for io.k8s.api.core.v1.PersistentVolumeSpec.persistentVolumeReclaimPolicy: got "array", expected "string"]; if you choose to ignore these errors, turn validation off with --validate=false
```

There are now the following syntax errors in this YAML file.

* `PersistentVolume.spec.accessModes:` should be an array and not a string.
* `PersistentVolume.spec.persistentVolumeReclaimPolicy` should be a string and not an array.

```bash
# Fix the syntax errors.
vi /home/thor/mysql_deployment.yml
```

```yaml
# Change string to array.
accessModes: ReadWriteOnce

accessModes:
  - ReadWriteOnce

# Change array to string.
persistentVolumeReclaimPolicy: 
  - Retain

persistentVolumeReclaimPolicy: Retain  
```

```bash
# Deploy the app.
k apply -f /home/thor/mysql_deployment.yml
```

```
persistentvolume/mysql-pv created
error: error validating "/home/thor/mysql_deployment.yml": error validating data: ValidationError(PersistentVolumeClaim.spec.accessModes): invalid type for io.k8s.api.core.v1.PersistentVolumeClaimSpec.accessModes: got "string", expected "array"; if you choose to ignore these errors, turn validation off with --validate=false
```

There are now the following syntax errors in this YAML file.

* `PersistentVolumeClaim.spec.accessModes` should be an array and not a string.

```bash
# Fix the syntax errors/
vi /home/thor/mysql_deployment.yml
```

```yaml
# Change string to array.
accessModes: ReadWriteOnce

accessModes:
  - ReadWriteOnce
```

```bash
# Deploy the app.
k apply -f /home/thor/mysql_deployment.yml
```

```
persistentvolume/mysql-pv unchanged
service/mysql created
error parsing /home/thor/mysql_deployment.yml: error converting YAML to JSON: yaml: line 28: mapping values are not allowed in this context
Error from server (BadRequest): error when creating "/home/thor/mysql_deployment.yml": PersistentVolumeClaim in version "v1" cannot be handled as a PersistentVolumeClaim: v1.PersistentVolumeClaim.Spec: v1.PersistentVolumeClaimSpec.StorageClassName: Resources: v1.ResourceRequirements.Requests: unmarshalerDecoder: quantities must match the regular expression '^([+-]?[0-9.]+)([eEinumkKMGTP]*[-+]?[0-9]*)$', error found in #10 byte of ...|e":"250MB"}},"storag|..., bigger context ...|eOnce"],"resources":{"requests":{"storage":"250MB"}},"storageClassName":"standard"}}
|...
```

There are now the following syntax errors in this YAML file.

* PV and PVC has size mismiatch, base 2 bytes versus base 10 bytes.

MB to Mi

```bash
# Fix the syntax errors/
vi /home/thor/mysql_deployment.yml
```

```yaml
# Change from base10 to base2
resources:
  requests:
    storage: 250MB

resources:
  requests:
    storage: 250Mi
```

```bash
# Deploy the app.
k apply -f /home/thor/mysql_deployment.yml
```

```
persistentvolume/mysql-pv unchanged
persistentvolumeclaim/mysql-pv-claim created
service/mysql unchanged
error: error parsing /home/thor/mysql_deployment.yml: error converting YAML to JSON: yaml: line 28: mapping values are not allowed in this context
```

There are now the following syntax errors in this YAML file.

* A syntax error with line 28 of the Deployment, not the entire file. The indentation for the values in `secretKeyRef` is worng.
* `app/v1` is wrong for Deployment.

```bash
# Fix the syntax errors.
sed -i -r 's|app/v1|apps/v1|g' mysql_deployment.yml

vi /home/thor/mysql_deployment.yml
```

```yaml
# Fix indentation for all values
- name: MYSQL_ROOT_PASSWORD 
  valueFrom:     
    secretKeyRef:
    name: mysql-root-pass 
      key: password

- name: MYSQL_ROOT_PASSWORD 
  valueFrom:     
    secretKeyRef:
      name: mysql-root-pass 
      key: password
```

```bash
# Deploy the app.
k apply -f /home/thor/mysql_deployment.yml
```

```
persistentvolume/mysql-pv unchanged
persistentvolumeclaim/mysql-pv-claim unchanged
service/mysql unchanged
error: error parsing mysql_deployment.yml: error converting YAML to JSON: yaml: line 52: mapping values are not allowed in this context
```

There are now the following syntax errors in this YAML file.

* A syntax error with line 52 of the Deployment, not the entire file. The indentation for the values in `persistentVolumeClaim` is worng.

```bash
# Fix the syntax errors/
vi /home/thor/mysql_deployment.yml
```

```yaml
# Fix indentation of all values
- name: mysql-persistent-storage
    persistentVolumeClaim: 
    claimName: mysql-pv-claim

- name: mysql-persistent-storage
  persistentVolumeClaim: 
    claimName: mysql-pv-claim
```

```bash
# Deploy the app.
k apply -f /home/thor/mysql_deployment.yml
```

```
persistentvolume/mysql-pv unchanged
persistentvolumeclaim/mysql-pv-claim unchanged
service/mysql unchanged
error: error validating "mysql_deployment.yml": error validating data: [ValidationError(Deployment.metadata): unknown field "app" in io.k8s.apimachinery.pkg.apis.meta.v1.ObjectMeta, ValidationError(Deployment.spec.selector): unknown field "app" in io.k8s.apimachinery.pkg.apis.meta.v1.LabelSelector, ValidationError(Deployment.spec.selector): unknown field "tier" in io.k8s.apimachinery.pkg.apis.meta.v1.LabelSelector]; if you choose to ignore these errors, turn validation off with --validate=false
```

There are now the following syntax errors in this YAML file.

* Indentation issues with the labels and selectors.
* `matchlabels` is wrong.

```bash
# Fix the syntax errors/
vi /home/thor/mysql_deployment.yml
```

```yaml
# Fix indentation of all values
metadata:
  name: mysql-deployment           
  labels:                         
  app: mysql-app   
spec:
  selector:
    matchlabels:                  
    app: mysql-app 
    tier: mysql 

metadata:
  name: mysql-deployment           
  labels:                         
    app: mysql-app   
spec:
  selector:
    matchLabels:                  
      app: mysql-app 
      tier: mysql 
```

```bash
# Deploy the app.
k apply -f /home/thor/mysql_deployment.yml
```

```
persistentvolume/mysql-pv unchanged
persistentvolumeclaim/mysql-pv-claim unchanged
service/mysql unchanged
deployment.apps/mysql-deployment created
```

Finally this app has been deployed.
