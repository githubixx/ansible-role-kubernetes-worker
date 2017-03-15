ansible-role-kubernetes-worker
==============================

This Ansible playbook is used in [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 7 - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-7/). This playbook installs Kubernetes worker including kubelet and kube-proxy.

Requirements
------------

This playbook expects that you already have rolled out the Kubernetes controller components (see https://galaxy.ansible.com/githubixx/kubernetes-controller/ and https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/). You also need Docker installed (you can use https://galaxy.ansible.com/githubixx/docker/).

Role Variables
--------------

```
local_cert_dir: /etc/cfssl

k8s_download_dir: /opt/docker

k8s_interface: tap0
k8s_conf_dir: /var/lib/kubernetes
k8s_bin_dir: /usr/local/bin
k8s_release: 1.5.1
k8s_cni_dir: /opt/cni
k8s_cni_plugins: cni-07a8a28637e97b22eb8dfe710eeae1344f69d16e
k8s_worker_binaries:
  - kube-proxy
  - kubelet
k8s_certificates:
  - ca.pem
  - kubernetes-key.pem
  - kubernetes.pem

k8s_kubelet_conf_dir: /var/lib/kubelet
k8s_kubelet_token: chAng3m3
```

Dependencies
------------

https://galaxy.ansible.com/githubixx/kubernetes-controller/

Example Playbook
----------------

```
- hosts: k8s-worker
  roles:
    - githubixx.kubernetes-worker
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
