
ssh Baiju@172.27.63.170 -p 5522


ssh Baiju@172.27.63.171 -p 5522


## Set the hostname

### Set the hostname for the machine

```sh
sudo hostnamectl set-hostname new-node-one (on first node)
sudo hostnamectl set-hostname new-node-two (on second node and so on ..)
```

### also add the new hostname to /etc/hosts file.

```sh
vi /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 new-node-one
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6 new-node-one

```

### Turn off swap

sudo swapoff -a

sudo vi /etc/fstab

```
#/swap.img      none    swap    sw      0       0
```


### Restart the node
```sh
reboot
```

### after rebooting, add the route (specific to railtel servers)

```
ip route add default via 10.31.31.1 dev ens224
```


## Mounting secondary disk steps

Identifying the Disk
First, ensure that the new disk is recognized by your system. You can use the lsblk command to list all block devices:

```
lsblk
```

Look for sdb in the output.

2. Creating a Filesystem
Before mounting, you need to format the disk. For example, to format it with the ext4 filesystem, use:

```
sudo mkfs.ext4 /dev/sdb
```

Warning: This will erase all data on /dev/sdb.

3. Creating the Mount Point
Create the directory where the disk will be mounted:

```
sudo mkdir -p /mnt/additional
```

4. Mounting the Disk Temporarily
To mount the disk temporarily, use:

```
sudo mount /dev/sdb /mnt/additional
```

5. Editing fstab for Permanent Mounting
To mount the disk automatically at boot, you need to add an entry to the /etc/fstab file. First, find the UUID of your new disk:

```
sudo blkid
```
Locate /dev/sdb in the output and note the UUID.

Then, edit the fstab file:

```
sudo nano /etc/fstab
```

```
UUID=YOUR_UUID /mnt/additional ext4 defaults 0 2
```

This line specifies the disk by UUID, mount point, filesystem type, and mounting options.

6. Mounting the Disk Automatically
To apply the changes in fstab without rebooting, use:
```
sudo mount -a
```

## Installing Kubernetes

### Set SELinux in permissive mode (effectively disabling it)

```sh
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

### Add the packages

```sh
# This overwrites any existing configuration in /etc/yum.repos.d/kubernetes.repo
cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
EOF
```


```sh
sudo yum update
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```

## Setting variables to load kernet modules

```sh
cat > /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
uio
uio_pci_generic
nvme-tcp
iscsi_tcp
EOF
```

```sh
modprobe overlay # for kubernetes
modprobe br_netfilter # for kubernetes
modprobe uio # for longhorn
modprobe uio_pci_generic # for longhorn
modprobe nvme-tcp # for longhorn
modprobe iscsi_tcp # for longhorn
```

## setting sysctl variables


```sh
cat > /etc/sysctl.d/99-zk8s.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

inside the file `/etc/sysctl.d/99-sysctl.conf`, set the `net.ipv4.ip_forward = 1`


### load the sysctl variables

`sudo sysctl --system`


## Installing containerd

```sh
dnf install -y  yum-utils device-mapper-persistent-data lvm2
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf update -y && dnf install -y containerd.io

```

```sh
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```

```
systemctl restart containerd
systemctl enable containerd
```


## installing jq for longhorn

```sh
yum install jq
```

## allowing masquerading on the machine

```sh
firewall-cmd --list-all
  ... 
  masquerade: yes

Enable if it's "no":

firewall-cmd --add-masquerade --permanent
firewall-cmd --reload

```


only after this node will be ready.
and after coredns will be ready


### open http port

```
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --zone=public --add-port=6443/tcp --permanent


sudo firewall-cmd --zone=public --add-port=2379-2380/tcp --permanent
sudo firewall-cmd --zone=public --add-port=10251/tcp --permanent
sudo firewall-cmd --zone=public --add-port=10252/tcp --permanent


```

### Open VXLAN port for Flannel

```
sudo firewall-cmd --zone=public --add-port=8472/udp --permanent
sudo firewall-cmd --zone=public --add-port=8285/udp --permanent

```

### Open Kubelet API port

