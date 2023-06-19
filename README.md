# OCP4 Agent Installation

```bash
# Reference Blog
# https://cloud.redhat.com/blog/meet-the-new-agent-based-openshift-installer-1

mkdir -p ocp4-agent-install
cd ocp4-agent-install

cat << EOF > agent-config.yaml
  apiVersion: v1alpha1
  kind: AgentConfig
  rendezvousIP: 192.168.111.100
  hosts:
    - hostname: master0
      role: master
      interfaces:
        - name: enp0s4
          macAddress: 00:21:50:90:c0:10
        - name: enp0s5
          macAddress: 00:21:50:90:c0:20
      networkConfig:
        interfaces:
          - name: bond0
            type: bond
            state: up
            mac-address: 00:21:50:90:c0:10
            ipv4:
              enabled: true
              address:
                - ip: 192.168.111.100
                  prefix-length: 24
            ipv6:
              enabled: false
            link-aggregation:
              mode: active-backup
              options:
                miimon: "150"
              port:
                - enp0s4
                - enp0s5
        dns-resolver:
          config:
            server:
              - 10.10.10.11
              - 10.10.10.12
        routes:
          config:
            - destination: 0.0.0.0/0
              next-hop-address: 192.168.111.1
              next-hop-interface: bond0
              table-id: 254
EOF

cat << EOF > install-config.yaml
apiVersion: v1
baseDomain: example.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: three-node-cluster
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 192.168.111.0/24
  serviceNetwork:
  - 172.30.0.0/16
  networkType: OpenShiftSDN
platform:
  baremetal:
    apiVIPs:
    - 192.168.111.5
    ingressVIPs:
    - 192.168.111.6
fips: false
pullSecret: |
  {"auths":{"cl......
sshKey: |
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAA.....
EOF

# Install dependencies (nmstatectl)
# https://github.com/nmstate/nmstate
sudo dnf install nmstate

# Install (oc and kubectl)
# https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
tar -xvf openshift-client-linux.tar.gz
sudo mv oc /usr/local/bin/

# Install (openshift-install)
# https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
tar -xvf openshift-client-linux.tar.gz

# Create image
./openshift-install agent create image

# After image is created:
tree
# .
# ├── agent.x86_64.iso
# ├── auth
# │   ├── kubeadmin-password
# │   └── kubeconfig
# ├── clients
# │   ├── kubectl
# │   └── oc
# ├── ocp4-huk-3node-cluster
# ├── openshift-install
# ├── original
# │   ├── agent-config.yaml
# │   └── install-config.yaml
# ├── README.md
# └── rendezvousIP
```
