 [k3s](https://k3s.io/) - lightweight Kubernetes certified distro
-----------------------------------------------

### k3s installation

After updating hosts ips in **hosts** file and k3s version in **group_vars/all.yml**, run playbook

```
ansible-playbook k3s.yml -i hosts
```

k3s uses server (master) and agent (worker) services.
Default server installation configures agent, that allows pods to run on master node as well;
master node does not have defined taints.

```
kubectl get nodes
NAME          STATUS   ROLES    AGE     VERSION
km1.hack.me   Ready    master   3m25s   v1.14.3-k3s.1
ks1.hack.me   Ready    worker   3m15s   v1.14.3-k3s.1
```

```
kubectl describe node <master_node>

Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=<master_hostname>
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=true
Annotations:        flannel.alpha.coreos.com/backend-data: {"VtepMAC":"be:81:ef:4b:15:b4"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 192.168.xxx.xxx
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
Taints:             <none>
```

Optionally, flag '--disable-agent' skips installing the agent on the master node.

To separate server and agent services on master node, update the **roles/server/tasks/main.yml** file adding '--disable-agent' flag then in **hosts** file add the server ip in agents list.

In this use case the master role in roles column is marked as worker, instead of master.

```
kubectl get nodes
NAME                STATUS   ROLES    AGE   VERSION
<master_hostname>   Ready    worker   30s   v1.14.3-k3s.1
<worker_hostname>   Ready    worker   17s   v1.14.3-k3s.1
```

```
kubectl describe node <master_node>
Roles:              worker
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=<master_hostname>
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/worker=true
Annotations:        flannel.alpha.coreos.com/backend-data: {"VtepMAC":"26:ef:01:ad:c6:a2"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 192.168.xxx.xxx
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
Taints:             <none>
```

traefik is installed by default. '--no-deploy=servicelb' & '--no-deploy=traefik' flags disable installing the traefik & lb during server installation.

flannel is the default CNI. '--no-flannel' flag during agent installation skips installing the flannel.

### k3s backup/restore sqlite key value store

db files are located at /var/lib/rancher/k3s/server/db/
```
state.db
state.db-shm
state.db-wal
```

backup current state.db state
```
sqlite3 state.db .dump > state.db.<back_up>
```

move/delete active state.db
```
rm -f state.db
```

restore sqlite
```
sqlite3 state.db < state.db.<back_up>
```
could require k3s server restart
```
systemctl restart k3s
```

k3s comes with default state.db content, which will be used in case the state.db does not exists and the master service re/starts.

[k3s documentation](https://github.com/rancher/k3s "k3s")

