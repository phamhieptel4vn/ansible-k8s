[![GitHub issues](https://img.shields.io/github/issues/garutilorenzo/ansible-role-linux-kubernetes)](https://github.com/garutilorenzo/ansible-role-linux-kubernetes/issues)
![GitHub](https://img.shields.io/github/license/garutilorenzo/ansible-role-linux-kubernetes)
[![GitHub forks](https://img.shields.io/github/forks/garutilorenzo/ansible-role-linux-kubernetes)](https://github.com/garutilorenzo/ansible-role-linux-kubernetes/network)
[![GitHub stars](https://img.shields.io/github/stars/garutilorenzo/ansible-role-linux-kubernetes)](https://github.com/garutilorenzo/ansible-role-linux-kubernetes/stargazers)

# Install and configure a high available Kubernetes cluster

This ansible role will install and configure a high available Kubernetes cluster. This repo automate the installation process of Kubernetes using [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/).

This repo is only a example on how to use Ansible automation to install and configure a Kubernetes cluster. For a production environment use [Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)

## Requirements

Install ansible, ipaddr and netaddr:

```
pip install -r requirements.txt
```

Download the role form GitHub:

```
ansible-galaxy install git+https://github.com/garutilorenzo/ansible-role-linux-kubernetes.git
```

## Role Variables

This role accept this variables:

| Var   | Required |  Default | Desc |
| ------- | ------- | ----------- |  ----------- |
| `kubernetes_subnet`       | `yes`       |  `172.16.146.0/24` | Subnet where Kubernetess will be deployed. If the VM or bare metal server has more than one interface, Ansible will filter the interface used by Kubernetes based on the interface subnet |
| `disable_firewall`       | `no`       | `no`       | If set to yes Ansible will disable the firewall.   |
| `kubernetes_version`       | `no`       | `1.28.0`       | Kubernetes version to install  |
| `kubernetes_cri`       | `no`       | `containerd`       | Kubernetes [CRI](https://kubernetes.io/docs/concepts/architecture/cri/) to install.   |
| `kubernetes_cni`       | `no`       | `flannel`       | Kubernetes [CNI](https://github.com/containernetworking/cni) to install.  |
| `kubernetes_dns_domain`       | `no`       | `cluster.local`       | Kubernetes default DNS domain  |
| `kubernetes_pod_subnet`       | `no`       | `10.10.0.0/16`       | Kubernetes pod subnet  |
| `kubernetes_service_subnet`       | `no`       | `10.96.0.0/16`       | Kubernetes service subnet  |
| `kubernetes_api_port`       | `no`       | `6443`       | kubeapi listen port  |
| `setup_vip`       | `no`       | `no`       | Setup kubernetes VIP addres using [kube-vip](https://kube-vip.io/)   |
| `kubernetes_vip_ip`       | `no`       | `172.16.146.200`       | **Required** if setup_vip is set to *yes*. Vip ip address for the control plane  |
| `kubevip_version`       | `no`       | `v0.4.3`       | kube-vip container version  |
| `install_longhorn`       | `no`       | `no`       | Install [Longhorn](#longhorn), Cloud native distributed block storage for Kubernetes.  |
| `longhorn_version`       | `no`       | `v1.3.1`       | Longhorn release.  |
| `install_nginx_ingress`       | `no`       | `no`       | Install [nginx ingress controller](#nginx-ingress-controller)  |
| `nginx_ingress_controller_version`       | `no`       | `controller-v1.3.0`       | nginx ingress controller version  |
| `nginx_ingress_controller_http_nodeport`       | `no`       | `30080`       | NodePort used by nginx ingress controller for the incoming http traffic  |
| `nginx_ingress_controller_https_nodeport`       | `no`       | `30443`       |  NodePort used by nginx ingress controller for the incoming https traffic  |
| `enable_nginx_ingress_proxy_protocol`       | `no`       | `no`       | Enable  nginx ingress controller proxy protocol mode |
| `enable_nginx_real_ip`       | `no`       | `no`       | Enable nginx ingress controller real-ip module |
| `nginx_ingress_real_ip_cidr`       | `no`       | `0.0.0.0/0`       | **Required** if enable_nginx_real_ip is set to *yes* Trusted subnet to use with the real-ip module  |
| `nginx_ingress_proxy_body_size`       | `no`       | `20m`       | nginx ingress controller max proxy body size  |
| `sans_base`       | `no`       | `[list of values, see defaults/main.yml]`       | list of ip addresses or FQDN uset to sign the kube-api certificate  |
| `install_calio`       | `no`       | `no`       | Install [Calico](https://www.projectcalico.org/) CNI  |
| `calico_version`       | `no`       | `v3.26.3`       | Calico version  |

## Extra Variables

This role accept an extra variable *kubernetes_init_host*. This variable is used when the cluster is bootstrapped for the first time. The value of this variable must be the hostname of one of the master nodes. When ansible will run on the matched host kubernetes will be initialized.

## Cluster resource deployed

Whit this role [Nginx ingress controller](#nginx-ingress-controller) and [Longhorn](#longhorn) will be installed.

### Nginx ingress controller

[Nginx ingress controller](https://kubernetes.github.io/ingress-nginx/) is used as ingress controller.

The installation is the [bare metal](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters) installation, the ingress controller then is exposed via a NodePort Service.
You can customize the ports exposed by the NodePort service, use [Role Variables](#role-variables) to change this values.

### Longhorn

[Longhorn](https://longhorn.io) is a lightweight, reliable, and powerful distributed block storage system for Kubernetes.

Longhorn implements distributed block storage using containers and microservices. Longhorn creates a dedicated storage controller for each block device volume and synchronously replicates the volume across multiple replicas stored on multiple nodes. The storage controller and replicas are themselves orchestrated using Kubernetes.

## Vagrant

To test this role you can use [Vagrant](https://www.vagrantup.com/) and [Virtualbox](https://www.virtualbox.org/) to bring up a example infrastructure. Once you have downloaded this repo use Vagrant to start the virtual machines:

```
vagrant up
```

In the Vagrantfile you can inject your public ssh key directly in the authorized_keys of the vagrant user. You have to change the *CHANGE_ME* placeholder in the Vagrantfile. You can also adjust the number of the vm deployed by changing the NNODES variable (Default: 6)

## Using this role

To use this role you follow the example in the [examples/](examples/) dir. Adjust the hosts.ini file with your hosts and run the playbook:

```
root@jenkins1:/opt/ansible-k8s# ansible-playbook -i hosts-ubuntu.ini site.yaml -e kubernetes_init_host=masternode1

PLAY [kubemaster] ***************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************
ok: [masternode1]
ok: [node01]
ok: [node01]

PLAY RECAP ********************************************************************************************************************************************************************************************************************************
masternode1                : ok=82   changed=36   unreachable=0    failed=0    skipped=24   rescued=0    ignored=3   
node01                     : ok=56   changed=28   unreachable=0    failed=0    skipped=39   rescued=0    ignored=1   
node02                     : ok=50   changed=26   unreachable=0    failed=0    skipped=30   rescued=0    ignored=1   
```

Now we have a Kubernetes cluster deployed in high available mode, we can check the status of the cluster:

```
root@masternode1:~# kubectl get nodes
NAME          STATUS   ROLES           AGE   VERSION
masternode1   Ready    control-plane   19m   v1.27.3
node01        Ready    <none>          14m   v1.27.3
node02        Ready    <none>          14m   v1.27.3
```

Check the pods status:

```
root@masternode1:~# kubectl get pods --all-namespaces
NAMESPACE         NAME                                                  READY   STATUS              RESTARTS        AGE
ingress-nginx     ingress-nginx-admission-create-2fcc5                  0/1     Completed           0               19m
ingress-nginx     ingress-nginx-admission-patch-ckhq5                   0/1     Completed           1               19m
ingress-nginx     ingress-nginx-controller-5c778bffff-87ctp             1/1     Running             0               19m
kube-flannel      kube-flannel-ds-b5q9x                                 1/1     Running             0               14m
kube-flannel      kube-flannel-ds-kb8z7                                 1/1     Running             0               14m
kube-flannel      kube-flannel-ds-wqqcs                                 1/1     Running             0               19m
kube-system       coredns-5d78c9869d-284vn                              1/1     Running             0               19m
kube-system       coredns-5d78c9869d-nfcvx                              1/1     Running             0               19m
kube-system       etcd-masternode1                                      1/1     Running             0               19m
kube-system       kube-apiserver-masternode1                            1/1     Running             0               19m
kube-system       kube-controller-manager-masternode1                   1/1     Running             0               19m
kube-system       kube-proxy-lm5lg                                      1/1     Running             0               19m
kube-system       kube-proxy-mssj6                                      1/1     Running             0               14m
kube-system       kube-proxy-prnj6                                      1/1     Running             0               14m
kube-system       kube-scheduler-masternode1                            1/1     Running             0               19m
kube-system       kube-vip-masternode1                                  1/1     Running             0               19m
longhorn-system   csi-attacher-689fdb475f-2s6l9                         1/1     Running             0               9m48s
longhorn-system   csi-attacher-689fdb475f-7lzh6                         1/1     Running             0               9m48s
longhorn-system   csi-attacher-689fdb475f-dmjnm                         1/1     Running             0               9m48s
longhorn-system   csi-provisioner-65dd48db7b-hlq62                      1/1     Running             0               9m48s
longhorn-system   csi-provisioner-65dd48db7b-tw8hs                      1/1     Running             0               9m48s
longhorn-system   csi-provisioner-65dd48db7b-xmsjw                      1/1     Running             0               9m48s
longhorn-system   csi-resizer-847d645569-2hkrq                          1/1     Running             0               9m48s
longhorn-system   csi-resizer-847d645569-cbqng                          1/1     Running             0               9m48s
longhorn-system   csi-resizer-847d645569-vsbdp                          1/1     Running             0               9m48s
longhorn-system   csi-snapshotter-59ff78b47b-8vf6d                      1/1     Running             0               9m47s
longhorn-system   csi-snapshotter-59ff78b47b-h4w7z                      1/1     Running             0               9m47s
longhorn-system   csi-snapshotter-59ff78b47b-jpl5w                      0/1     ContainerCreating   0               9m47s
longhorn-system   engine-image-ei-1d169b76-s8dj5                        1/1     Running             0               9m52s
longhorn-system   engine-image-ei-1d169b76-svk9s                        1/1     Running             0               9m52s
longhorn-system   instance-manager-e-0a729c5ba5b5f7e4cf93fc036ac30566   1/1     Running             0               9m50s
longhorn-system   instance-manager-e-fbd274715cd125407eb86cefed555a97   1/1     Running             0               9m51s
longhorn-system   instance-manager-r-0a729c5ba5b5f7e4cf93fc036ac30566   1/1     Running             0               9m50s
longhorn-system   instance-manager-r-fbd274715cd125407eb86cefed555a97   1/1     Running             0               9m51s
longhorn-system   longhorn-admission-webhook-7bfdcc7b9b-dlwrv           1/1     Running             0               19m
longhorn-system   longhorn-admission-webhook-7bfdcc7b9b-s5sq4           1/1     Running             0               19m
longhorn-system   longhorn-conversion-webhook-64d6fd986b-lsh4z          1/1     Running             0               19m
longhorn-system   longhorn-conversion-webhook-64d6fd986b-rpsvb          1/1     Running             0               19m
longhorn-system   longhorn-csi-plugin-9ql4t                             3/3     Running             0               9m47s
longhorn-system   longhorn-csi-plugin-d5xdx                             0/3     ContainerCreating   0               9m47s
longhorn-system   longhorn-driver-deployer-d6dcbbdcd-dcgsv              1/1     Running             0               19m
longhorn-system   longhorn-manager-2gqmj                                1/1     Running             0               11m
longhorn-system   longhorn-manager-vs4rf                                1/1     Running             1 (9m52s ago)   13m
longhorn-system   longhorn-recovery-backend-b84fd87bf-4cdvk             1/1     Running             0               19m
longhorn-system   longhorn-recovery-backend-b84fd87bf-mrjp4             1/1     Running             0               19m
longhorn-system   longhorn-ui-59cdc947cf-pj6bq                          1/1     Running             0               19m
longhorn-system   longhorn-ui-59cdc947cf-zpm9r                          1/1     Running             0               19m
```

we can see, longhorn, nginx ingress and all the kube-system pods.

We can also inspect the service of the nginx ingress controller:

```
root@jenkins1:/opt/ansible-k8s#     
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.104.251.66   <none>        80:30080/TCP,443:30443/TCP   17m
ingress-nginx-controller-admission   ClusterIP   10.98.230.21    <none>        443/TCP                      17m
```

we can see the nginx ingress controller listening port, in this case the http port is 30080 and  the https port is 30443. From an external machine we can test the ingress controller:

```
root@masternode1:~# curl -v http://172.16.146.164:30080
*   Trying 172.16.146.164:30080...
* TCP_NODELAY set
* Connected to 172.16.146.164 (172.16.146.164) port 30080 (#0)
> GET / HTTP/1.1
> Host: 172.16.146.164:30080
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< Date: Thu, 26 Oct 2023 04:20:38 GMT
< Content-Type: text/html
< Content-Length: 146
< Connection: keep-alive
< 
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
* Connection #0 to host 172.16.146.164 left intact
```