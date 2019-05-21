ansible-role-kubernetes-worker
==============================

This Ansible playbook is used in [Kubernetes the not so hard way with Ansible - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-worker/). This playbook installs Kubernetes worker including kubelet and kube-proxy. For more information please see [Kubernetes the not so hard way with Ansible - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-worker/).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `8.0.0+1.13.2` means this is release `8.0.0` of this role and it's meant to be used with Kubernetes version `1.13.2` (but should work with any K8s 1.13.x release of course). If the role itself changes `X.Y.Z` before `+` will increase. If the Kubernetes version changes `X.Y.Z` after `+` will increase too. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release. That's especially useful for Kubernetes major releases with breaking changes.

Requirements
------------

This playbook expects that you already have rolled out the Kubernetes controller components (see https://galaxy.ansible.com/githubixx/kubernetes-controller/ and https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-control-plane/). You also need Docker (https://galaxy.ansible.com/githubixx/docker/) and flannel (https://galaxy.ansible.com/githubixx/flanneld/) roles installed.

Changelog
---------

see [CHANGELOG.md](https://github.com/githubixx/ansible-role-kubernetes-worker/blob/master/CHANGELOG.md)

Role Variables
--------------

```
# The directory to store the K8s certificates and other configuration
k8s_conf_dir: "/var/lib/kubernetes"
# The directory to store the K8s binaries
k8s_bin_dir: "/usr/local/bin"
# K8s release
k8s_release: "1.14.2"
# The interface on which the K8s services should listen on. As all cluster
# communication should use a VPN interface the interface name is
# normally "wg0" (WireGuard),"peervpn0" (PeerVPN) or "tap0".
k8s_interface: "tap0"

# The directory from where to copy the K8s certificates. By default this
# will expand to user's LOCAL $HOME (the user that run's "ansible-playbook ..."
# plus "/k8s/certs". That means if the user's $HOME directory is e.g.
# "/home/da_user" then "k8s_ca_conf_directory" will have a value of
# "/home/da_user/k8s/certs".
k8s_ca_conf_directory: "{{ '~/k8s/certs' | expanduser }}"
# Directory where kubeconfig for Kubernetes worker nodes and kube-proxy
# is stored among other configuration files. Same variable expansion
# rule applies as with "k8s_ca_conf_directory"
k8s_config_directory: "{{ '~/k8s/configs' | expanduser }}"

# K8s worker binaries to download
k8s_worker_binaries:
  - kube-proxy
  - kubelet
  - kubectl

# Certificate/CA files for API server and kube-proxy
k8s_worker_certificates:
  - ca-k8s-apiserver.pem
  - ca-k8s-apiserver-key.pem
  - cert-k8s-apiserver.pem
  - cert-k8s-apiserver-key.pem

# Download directory for archive files
k8s_worker_download_dir: "/opt/tmp"

# Directory to store kubelet configuration
k8s_worker_kubelet_conf_dir: "/var/lib/kubelet"

# kubelet settings
k8s_worker_kubelet_settings:
  "allow-privileged": "true"
  "config": "{{k8s_worker_kubelet_conf_dir}}/kubelet-config.yaml"
  "node-ip": "{{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}"
  "container-runtime": "docker"
  "image-pull-progress-deadline": "2m"
  "kubeconfig": "{{k8s_worker_kubelet_conf_dir}}/kubeconfig"
  "network-plugin": "cni"
  "cni-conf-dir": "{{k8s_cni_conf_dir}}"
  "cni-bin-dir": "{{k8s_cni_bin_dir}}"
  "cloud-provider": ""
  "register-node": "true"

# kublet kubeconfig
k8s_worker_kubelet_conf_yaml: |
  kind: KubeletConfiguration
  apiVersion: kubelet.config.k8s.io/v1beta1
  address: {{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}
  authentication:
    anonymous:
      enabled: false
    webhook:
      enabled: true
    x509:
      clientCAFile: "{{k8s_conf_dir}}/ca-k8s-apiserver.pem"
  authorization:
    mode: Webhook
  clusterDomain: "cluster.local"
  clusterDNS:
    - "10.32.0.254"
  failSwapOn: true
  healthzBindAddress: "{{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}"
  healthzPort: 10248
  runtimeRequestTimeout: "15m"
  serializeImagePulls: false
  tlsCertFile: "{{k8s_conf_dir}}/cert-k8s-apiserver.pem"
  tlsPrivateKeyFile: "{{k8s_conf_dir}}/cert-k8s-apiserver-key.pem"

# Directroy to store kube-proxy configuration
k8s_worker_kubeproxy_conf_dir: "/var/lib/kube-proxy"

# kube-proxy settings
k8s_worker_kubeproxy_settings:
  "config": "{{k8s_worker_kubeproxy_conf_dir}}/kubeproxy-config.yaml"

k8s_worker_kubeproxy_conf_yaml: |
  kind: KubeProxyConfiguration
  apiVersion: kubeproxy.config.k8s.io/v1alpha1
  bindAddress: {{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}
  clientConnection:
    kubeconfig: "{{k8s_worker_kubeproxy_conf_dir}}/kubeconfig"
  healthzBindAddress: {{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}:10256
  mode: "ipvs"
  ipvs:
    minSyncPeriod: 0s
    scheduler: ""
    syncPeriod: 2s
  iptables:
    masqueradeAll: true
  clusterCIDR: "10.200.0.0/16"

# CNI network plugin settings
k8s_cni_dir: "/opt/cni"
k8s_cni_bin_dir: "{{k8s_cni_dir}}/bin"
k8s_cni_conf_dir: "/etc/cni/net.d"
k8s_cni_plugin_version: "0.7.5"
# SHA512 checksum (see https://github.com/containernetworking/plugins/releases)
k8s_cni_plugin_checksum: "1abfb567c13f87aab94efd6f2c6bb17f3581cbcce87bf8c6216510a92486cb44e6b8701fd3a6cb85273a97981feecbd0c4eb624eab0eca5a005fd85a7d95c284"
```

Dependencies
------------

- https://galaxy.ansible.com/githubixx/kubernetes-controller/
- https://galaxy.ansible.com/githubixx/docker/
- https://galaxy.ansible.com/githubixx/flanneld/

Example Playbook
----------------

```
- hosts: k8s_worker
  roles:
    - githubixx.kubernetes-worker
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
