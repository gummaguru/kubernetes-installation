# kubernetes-installation
## Kubernetes v1.20 installation - Ubuntu
 
The issue is with the latest version of Kubernetes. This is a bug in version 1.22. Installing an older version (1.20) will resolve the issue. Create a new instance and follow the below steps,
 
1. Install docker.
 
Execute the below command to switch to the root user,
   sudo su
 
Create a file with the below command,
   nano docker.sh
 
Add the below contents in this file,
 
   swapoff -a
   apt update -y
   apt install -y docker.io
   systemctl enable docker
   systemctl start docker
 
Now execute the below commands to run the above file and install docker,
   chmod -R 777 docker.sh
   ./docker.sh
 
 
2. Install Kubernetes
 
Create a file with the below command,
   nano kubernetes.sh
 
Add the below contents in this file,
  sudo apt-get update && sudo apt-get install -y apt-transport-https curl
   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
   deb https://apt.kubernetes.io/ kubernetes-xenial main
   EOF
   sudo apt-get update
   sudo apt-get install -y kubelet=1.20.2-00 kubeadm=1.20.2-00 kubectl=1.20.2-00
   sudo apt-mark hold kubelet kubeadm kubectl
 
Now execute the below commands to run the above file and install Kubernetes,
   chmod -R 777 kubernetes.sh
   ./kubernetes.sh
 
3. Initialize the cluster with the below command,
   kubeadm init --apiserver-advertise-address=<your private IP address> --pod-network-cidr=192.168.0.0/16 --ignore-preflight-errors all
   Change the private IP accordingly.
 
 
When the cluster is initialized Successfully it will print the kubeadm join command. Note that command as you will have to execute it in the worker node to connect it to the master later.
 
Now, execute the below commands in the master one-by-one,
 exit
 
    mkdir -p $HOME/.kube
 
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

4. Follow the Step 1 and 2 in worker node as well
 
5. Now execute the kubeadm join command in the worker node from the root user
 
6. Now execute the below command in the master node,
    kubectl get nodes

  
  ==========================================================================================================================================================================
  =======================================================================================================================================================================
  ==================================================== MULTI-MASTER-CLUSTER FOR HIGH AVAILABLE =================================================
  
### Multi Master Kubernetes Cluster
   ####    https://youtu.be/q92MYG-EW-w    ##############
 Create a multi master kubernetes cluster

Steps to create a Multi Master kubernetes cluster

Servers -

    1. adminnode - Loadbalancer (10.142.0.8)
    2. 3 Masters - Kmaster1, kmaster2, kmaster3 (10.142.0.2, 10.142.0.3, 10.142.0.4)
    3. 3 Nodes - knode1, knode2, knode3 (10.142.0.5, 10.142.0.6, 10.142.0.7)
    4. Pre- requisites - a. SSH enabled on all servers b. servers are able to ssh to each other
Admin Node

    5.cd multimaster

    6. Run : ./1_install_cffsl.sh to install cffsl (used to generate SSL keys)

    7.Run : ./2_install_kubectl_admin.sh to install kubectl executable

    8. Run : ./3_install_haproxy.sh to install haproxy for Loadbalancing 3 masters

    9. As root : cat haproxy_cfg >> /etc/haproxy/haproxy.cfg - This will add haproxy configurations to haproxy.cfg for frontend and backend

    10. As root run sudo systemctl restart haproxy

    11. Generate certificates for CA and ETCD cluster as below

        a. cd certs

        b. ca-config.json and ca-csr.json is already provided with dummy examples. Make changes accordingly as per your organization standards

        c. execute - ./4_generateca.sh to generate the root CA file and private key

        d. Files kubernetes-csr.json is already provided with dummy examples. Make changes to them accordingly

        e. execute - ./5_generateetcdcert.sh to generate certificates for ETCD cluster. Make sure to make changes related to Master IP addresses inside 5_generateetcdcert.sh            script.

        f. scp ca.pem kubernetes.pem kubernetes-key.pem to all nodes from admin Node

Master Nodes and minion nodes -

On all the nodes master + minion - run the below -

    12. cd multimaster/install_kubeadm

    13. Run - ./6_install_docker_kubeadm.sh to install docker + kubeadm + kubelet + kubectl

Only on Master Nodes - Install etcd on master nodes as below -

    14. cd multimaster/install_etcd

    15. Run ./7_install_etcd.sh - This will install ETCD and will copy the certificates from your home account to /etc/etcd

    16. The files etcd.service.kmaster1/2/3 are provided with etcd default configurations. Please make sure that you edit them according to the IP addresses used to generate the certificates.

    17. cp etcd.service.kmaster1 /etc/systemd/system/etcd.service - on kmaster1

    18. cp etcd.service.kmaster2 /etc/systemd/system/etcd.service - on kmaster2

    19. cp etcd.service.kmaster3 /etc/systemd/system/etcd.service - on kmaster3

    20. Run systemctl daemon-reload

    21. Run systemctl enable etcd

    22. Run systemctl start etcd

    23. Verify the cluster using - ETCDCTL_API=3 etcdctl member list

      Output as below - root@kmaster1:~# ETCDCTL_API=3 etcdctl member list 56fbe3d44c581b82, started, kmaster2, https://10.142.0.3:2380, https://10.142.0.3:2379                80bd93ff16829bfc, started, kmaster3, https://10.142.0.4:2380, https://10.142.0.4:2379 a3da5fa43de07c2f, started, kmaster1, https://10.142.0.2:2380, https://10.142.0.2:2379

Only on master nodes -

Initialize cluster as below -

    24. Login to any one of the master (kmaster1)

    25. cd multimaster/init_kubeadm

    26. File config.yaml is provided with default kubeadm configuration required to initialize kubeadm. Please modify the ip addresses or names of certificates accordingly.            You can also edit the pod subnet in the default configuration. Additionally - apiServerCertSANs and controlPlaneEndpoint must be the ip address of the loadbalancer            or the adminnode.

                kubeadm config migrate --old-config config.yaml --new-config new.config.yaml

    27. Run : 9_kubeadminit.sh to initialize the cluster with the above config.yaml. Make sure to copy the last few lines for kubeadm join for nodes

    28. on the root home of kmaster1 -

         a. tar cvf pki.tar /etc/kubernetes/pki

         b. scp this tar to kmaster2 and kmaster3

    29. Login to Kmaster2 and kmaster3 and do the below

    30. tar xvf pki.tar

    31. cd inside pki directory and run - rm apiserver.*

    32. mv pki /etc/kubernetes/

    33. cd multimaster/init_kubeadm

    34. execute - ./9_kubeadminit.sh

On Worker / Minion Nodes -

    35. Run the kubeadm join command on all nodes

    36. Login to any master and execute - kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes to see if all nodes and masters are joined

Initialize the cluster on admin node for secure access -

    37. Login to kmaster and run - chmod +r /etc/kubernetes/admin.conf

    38. scp admin.conf to adminode root home directory

    39. login to adminnode

    40. mkdir ~/.kube

    41. cp admin.conf ~/.kube/config

    42. chmod 600 ~/.kube/config

    43. on kmaster1 chmod 600 /etc/kubernetes/admin.conf

    44. You can now start using adminnode for all kubernetes cluster related activities without giving master access to everyone.

    45. Add CNI from admin Node: kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
