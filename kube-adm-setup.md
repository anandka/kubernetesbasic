#For each host

SSH into the machine and become root if you are not already (for example, run sudo su -)

If the machine is running CentOS 7, run:


# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://yum.kubernetes.io/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

~~~~~~~~~~
 setenforce 0
 yum install -y docker kubelet kubeadm kubectl kubernetes-cni
 systemctl enable docker && systemctl start docker
 systemctl enable kubelet && systemctl start kubelet
 systemctl disable firewalld && systemctl stop firewalld

~~~~~~~~~~

The kubelet is now restarting every few seconds, as it waits in a crashloop for 




#On Master

~~~~~~~~
 kubeadm init

~~~~~~~~

The output should look like

~~~~~~~~~~~~
<master/tokens> generated token: "f0c861.753c505740ecde4c"
<master/pki> created keys and certificates in "/etc/kubernetes/pki"
<util/kubeconfig> created "/etc/kubernetes/kubelet.conf"
<util/kubeconfig> created "/etc/kubernetes/admin.conf"
<master/apiclient> created API client configuration
<master/apiclient> created API client, waiting for the control plane to become ready
<master/apiclient> all control plane components are healthy after 61.346626 seconds
<master/apiclient> waiting for at least one node to register and become ready
<master/apiclient> first node is ready after 4.506807 seconds
<master/discovery> created essential addon: kube-discovery
<master/addons> created essential addon: kube-proxy
<master/addons> created essential addon: kube-dns

Kubernetes master initialised successfully!

You can connect any number of nodes by running:

kubeadm join --token <token> <master-ip>
~~~~~~~~~~~~~


Note down the join command you will need it to add new nodes

Run the below command and verify to check all pods must be running expect kube-dns

~~~~~~~~~~
  kubectl  get pods  --all-namespaces

~~~~~~~~~



#Installing a pod network


You must install a pod network add-on so that your pods can communicate with each other.

Run below command to start pod network

~~~~~~~~~
 kubectl apply -f https://git.io/weave-kube

~~~~~~~

Wait and check if kube-dns pod is now running once it is running you can go ahead
and join nodes

~~~~~~~
	kubectl  get pods  --all-namespaces

~~~~~~~


#On Node

Run the join command which was output on master

~~~~~~~~~~
 kubeadm join --token <token> <master-ip>

~~~~~~~~



Now go on master and verify node has been added

~~~~~~~~
 kubectl get nodes

~~~~~~~~~



Deploying dashboard

~~~~~~~~~~~~
kubectl create -f https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml

~~~~~~~~~~~~


~~~~~~~~~~~
kubectl describe svc kubernetes-dashboard -n kube-system
~~~~~~~~~~~


see the nodeport and open http://masterip:nodeport/ui
