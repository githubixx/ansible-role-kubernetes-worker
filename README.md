ansible-role-kubernetes-worker
==============================

For more information have a look at: [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 7 - Worker](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-7/). This playbook installs Kubernetes worker including Docker, kubelet and kube-proxy.

Requirements
------------

This playbook expects that you already have rolled out the Kubernetes controller components (see https://galaxy.ansible.com/githubixx/kubernetes-controller/ and https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-6/). Since we use Ubuntu 16.04 at Scaleway we should have already a very recent kernel running (at time of writing it was kernel 4.8.x on my VPS instance). It makes sense to use a recent kernel for Docker in general. Ubuntu 16.04 additionally provides kernel 4.4.x and 4.8.x. I recommend to use 4.8.x if possible. Verify that you have `overlayfs` filesystem available (execute `cat /proc/filesystems | grep overlay`. If you see an output you should be fine) on your worker instances. In my case the `overlayfs` is compiled into the kernel. If it's not compiled in you can normally load it via `modprobe -v overlay` (`-v` gives us a little bit more information). We'll configure Docker to use overlayfs by default because it's one of the best choises (Docker 1.13.x started to use overlayfs by default if available). But you can change the storage driver via the `docker_storage_driver` variable if you like.

Role Variables
--------------

```
local_cert_dir: /etc/cfssl

download_dir: /opt/docker

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

docker_version: 1.13.1
docker_user: docker
docker_group: docker
docker_uid: 666
docker_gid: 666
docker_bin_dir: /usr/local/bin
docker_storage_driver: overlay
docker_log_level: error
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
