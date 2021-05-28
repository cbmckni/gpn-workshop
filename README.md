Doing Genomics in the Cloud: From Repository to Result
====

Computational genomics has become a core toolkit in the study of biological systems at the molecular level.  To run genomics workflows, a researcher needs access to advanced computer systems including compute, storage, memory, and networks to move and mine huge genomics datasets.  The Cloud provides scalable compute solutions to run workflows.  In this workshop,  we will demonstrate how one can create a compute cluster in the Cloud using Kubernetes and run containerized genomics workflows.    The Cloud workflows will include pulling high-throughput DNA datasets from the NCBI-SRA data repository, performing reference genome mapping of SRA RNAseq datasets, and building a gene co-expression network.

The Demo:

We will cover the complete deployment life cycle that enables scientific workflows to be run in the cloud. This includes:
 - Deployment/Access to a Kubernetes(K8s) cluster using Cisco Container Platform(CCP). 
 - Creating a persistent NFS server to store workflow data, then loading a Gene Expression Matrix(GEM) onto it.
 - Deploying Knowledge Independent Network Construction(KINC), a genomic workflow, to the K8s cloud.
 - Downloading the resulting Gene Coexpression Network(GCN) from the NFS server, then visualizing the network.

## 0. Prerequisites(BOTH SESSIONS)

The following software is necessary to participate in the demo:
 - helm
 - Cisco Container Platform CLI
 - kubectl - Kubernetes CLI 
 - Nextflow - Workflow Manager
 - Java
 - Files/scripts from this repo.

To streamline the workshop, all software has been packaged into a virtual machine that has been replicated for each user. 

An additional requirement is access to the kubernetes clusters that will be used for the workshop.

**If you do not have your CCP cluster credentials(kubeconfig) and access to your personal VM, please let us know.**

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

## 1. Access Kubernertes Cluster

Download or copy/paste the kubeconfig you were provided to a file named `config`.

Move the kubeconfig to your .kube folder: 

`mv config.yaml ~/.kube`

`chmod 600 ~/.kube/config`

Confirm your cluster name:

`kubectl config current-context`

The output should match the name of your cluster.

You now have access to your K8s cluster!

Issue an API call to view current pods(containers) that are deployed:

`kubectl get pods`

## 2. Create Persistant Data Storage to Host Workflow Data

Now it is time to provision a NFS server to store workflow data. We will streamline this process by using Helm. Helm is a kubernetes package manager!

Install Helm:

`cd PATH`

`wget https://get.helm.sh/helm-v3.6.0-linux-amd64.tar.gz`

`tar -xvf helm-v3.6.0-linux-amd64.tar.gz`

`sudo cp linux-amd64/helm /usr/local/bin`

Add the `stable` repo:

`helm repo add stable https://charts.helm.sh/stable`

Update Helm's repositories(similar to `apt-get update)`:

`helm repo update`

Next, install a NFS provisioner onto the K8s cluster to permit dynamic provisoning for 50Gb of persistent data:

`helm install kf stable/nfs-server-provisioner \`

`--set=persistence.enabled=true,persistence.storageClass=standard,persistence.size=52Gi`

Check that the `nfs` storage class exists:

`kubectl get sc`

Next, deploy a 42Gb Persistant Volume Claim(PVC) to the cluster:

`cd ~/PATH/techex-demo`

`kubectl create -f task-pv-claim.yaml`

Check that the PVC was deployed successfully:

`kubectl get pvc`

Give Nextflow the necessary permissions to deploy jobs to your K8s cluster:

```
kubectl create rolebinding default-edit --clusterrole=edit --serviceaccount=default:default 
kubectl create rolebinding default-view --clusterrole=view --serviceaccount=default:default
```

Finally, login to the PVC to get a shell, enabling you to view and manage files:

`nextflow kuberun login -v task-pv-claim`

**This tab now on your cluster's persistent filesystem.** 

**To continue, open a new tab with File -> New -> Terminal**

## 3. Deploy a Container to Pull Genomic Data

Navigate to the *yamls* directory and deploy the sra-tools container:

`kubectl create -f /PATH/sra-tools.yaml`

Get the name of your pod:

`kubectl get pods`

Get a Bash session inside your pod:

`kubectl exec -ti <NAME> -- /bin/bash`

Once inside the pod, navigate to the persistent directory `/workspace`:

`cd /workspace`

Initialize SRA-Tools:

`printf '/LIBS/GUID = "%s"\n' `uuidgen` > /root/.ncbi/user-settings.mkfg`

Pull the sequence: `fasterq-dump --split-files SRR5139429`

**While the file is downloading, switch tabs to your other terminal.**


## 4. Configure GEMmaker 

**On the cluster's filesystem....**

We can now start creating the input directory structure:

Create a workflow directory in `/workspace`:

`mkdir -p /workspace/gm-<YOUR_NAME>` 

Install git in your container and download the workflow:

`apk add --no-cache git`

`git clone https://github.com/SystemsGenetics/GEMmaker.git`

Copy the GEMmaker input folder to your workflow directory:

`cp -R GEMmaker/input /workspace/gm-<YOUR_NAME>`

Move the sequence into the input folder:

`mv /workspace/gm-<YOUR_NAME>/SRR5139429_* /workspace/gm-<YOUR_NAME>/input`

**Switch tabs**

## 4. Deploy GEMmaker

**On your local VM's filesystem....**

Clone GEMmaker and edit the nextflow.config file:

`git clone https://github.com/SystemsGenetics/GEMmaker.git`

`nano GEMmaker/nextflow.config`

Change the `input_dir` and `reference_dir` to `/workspace/gm-<YOUR_NAME>/input` and `/workspace/gm-<YOUR_NAME>/input/references`. 

Deploy GEMmaker using `nextflow-kuberun`:

`nextflow kuberun systemsgenetics/kinc-nf -v task-pv-claim` -C GEMmaker/nextflow.config










# Session 2

## 1. Deploy KINC

Load the input data onto the PVC:

`./kube-load.sh task-pv-claim input`

Deploy KINC using `nextflow-kuberun`:

`nextflow kuberun systemsgenetics/kinc-nf -v task-pv-claim`

**The workflow should take about 10-15 minutes to execute.**

## 5. Retreive and Visualize Gene Co-expression Network

Copy the output of KINC from the PVC to your VM:

`./kube-save.sh task-pv-claim output`

Open Cytoscape. (Applications -> Other -> Cytoscape)

Go to your desktop and open a file browsing window, navigate to the output folder:

`cd ~/Downloads/techex-demo/output/Yeast`

Finally, drag the file `Yeast.coexpnet.txt` from the file browser to Cytoscape!

The network should now be visualized! 






