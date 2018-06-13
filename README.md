ansible-role-kubernetes-worker
==============================

This Ansible playbook is used in [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 7 - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-7/). This playbook installs Kubernetes worker including kubelet and kube-proxy. For more information please see [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 7 - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-7/).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `r1.0.0_v1.8.0` means this is release 1.0.0 of this role and it's meant to be used with Kubernetes version 1.8.0. If the role itself changes `rX.Y.Z` will increase. If the Kubernetes version changes `vX.Y.Z` will increase. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Kubernetes release.

Requirements
------------

This playbook expects that you already have rolled out the Kubernetes controller components (see https://galaxy.ansible.com/githubixx/kubernetes-controller/ and https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/). You also need Docker (https://galaxy.ansible.com/githubixx/docker/) and flannel (https://galaxy.ansible.com/githubixx/flanneld/) roles installed.

Changelog
---------

**r5.0.0_v1.10.4**

- update `k8s_release` to `1.10.4`
- introduce `k8s_worker_kubelet_conf_yaml` variable
- removed deprecated setttings in `k8s_worker_kubelet_settings`
- moved settings in `k8s_worker_kubelet_settings` to `k8s_worker_kubelet_conf_yaml`:
  see https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/
  see https://github.com/kubernetes/kubernetes/blob/release-1.10/pkg/kubelet/apis/kubeletconfig/v1beta1/types.go
- introduce `k8s_worker_kubeproxy_conf_yaml` variable
- removed deprecated settings in `k8s_worker_kubeproxy_settings`
- moved settings in `k8s_worker_kubeproxy_settings` to `k8s_worker_kubeproxy_conf_yaml`:
  see: https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/apis/kubeproxyconfig/v1alpha1/types.go
- remove cert-k8s-proxy* from k8s_worker_certificates because no longer needed

**r4.1.1_v1.9.8**

- set `k8s_release` to `1.9.8`

**r4.1.1_v1.9.3**

- changed deprecated Ansible state (https://github.com/githubixx/ansible-role-kubernetes-worker/issues/8)

**r4.1.0_v1.9.3**

- remove obsolet kubeconfig.j2 template

**r4.0.0_v1.9.3**

- set `k8s_release` to `1.9.3`

**r4.0.0_v1.9.1**

- move bind-address,healthz-bind-address out of kube-proxy.service.j2
- remove unneeded macro from kubelet.service.j2 / move address,node-ip,healthz-bind-address out of kubelet.service.j2
- restart kube-proxy/kubelet after service file change

**r3.0.1_v1.9.1**

- update to Kubernetes v1.9.1

**r3.0.0_v1.9.0**

- Disable fail-swap-on to evict fail when running kubelet on a machine with swap. With this option disabled, only show a warning in log files
- update to Kubernetes v1.9.0
- change defaults for k8s_ca_conf_directory and k8s_config_directory
- more documentation for defaults
- introduce flexible parameter settings for kubelet via k8s_worker_kubelet_settings and k8s_worker_kubelet_settings_user
- introduce flexible parameter settings for kube-proxy vi k8s_worker_kubeproxy_settings and k8s_worker_kubeproxy_settings_user
- add kube-proxy healthz-bind-address setting
- remove k8s_api_server/k8s_api_server_ip variables from kube-proxy.service.j2 (no longer needed)

No changelog for releases < r3.0.0_v1.9.0 (see commit history if needed)

Role Variables
--------------

```
# The directory to store the K8s certificates and other configuration
k8s_conf_dir: "/var/lib/kubernetes"
# The directory to store the K8s binaries
k8s_bin_dir: "/usr/local/bin"
# K8s release
k8s_release: "1.10.4"
# The interface on which the K8s services should listen on. As all cluster
# communication should use the PeerVPN interface the interface name is
# normally "tap0" or "peervpn0". But in general you can use any interface
# you want as long as this interface is reachable for the other K8s nodes.
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

# kubelet settings (can be overriden or additional added by defining
# "k8s_worker_kubelet_settings_user")
k8s_worker_kubelet_settings:
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

# kube-proxy settings (can be overriden or additional added by defining
# "k8s_worker_kubeproxy_settings_user")
k8s_worker_kubeproxy_settings:
  "config": "{{k8s_worker_kubeproxy_conf_dir}}/kubeproxy-config.yaml"

k8s_worker_kubeproxy_conf_yaml: |
  kind: KubeProxyConfiguration
  apiVersion: kubeproxy.config.k8s.io/v1alpha1
  bindAddress: {{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}
  clientConnection:
    kubeconfig: "{{k8s_worker_kubeproxy_conf_dir}}/kubeconfig"
  healthzBindAddress: {{hostvars[inventory_hostname]['ansible_' + k8s_interface].ipv4.address}}:10256
  masqueradeAll: true
  mode: "iptables"
  clusterCIDR: "10.200.0.0/16"

# CNI network plugin settings
k8s_cni_dir: "/opt/cni"
k8s_cni_bin_dir: "{{k8s_cni_dir}}/bin"
k8s_cni_conf_dir: "/etc/cni/net.d"
k8s_cni_plugin_version: "0.6.0"
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
