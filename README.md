ansible-role-kubernetes-worker
==============================

This Ansible playbook is used in [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 7 - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-7/). This playbook installs Kubernetes worker including kubelet and kube-proxy.

Requirements
------------

This playbook expects that you already have rolled out the Kubernetes controller components (see https://galaxy.ansible.com/githubixx/kubernetes-controller/ and https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/). You also need Docker installed (you can use https://galaxy.ansible.com/githubixx/docker/).

Role Variables
--------------

```
k8s_ca_conf_directory: /etc/k8s/certs

k8s_worker_download_dir: /opt/tmp

k8s_interface: tap0
k8s_conf_dir: /var/lib/kubernetes
k8s_bin_dir: /usr/local/bin
k8s_release: 1.8.0
k8s_cni_dir: /opt/cni
k8s_cni_plugin_version: 0.6.0
k8s_worker_binaries:
  - kube-proxy
  - kubelet
  - kubectl
k8s_certificates:
  - ca.pem
  - kubernetes-key.pem
  - kubernetes.pem

k8s_worker_kubelet_conf_dir: /var/lib/kubelet
```

Dependencies
------------

https://galaxy.ansible.com/githubixx/kubernetes-controller/

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
