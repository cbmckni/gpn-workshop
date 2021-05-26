Doing Genomics in the Cloud: From Repository to Result
====

Computational genomics has become a core toolkit in the study of biological systems at the molecular level.  To run genomics workflows, a researcher needs access to advanced computer systems including compute, storage, memory, and networks to move and mine huge genomics datasets.  The Cloud provides scalable compute solutions to run workflows.  In this workshop,  we will demonstrate how one can create a compute cluster in the Cloud using Kubernetes and run containerized genomics workflows.    The Cloud workflows will include pulling high-throughput DNA datasets from the NCBI-SRA data repository, performing reference genome mapping of SRA RNAseq datasets, and building a gene co-expression network.

The Demo:

We will cover the complete deployment life cycle that enables scientific workflows to be run in the cloud. This includes:
 - Deployment of a Kubernetes(K8s) cluster using Cisco Container Platform(CCP). 
 - Creating a persistent NFS server to store workflow data, then loading a Gene Expression Matrix(GEM) onto it.
 - Deploying Knowledge Independent Network Construction(KINC), a genomic workflow, to the K8s cloud.
 - Downloading the resulting Gene Coexpression Network(GCN) from the NFS server, then visualizing the network.

## 0. Prerequisites(BOTH SESSIONS)

The following software is necessary to participate in the demo:
 - Cisco Container Platform CLI
 - Helm - Kubernetes Package Management Utility
 - kubectl - Kubernetes CLI 
 - Nextflow - Workflow Manager
 - Java
 - Files/scripts from this repo.

To streamline the demo, all software has been packaged into a virtual machine that has been replicated for each user. 

An additional requirement is access to the CCP environment where the K8s clusters will be created.

**If you do not have your CCP login credentials and access to your personal VM, please let us know.**

### Access Praxis

Navigate to [the Praxis portal](https://dcm.toolwire.com/alai/admin/login.jsp)

Enter your credentials.

Select the class *Running Scientific Workflows on Regional R&E Kubernetes Clusters Workshop*

Select *Learning* at the upper right side of the menu bar.

Select the appropriate lab session, and when prompted start the live lab.

Once the Jupyter notebook is provisioned, select *Terminal* from the menu to access a Bash terminal from within your VM! 

Finally, please clone this repo to a folder with persistent storage:

`git clone <link> /PATH`

# Session 1

In this session we will cover Kubernetes cluster creation and usage, while also performing genomic analysis on human RNA sequences. 

## 1. Deploy Kubernertes Cluster Using Cisco Container Platform

There are two ways to use CCP: the web GUI and command line interface.

For this workshop, we will use the CLI. Instructions for using the web interface are below. 

### Create a K8s Cluster Using the CCP Command Line Interface

There should be an open terminal on the VM that is already running the CCP CLI environment.

**If there is not a CCP CLI terminal open, or you closed the terminal, run the following in a new terminal:**

`cd Desktop/ccp_cli`

`source venv/bin/activate`

This will activate the CCP CLI environment.

We will create a Kubernetes cluster with 1 master node and 3 workers. Give the cluster the same name as your CCP username, or *workshopX*

To create the cluster, run:

`ccp create cluster -name <CCP_Username> -master 1 -m-mem 32768 -worker 3 -w-mem 32768 -vcpu 4 -load 1

The command should run until your cluster has been created.

Next, run `ccp cluster list` to list the clusters, find your cluster and copy the UUID.

To download the kubeconfig for your cluster, run:

`ccp create kubeconfig -uuid <UUID> -file kubeconfig`

Finally, move the kubeconfig to your .kube folder: 

`mv kubeconfig.yaml ~/.kube`

You now have access to your own personal K8s cluster!

## 2. Create Persistant Data Storage to Host Workflow Data

Now that you have a K8s cluster, it is time to access it from the VM.

First, tell the VM where to look for the Kubernetes configuration file:

`echo 'export KUBECONFIG=~/Downloads/kubeconfig.yaml' >> ~/.bashrc && source ~/.bashrc`

Test your connection with:

`kubectl config current-context`

The output should match the name of your cluster.

Now it is time to provision a NFS server to store workflow data. We will streamline this process by using Helm.

Update Helm's repositories(similar to `apt-get update)`:

`helm repo update`

Next, install a NFS provisioner onto the K8s cluster to permit dynamic provisoning for 50Gb of persistent data:

`helm install kf stable/nfs-server-provisioner \`

`--set=persistence.enabled=true,persistence.storageClass=standard,persistence.size=50Gi`

Check that the `nfs` storage class exists:

`kubectl get sc`

Next, deploy a 42Gb Persistant Volume Claim(PVC) to the cluster:

`cd ~/PATH/techex-demo`

`kubectl create -f task-pv-claim.yaml`

Check that the PVC was deployed successfully:

`kubectl get pvc`

Finally, login to the PVC to get a shell, enabling you to view and manage files:

`nextflow kuberun login -v task-pv-claim`

## 3. Deploy a container to pull genomic data:

Navigate to the *yamls* directory and deploy the sra-tools container:

`kubectl create -f /PATH/sra-tools.yaml`

Get the name of your pod:

`kubectl get pods`

Get a Bash session inside your pod:

`kubectl exec -ti <NAME> -- /bin/bash`

Once inside the pod, navigate to the persistent directory `/workspace`:

`cd /workspace`

Initialize SRA-Tools:

`vdb-config --interactive`

Pull the sequence: `fasterq-dump -S SRR5139429`

## 4. Deploy a Genomic Workflow to Your Cloud

**In a new terminal window....**

Go to the **techex-demo** folder:

`cd ~/Downloads/techex-demo`

Give Nextflow the necessary permissions to deploy jobs to your K8s cluster:

```
kubectl create rolebinding default-edit --clusterrole=edit --serviceaccount=default:default 
kubectl create rolebinding default-view --clusterrole=view --serviceaccount=default:default
```

Load the input data onto the PVC:

`./kube-load.sh task-pv-claim input`

Deploy KINC using `nextflow-kuberun`:

`nextflow kuberun systemsgenetics/kinc-nf -v task-pv-claim`

**The workflow should take about 10-15 minutes to execute.**

### 5. Retreive and Visualize Gene Co-expression Network

Copy the output of KINC from the PVC to your VM:

`./kube-save.sh task-pv-claim output`

Open Cytoscape. (Applications -> Other -> Cytoscape)

Go to your desktop and open a file browsing window, navigate to the output folder:

`cd ~/Downloads/techex-demo/output/Yeast`

Finally, drag the file `Yeast.coexpnet.txt` from the file browser to Cytoscape!

The network should now be visualized! 






