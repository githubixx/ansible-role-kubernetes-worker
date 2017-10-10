role-kubernetes-worker
======================

This Ansible playbook is used in [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 7 - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-7/). This playbook installs Kubernetes worker including kubelet and kube-proxy. For more information please see [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 7 - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-7/).

Requirements
------------

This playbook expects that you already have rolled out the Kubernetes controller components (see https://galaxy.ansible.com/githubixx/kubernetes-controller/ and https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/). You also need Docker (https://galaxy.ansible.com/githubixx/docker/) and flannel (https://galaxy.ansible.com/githubixx/flanneld/) roles installed.

Role Variables
--------------

```
k8s_conf_dir: "/var/lib/kubernetes"
k8s_bin_dir: "/usr/local/bin"
k8s_release: "1.8.0"
k8s_interface: "tap0"

k8s_ca_conf_directory: "/etc/k8s/certs"
k8s_config_directory: "/etc/k8s/configs"

k8s_worker_binaries:
  - kube-proxy
  - kubelet
  - kubectl

k8s_worker_certificates:
  - ca-k8s-apiserver.pem
  - ca-k8s-apiserver-key.pem
  - cert-k8s-apiserver.pem
  - cert-k8s-apiserver-key.pem
  - cert-kube-proxy.pem
  - cert-kube-proxy-key.pem

k8s_worker_download_dir: "/opt/tmp"

k8s_worker_kubelet_conf_dir: "/var/lib/kubelet"
k8s_worker_kubelet_serialize_image_pulls: "false"
k8s_worker_kubelet_allow_privileged: "true"
k8s_worker_kubelet_container_runtime: "docker"
k8s_worker_kubelet_docker: "unix:///var/run/docker.sock"
k8s_worker_kubelet_image_pull_progress_deadline: "2m"
k8s_worker_kubelet_register_node: "true"
k8s_worker_kubelet_runtime_request_timeout: "10m"
k8s_worker_kubelet_cadvisor_port: "4194" # port or "0" to disable
k8s_worker_kubelet_cloud_provider: ""
k8s_worker_kubelet_healthz_port: "10248"

k8s_worker_kubeproxy_conf_dir: "/var/lib/kube-proxy"
k8s_worker_kubeproxy_proxy_mode: "iptables"

k8s_controller_manager_cluster_cidr: "10.200.0.0/16"

k8s_cluster_dns: "10.32.0.254"
k8s_cluster_domain: "cluster.local"

k8s_network_plugin: "cni"

k8s_cni_dir: "/opt/cni"
k8s_cni_bin_dir: "{{k8s_cni_dir}}/bin"
k8s_cni_conf_dir: "/etc/cni/net.d"
k8s_cni_plugin_version: "0.6.0"
```

Dependencies
------------

https://galaxy.ansible.com/githubixx/kubernetes-controller/
https://galaxy.ansible.com/githubixx/docker/
https://galaxy.ansible.com/githubixx/flanneld/

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
