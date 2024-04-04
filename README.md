# ansible-role-kubernetes-worker

This Ansible role is used in [Kubernetes the not so hard way with Ansible - Worker](https://www.tauceti.blog/posts/kubernetes-the-not-so-hard-way-with-ansible-worker-2020/). This Ansible role setup Kubernetes worker nodes. For more information please see [Kubernetes the not so hard way with Ansible - Worker](https://www.tauceti.blog/posts/kubernetes-the-not-so-hard-way-with-ansible-worker-2020/).

## Versions

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `24.0.0+1.27.8` means this is release `24.0.0` of this role and it's meant to be used with Kubernetes version `1.27.8` (but should work with any K8s 1.27.x release of course). If the role itself changes `X.Y.Z` before `+` will increase. If the Kubernetes version changes `X.Y.Z` after `+` will increase too. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release. That's especially useful for Kubernetes major releases with breaking changes.

## Requirements

This playbook expects that you already have rolled out the Kubernetes controller components (see [kubernetes-controller](https://github.com/githubixx/ansible-role-kubernetes-controller) and [Kubernetes the not so hard way with Ansible - Control plane](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-control-plane/).

You also need [containerd](https://github.com/githubixx/ansible-role-containerd), [CNI plugins](https://github.com/githubixx/ansible-role-cni) and [runc](https://github.com/githubixx/ansible-role-runc) installed. To enable Kubernetes `Pods` to communicate between different hosts it makes sense to install [Cilium](https://galaxy.ansible.com/githubixx/cilium_kubernetes) later once the worker nodes are running e.g. Of course `Calico`, `WeaveNet`, `kube-router` or [flannel](https://galaxy.ansible.com/githubixx/flanneld) or other Kubernetes network solutions are valid options.

## Supported OS

- Ubuntu 20.04 (Focal Fossa)
- Ubuntu 22.04 (Jammy Jellyfish)

## Changelog

**Change history:**

See full [CHANGELOG.md](https://github.com/githubixx/ansible-role-kubernetes-worker/blob/master/CHANGELOG.md)

**Recent changes:**

## 25.0.0+1.28.5

### PLEASE READ CAREFULLY

Version `24.0.0+1.27.8` had a lot of potential breaking changes. So if you upgrade from a version < `24.0.0+1.27.8` please read the CHANGELOG of that version too!

### UPDATE

- update `k8s_release` to `1.28.5`

### OTHER CHANGES

- adjust Github action because of Ansible Galaxy changes
- `.yamllint`: extend max line length from 200 to 300

### MOLECULE

- change to Ubuntu 22.04 for test-assets VM
- change IP addresses
- adjust common names for certificates / change algo to ecdsa and algo size
- remove `collections.yml"

## 24.0.0+1.27.8

### PLEASE READ CAREFULLY

This release contains quite a few potential breaking changes! So review carefully before rolling out the new version of this role! A bigger part of the whole changes are related to increase security. While most of the new variables
 and defaults should be just fine and should just work out of the box side effects might occur.

All the newly introduced or changed variables have detailed comments in [README](https://github.com/githubixx/ansible-role-kubernetes-worker/blob/master/README.md). So please read them carefully!

This refactoring was needed to make it possible to have `githubixx.kubernetes_controller` and `githubixx.kubernetes_worker` deployed on the same host e.g. They were some intersections between the two roles that had to be fixed.

### UPDATE

- update `k8s_release` to `1.27.8`

### BREAKING

- Rename variable `k8s_conf_dir` to `k8s_worker_conf_dir`. Additionally the default value changed from `/usr/lib/kubernetes` to `/etc/kubernetes/worker`.
- Rename variable `k8s_bin_dir` to `k8s_worker_bin_dir`.
- `k8s_worker_binaries` variable is no longer defined in `defaults/main.yml` but in `vars/main.yml`. Since this list is fixed anyways it makes no sense to allow to modify this list.
- `k8s_worker_certificates` variable is no longer defined in `defaults/main.yml` but in `vars/main.yml`. Since this list is fixed anyways it makes no sense to allow to modify this list.
- Introduce variable `k8s_worker_pki_dir`. All certificate files specified in `k8s_worker_certificates` (see `vars/main.yml`) will be stored here. Related to this: Certificate related settings in `k8s_worker_kubelet_conf_yaml` used
 `k8s_conf_dir` before and now use `k8s_ctl_pki_dir`. That's `clientCAFile`, `tlsCertFile` and `tlsPrivateKeyFile`.
- The default value for `k8s_interface` changed from `tap0` to `eth0`.
- The variable `k8s_config_directory` is gone. It's no longer in use. After the upgrade to this release you can delete this directory (if you accept the new default!) and it's content (make a backup esp. of `admin.kubeconfig` file
- just in case!)
- Remove variable `k8s_worker_download_dir` (no longer needed).
- Change default value of `k8s_worker_kubelet_conf_dir` to `{{ k8s_worker_conf_dir }}/kubelet`.
- Change default value of `k8s_worker_kubeproxy_conf_dir` to `{{ k8s_worker_conf_dir }}/kube-proxy`.

### FEATURE

- When downloading the Kubernetes binaries the task checks the SHA512 checksum.
- Introduce `k8s_worker_api_endpoint_host` and `k8s_worker_api_endpoint_port` variables. Previously `kubelet` and `kube-proxy` where configured to connect to the first host in the Ansible `k8s_controller` group and communicate with the `kube-apiserver` that was running there. This was hard-coded and couldn't be changed. If that host was down the K8s worker nodes didn't receive any updates. Now one can install and use a load balancer like `haproxy` e.g. that distributes requests between all `kube-apiserver`'s and takes a `kube-apiserver` out of rotation if that one is down (also see my Ansible [haproxy role](https://github.com/githubixx/ansible-role-haproxy) for that use case). The default is still to use the first host/kube-apiserver in the Ansible `k8s_controller` group. So behaviorwise nothing changed basically.
- Add task to generate `kubeconfig` for `kubelet` service (previously this was a separate [playbook](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/v15.0.0_r1.27.5/kubeauthconfig)).
- Add task to generate `kubeconfig` for `kube-proxy` service (previously this was a separate [playbook](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/v15.0.0_r1.27.5/kubeauthconfig)).

### OTHER CHANGES

- Use `kubernetes.core.*` modules instead of `kubectl` binary
- Fix some `ansible-lint` issues

### MOLECULE

- Updated all files to reflect the changes introduces with this version
- Tasks for creating `kubeconfig` for `kubelet` and `kube-proxy` are no longer needed as they're now part of `kubernetes_worker` role
- Add `haproxy` to Ubuntu 22 hosts to test new `k8s_worker_api_endpoint_host` and `k8s_worker_api_endpoint_port` settings
- Add tasks to install [ansible-role-cni](https://github.com/githubixx/ansible-role-cni) and [ansible-role-runc](https://github.com/githubixx/ansible-role-runc)
- Use `kubernetes.core.k8s_info` module instead of calling `kubectl` binary

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
    version: 25.0.0+1.28.5
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
k8s_worker_release: "1.28.8"

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
k8s_worker_api_endpoint_host: "{% set controller_host = groups['k8s_controller'][0] %}{{ hostvars[controller_host]['ansible_' + hostvars[controller_host]['k8s_interface']].ipv4.address }}"

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
  "node-ip": "{{ hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address }}"
  "kubeconfig": "{{ k8s_worker_kubelet_conf_dir }}/kubeconfig"

# kubelet kubeconfig
k8s_worker_kubelet_conf_yaml: |
  kind: KubeletConfiguration
  apiVersion: kubelet.config.k8s.io/v1beta1
  address: {{ hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address }}
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
  healthzBindAddress: "{{ hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address }}"
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
  bindAddress: {{ hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address }}
  clientConnection:
    kubeconfig: "{{ k8s_worker_kubeproxy_conf_dir }}/kubeconfig"
  healthzBindAddress: {{ hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address }}:10256
  mode: "ipvs"
  ipvs:
    minSyncPeriod: 0s
    scheduler: ""
    syncPeriod: 2s
  iptables:
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

This role has a small test setup that is created using [Molecule](https://github.com/ansible-community/molecule), libvirt (vagrant-libvirt) and QEMU/KVM. Please see my blog post [Testing Ansible roles with Molecule, libvirt (vagrant-libvirt) and QEMU/KVM](https://www.tauceti.blog/posts/testing-ansible-roles-with-molecule-libvirt-vagrant-qemu-kvm/) how to setup. The test configuration is [here](https://github.com/githubixx/ansible-role-kubernetes-worker/tree/master/molecule/default).

Afterwards Molecule can be executed. This will setup a few virtual machines (VM) with supported Ubuntu OS and installs a Kubernetes cluster:

```bash
molecule converge
```

At this time the cluster isn't fully functional as a network plugin is missing e.g. So Pod to Pod communication between two different nodes isn't possible yet. To fix this the following command can be used to install [Cilium](https://github.com/githubixx/ansible-role-cilium-kubernetes) for all Kubernetes networking needs and [CoreDNS](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/master/coredns) for Kubernetes DNS stuff:

```bash
molecule converge -- --extra-vars k8s_worker_setup_networking=install
```

After this you basically have a fully functional Kubernetes cluster.

A small verification step is also included:

```bash
molecule verify
```

To clean up run

```bash
molecule destroy
```

## License

GNU GENERAL PUBLIC LICENSE Version 3

## Author Information

[http://www.tauceti.blog](http://www.tauceti.blog)
