# k3s
------

After updating hosts ips in **hosts** file, run playbook

```
ansible-playbook k3s.yml -i hosts
```

k3s uses server (master) and agent (worker) services.
Default server installation configures agent, that allows pods to run on master node as well;
master node does not have defined taints.

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

Provided sample installs server and agents on two nodes.
Master node runs server and agent services separately and worker node runs only agent service.

```
kubectl get nodes
NAME                STATUS   ROLES    AGE   VERSION
<master_hostname>   Ready    worker   30s   v1.14.3-k3s.1
<worker_hostname>   Ready    worker   17s   v1.14.3-k3s.1
```

In this case the master role in roles column is marked as worker, instead of master.

```
kubectl describe node km1.hack.me
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


[k3s documentation](https://github.com/rancher/k3s "k3s")

