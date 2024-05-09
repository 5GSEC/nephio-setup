# 6. Deploy UPF, AMF and SMF

Once Free5gc-CP is properly configured, you can deploy `UPF`, `AMF`, and `SMF`. Note that this process may take some time.

### Deploy UPF, AMF, and SMF

```bash
##### -----=[ In mgmt cluster ]=----- ####

kubectl apply -f test-infra/e2e/tests/free5gc/005-edge-free5gc-upf.yaml
kubectl apply -f test-infra/e2e/tests/free5gc/006-regional-free5gc-amf.yaml
kubectl apply -f test-infra/e2e/tests/free5gc/006-regional-free5gc-smf.yaml
``` 

SMF in the `regional` cluster connects to `UPFs` in the `edge1` and `edge02` clusters.

### Check SMF's log in the `regional` cluster.

```bash
##### -----=[ In regional cluster ]=----- ####

$ kubectl logs -n free5gc-cp -l name=smf-regional

[INFO][SMF][App] Received PFCP Association Setup Accepted Response from UPF[172.1.0.254]
[INFO][SMF][App] Sending PFCP Association Request to UPF[172.1.2.254]
[INFO][LIB][PFCP] Remove Request Transaction [2]
[INFO][SMF][App] Received PFCP Association Setup Accepted Response from UPF[172.1.2.254]
```

If both `UPFs` were successfully connected, this means that the `N4` connection was successful.

<br></br>
---
|Before|Next|
|--|--|
|[ Go to Before Page](5_deploy_free5gc_cp.md) | [ Go to Next Page ](7_deploy_ueransim.md)|
