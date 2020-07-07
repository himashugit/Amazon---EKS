# Amazon---EKS

## AWS EKS – Elastic Kubernetes Service ##

> Amazon Elastic Kubernetes Service (Amazon EKS) is a managed service that makes it easy for you to
run Kubernetes on AWS without needing to stand up or maintain your own Kubernetes control plane.

> Kubernetes is an open-source system for automating the deployment, scaling, and management of
containerized applications.

> Amazon EKS runs Kubernetes control plane instances across multiple Availability Zones to ensure high
availability. Amazon EKS automatically detects and replaces unhealthy control plane instances, and it
provides automated version upgrades and patching for them.

> Amazon EKS is also integrated with many AWS services to provide scalability and security for your
 applications, including the following:
 
•  Amazon ECR for container images
•  Elastic Load Balancing for load distribution
•  IAM for authentication
•  Amazon VPC for isolation

> Amazon EKS runs a single tenant Kubernetes control plane for each cluster, and control plane
infrastructure is not shared across clusters or AWS accounts.

> This control plane consists of at least two API server nodes and three etcd nodes that run across three
Availability Zones within a Region. Amazon EKS automatically detects and replaces unhealthy control
plane instances, restarting them across the Availability Zones within the Region as needed. Amazon EKS
leverages the architecture of AWS Regions in order to maintain high availability. Because of this, Amazon EKS is able to offer an SLA for API server endpoint availability.

> Amazon EKS uses Amazon VPC network policies to restrict traffic between control plane components to
within a single cluster. Control plane components for a cluster cannot view or receive communication
from other clusters or other AWS accounts, except as authorized with Kubernetes RBAC policies.
This secure and highly available configuration makes Amazon EKS reliable and recommended for
production workloads.

 
## There are 2 ways to start your K8s cluster on AWS
1.	Command Line – eksctl
2.	Management Console – Manually create from the console or using cloud formation console


## Install AWS Cli and check and confirm with version command.
https://awscli.amazonaws.com/AWSCLIV2.msi

2.  Run the downloaded MSI installer and follow the onscreen instructions. By default, the AWS CLI install c:/Programfiles/

 


> Create an IAM user and configure AWS, put below detail, more info https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html

 


  Also download eksctl to work with EKS cluster

> Once you downloaded you can write a yaml based config file and deploy this to AWS

 

Use -           **eksctl create -f cluster.yml**

& this will take 5-10 min to deploy and launch. Behind the scene Cloud Formation stack will do the setup for you, networking,SG, etc….

 

> You will see 3 nodes are deployed by EKS for you using Cloud Formation stack

 

> Now lets see using kubectl command to interact with the cluster and nodes

 

> If you want to see what node contain. You can use describe command to see

Use kubectl describe node (nodeid)



 
> Let’s create a namespace to mange the resources(rs/deployment) and policies which we are going to set on the nodes in it with a secure token

 


> Now Let’s create a deployment with a httpd image at my docker hub account


**Kubectl create deployment apachewebserver1 –image=himashu07/apacheweb:latest**

 

You can check your POD should be running using – kubectl get pods

 

IF you need to check on which node this pod has been launched in AWS EKS. Check out Node IP

Kubectl get pods -o wide

 

> You can scale your deployment so that you have scalability of your pods, Once you scale your deployment, behind the scene EKS Master deploy the pods on differen nodes so that you have all time availability of your pods. Scheduler is the service on the Master which will place the pods on separate nodes.

**Kubectl scale deployment apachewebserver1 –replicas=4**

 

> Let’s expose the deployment(port 80) so that apache web server can be access outside world and this is backed by LB to share web server load. If you check the service you can   grab the external IP and access it.
**Kubectl expose deployment apachewebserver1 –type=LoadBalancer –port=80**

 

> Let’s create a new deployment with a new apache web server image

**kubectl create deployment apachewebserver --image=vimal13/apache-webserver-php**

 

> I will create a new index.php page and replace the existing index.php folder inside this pod. My page is running @ cd /var/www/html and I will copy a new page here. Once you deploy a new page will be display to you at your LB external IP

**kubectl cp index.php apachewebserver-86bb9d8c54-vqzzp:/var/www/html/index.php**

 

> But this storage is ephemeral(temporary), as soon as this pod down you will lose this page
> So, I need a permanent storage/page for this. So, I am going to attach a persistent storage to it.
> Storage options are Block. File, object. I’ll attach block storage to this POD and mount this to pod web page location. Storage attach is external hard disk so even if pod       down data is there. In AWS we have EBS so we will connect EBS volume to this POD.

> I’ll create an yml file for PVC storage on EKS cluster. And deploy this pvc
**Kubectl create -f pvc.yml**
 

 

> But you will see PV is not created. Why ? because storage class is the one who create HDD. Acc to storage class has one annotation, that volume binding mode wait for first consumer. Anyone uses this gp2 they get storage from ebs. So once pod request for this they will create a PV and attach it. I’ll edit the deployment and attach the persistentvolume to this PVC


 

> Let’s now create a new Storage Class 

 



> Once Storage Class is created, we need to attach this to PVC so that it uses new IO StorageClass
 

> Now you have PV created and permanent as reclaim policy is retaining. 
  You can mount this to any other node as well.

