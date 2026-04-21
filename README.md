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

2. NAD configuration
```
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: coe-ovs-2003
  namespace: default
spec:
  config: '{ "name":"localnet1", "type":"ovn-k8s-cni-overlay", "cniVersion":"0.4.0", "topology":"localnet", "netAttachDefName":"default/coe-ovs-2003", "vlanID":2003 }'
```