```
sudo firewall-cmd --zone=public --add-port=10250/tcp --permanent
```

### Open NodePort range ports

```
sudo firewall-cmd --zone=public --add-port=30000-32767/tcp --permanent
sudo firewall-cmd --zone=public --add-port=30000-32767/udp --permanent
```


### Reload the firewall to apply changes

```
sudo firewall-cmd --reload
```



## Starting kubeadm

RUN ONLY ON MASTER

### master specific 
```sh
mkdir -p $HOME/.kube
sudo kubeadm config images pull
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket /run/containerd/containerd.sock
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Installing Flannel ( only for master)

```sh
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```


### worker specific (needs to be setup after master is running properly, which happens after flannel is installed)

#### Run this command on the master

```sh
kubeadm token create --print-join-command
```

Run the join command on the worker


### After setting up each node in the cluster

```
sudo ethtool -K flannel.1 tx-checksum-ip-generic off
```
https://github.com/k3s-io/k3s/issues/5013



## After cluster is up we need to install longhorn

### Open SCSI


https://longhorn.io/docs/1.5.3/deploy/install/#installing-open-iscsi


#### first way
```sh
yum --setopt=tsflags=noscripts install iscsi-initiator-utils
echo "InitiatorName=$(/sbin/iscsi-iname)" > /etc/iscsi/initiatorname.iscsi
systemctl enable iscsid
systemctl start iscsid
```
modprobe iscsi_tcp



#### alternative way
```
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.5.3/deploy/prerequisite/longhorn-iscsi-installation.yaml

```

### NFS V4 client

#### first way

yum install nfs-utils


#### alternative way

```
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.5.3/deploy/prerequisite/longhorn-nfs-installation.yaml
```


### For Kernel modules and huge pages


#### first way

```
echo 512 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages

echo "vm.nr_hugepages=512" >> /etc/sysctl.conf

modprobe iscsi_tcp
```

#### alternate way

https://longhorn.io/docs/1.5.3/spdk/quick-start/#configure-kernel-modules-and-huge-pages


```
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.5.3/deploy/prerequisite/longhorn-spdk-setup.yaml

```


### Load NVME TCP Kernel module

https://longhorn.io/docs/1.5.3/spdk/quick-start/#load-nvme-tcp-kernel-module

```
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.5.3/deploy/prerequisite/longhorn-nvme-cli-installation.yaml

```


### Script to check env for potential issues, run on the nodes

```
curl -sSfL https://raw.githubusercontent.com/longhorn/longhorn/v1.5.3/scripts/environment_check.sh | bash
```

## Temporary ##
Add route for 10.0.0.0/16 towards the vpn instance 
remove 8.8.8.8 from resolv.conf

## few tools for easy switching of namespace




https://github.com/blendle/kns
fzf


## to allow pods to be deployed on control plane (master node) also

Remove the control plane taint from the control plane node

kubectl edit node new-node-one


## How to refresh firewall rules in redhat linux

https://bugzilla.redhat.com/show_bug.cgi?id=1531545

@localhost /]# rm -rf  /etc/firewalld/zones/ (delete modified one)
@localhost /]# cp -r /usr/lib/firewalld/zones  /etc/firewalld/zones (copy stock options)
@localhost /]# firewall-cmd --reload
@localhost /]# firewall-cmd --zone=public --list-all



## Mounting secondary disks in longhorn

In longhorn frontend UI, we can make the root disk unschedulable, because we don't want to create pvc on the root node.
Instead we mount the secondary disk and mark it as schedulable.

Go to the longhorn frontend using the Nodeport, (check kubernetes svc in longhorn-system namespace)
VIsit Nodes in the page, and go to the node.(open the breadcrumb in the right)
Mark the root as unschedulable  and add a new volume.
Add Disk and specificy the path where the disk is mounted on the instance (as in the fstab)
reserve some storage  (for eg for 5 tb reserve 30 GB)


## Setting ELK
refer elk.md file

## Setting lighthouse
refer lighthouse.md file
update the lighthouse server IP in common_cm.yaml file