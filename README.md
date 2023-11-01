# Calico Cloud on EKS - Workshop Environment Preparation

This repository will guide you through the preparation process of building an EKS cluster using the AWS CloudShell that will be used in Tigera's Calico Cloud workshop. The goal is to reduce the time used for setting up infrastructure during the workshop, optimizing the Calico Cloud learning and ensuring everyone has the same experience.

## Getting Started with AWS CloudShell

The following are the basic requirements to **start** the workshop.

* AWS Account [AWS Console](https://portal.aws.amazon.com)
* AWS CloudShell [https://portal.aws.amazon.com/cloudshell](https://portal.aws.amazon.com/cloudshell)
* Amazon EKS Cluster - to be created here!

# Instructions

1. Login to AWS Portal at https://portal.aws.amazon.com.
2. Open the AWS CloudShell.

   ![cloudshell](https://github.com/tigera-solutions/eks-workshop-prep/assets/104035488/a1f0b555-018d-488f-8d8c-975b5c391ede)

3. Install the bash-completion on the AWS CloudShell.

   ```bash
   sudo yum -y install bash-completion
   ```

4. Configure the kubectl autocomplete.

   ```bash
   source <(kubectl completion bash) # set up autocomplete in bash into the current shell, bash-completion package should be installed first.
   echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
   ```

   You can also use a shorthand alias for kubectl that also works with completion:

   ```bash
   alias k=kubectl
   complete -o default -F __start_kubectl k
   echo "alias k=kubectl"  >> ~/.bashrc
   echo "complete -o default -F __start_kubectl k" >> ~/.bashrc
   ```

5. Install the eksctl - [Installation instructions](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

   ```bash
   mkdir ~/.local/bin
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   sudo mv /tmp/eksctl ~/.local/bin
   eksctl version
   ```

6. Install the K9S, if you like it.

   ```bash
   curl --silent --location "https://github.com/derailed/k9s/releases/download/v0.27.4/k9s_Linux_amd64.tar.gz" | tar xz -C /tmp
   sudo mv /tmp/k9s ~/.local/bin
   k9s version
   ```

## Create an Amazon EKS Cluster

1. Define the environment variables to be used by the resources definition.

   > **NOTE**: In this section, we'll create some environment variables. If your terminal session restarts, you may need to reset these variables. You can do that using the following command:
   >
   > ```console
   > source ~/workshopvars.env
   > ```

   ```bash
   # Feel free to use the cluster name and the region that better suits you.
   export CLUSTERNAME=tigera-workshop
   export REGION=ca-central-1
   # Persist for later sessions in case of disconnection.
   echo "# Start Lab Params" > ~/workshopvars.env
   echo export CLUSTERNAME=$CLUSTERNAME >> ~/workshopvars.env
   echo export REGION=$REGION >> ~/workshopvars.env
   ```
 
2. Create the EKS cluster.
   
   ```bash
   eksctl create cluster \
     --name $CLUSTERNAME \
     --version 1.27 \
     --region $REGION \
     --node-type m5.xlarge
   ```

3. Verify your cluster status. The `status` should be `ACTIVE`.

   ```bash
   aws eks describe-cluster \
     --name $CLUSTERNAME \
     --region $REGION \
     --no-cli-pager \
     --output yaml
   ```

4. Verify you have API access to your new EKS cluster

   ```bash
   kubectl get nodes
   ```

   The output will be something similar to:

   <pre>
   NAME                                              STATUS   ROLES    AGE   VERSION
   ip-192-168-4-255.ca-central-1.compute.internal    Ready    <none>   17m   v1.27.5-eks-43840fb
   ip-192-168-54-212.ca-central-1.compute.internal   Ready    <none>   17m   v1.27.5-eks-43840fb
   </pre>

   To see more details about your cluster:

   ```bash
    kubectl cluster-info
   ```

   The output will be something similar to:
   <pre>
   Kubernetes control plane is running at https://16AAA7FFCE2B8F8C4C449E9264BA3612.yl4.ca-central-1.eks.amazonaws.com
   CoreDNS is running at https://16AAA7FFCE2B8F8C4C449E9264BA3612.yl4.ca-central-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy  </br>

   To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
   </pre>

   You should now have a Kubernetes cluster running with 2 nodes. You do not see the master servers for the cluster because these are managed by AWS. The Control Plane services which manage the Kubernetes cluster such as scheduling, API access, configuration data store and object controllers are all provided as services to the nodes.


## Scale down the nodegroup to 0 nodes until the workshop starts.

1. Save the nodegroup name in an environment variable:

   ```bash
   export NGNAME=$(eksctl get nodegroups --cluster $CLUSTERNAME --region $REGION | grep $CLUSTERNAME | awk -F ' ' '{print $2}') && \
   echo export NGNAME=$NGNAME >> ~/workshopvars.env
   ```

2. Scale the nodegroup down to 0 nodes, to reduce the cost.

   ```bash
   eksctl scale nodegroup $NGNAME \
     --cluster $CLUSTERNAME \
     --region $REGION \
     --nodes 0 \
     --nodes-max 1 \
     --nodes-min 0
   ```

3. It will take a minute or two until all nodes are deleted. You can monitor the process using the following command: 

   ```bash
   watch kubectl get nodes
   ```

   When there are no more worker nodes in your EKS cluster, you should see:
   <pre>
   Every 2.0s: kubectl get nodes
   
   No resources found
   </pre>


 ## Scale up the nodegroup to 2 nodes before the workshop starts.

1. Connect back to your AWS CloudShell and load the environment variables:

   ```bash
   source ~/workshopvars.env
   ```

2. Scale up the nodegroup back to 2 nodes:

   ```bash
   eksctl scale nodegroup $NGNAME \
     --cluster $CLUSTERNAME \
     --region $REGION \
     --nodes 2 \
     --nodes-max 2 \
     --nodes-min 2
   ```

3. It will take a few minutes until the nodes are back in `Ready` status. You can monitor it with the following command:

   ```bash
   watch kubectl get nodes
   ```

   Wait until the output becomes:
   <pre>
   Every 2.0s: kubectl get nodes  

   NAME                                              STATUS   ROLES    AGE    VERSION
   ip-192-168-13-66.ca-central-1.compute.internal    Ready    <none>   2m2s   v1.27.5-eks-43840fb
   ip-192-168-69-111.ca-central-1.compute.internal   Ready    <none>   2m2s   v1.27.5-eks-43840fb
   </pre>

### You are now ready to start the workshop!