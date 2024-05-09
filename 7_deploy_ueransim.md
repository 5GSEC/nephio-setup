# 7. Deploy UERANSIM

### Check the Free5GC Web UI

```bash
##### -----=[ In regional cluster ]=----- ####

kubectl get svc -n free5gc-cp

udr-nudr  ClusterIP 10.104.71.182  <none>  80/TCP 5d15h
webui-service NodePort  10.104.14.50 <none>  5000:30500/TCP 5d15h
```

Now, login to the Free5GC login page. You can access the server using the `webui-service` node port. 

Follow https://free5gc.org/guide/Webconsole/Create-Subscriber-via-webconsole/ to create a subscriber

### Create `ueransim` namespace and `ueransimgnb` deployments

```bash
##### -----=[ In mgmt cluster ]=----- ####

kubectl apply -f test-infra/e2e/tests/free5gc/007-edge01-ueransim.yaml
```

This action establishes a `ueransim` namespace in the `edge01` cluster, featuring two deployments: `ueransimgnb` (gNodeB) and `ueransimue` (UE).

### Check `ueransimnb` and `ueransimue`

```bash
##### -----=[ In edge01 cluster ]=----- ####

kubectl get pods -n ueransim

NAME  READY STATUS  RESTARTS AGE
ueransimgnb-edge01-7b764c4f9c-92spz 1/1 Running 0  37m
ueransimue-edge01-7b44fcd85b-r2j2n  1/1 Running 0  37m
```

Once the `ueransimgnb-edge01` deployment is initiated, it establishes an `N2` connection to the `AMF` in the `regional` cluster.

### Check AMF connection

Review the logs in the `regional` cluster to confirm the success of the `AMF` connection.

```bash
##### -----=[ In regional cluster ]=----- ####

kubectl logs -n free5gc-cp -l name=amf-regional

[INFO][AMF][NGAP][172.2.0.254:38273][AMF_UE_NGAP_ID:23] Uplink NAS Transport (RAN UE NGAP ID: 1)
[INFO][LIB][FSM] Handle event[Gmm Message], transition from [Registered] to [Registered]
[INFO][AMF][GMM][AMF_UE_NGAP_ID:23][SUPI:imsi-208930000000003] Handle UL NAS Transport
[INFO][AMF][GMM][AMF_UE_NGAP_ID:23][SUPI:imsi-208930000000003] Transport 5GSM Message to SMF
[INFO][AMF][GMM][AMF_UE_NGAP_ID:23][SUPI:imsi-208930000000003] Select SMF [snssai: {Sst:1 Sd:010203}, dnn: internet]
[INFO][AMF][GMM][AMF_UE_NGAP_ID:23][SUPI:imsi-208930000000003] create smContext[pduSessionID: 1] Success
[INFO][AMF][Producer] Handle N1N2 Message Transfer Request
[INFO][AMF][NGAP][172.2.0.254:38273][AMF_UE_NGAP_ID:23] Send PDU Session Resource Setup Request
[INFO][AMF][GIN] | 200 | 10.121.0.52 | POST  | /namf-comm/v1/ue-contexts/imsi-208930000000003/n1-n2-messages |
[INFO][AMF][NGAP][172.2.0.254:38273][AMF_UE_NGAP_ID:23] Handle PDU Session Resource Setup Response
```

The log shows that the `AMF` receives a request from `gNodeB` in the `edge01` cluster.

<br></br>
---
|Before|Index|
|--|--|
|[ Go to Before Page](6_deploy_upf_amf_smf.md) | [ Go to Index Page ](README.md)|
