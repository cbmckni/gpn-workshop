Doing Genomics in the Cloud: From Repository to Result
====

Computational genomics has become a core toolkit in the study of biological systems at the molecular level.  To run genomics workflows, a researcher needs access to advanced computer systems including compute, storage, memory, and networks to move and mine huge genomics datasets.  The Cloud provides scalable compute solutions to run workflows.  In this workshop,  we will demonstrate how one can create a compute cluster in the Cloud using Kubernetes and run containerized genomics workflows.    The Cloud workflows will include pulling high-throughput DNA datasets from the NCBI-SRA data repository, performing reference genome mapping of SRA RNAseq datasets, and building a gene co-expression network.

The Workshop:

We will cover the complete deployment life cycle that enables scientific workflows to be run in the cloud. This includes:
 - Deployment/Access to a Kubernetes(K8s) cluster using Cisco Container Platform(CCP). 
 - Creating a persistent NFS server to store workflow data, then loading a Gene Expression Matrix(GEM) onto it.
 - Pulling genomic data from the NCBI's SRA database.
 - Deploying GEMmaker to create a Gene Expression Matrix
 - Deploying Knowledge Independent Network Construction(KINC), a genomic workflow, to the K8s cloud.
 - Downloading the resulting Gene Coexpression Network(GCN) from the NFS server, then visualizing the network.

There will also be presentations and talks on cutting edge technology and methodology!

Sessions 1 and 2 will be split up by a lunch break. Sessions do not overlap and have different concepts so try to stay for both!

# Session 1

## 0. Prerequisites

The following software is necessary to participate in the demo:
 - Cisco Container Platform CLI
 - kubectl - Kubernetes CLI 
 - Nextflow - Workflow Manager
 - Java
 - Helm

To streamline the workshop, all software has been packaged into a virtual machine that has been replicated for each user. 

An additional requirement is access to the kubernetes clusters that will be used for the workshop.

**If you do not have your CCP cluster credentials(kubeconfig) and access to your personal VM, please let us know.**

### Access Praxis

Navigate to [the Praxis portal](https://dcm.toolwire.com/alai/admin/login.jsp)

Enter your credentials.

Select the class *Running Scientific Workflows on Regional R&E Kubernetes Clusters Workshop*

Select *Learning* at the upper right side of the menu bar.

Select the lab session *Accessing the Cloud through c-Light CCP/IKS Cluster*, when prompted start the live lab.

Once the Jupyter notebook is provisioned, select *Terminal* from the menu to access a Bash terminal from within your VM! 

Finally, please clone this repo to a folder with persistent storage:

`git clone https://github.com/cbmckni/gpn-workshop.git ~/Desktop/classroom`

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

**Only one person per cluster should run this command:**

```
helm install kf stable/nfs-server-provisioner \
--set=persistence.enabled=true,persistence.storageClass=standard,persistence.size=300Gi
```
**Everyone:**

Check that the `nfs` storage class exists:

`kubectl get sc`

Next, deploy a 50Gb Persistant Volume Claim(PVC) to the cluster:

`cd ~/Desktop/classroom/gpn-workshop`

Edit the file and enter your name for your own PVC!

```
metadata:
  name: task-pv-claim-<YOUR_NAME>
```

`kubectl create -f task-pv-claim.yaml`

Check that the PVC was deployed successfully:

`kubectl get pvc`

Give Nextflow the necessary permissions to deploy jobs to your K8s cluster:

```
kubectl create rolebinding default-edit --clusterrole=edit --serviceaccount=default:default 
kubectl create rolebinding default-view --clusterrole=view --serviceaccount=default:default
```

Finally, login to the PVC to get a shell, enabling you to view and manage files:

`nextflow kuberun login -v task-pv-claim-<YOUR_NAME>`

Take note of the pod that gets deployed, use the name when you see **<POD_NAME>**

**This tab is now on your cluster's persistent filesystem.** 

**To continue, open a new tab with File -> New -> Terminal**

## 3. Deploy a Container to Pull Genomic Data

**On your local VM....**

Go to the repo:

`cd ~/Desktop/classroom/gpn-workshop`

Edit the file `sra-tools.yaml`:

```
metadata:
  name: sra-tools-<YOUR_NAME>
  labels:
    app: sra-tools-<YOUR_NAME>
spec:
  containers:
  - name: sra-tools-<YOUR_NAME>
```

Deploy the sra-tools container:

`kubectl create -f sra-tools.yaml`

Get the name of your pod:

`kubectl get pods`

Get a Bash session inside your pod:

`kubectl exec -ti <NAME> -- /bin/bash`

Once inside the pod, navigate to the persistent directory `/workspace`:

`cd /workspace`

Make a folder and enter:

`mkdir -p /workspace/sra-data-<YOUR_NAME> && cd /workspace/sra-data-<YOUR_NAME>`

Initialize SRA-Tools:

`printf '/LIBS/GUID = "%s"\n' `uuidgen` > /root/.ncbi/user-settings.mkfg`

Pull the sequence: `fasterq-dump --split-files SRR5139429`

**While the file is downloading, switch tabs to your other terminal.**

## 4. Configure GEMmaker 

**On your local VM....**

Go to the repo:

`cd ~/Desktop/classroom/gpn-workshop`

Edit the file `gemmaker.yaml`:

```
metadata:
  name: gm-<YOUR_NAME>
  labels:
    app: gm-<YOUR_NAME>
spec:
  containers:
  - name: gm-<YOUR_NAME>
```

Deploy the GEMMaker container:

`kubectl create -f gemmaker.yaml`

**Switch tabs**

## 4. Deploy GEMmaker

**On your local VM's filesystem....**







**Enjoy your lunch! :)**


