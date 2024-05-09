# 4. Configure Network Topology

Nephio employs SR Linux to facilitate inter-connectivity among clusters. However, given the use of multiple servers, we need to ensure they are interconnected effectively. To achieve this, we use Open vSwitch (OVS) to establish connections from SR Linux to each cluster.

## 4.1 Setup Containerlab

### Create a network topology file:

```yaml
##### -----=[ In mgmt cluster ]=----- ####

name: 5g
prefix: net
topology:
  kinds:
    srl:
      type: ixrd3
      image: ghcr.io/nokia/srlinux:22.11.2-116
  nodes:
    leaf:
      kind: srl
      ports:
        - 57400:57400
      mgmt-ipv4: 172.20.20.120
  links:
    - endpoints: ["leaf:e1-1", "host:sr-r"]
    - endpoints: ["leaf:e1-2", "host:sr-e1"]
    - endpoints: ["leaf:e1-3", "host:sr-e2"]
```

This will deploy an SR Linux instance with interfaces `e1-1`, `e1-2`, and `e1-3`, which are connected to the host's `sr-r`, `sr-e1`, and `sr-e1`. Also, in the following steps, we will connect each interface to an OVS tunnel that utilizes VXLAN to link to the remote server.

### Deploy topology using containerlab

```bash
##### -----=[ In mgmt cluster ]=----- ####

sudo containerlab deploy --topo topology.yaml
```

### Create an OVS bridge in the `mgmt` cluster by:

```bash
##### -----=[ In mgmt cluster ]=----- ####

# add bridge
sudo ovs-vsctl add-br br-tun-r
sudo ovs-vsctl add-br br-tun-e1
sudo ovs-vsctl add-br br-tun-e2

# set interface
sudo ifconfig br-tun-r up
sudo ifconfig br-tun-e1 up
sudo ifconfig br-tun-e2 up
```

### Then, prepare to connect vxlans in the `mgmt` cluster by:

```bash
##### -----=[ In mgmt cluster ]=----- ####

# change to regional, edge01, edge02 IP address
sudo ovs-vsctl add-port br-tun-r vxlan0 -- set interface vxlan0 type=vxlan options:remote_ip=172.18.0.121 options:dst_port=48317 options:tag=321
sudo ovs-vsctl add-port br-tun-r vxlan0 -- set interface vxlan0 type=vxlan options:remote_ip=172.18.0.122 options:dst_port=48318 options:tag=321
sudo ovs-vsctl add-port br-tun-r vxlan0 -- set interface vxlan0 type=vxlan options:remote_ip=172.18.0.123 options:dst_port=48319 options:tag=321
```

## 4.2 Set up Open vSwitch

### In each worker cluster, establish connectivity with the counterpart by:

```bash
##### -----=[ In regional cluster ]=----- ####

# set ovs
sudo ovs-vsctl add-br eth1
sudo ifconfig eth1 up
sudo ovs-vsctl add-port eth1 vxlan0 -- set interface vxlan0 type=vxlan options:remote_ip=[mgmt_ip_address] options:dst_port=48317 options:tag=321
```

```bash
##### -----=[ In edge01 cluster ]=----- ####

# set ovs
sudo ovs-vsctl add-br eth1
sudo ifconfig eth1 up
sudo ovs-vsctl add-port eth1 vxlan0 -- set interface vxlan0 type=vxlan options:remote_ip=[mgmt_ip_address] options:dst_port=48318 options:tag=321
```

```bash
##### -----=[ In edge02 cluster ]=----- ####

# set ovs
sudo ovs-vsctl add-br eth1
sudo ifconfig eth1 up
sudo ovs-vsctl add-port eth1 vxlan0 -- set interface vxlan0 type=vxlan options:remote_ip=[mgmt_ip_address]  options:dst_port=48319 options:tag=321
```

These interfaces will later be connected to nodes `n3`, `n4`, and `n6`.

Also, each of these interfaces will be linked to the `eth1` OVS bridge on each worker node.

### Create interfaces for `eth1.2`-`eth1.6`

```bash
##### -----=[ In regional, edge01, edge02 clusters ]=----- ####

# link eth1.2-6 to eth1
sudo ip link add link eth1 name "eth1.2" type vlan id 2
sudo ip link add link eth1 name "eth1.3" type vlan id 3
sudo ip link add link eth1 name "eth1.4" type vlan id 4
sudo ip link add link eth1 name "eth1.5" type vlan id 5
sudo ip link add link eth1 name "eth1.6" type vlan id 6

#set interface
sudo ifconfig eth1.2 up
sudo ifconfig eth1.3 up
sudo ifconfig eth1.4 up
sudo ifconfig eth1.5 up
sudo ifconfig eth1.6 up
```

### Add ports with ovs-vsctl

```bash
##### -----=[ In mgmt cluster ]=----- ####

sudo ovs-vsctl add-port br-tun-r sr-r
sudo ovs-vsctl add-port br-tun-e1 sr-e1
sudo ovs-vsctl add-port br-tun-e2 sr-e2
```

## 4.3 Configure Nephio Topology

### Apply network settings to Nephio

```bash
##### -----=[ In mgmt cluster ]=----- ####

kubectl apply -f test-infra/e2e/tests/free5gc/002-network.yaml
kubectl apply -f test-infra/e2e/tests/free5gc/002-secret.yaml
```

### Create a `RawTopology` file

```yaml
##### -----=[ In mgmt cluster ]=----- ####

# change ip address to mgmt cluster's ip!
apiVersion: topo.nephio.org/v1alpha1
kind: RawTopology
metadata:
  name: nephio
spec:
  nodes:
    srl:
      address: [mgmt_ip_address]:57400
      provider: srl.nokia.com
    mgmt:
      provider: host
    regional:
      provider: host
      labels:
        nephio.org/cluster-name: regional
    edge01:
      provider: host
      labels:
        nephio.org/cluster-name: edge01
    edge02:
      provider: host
      labels:
        nephio.org/cluster-name: edge02
  links:
  - endpoints:
    - { nodeName: srl, interfaceName: e1-1}
    - { nodeName: regional, interfaceName: eth1}
  - endpoints:
    - { nodeName: srl, interfaceName: e1-2}
    - { nodeName: edge01, interfaceName: eth1}
  - endpoints:
    - { nodeName: srl, interfaceName: e1-3}
    - { nodeName: edge02, interfaceName: eth1}
```

### Apply this topology to Nephio

```bash
##### -----=[ In mgmt cluster ]=----- ####

kubectl create -f topo.yaml
```

<br></br>
---
|Before|Next|
|--|--|
|[ Go to Before Page](3_add_k8s_clusters_to_nephio.md) | [ Go to Next Page ](5_deploy_free5gc_cp.md)|
