1. Localnet network
```
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: mapping
spec:
  desiredState:
    ovn:
      bridge-mappings:
        - bridge: br-ex
          localnet: localnet1
          state: present
  nodeSelector:
    external-network: 'true'
    node-role.kubernetes.io/worker: ''
```

2. CUDN configuration for vlan 2003
```
apiVersion: k8s.ovn.org/v1
kind: ClusterUserDefinedNetwork
metadata:
  name: cudn-localnet1-2003
spec:
  namespaceSelector:
    matchExpressions:
      - key: kubernetes.io/metadata.name
        operator: In
        values:
          - default
  network:
    localnet:
      ipam:
        mode: Disabled
      physicalNetworkName: localnet1
      role: Secondary
      vlan:
        access:
          id: 2003
        mode: Access
    topology: Localnet
```

3. CUDN configuration for vlan 2005
```
apiVersion: k8s.ovn.org/v1
kind: ClusterUserDefinedNetwork
metadata:
  name: cudn-localnet1-2005
spec:
  namespaceSelector:
    matchExpressions:
      - key: kubernetes.io/metadata.name
        operator: In
        values:
          - default
  network:
    localnet:
      ipam:
        mode: Disabled
      physicalNetworkName: localnet1
      role: Secondary
      vlan:
        access:
          id: 2005
        mode: Access
    topology: Localnet
```


4. MetalLB VLAN2003 pool
```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ip-addresspool-vlan2003
  namespace: metallb-system
spec:
  addresses:
    - 192.168.203.40-192.168.203.49
  autoAssign: true
  avoidBuggyIPs: false
```

4. MetalLB VLAN2005 pool
```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ip-addresspool-vlan2005
  namespace: metallb-system
spec:
  addresses:
    - 192.168.205.40-192.168.205.49
  autoAssign: true
  avoidBuggyIPs: false
```


6. L2 Advertissment
```
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: vlan2003-adv
  namespace: metallb-system
spec:
  ipAddressPools:
  - ip-addresspool-vlan2003
  nodeSelector:
  - matchLabels:
      coe.muc.redhat.com/second-nic: enp11s0
      node-role.kubernetes.io/worker: ''
  interfaces:
  - enp11s0.100
```