# Session 2

## 0. Prerequisites

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

Select the lab session *Making Gene Networks with KINC: GEMs to GCNs*, when prompted start the live lab.

Once the Jupyter notebook is provisioned, select *Terminal* from the menu to access a Bash terminal from within your VM! 

Finally, please clone this repo to a folder with persistent storage:

`git clone https://github.com/cbmckni/gpn-workshop.git ~/Desktop/classroom`

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

**If you were not present for the first session:**

Check that the `nfs` storage class exists:

`kubectl get sc`

Next, deploy a 50Gb Persistant Volume Claim(PVC) to the cluster:

`cd ~/Desktop/classroom/gpn-workshop`

Edit the file and enter your name for your own PVC!

```
metadata:
  name: task-pv-claim-<YOUR_NAME>
```

`kubectl create -f task-pv-claim.yaml`

Check that the PVC was deployed successfully:

`kubectl get pvc`

**Everyone:**

To view and manage files on the cluster:

`nextflow kuberun login -v task-pv-claim-<YOUR_NAME>`

Take note of the pod that gets deployed, use the name when you see **<POD_NAME>**

**To continue, open a new tab with File -> New -> Terminal**

## 2. Deploy KINC

**On your local VM....**

Go to the repo: 

`cd ~/Desktop/classroom/gpn-workshop`

Edit the file `nextflow.config`:

```
params {
    input {
        dir = "/workspace/gcn-<YOUR_NAME>/input"
        emx_txt_files = "*.emx.txt"
        emx_files = "*.emx"
        ccm_files = "*.ccm"
        cmx_files = "*.cmx"
    }

    output {
        dir = "/workspace/gcn-<YOUR_NAME>/output"
    }
```

Load the input data onto the PVC:

`kubectl exec <POD_NAME> -- bash -c "mkdir -p /workspace/gcn-<YOUR_NAME>"`

`kubectl cp "input" "<POD_NAME>:/workspace/gcn-<YOUR_NAME>"`

Deploy KINC using `nextflow-kuberun`:

`nextflow kuberun -C nextflow.config systemsgenetics/kinc-nf -v task-pv-claim-<YOUR_NAME>`

**The workflow should take about 10-15 minutes to execute.**

## 3. Retreive and Visualize Gene Co-expression Network

Copy the output of KINC from the PVC to your VM:

`cd ~/Desktop/classroom/gpn-workshop`

```
kubectl exec <POD_NAME> -- bash -c \
"for f in \$(find /workspace/gcn-<YOUR_NAME>/output/Yeast -type l); do cp --remove-destination \$(readlink \$f) \$f; done"
```

`kubectl cp "<POD_NAME>:/workspace/gcn-<YOUR_NAME>/output/Yeast" "Yeast"`

Open Cytoscape. (Applications -> Other -> Cytoscape)

Go to your desktop and open a file browsing window, navigate to the output folder:

`cd ~/Desktop/classroom/gpn-workshop/Yeast`

Finally, drag the file `Yeast.coexpnet.txt` from the file browser to Cytoscape!

The network should now be visualized! 






