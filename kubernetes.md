# Kubernetes

_Kubernetes is a portable, extensible open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation._

Containers runs inside a construct called **Pods**.

Following are the tools to be used:
-   `kubeadm`: the command to bootstrap the cluster.
    
-   `kubelet`: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
    
-   `kubectl`: the command line util to talk to your cluster.

### Setup & Install `kubectl`

```bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin/kubectl
$ kubectl version # Check the installation correctly
$ echo "source <(kubectl completion bash)" >> ~/.bashrc # Enables kubectl auto completions on pressing Tab key
```

### Prerequisites:

- A _Master_ (VM, VPS, etc) with atleast 2 core CPUs & 4GB RAM 
- At least 1 _Node_ (VM, VPS, etc) with atleast 2 core CPU & 4GB RAM

_For this setup, we assume that the Master's IP address is `78.47.170.89` & Node's IP address is `88.99.173.101`_

### Initial setup _[To be done on Master & all other Nodes]_:
```bash
$ sudo su
$ apt-get update
$ swapoff -a	# disable devices and files for paging and swapping
# Swap is a space on a disk that is used when the amount of physical RAM memory is full. 
# When a Linux system runs out of RAM, inactive pages are moved from the RAM to the swap space.
$ nano /etc/fstab	# Look for /swapfile and if you find it.. comment that line with and save the file
$ nano /etc/hostname # Update the Hostname according to your convinience (this is optional)
$ ifconfig # To get the IP address

$  nano /etc/network/interfaces # Add the below at the end of the file
auto enp0s8
iface enp0s8 inet static
address <ip-address-of-system (obtained from ifconfig)>

$ nano /etc/hosts # add all the nodes ip's, name's including with master details (hostnames are obtained from /etc/hostname)
Example: 	78.47.170.89 <master-hostname>
		88.99.173.101 <node-hostname>
$ apt-get install openssh-server
```

### Install `docker` _[To be done on Master & all other Nodes]_:
Follow the steps mentioned [here](https://drive.google.com/open?id=18TUr_Vc_FDQKhBG-AxIPzHjdBm6x2-88) to install `docker`

### Install `kubectl`, `kubelet` and `kubeadm` _[To be done on Master & all other Nodes]_:

```bash
$ sudo apt-get update && sudo apt-get install -y apt-transport-https curl
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl

# Update the Kubernetes configuration
nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# Add the below after the Environment last line
Environment="cgroup-driverer=systemd/cgroup-driver=cgroupfs"
```
### Start the pods:

#### On Master:
```bash
$ sudo kubeadm init --apiserver-advertise-address=78.47.170.89 # Use the correct IP address of master
# At the end of running this command, you will obtain a `kubeadm join` command to be entered in each Node 

$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### On Node:
Run the `kubeadm join` command obtained from Master

### Install pod network add-on _[On Master]_:
A pod network add-on must be installed so that pods can communicate with each other. 
Out of several options available, we can choose any add-on. Here, we use Weave Net add-on:

```bash
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
``` 

### Start using Kubernetes:

Check if the versions of `kubeadm` & `kubectl` are same for the Master & Node. If there is a discrepancy, upgrade the tools using steps mentioned [here]()

```bash
# Get status of Nodes
$ kubectl get nodes

# Get status of pods
$ kubectl get pods --all-namespaces

# To check the status the pods status
$ kubectl get -o wide pod --all-namespaces
```