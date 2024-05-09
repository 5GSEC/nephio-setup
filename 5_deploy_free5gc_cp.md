# 5. Deploy Free5gc-cp

Now, deploy Free5gc-CP as usual: https://docs.nephio.org/docs/guides/user-guides/exercise-1-free5gc/#step-4-deploy-free5gc-control-plane-functions. 

The Nephio web UI is accessible at `172.18.0.132:7007` (as an example).

The `regional` cluster utilizes a hostpath PV to store data for `mongodb`.

### Create a PV file

```yaml
##### -----=[ In regional cluster ]=----- ####

apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-mongodb-0
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  hostPath:
    path: /home/[User]/nephio/mongodb/ # change here
```

### Apply the PV file

```bash
##### -----=[ In regional cluster ]=----- ####

kubectl apply -f mongodb-pv.yaml
```

Similarly to the `gitea`'s Persistent Volumes in the `mgmt` cluster, we also need to manually adjust the permissions of the local directory using `chmod`.

### Change file permission in PV's hostPath

```bash
##### -----=[ In regional clusters ]=----- ####

# change file permission of the hostpath for the PV
sudo chmod 777 -R ~/nephio/mongodb
 ```

### Deploy free5gc operators

```bash
##### -----=[ In mgmt cluster ]=----- ####

kubectl apply -f test-infra/e2e/tests/free5gc/004-free5gc-operator.yaml
```

### Install other packages

```bash
##### -----=[ In regional, edge01, edge02 clusters ]=----- ####

kpt pkg get --for-deployment https://github.com/nephio-project/catalog.git/nephio/core/workload-crds@main
kpt fn render workload-crds
kpt live init workload-crds
kpt live apply workload-crds --reconcile-timeout=15m --output=table

kpt pkg get --for-deployment https://github.com/nephio-project/catalog.git/infra/capi/multus@main
kpt fn render multus
kpt live init multus
kpt live apply multus --reconcile-timeout=15m --output=table

kubectl rollout restart deployment -n free5gc
```

This will deploy

1 -  `free5gc/free5gc-operator` pods in the `edge01`, `edge02` and `regional` clusters. \
2 -  `free5gc-cp/free5gc-NFV` pods (e.g., `free5gc-ausf`, `nrf`, `nssf`,`pcf`, `udm`, etc.) in the `regional` cluster.

<br></br>
---
|Before|Next|
|--|--|
|[ Go to Before Page](4_configure_network_topology.md) | [ Go to Next Page ](6_deploy_upf_amf_smf.md)|
