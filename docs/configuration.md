# MKE configuration

## Control plane

MKE Control plane can be configured via a YAML config file. By default `mke server` command reads a file called `mke.yaml` but can be told to read any yaml file via `--config` option.

An example config file with defaults:

```yaml
apiVersion: mke.mirantis.com/v1beta1
kind: Cluster
metadata:
  name: mke
spec:
  storage:
    type: etcd
    etcd:
      peerAddress: 1.2.3.4 # Defaults to first non-local address found on the node.
  api:
    address: 172.17.0.3 # Address where the k8s API is accessed at.
    sans: # If you want to incorporate multiple addresses into the generates api server certs
      - 13.48.10.8
  network:
    provider: calico
    podCIDR: "10.244.0.0/16"
    serviceCIDR: "10.96.0.0/12"
    calico:
      mode: vxlan # ipip also supported
      mtu: 1450
      vxlanPort: 4789
      vxlanVNI: 4096

```

### `spec.storage`

- `type`: Type of the data store, either `etcd` or `kine`.
- `etcd.peerAddress`: Nodes address to be used for etcd cluster peering.
- `kine.dataSource`: [kine](https://github.com/rancher/kine/) datasource URL.

Using type `etcd` will make mke to create and manage an elastic etcd cluster within the controller nodes.

### `spec.api`

- `address`: The local address to bind API on. Also used as one of the addresses pushed on the mke create service certificate on the API. Defaults to first non-local address found on the node.
- `sans`: List of additional addresses to push to API servers serving certificate

### `spec.network`

- `provider`: Network provider, either `calico` or `custom`. In case of `custom` user can push any network provider.
- `podCIDR`: Pod network CIDR to be used in the cluster
- `serviceCIDR`: Network CIDR to be used for cluster VIP services.

#### `spec.network.calico`

- `mode`: `vxlan` (default) or `ipip`
- `vxlanPort`: The UDP port to use for VXLAN (default `4789`)
- `vxlanVNI`: The virtual network ID to use for VXLAN. (default: `4096`)
- `mtu`: MTU to use for overlay network (default `1450`)

## Configuring multi-node controlplane

When configuring an elastic/HA controlplane one must use same configuration options on each node for the cluster level options. Following options need to match on each node, otherwise the control plane component will end up in very unknown states:
- `network`
- `storage`: Needless to say, one cannot create a clustered controlplane with each node only storing data locally on SQLite.