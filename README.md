# ansible-k3d

Ansible role for Rancher's [`k3d`](https://github.com/rancher/k3d) (assumes MacOS and Docker). Deploys [`tuntap`](https://formulae.brew.sh/cask/tuntap) since HyperKit doesn't uses bridged networking (this makes [`metallb`](https://metallb.universe.tf) work on the same L2). Basically, following [this](https://blog.kubernauts.io/k3s-with-k3d-and-metallb-on-mac-923a3255c36e) nice writeup. Also, installs the ever-so-lovely [`k9s`](https://github.com/derailed/k9s) interface. Keep the dev-ing local!

```bash
❯ uname -a
Darwin honeycrisp.localdomain 19.4.0 Darwin Kernel Version 19.4.0: Wed Mar  4 22:28:40 PST 2020; root:xnu-6153.101.6~15/RELEASE_X86_64 x86_64

❯ docker -v
Docker version 19.03.8, build afacb8b

❯ ifconfig tap1 2>&1 | head -1
tap1: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500

❯ kubectl --kubeconfig ~/.kube/config.k3s-default get nodes
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

❯ kubectl --kubeconfig ~/.kube/config.k3s-default get svc -n kube-system traefik -o json | jq -r '.status.loadBalancer.ingress[0].ip'
172.22.0.3

❯ curl 172.22.0.3
404 page not found # this is good, means we're getting through
```

Running the role locally:

```bash
$ virtualenv -p python3 venv

$ source venv/bin/activate

$ pip install -r requirements.txt

$ ANSIBLE_ROLES_PATH=ansible-k3d ansible-playbook ansible-k3d/tests/test.yml
```

Verify again if you like:

```
❯ kubectl --kubeconfig ~/.kube/config.k3s-default get svc whoareyou-service
NAME                TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
whoareyou-service   LoadBalancer   10.43.10.189   172.23.0.101   80:31094/TCP   11m

❯ curl 172.23.0.101
Hostname: whoareyou-deployment-74bff9c468-wn4n5
IP: 127.0.0.1
IP: ::1
IP: 10.42.2.4
IP: fe80::e07b:3bff:fe36:2e74
RemoteAddr: 10.42.2.1:32404
GET / HTTP/1.1
Host: 172.23.0.101
User-Agent: curl/7.64.1
Accept: */*
```

You'll have to manually remove this route when you're done (`sudo route delete -net 172.23/24`):

```
❯ netstat -rn | grep 172
172.23/24          10.0.75.2          UGSc          tap1
```
