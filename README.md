# ansible-role-kubernetes-worker

This Ansible role is used in [Kubernetes the not so hard way with Ansible - Worker](https://www.tauceti.blog/posts/kubernetes-the-not-so-hard-way-with-ansible-worker-2020/). This Ansible role setup Kubernetes worker nodes. For more information please see [Kubernetes the not so hard way with Ansible - Worker](https://www.tauceti.blog/posts/kubernetes-the-not-so-hard-way-with-ansible-worker-2020/).

## Versions

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `33.0.0+1.36.6` means this is release `33.0.0` of this role and it's meant to be used with Kubernetes version `1.36.6` (but should work with any K8s 1.36.x release of course). If the role itself changes `X.Y.Z` before `+` will increase. If the Kubernetes version changes `X.Y.Z` after `+` will increase too. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release. That's especially useful for Kubernetes major releases with breaking changes.

## Requirements

This playbook expects that you already have rolled out the Kubernetes controller components (see [kubernetes-controller](https://github.com/githubixx/ansible-role-kubernetes-controller) and [Kubernetes the not so hard way with Ansible - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-control-plane/).

You also need [containerd](https://github.com/githubixx/ansible-role-containerd) **2.x**, [CNI plugins](https://github.com/githubixx/ansible-role-cni) and [runc](https://github.com/githubixx/ansible-role-runc) installed. This role checks the installed containerd version before provisioning workers; containerd 1.x is no longer supported by this role. To enable Kubernetes `Pods` to communicate between different hosts it makes sense to install [Cilium](https://galaxy.ansible.com/githubixx/cilium_kubernetes) later once the worker nodes are running e.g. Of course `Calico`, `WeaveNet`, `kube-router` or other Kubernetes network solutions are valid options.

For Kubernetes `1.36.x` worker hosts should use cgroup v2. This role retains `cgroupDriver: "systemd"` in the kubelet configuration, but it does not change the host cgroup mode. Kubernetes 1.36 queries containerd 2.x through the CRI `RuntimeConfig` API and uses the reported cgroup driver instead of this kubelet setting. Configure containerd's runc runtime with `SystemdCgroup = true`; leaving the kubelet setting in place is harmless but does not override the runtime. Requiring containerd 2.x is this role's support policy, not an upstream Kubernetes 1.36 minimum.

The default kube-proxy backend is now `nftables`. It requires Linux kernel **5.13 or newer** and **nft 1.0.0 or newer**. The role installs nftables tooling and checks these requirements when nftables mode is selected. Verify that your CNI and host firewall support this backend.

## Supported OS

- Ubuntu 22.04 (Jammy Jellyfish)
- Ubuntu 24.04 (Noble Numbat) (recommended)
- Ubuntu 26.04 (Resolute Raccoon)

## Changelog

**Change history:**

See full [CHANGELOG.md](https://github.com/githubixx/ansible-role-kubernetes-worker/blob/master/CHANGELOG.md)

**IMPORTANT** Version `24.0.0+1.27.8` had a lot of potential breaking changes. So if you upgrade from a version < `24.0.0+1.27.8` please read the CHANGELOG of that version too!

**Recent changes:**

## 33.0.0+1.36.6

- **BREAKING**
  - require containerd 2.x before provisioning workers; containerd 1.x is no longer supported by this role
  - default kube-proxy to `nftables` instead of deprecated `ipvs`, as announced in the previous release
  - nftables mode requires Linux kernel 5.13+ and nft 1.0.0+; NodePorts default to primary node addresses and require explicit host firewall allowances

- **UPDATE**
  - update `k8s_worker_release` to `1.36.6`
  - add Ubuntu 26.04 support and install nftables tooling
  - preserve the existing masquerading policy, explicitly select primary node addresses for NodePorts, and use upstream nftables synchronization intervals (`minSyncPeriod: 1s`, `syncPeriod: 30s`)
  - restart kubelet and kube-proxy when their configuration files change

- **MOLECULE**
  - cover Ubuntu 26.04 on a controller and a dedicated worker; align controller and assets-host kubectl with Kubernetes 1.36.6
  - include idempotence and verify in the test sequence; install networking, DNS, and persistent smoke fixtures during converge
  - disable Cilium kube-proxy replacement and socket load balancing so service checks exercise kube-proxy
  - install CoreDNS through `githubixx.coredns`, preserving the DNS Service address and Corefile; prepare Helm 3 and Helm-diff on the assets host
  - verify runtime dependencies, worker versions, health endpoints, nftables rules, fine-grained kubelet RBAC, cross-node service traffic, DNS, NodePorts, and exec streaming without creating or deleting workloads

- **UPGRADE NOTES**
  - upgrade the control plane first; controller role `31.0.0+1.36.6` includes the fine-grained kubelet authorization permissions that became stable in Kubernetes 1.36
  - upgrade containerd to 2.x and configure its runc runtime with `SystemdCgroup = true`; keep cgroup v2 and the systemd cgroup driver
  - preserve kubelet TLS and webhook authorization for direct WebSocket streaming; regenerate embedded kubeconfig certificates when rotating their CA
  - custom `k8s_worker_kubeproxy_conf_yaml` values are not rewritten; see [CHANGELOG.md](CHANGELOG.md) for migration and an explicit IPVS override

## 32.0.0+1.35.5

- **BREAKING**
  - Kubernetes v1.35+ requires worker hosts to use cgroup v2 by default. This role does not manage the host cgroup mode.

- **UPDATE**
  - update `k8s_worker_release` to `1.35.5`

- **OTHER**
  - keep `kube-proxy` default mode `ipvs` although Kubernetes v1.35 deprecates it. This will change with the next release of this role and then default to `nftables`. Also see [NFTables mode for kube-proxy](https://kubernetes.io/blog/2025/02/28/nftables-kube-proxy/).
  - document that user supplied kubelet flags must not add the removed `--pod-infra-container-image` option

## 31.0.0+1.34.4

- **UPDATE**
  - update `k8s_ctl_release` to `1.34.4`

- **OTHER**
  - replace injected `ansible_*` facts usage with `ansible_facts[...]` (prepares for ansible-core 2.24 where `INJECT_FACTS_AS_VARS` default changes)

- **MOLECULE**
  - use own [githubixx Vagrant boxes](https://portal.cloud.hashicorp.com/vagrant/discover/githubixx)
  - add more checks in `verify.yml`

## Installation

- Directly download from Github (Change into Ansible roles directory before cloning. You can figure out the role path by using `ansible-config dump | grep DEFAULT_ROLES_PATH` command):
`git clone https://github.com/githubixx/ansible-role-kubernetes-worker.git githubixx.kubernetes_worker`

- Via `ansible-galaxy` command and download directly from Ansible Galaxy:
`ansible-galaxy install role githubixx.kubernetes_worker`

- Create a `requirements.yml` file with the following content (this will download the role from Github) and install with
`ansible-galaxy role install -r requirements.yml` (change `version` if needed):

```yaml
---
roles:
  - name: githubixx.kubernetes_worker
    src: https://github.com/githubixx/ansible-role-kubernetes-worker.git
    version: 33.0.0+1.36.6
```

## Role Variables

```yaml
# The base directory for Kubernetes configuration and certificate files for
# everything worker nodes related. After the playbook is done this directory
# contains various sub-folders.
k8s_worker_conf_dir: "/etc/kubernetes/worker"

# All certificate files (Private Key Infrastructure related) specified in
# "k8s_worker_certificates" (see "vars/main.yml") will be stored here.
# Owner and group of this new directory will be "root". File permissions
# will be "0640".
k8s_worker_pki_dir: "{{ k8s_worker_conf_dir }}/pki"

# The directory to store the Kubernetes binaries (see "k8s_worker_binaries"
# variable in "vars/main.yml"). Owner and group of this new directory
# will be "root" in both cases. Permissions for this directory will be "0755".
#
# NOTE: The default directory "/usr/local/bin" normally already exists on every
# Linux installation with the owner, group and permissions mentioned above. If
# your current settings are different consider a different directory. But make sure
# that the new directory is included in your "$PATH" variable value.
k8s_worker_bin_dir: "/usr/local/bin"

# K8s release
k8s_worker_release: "1.36.6"

# The interface on which the Kubernetes services should listen on. As all cluster
# communication should use a VPN interface the interface name is
# normally "wg0" (WireGuard),"peervpn0" (PeerVPN) or "tap0".
#
# The network interface on which the Kubernetes worker services should
# listen on. That is:
#
# - kube-proxy
# - kubelet
#
k8s_interface: "eth0"

# The directory from where to copy the K8s certificates. By default this
# will expand to user's LOCAL $HOME (the user that run's "ansible-playbook ..."
# plus "/k8s/certs". That means if the user's $HOME directory is e.g.
# "/home/da_user" then "k8s_ca_conf_directory" will have a value of
# "/home/da_user/k8s/certs".
k8s_ca_conf_directory: "{{ '~/k8s/certs' | expanduser }}"

# The IP address or hostname of the Kubernetes API endpoint. This variable
# is used by "kube-proxy" and "kubelet" to connect to the "kube-apiserver"
# (Kubernetes API server).
#
# By default the first host in the Ansible group "k8s_controller" is
# specified here. NOTE: This setting is not fault tolerant! That means
# if the first host in the Ansible group "k8s_controller" is down
# the worker node and its workload continue working but the worker
# node doesn't receive any updates from Kubernetes API server.
#
# If you have a loadbalancer that distributes traffic between all
# Kubernetes API servers it should be specified here (either its IP
# address or the DNS name). But you need to make sure that the IP
# address or the DNS name you want to use here is included in the
# Kubernetes API server TLS certificate (see "k8s_apiserver_cert_hosts"
# variable of https://github.com/githubixx/ansible-role-kubernetes-ca
# role). If it's not specified you'll get certificate errors in the
# logs of the services mentioned above.
k8s_worker_api_endpoint_host: "{{ hostvars[groups['k8s_controller'] | first]['ansible_facts'][hostvars[groups['k8s_controller'] | first]['k8s_interface']]['ipv4']['address'] }}"

# As above just for the port. It specifies on which port the
# Kubernetes API servers are listening. Again if there is a loadbalancer
# in place that distributes the requests to the Kubernetes API servers
# put the port of the loadbalancer here.
k8s_worker_api_endpoint_port: "6443"

# OS packages needed on a Kubernetes worker node. You can add additional
# packages at any time. But please be aware if you remove one or more from
# the default list your worker node might not work as expected or doesn't work
# at all.
k8s_worker_os_packages:
  - ebtables
  - ethtool
  - ipset
  - conntrack
  - iptables
  - nftables
  - iptstate
  - netstat-nat
  - socat
  - netbase

# Directory to store kubelet configuration
k8s_worker_kubelet_conf_dir: "{{ k8s_worker_conf_dir }}/kubelet"

# kubelet settings
#
# If you want to enable the use of "RuntimeDefault" as the default seccomp
# profile for all workloads add these settings to "k8s_worker_kubelet_settings":
#
# "seccomp-default": ""
#
# Also see:
# https://kubernetes.io/docs/tutorials/security/seccomp/#enable-the-use-of-runtimedefault-as-the-default-seccomp-profile-for-all-workloads
k8s_worker_kubelet_settings:
  "config": "{{ k8s_worker_kubelet_conf_dir }}/kubelet-config.yaml"
  "node-ip": "{{ hostvars[inventory_hostname]['ansible_facts'][k8s_interface]['ipv4']['address'] }}"
  "kubeconfig": "{{ k8s_worker_kubelet_conf_dir }}/kubeconfig"

# kubelet kubeconfig
#
# NOTE: Kubernetes v1.35+ no longer starts with cgroup v1 by default. This
# role configures kubelet for the systemd cgroup driver but does not manage the
# host cgroup mode itself, so worker hosts should use cgroup v2.
#
# Also note that this role does not add the removed kubelet
# "--pod-infra-container-image" flag by default. If you use
# "k8s_worker_kubelet_settings_user" do not add that flag on Kubernetes v1.35+.
k8s_worker_kubelet_conf_yaml: |
  kind: KubeletConfiguration
  apiVersion: kubelet.config.k8s.io/v1beta1
  address: {{ hostvars[inventory_hostname]['ansible_facts'][k8s_interface]['ipv4']['address'] }}
  authentication:
    anonymous:
      enabled: false
    webhook:
      enabled: true
    x509:
      clientCAFile: "{{ k8s_worker_pki_dir }}/ca-k8s-apiserver.pem"
  authorization:
    mode: Webhook
  clusterDomain: "cluster.local"
  clusterDNS:
    - "10.32.0.254"
  failSwapOn: true
  healthzBindAddress: "{{ hostvars[inventory_hostname]['ansible_facts'][k8s_interface]['ipv4']['address'] }}"
  healthzPort: 10248
  runtimeRequestTimeout: "15m"
  serializeImagePulls: false
  tlsCertFile: "{{ k8s_worker_pki_dir }}/cert-{{ inventory_hostname }}.pem"
  tlsPrivateKeyFile: "{{ k8s_worker_pki_dir }}/cert-{{ inventory_hostname }}-key.pem"
  cgroupDriver: "systemd"
  registerNode: true
  containerRuntimeEndpoint: "unix:///run/containerd/containerd.sock"

# Directory to store kube-proxy configuration
k8s_worker_kubeproxy_conf_dir: "{{ k8s_worker_conf_dir }}/kube-proxy"

# kube-proxy settings
k8s_worker_kubeproxy_settings:
  "config": "{{ k8s_worker_kubeproxy_conf_dir }}/kubeproxy-config.yaml"

k8s_worker_kubeproxy_conf_yaml: |
  kind: KubeProxyConfiguration
  apiVersion: kubeproxy.config.k8s.io/v1alpha1
  bindAddress: {{ hostvars[inventory_hostname]['ansible_facts'][k8s_interface]['ipv4']['address'] }}
  clientConnection:
    kubeconfig: "{{ k8s_worker_kubeproxy_conf_dir }}/kubeconfig"
  healthzBindAddress: {{ hostvars[inventory_hostname]['ansible_facts'][k8s_interface]['ipv4']['address'] }}:10256
  mode: "nftables"
  nodePortAddresses:
    - primary
  nftables:
    minSyncPeriod: 1s
    syncPeriod: 30s
    masqueradeAll: true
  clusterCIDR: "10.200.0.0/16"
```

## Dependencies

- [kubernetes_controller](https://github.com/githubixx/ansible-role-kubernetes-controller)
- [containerd](https://github.com/githubixx/ansible-role-containerd)
- [runc](https://github.com/githubixx/ansible-role-runc)
- [CNI plugins](https://github.com/githubixx/ansible-role-cni)

## Example Playbook

```yaml
- hosts: k8s_worker
  roles:
    - githubixx.kubernetes_worker
```

## Testing

This role has a small test setup that is created using [Molecule](https://github.com/ansible-community/molecule), libvirt (vagrant-libvirt) and QEMU/KVM. Please see my blog post [Testing Ansible roles with Molecule, libvirt (vagrant-libvirt) and QEMU/KVM](https://www.tauceti.blog/posts/testing-ansible-roles-with-molecule-libvirt-vagrant-qemu-kvm/) how to setup. The Molecule test configuration is in [molecule/default](https://github.com/githubixx/ansible-role-kubernetes-worker/tree/master/molecule/default).

The scenario creates ten VMs covering Ubuntu 22.04, 24.04, and 26.04. A normal converge installs workers, Cilium, CoreDNS, and persistent smoke-test workloads:

```bash
molecule converge
```

For the complete automated sequence (`prepare`, `converge`, `idempotence`, `verify`, preceded by VM creation), run:

```bash
molecule test
```

The two-stage workflow is still available. The first command deliberately leaves networking uninstalled; the second installs Cilium, CoreDNS, and smoke-test fixtures without reinstalling workers:

```bash
molecule converge -- --extra-vars k8s_worker_setup_networking=skip
molecule converge -- --extra-vars k8s_worker_setup_networking=install
```

Verification requires networking and fixtures to be present; missing dependencies fail rather than silently skipping smoke checks. It checks service health, exact worker versions, fine-grained kubelet RBAC, nftables rules, NodePorts, cross-node ClusterIP traffic, DNS, and kubelet exec streaming. Cilium's kube-proxy replacement and socket load balancing are disabled so service checks exercise kube-proxy. Verification does not create or delete cluster resources:

```bash
molecule verify
```

Cilium is installed through `githubixx.cilium_kubernetes`, using the scenario's custom Helm values template. Its installation tasks create and remove temporary values files on each run, which currently causes the full scenario's idempotence check to report changes.

CoreDNS is installed through `githubixx.coredns`. Preparation installs Helm 3 and the Helm-diff plugin for `vagrant` on the assets host. The scenario preserves the `kube-dns` Service at `10.32.0.254` with UDP/TCP port 53, two replicas, resource requests and memory limit, tolerations, and the previous Corefile (including upstreams `1.1.1.1` and `9.9.9.9`). The role supplies its chart's current image, labels/annotations, readiness probe, disruption budget, and safer rollout settings.

On `test-assets`, `vagrant` can use `/home/vagrant/.kube/config`, which links to `/home/vagrant/k8s/configs/admin.kubeconfig`.

To clean up run

```bash
molecule destroy
```

## License

GNU GENERAL PUBLIC LICENSE Version 3

## Author Information

[http://www.tauceti.blog](http://www.tauceti.blog)
