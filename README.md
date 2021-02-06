# Kubernetes-Config_of_Multi_Nodes_Cluster_Over_AWS_Cloud
Configuration of a Multi Nodes in AWS for Kubernetes

     ### Bref Introduction: ###
              First, what is kubernetes ? Kubernetes is an open-source container-orchestration system for automating computer application deployment, scaling, and management. So in the process of becoming an Sysadmin i encounter the necessity to learn kubernetes to automate the deployment and configuration  of cluster in the cloud to help developpers manage their codes easily. 
              Now why we need multi nodes in cluster ? We need multi nodes Cluster (cluster is like a department/room inside which a master node work together with multi workers nodes/slaves to provide a solution), because in production the down time of a node isn't acceptable so we have to work with multi nodes to have a high availability of nodes running and avoiding loss of data during working services. Below i describe the steps to achieve our goals..

## I) Launch the instances from AWS
        An instance is a virtual server in the AWS cloud. With Amazon EC2, you can set up and configure the operating system and applications that run on your instance.  To know more about instances i refer you to Amazon AWS doc.
        My plan is to configure a three nodes in AWS for kubernetes, and first we have to launch ec2 instances . Go to this link to know how to launch an instance: "https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-1-launch-instance.html"
        
 ## II) Setup the Master Node
         Now come the main work, so after launch the 3 instances we will name one as a Master and the 2 remains as worker nodes.
         
         Inside the Master node,  below are few steps to procceed with:
         ## a) Install docker by:##
             "yum install docker -y"   
             Why install docker ? Because Kubernetes run by default on a container engine to manage the nodes, and many containers engine or runtime are available on the market as docker, containerd, CRIO....
             
         ## b) Enable the docker service and start it by:##
                    "systemctl enable --now docker"
         
         ## c) Install kubeadm (Kubeadm is command used in kubernetes to set multi nodes) by running:##
                                "yum install kubeadm"
            **Note: Be sure to have a kubernete repo in your instance before installing the cmd kubeadm.**
                *Ressources to help configure a kubernetes repo: <"https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/">*
         
         ## d) Enable the program "kubelet" which come under the installation of kubeadm:## 
                          "systemctl enable --now kubelet"
              Kubelet is just a program in the worker node who checks the state/if alive of the worker node and signal it to the master node.
              
         ##e) Downloads the required images used by kubernetes using:##
                          "kubeadm config images pull"
        
         ## f) Initialize the master with: "kubeadm  init --pod-network-cidr=10.240.0.0/16 --ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem" which will launch all services required.##
    **Note1: --pod-network-cidr=10.240.0.0/16 => defines the net/w range givem to our nodes.**
    **Note2: "--ignore-preflight-errors=NumCPU && --ignore-preflight-errors=Mem" are used as parameters to overcomes CPU and RAM quantities of the Ec2 instances used in AWS.**
    **Note1: Before initialize make sure your docker driver is set to __systemd__ ; and you can check it by:  " docker info | grep Driver "**
    **Note2: Make also sure to install before a networking program called "iproute-tc" which is used by kubernetes to manage some routing tables on behalf net/w cards: "yum install iproute-tc -y"**
    
         ## g) Configure the kubernates directory as a regular user by doing:
             mkdir -p $HOME/.kube
             sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
             sudo chown $(id -u):$(id -g) $HOME/.kube/config
         
         ## h) Verify that your master node is up by following cmd: 
                "kubectl get nodes"  and you will remark that  the master node isn't ready, this means that you have to join a worker node. 
                
 ## II) Setup the first Worker Node:
     Here as this node has to join the master node you have to reproduce steps from a)  to d) from the setup of Master node.
     
 ## III) Join the Worker Node to the Master Node:
      1) Inside the master Node, you have to create a token from which the worker node will join:
         **Kubeadm token create --print-join-command**
         
      2) Now go in the worker node and paste in the terminal the token received from Master Node:
              kubeadm join <IP_ofMaster_Node:Port_no_To_be_join> --token <toke_ID>
              **Note: Make sure to configure the brigde card for kubernetes in : "/etc/sysctl.d/k8s.conf"  as per this ressource:=> <https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm>
              
       3) Reload the system through with: "sysctl --system" 
      
       4) Now running again in the Master Node : "kubectl get nodes"  you will see the Worker node has successfully joined the Master Node. 
         
 ## IV) Enable the flannel program to allow network routing flow between the Master && Worker Node:
   In the previous, step we have successfully joined a Worker Node in the Master Node but you will see that when lauching a pod/deployemnt we don't have connectivity inside so we have to install the kubernetes flannel. 
   What's flannel? It's a program running in the backend which uses VXLAN to provide a tunnel for an overlay net/w in the cluster.  More info refer this=> <https://github.com/coreos/flannel/blob/master/Documentation/kubernetes.md>
   
  So this program can be installed on the Master Node by: "<kubectl apply  -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml>"
  
  ## V) Conclusion
   By running "kubectl get nodes" , you can verify that all nodes are now ready to communicate and exchange datas. You can even launching a deployment or pods for confirmation.
   
   
 #Happy Automate && Administrate !!! Any suggestions will be the welcome.
