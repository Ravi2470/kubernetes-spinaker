# Install Kubernetes Cluster using kubeadm
Follow this documentation to set up a Kubernetes cluster on __CentOS 7__ Virtual machines.

This documentation guides you in setting up a cluster with one master node and one worker node.

## Assumptions
|Role|FQDN|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Master|kmaster.example.com|172.42.42.100|CentOS 7|2G|1|
|Worker|kworker.example.com|172.42.42.101|CentOS 7|1G|1|

## On Kmaster
Perform all the commands as root user unless otherwise specified
### Pre-requisites
##### Update /etc/hosts
So that we can talk to each of the nodes in the cluster
```
cat >>/etc/hosts<<EOF
172.42.42.100 kmaster.example.com kmaster
172.42.42.101 kworker.example.com kworker
EOF
```
##### Install, enable and start docker service
```
yum install -y docker
systemctl enable docker
systemctl start docker
```
##### Disable SELinux
```
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```
##### Disable Firewall
```
systemctl disable firewalld
systemctl stop firewalld
```
##### Disable swap
```
sed -i '/swap/d' /etc/fstab
swapoff -a
```
##### Update sysctl settings for Kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
### Kubernetes Setup
##### Add yum repository
```
cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
##### Install Kubernetes
```
yum install -y kubeadm kubelet kubectl
```
##### Enable and Start kubelet service
```
systemctl enable kubelet
systemctl start kubelet
```
##### Initialize Kubernetes Cluster
```
kubeadm init --apiserver-advertise-address=172.42.42.100 --pod-network-cidr=10.244.0.0/16
```
##### Copy kube config
To be able to use kubectl command to connect and interact with the cluster, the user needs kube config file.

In my case, the user account is venkatn
```
mkdir /home/venkatn/.kube
cp /etc/kubernetes/admin.conf /home/venkatn/.kube/config
chown -R venkatn:venkatn /home/venkatn/.kube
```
##### Deploy flannel network
This has to be done as the user in the above step (in my case it is __venkatn__)
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
##### Cluster join command
```
kubeadm token create --print-join-command
```
## On Kworker
##### Join the cluster
Use the output from __kubeadm token create__ command in previous step from the master server and run here.

## Verifying the cluster
##### Get Nodes status
```
kubectl get nodes
```
##### Get component status
```
kubectl get cs
```

Have Fun!!