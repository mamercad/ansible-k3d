# ansible-k3d

Ansible role for Rancher's `k3d` (assumes MacOS and Docker). Deploys `tuntap` since HyperKit doesn't uses bridged networking (this makes `metallb` work on the same L2). Basically, following [this](https://blog.kubernauts.io/k3s-with-k3d-and-metallb-on-mac-923a3255c36e) nice writeup.

```bash
❯ uname -a
Darwin honeycrisp.localdomain 19.4.0 Darwin Kernel Version 19.4.0: Wed Mar  4 22:28:40 PST 2020; root:xnu-6153.101.6~15/RELEASE_X86_64 x86_64

❯ docker -v
Docker version 19.03.8, build afacb8b

❯ ifconfig tap1 2>&1 | head -1
tap1: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500

❯ kubectl --kubeconfig ~/.kube/config.k3d-default get nodes
NAME                       STATUS   ROLES    AGE   VERSION
k3d-k3s-default-worker-0   Ready    <none>   13h   v1.18.4+k3s1
k3d-k3s-default-worker-1   Ready    <none>   13h   v1.18.4+k3s1
k3d-k3s-default-worker-2   Ready    <none>   13h   v1.18.4+k3s1
k3d-k3s-default-server     Ready    master   13h   v1.18.4+k3s

❯ route get -net 172.22.0.0/24 2>&1 | head -4
   route to: 172.22.0.0
destination: 172.22.0.0
       mask: 255.255.255.0
    gateway: 10.0.75.2

❯ kubectl --kubeconfig ~/.kube/config.k3d-default get svc -n kube-system traefik -o json | jq -r '.status.loadBalancer.ingress[0].ip'
172.22.0.3

❯ curl 172.22.0.3
404 page not found # this is good, means we're getting through
```

Running the role locally:

```bash
$ ANSIBLE_ROLES_PATH=ansible-k3d ansible-playbook ansible-k3d/tests/test.yml
```
