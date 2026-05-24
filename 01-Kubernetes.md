# Sources:
- kubeadm: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
- kubeadm create cluster: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm
- kubectl: source: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux
- docker: https://docs.docker.com/engine/install/rhel/
- cri-dockerd: https://github.com/Mirantis/cri-dockerd
- cilium: https://github.com/cilium/cilium
- multi control node kubernetes: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

# Prerequisites

## disable SELinux
- sudo setenforce 0
- sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

## add Kubernetes yum repository
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.36/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.36/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

## add centos repo
cat <<EOF | sudo tee /etc/yum.repos.d/centos-baseos.repo
[centos-baseos]
name=CentOS Stream 9 - BaseOS
baseurl=https://mirror.stream.centos.org/9-stream/BaseOS/x86_64/os/
enabled=1
gpgcheck=1
gpgkey=https://www.centos.org/keys/RPM-GPG-KEY-CentOS-Official-SHA256
EOF

## add appstream reo
cat <<EOF | sudo tee /etc/yum.repos.d/centos-appstream.repo
[centos-appstream]
name=CentOS Stream 9 - AppStream
baseurl=https://mirror.stream.centos.org/9-stream/AppStream/x86_64/os/
enabled=1
gpgcheck=1
gpgkey=https://www.centos.org/keys/RPM-GPG-KEY-CentOS-Official-SHA256
EOF

## add docker repo
- sudo dnf -y install dnf-plugins-core
- sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

## install iptables and cri-tools
- sudo yum install -y iptables
- sudo yum install -y cri-tools --disableexcludes=kubernetes


# Install kubeadmin

## install kubelet, kubeadm, kubectl and start kubelet
- sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
- sudo systemctl enable --now kubelet

# Install docker and cri-dockerd

## install docker engine
- sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
- confirm with y twice when prompted

## start and test docker
- sudo systemctl enable --now docker
- sudo docker run hello-world

## install cri-dockerd
- curl -LO https://github.com/Mirantis/cri-dockerd/releases/download/v0.4.3/cri-dockerd-0.4.3.amd64.tgz
- tar xvf cri-dockerd-0.4.3.amd64.tgz
- sudo install -o root -g root -m 0755 cri-dockerd/cri-dockerd /usr/local/bin/cri-dockerd

## setup cri-dockerd systemd service
- curl -LO https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
- curl -LO https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket
- sudo mv cri-docker.service cri-docker.socket /etc/systemd/system/
- sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
- sudo systemctl daemon-reload
- sudo systemctl enable --now cri-docker.socket

## verify cri-dockerd
- Should show active (listening): sudo systemctl status cri-docker.socket
- verifiy socket exists: ls -la /var/run/cri-dockerd.sock
- test cri-dockerd respond:  sudo crictl --runtime-endpoint unix:///var/run/cri-dockerd.sock info

# Initialize Kubernetes cluster

## Single Control node Init
- sudo kubeadm init --cri-socket=unix:///var/run/cri-dockerd.sock --pod-network-cidr=10.244.0.0/16
This will produce the kubeadm join command you need in order to add worker nodes (copy it for later use)

## Multi Control node init
- Make sure the loadbalancer points to all your virtual machines
- get the ip and port of the loadbalancer
- sudo kubeadm init --control-plane-endpoint "<LB_IP>:<LB_Port>" --upload-certs --cri-socket=unix:///var/run/cri-dockerd.sock   --pod-network-cidr=10.244.0.0/16
This will produce the kubeadm join command you need in order to add worker nodes and other control nodes (copy it for later use)

## make cluster usable
- mkdir -p $HOME/.kube
- sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
- sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install network plugin (container network interface: cilium)

## install Cilium CLI
- curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/v0.19.2/cilium-linux-amd64.tar.gz
- sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin
- rm cilium-linux-amd64.tar.gz

## install Cilium into the cluster
- cilium install

## verify Cilium is running
- cilium status --wait

## verify Pod Network
- check for CoreDNS pod: kubectl get pods --all-namespaces

# post Install

## admin Access from local machine
- copy admin config: scp root@<control-plane-host>:/etc/kubernetes/admin.conf .
- kubectl --kubeconfig ./admin.conf get nodes

## user account creation (better then admin access)
- this creates a user and returns the user config: sudo kubeadm kubeconfig user --client-name <ClientName> > /etc/kubernetes/user.conf
- The user ocnfig can then be used to access the cluster like above:
- copy user config: scp root@<control-plane-host>:/etc/kubernetes/user.conf .
- this will fail: kubectl --kubeconfig ./user.conf get pods -n kube-system
- grant the new user at least readonly access: kubectl create clusterrolebinding <ClientName>-binding --clusterrole=view --user=<ClientName>
- now this will work: kubectl --kubeconfig ./user.conf get pods -n kube-system
- retrieve yaml files for repository: kubectl get clusterrolebinding <ClientName>-binding -o yaml

## some user accout tests
- will fail: kubectl --kubeconfig ./user.conf get nodes
- create a custom cluster role to remedy the situation: kubectl create clusterrole node-viewer --verb=get,list,watch --resource=nodes
- create clusterrolebinding: kubectl create clusterrolebinding <ClientName>-node-viewer-binding --clusterrole=node-viewer --user=<ClientName>
- now this will work: kubectl --kubeconfig ./user.conf get nodes
- retrieve yaml files for repository: 
- kubectl get clusterrolebinding <ClientName>-node-viewer-binding -o yaml
- kubectl get clusterrole node-viewer -o yaml

## proxy api server to localhost (optional)
- scp root@<control-plane-host>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf proxy
- kubectl --kubeconfig ./admin.conf proxy
- access API Server locally at http://localhost:8001/api/v1


## FYI: to allow pod sceduling on control nodes for single node clusters
don't run this! kubectl taint nodes --all node-role.kubernetes.io/control-plane-
don't run this! kubectl label nodes --all node.kubernetes.io/exclude-from-external-load-balancers-

## Install Helm
Helm can be installed on a local machine that connects to the cluster using kubeconfig. Alternatively it works on a controlplane node as well
- curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
- chmod 700 get_helm.sh
- ./get_helm.sh
- verify: helm version
- rm get_helm.sh


# Join Nodes

## Install worker and control plane nodes
- Install Kubernetes and Docker (including cri-dockerd) as described above
- Make sure not to run kubeadm init!
- Cilium only needs to be installed on the first control plane.

## Get the join command
- If you have stored the join command after the kubeadm init command, then just use it otherwise create a fresh token and join command
- kubeadm token create --print-join-command
- make sure to append the necessary commands like --cri-socket (see below)

## join worker nodes: (for single control plane clusters use the control plane ip instead of the loadbalancer_ip)
- sudo kubeadm join <Loadbalancer_IP>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --cri-socket=unix:///var/run/cri-dockerd.sock

## join control plane nodes:
- sudo kubeadm join <Loadbalancer_IP>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --control-plane --certificate-key <key> --cri-socket=unix:///var/run/cri-dockerd.sock
- The certificate-key is printed by kubeadm init --upload-certs (expires after 2 hours)
- To regenerate: sudo kubeadm init phase upload-certs --upload-certs



# Remove a node
- list nodes: kubectl get nodes
- kubectl drain <node name> --delete-emptydir-data --force --ignore-daemonsets
- sudo kubeadm reset --cri-socket=unix:///var/run/cri-dockerd.sock
- kubectl delete node <node name>