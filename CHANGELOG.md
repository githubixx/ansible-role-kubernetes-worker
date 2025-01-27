# Changelog

## 28.0.0+1.31.5

- **UPDATE**
  - update `k8s_worker_release` to `1.31.5`

## 27.0.1+1.30.9

- **UPDATE**
  - update `k8s_worker_release` to `1.30.9`

## 27.0.0+1.30.5

- **UPDATE**
  - update `k8s_worker_release` to `1.30.5`

- **OTHER CHANGES**
  - support Ubuntu 24.04
  - update `.yamllint`

## 26.0.2+1.29.9

- **OTHER CHANGES**
  - fix download URLs for Kubernetes binaries (see: [Download Kubernetes - Binaries](https://kubernetes.io/releases/download/#binaries)

## 26.0.1+1.29.9

- **UPDATE**
  - update `k8s_worker_release` to `1.29.9`

## 26.0.0+1.29.4

- **PLEASE READ CAREFULLY**

  Version `24.0.0+1.27.8` had a lot of potential breaking changes. So if you upgrade from a version < `24.0.0+1.27.8` please read the CHANGELOG of that version too!

- **UPDATE**
  - update `k8s_worker_release` to `1.29.4`

- **MOLECULE**
  - use `alvistack` instead of `generic` Vagrant boxes

## 25.0.1+1.28.8

- **UPDATE**
  - update `k8s_worker_release` to `1.28.8`

## 25.0.0+1.28.5

- **PLEASE READ CAREFULLY**

  Version `24.0.0+1.27.8` had a lot of potential breaking changes. So if you upgrade from a version < `24.0.0+1.27.8` please read the CHANGELOG of that version too!

- **UPDATE**
  - update `k8s_worker_release` to `1.28.5`

- **OTHER CHANGES**
  - adjust Github action because of Ansible Galaxy changes
  - `.yamllint`: extend max line length from 200 to 300

- **MOLECULE**
  - change to Ubuntu 22.04 for test-assets VM
  - change IP addresses
  - adjust common names for certificates / change algo to ecdsa and algo size
  - remove `collections.yml"

## 24.0.0+1.27.8

- **PLEASE READ CAREFULLY**
  
  This release contains quite a few potential breaking changes! So review carefully before rolling out the new version of this role! A bigger part of the whole changes are related to increase security. While most of the new variables and defaults should be just fine and should just work out of the box side effects might occur.

  All the newly introduced or changed variables have detailed comments in [README](https://github.com/githubixx/ansible-role-kubernetes-worker/blob/master/README.md). So please read them carefully!

  This refactoring was needed to make it possible to have `githubixx.kubernetes_controller` and `githubixx.kubernetes_worker` deployed on the same host e.g. They were some intersections between the two roles that had to be fixed.

- **UPDATE**
  - update `k8s_worker_release` to `1.27.8`

- **BREAKING**
  - Rename variable `k8s_conf_dir` to `k8s_worker_conf_dir`. Additionally the default value changed from `/usr/lib/kubernetes` to `/etc/kubernetes/worker`.
  - Rename variable `k8s_bin_dir` to `k8s_worker_bin_dir`.
  - `k8s_worker_binaries` variable is no longer defined in `defaults/main.yml` but in `vars/main.yml`. Since this list is fixed anyways it makes no sense to allow to modify this list.
  - `k8s_worker_certificates` variable is no longer defined in `defaults/main.yml` but in `vars/main.yml`. Since this list is fixed anyways it makes no sense to allow to modify this list.
  - Introduce variable `k8s_worker_pki_dir`. All certificate files specified in `k8s_worker_certificates` (see `vars/main.yml`) will be stored here. Related to this: Certificate related settings in `k8s_worker_kubelet_conf_yaml` used `k8s_conf_dir` before and now use `k8s_ctl_pki_dir`. That's `clientCAFile`, `tlsCertFile` and `tlsPrivateKeyFile`.
  - The default value for `k8s_interface` changed from `tap0` to `eth0`.
  - The variable `k8s_config_directory` is gone. It's no longer in use. After the upgrade to this release you can delete this directory (if you accept the new default!) and it's content (make a backup esp. of `admin.kubeconfig` file - just in case!)
  - Remove variable `k8s_worker_download_dir` (no longer needed).
  - Change default value of `k8s_worker_kubelet_conf_dir` to `{{ k8s_worker_conf_dir }}/kubelet`.
  - Change default value of `k8s_worker_kubeproxy_conf_dir` to `{{ k8s_worker_conf_dir }}/kube-proxy`.

- **FEATURE**
  - When downloading the Kubernetes binaries the task checks the SHA512 checksum.
  - Introduce `k8s_worker_api_endpoint_host` and `k8s_worker_api_endpoint_port` variables. Previously `kubelet` and `kube-proxy` where configured to connect to the first host in the Ansible `k8s_controller` group and communicate with the `kube-apiserver` that was running there. This was hard-coded and couldn't be changed. If that host was down the K8s worker nodes didn't receive any updates. Now one can install and use a load balancer like `haproxy` e.g. that distributes requests between all `kube-apiserver`'s and takes a `kube-apiserver` out of rotation if that one is down (also see my Ansible [haproxy role](https://github.com/githubixx/ansible-role-haproxy) for that use case). The default is still to use the first host/kube-apiserver in the Ansible `k8s_controller` group. So behavior-wise nothing changed basically.
  - Add task to generate `kubeconfig` for `kubelet` service (previously this was a separate [playbook](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/v15.0.0_r1.27.5/kubeauthconfig)).
  - Add task to generate `kubeconfig` for `kube-proxy` service (previously this was a separate [playbook](https://github.com/githubixx/ansible-kubernetes-playbooks/tree/v15.0.0_r1.27.5/kubeauthconfig)).

- **OTHER CHANGES**
  - Use `kubernetes.core.*` modules instead of `kubectl` binary
  - Fix some `ansible-lint` issues

- **MOLECULE**
  - Updated all files to reflect the changes introduces with this version
  - Tasks for creating `kubeconfig` for `kubelet` and `kube-proxy` are no longer needed as they're now part of `kubernetes_worker` role
  - Add `haproxy` to Ubuntu 22 hosts to test new `k8s_worker_api_endpoint_host` and `k8s_worker_api_endpoint_port` settings
  - Add tasks to install [ansible-role-cni](https://github.com/githubixx/ansible-role-cni) and [ansible-role-runc](https://github.com/githubixx/ansible-role-runc)
  - Use `kubernetes.core.k8s_info` module instead of calling `kubectl` binary

## 23.1.2+1.27.5

- rename `githubixx.harden-linux` to `githubixx.harden_linux`

## 23.1.1+1.27.5

- rename `githubixx.kubernetes-ca` to `githubixx.kubernetes_ca`

## 23.1.0+1.27.5

- add support for Ubuntu 22.04
- `molecule/default/group_vars/all.yml`: Removed `container-runtime-endpoint` setting from `k8s_worker_kubelet_settings` (/etc/systemd/system/kubelet.service). It was moved to `k8s_worker_kubelet_conf_yaml` (kubelet-config.yaml)

## 23.0.0+1.27.5

- **BREAKING**: `meta/main.yml`: change role_name from `kubernetes-worker` to `kubernetes_worker`. This is a requirement since quite some time for Ansible Galaxy. But the requirement was introduced after this role already existed for quite some time. So please update the name of the role in your playbook accordingly!
- rename `kubernetes-controller` to `kubernetes_controller` as role name changed (requirement as before)
- update `k8s_release` to `1.27.5`
- `meta/main.yml`: remove Ubuntu 18.04 as supported OS (reached EOL)
- moved `container-runtime-endpoint` setting from `k8s_worker_kubelet_settings` variable (`/etc/systemd/system/kubelet.service`) to `k8s_worker_kubelet_conf_yaml` variable (`kubelet-config.yaml`). The name changed from `container-runtime-endpoint` to `containerRuntimeEndpoint`. For more information see pull request [kubelet: migrate container runtime endpoint flag to config](https://github.com/kubernetes/kubernetes/pull/112136).
- remove `Install some network packages` task for Red Hat based OSes (was actually never officially supported)

## 22.0.1+1.26.8

- update `k8s_release` to `1.26.8`
- `kube-proxy` needs to have network-online.target ready
- `kubelet` needs to have network-online.target ready

## 22.0.0+1.26.4

- update `k8s_release` to `1.26.4`
- add Molecule test
- add Github workflow

## 21.1.0+1.25.9

- update `k8s_release` to `1.25.9`
- move kubelet parameter `--register-node` to `kubelet.conf` (using the option as parameter is deprecated)
- `tasks/main.yml`: add `changed_when: false` to `Disable swap` task
- `tasks/main.yml`: use `ansible.posix.mount` instead of `ansible.builtin.mount`
- `kubelet`: remove `--container-runtime` (Flag `--container-runtime` has been deprecated, will be removed in 1.27 as the only valid value is `remote`)
- `kubelet`: update information about `seccomp-default` / remove `SeccompDefault` feature gate (now beta and enabled by default)

## 21.0.0+1.25.5

- update `k8s_release` to `1.25.5`

## 20.0.1+1.24.9

- update `k8s_release` to `1.24.9`

## 20.0.0+1.24.4

update `k8s_release` to `1.24.4`

## 19.1.0+1.23.10

- update `k8s_release` to `1.23.10`

## 19.0.0+1.23.3

- update `k8s_release` to `1.23.3`
- this role now requires Ansible >= 2.9

## 18.0.1+1.22.6

- update `k8s_release` to `1.22.6`

## 18.0.0+1.22.5

- update `k8s_release` to `1.22.5`
- default the `cgroupDriver` value in the `KubeletConfiguration` to `systemd` as `kubelet` runs as a `systemd` service. See [configure-cgroup-driver](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/) for more details. Before that the default was `cgroupfs`. Also see [Migrating to the systemd driver](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#migrating-to-the-systemd-driver)

## 17.0.0+1.21.8

- **BREAKING**: This role no longer installs `CNI plugins`. So the variables `k8s_cni_dir`, `k8s_cni_bin_dir`, `k8s_cni_conf_dir`, `k8s_cni_plugin_version` and `k8s_cni_plugin_checksum` are no longer relevant and are ignored. Please use Ansible role [containerd](https://galaxy.ansible.com/githubixx/containerd) to install `containerd`, `runc` and `CNI plugins` before installing this role. Also see [Kubernetes: Replace dockershim with containerd and runc](https://www.tauceti.blog/posts/kubernetes-replace-docker-with-containerd-runc/)
- **BREAKING**: [containerd](https://galaxy.ansible.com/githubixx/containerd) is a new dependency
- **BREAKING**: This role version no longer uses `Docker/dockershim`. Instead [containerd](https://containerd.io/) is used.
- **BREAKING**: Content of `k8s_worker_kubelet_settings` variable changed: The previous settings `image-pull-progress-deadline`, `network-plugin`, `cni-conf-dir` and `cni-bin-dir` will all be removed with the [dockershim removal](https://kubernetes.io/blog/2020/12/02/dockershim-faq/). `cloud-provider` will be removed in Kubernetes `v1.23`, in favor of removing cloud provider code from Kubelet. `container-runtime` has only two possible values and changed from `docker` to `remote`. And finally one new setting is needed which is `container-runtime-endpoint` which points to `containerd's` socket.
- **BREAKING**: `kubelet.service` has now a dependency on `containerd.service` instead of `docker.service`.
- `kubelet.service` is now enabled and started after Ansible's `flush_handlers` was run.
- update `k8s_release` to `1.21.8`

## 16.0.0+1.21.4

- update `k8s_release` to `1.21.4`
- remove Ubuntu 16.04 support

## 15.1.0+1.20.10

- update `k8s_release` to `1.20.10`

## 15.0.0+1.20.8

- update `k8s_release` to `1.20.8`

## 14.1.0+1.19.12

- update `k8s_release` to `1.19.12`

## 14.0.0+1.19.4

- update `k8s_release` to `1.19.4`

## 13.2.0+1.18.12

- update `k8s_release` to `1.18.12`
- update `k8s_cni_plugin_version` to `0.8.7`
- update `k8s_cni_plugin_checksum` value

## 13.1.0+1.18.6

- update `k8s_release` to `1.18.6`
- CNI plugins download location changed

## 13.0.1+1.18.5

- added Ubuntu 20.04 (Focal Fossa) as supported platform

## 13.0.0+1.18.5

- update `k8s_release` to `1.18.5`
- update `k8s_cni_plugin_version` to `0.8.6`

## 12.0.0+1.17.4

- update `k8s_release` to `1.17.4`

## 11.1.1+1.16.8

- update `k8s_release` to `1.16.8`

## 11.1.0+1.16.3

- `tlsCertFile` and `tlsPrivateKeyFile` options in `kubelet-config.yaml` used wrong certificate files

## 11.0.0+1.16.3

- update `k8s_release` to `1.16.3`

## 10.0.0+1.15.6

- update `k8s_release` to `1.15.6`

## 10.0.0+1.15.3

- update `k8s_release` to `1.15.3`
- removed deprecated `--allow-privileged` kubelet flag (see [Node](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.15.md#node) in K8s changelog)

## 9.0.1+1.14.6

- update `k8s_release` to `1.14.6`

## 9.0.0+1.14.2

- update `k8s_release` to `1.14.2`
- update `k8s_cni_plugin_version` to `0.7.5`
- introduce `k8s_cni_plugin_checksum` variable to determine if CNI plugin tarball has changed and needs to be unarchive

## 8.0.1+1.13.5

- update `k8s_release` to `1.13.5`

## 8.0.0+1.13.2

- update `k8s_release` to `1.13.2`
- use correct semantic versioning as described in [semver](https://semver.org). Needed for Ansible Galaxy importer as it now insists on using semantic versioning.
- make Ansible linter happy

## r7.0.1_v1.12.3

- update `k8s_release` to `1.12.3'

## r7.0.0_v1.11.3

- update `k8s_release` to `1.11.3`

## r6.1.1_v1.10.8

- update `k8s_release` to `1.10.8`

## r6.1.0_v1.10.4

- add task to disable swap in /etc/fstab and execute swapoff.

## r6.0.0_v1.10.4

- switch service routing from `iptables` to `ipvs`. IPVS (IP Virtual Server) is built on top of the Netfilter and implements transport-layer load balancing as part of the Linux kernel. Besides it increases scalability it's way easier to debug Kubernetes networking. Instead of having a look at hundreds or more iptables rules you just run `ipvsadm -Ln` and have a fast overview what Kubernetes service IP get's load balanced to which pod IPs. And if you have the pod IPs you can have a quick look with `ip route` about what routes exist and to figure out how packets for this service are handled. For further information see [IPVS-Based In-Cluster Load Balancing Deep Dive](https://kubernetes.io/blog/2018/07/09/ipvs-based-in-cluster-load-balancing-deep-dive/).

## r5.0.3_v1.10.4

- install `socat` and `netbase` packages

## r5.0.2_v1.10.4

- support Ubuntu 18.04
- update README

## r5.0.1_v1.10.4

- fix iptables bug in `k8s_worker_kubeproxy_conf_yaml`

## r5.0.0_v1.10.4

- update `k8s_release` to `1.10.4`
- introduce `k8s_worker_kubelet_conf_yaml` variable
- removed deprecated settings in `k8s_worker_kubelet_settings`
- moved settings in `k8s_worker_kubelet_settings` to `k8s_worker_kubelet_conf_yaml`:
  see [kubelet-config-file](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/)
  see [types.go](https://github.com/kubernetes/kubernetes/blob/release-1.10/pkg/kubelet/apis/kubeletconfig/v1beta1/types.go)
- introduce `k8s_worker_kubeproxy_conf_yaml` variable
- removed deprecated settings in `k8s_worker_kubeproxy_settings`
- moved settings in `k8s_worker_kubeproxy_settings` to `k8s_worker_kubeproxy_conf_yaml`:
  see: [types.go](https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/apis/kubeproxyconfig/v1alpha1/types.go)
- remove cert-k8s-proxy... entries from `k8s_worker_certificates` because no longer needed

## r4.1.1_v1.9.8

- set `k8s_release` to `1.9.8`

## r4.1.1_v1.9.3

- changed [deprecated Ansible state](https://github.com/githubixx/ansible-role-kubernetes-worker/issues/8)

## r4.1.0_v1.9.3

- remove obsolete kubeconfig.j2 template

## r4.0.0_v1.9.3

- set `k8s_release` to `1.9.3`

## r4.0.0_v1.9.1

- move bind-address,healthz-bind-address out of kube-proxy.service.j2
- remove unneeded macro from kubelet.service.j2 / move address,node-ip,healthz-bind-address out of kubelet.service.j2
- restart kube-proxy/kubelet after service file change

## r3.0.1_v1.9.1

- update to Kubernetes v1.9.1

## r3.0.0_v1.9.0

- Disable fail-swap-on to evict fail when running kubelet on a machine with swap. With this option disabled, only show a warning in log files
- update to Kubernetes v1.9.0
- change defaults for `k8s_ca_conf_directory` and `k8s_config_directory`
- more documentation for defaults
- introduce flexible parameter settings for kubelet via `k8s_worker_kubelet_settings` and `k8s_worker_kubelet_settings_user`
- introduce flexible parameter settings for kube-proxy vi `k8s_worker_kubeproxy_settings` and `k8s_worker_kubeproxy_settings_user`
- add kube-proxy healthz-bind-address setting
- remove `k8s_api_server/k8s_api_server_ip` variables from kube-proxy.service.j2 (no longer needed)

No changelog for releases < r3.0.0_v1.9.0 (see commit history if needed)
