---
title: "FSxN & ROSA = perfect Match"
date:  2023-10-05T00:00:00+02:00
draft: false
categories: ["NetApp","Docker","Kubernetes","Trident","DevOps","AWS","RedHat","AstraControl","Openshift"]
banner: "https://github.com/project-epicshit/project-epicshit.github.io/assets/36699674/6c80b9b6-c0c0-415a-9606-178ce4f9b743"
layout: post
---

# FSx for NetApp ONTAP and RedHat Openshift on AWS a perfect match

## Solution Overview
﻿ROSA is one of the latest services offered by RedHat on Amazon Web Services (AWS). This means that it is now possible to use the same products that are familiar from the DataCenter world in the public cloud, with the advantage that they are now available as a managed service. ROSA is enhanced with Amazon FSx for NetApp ONTAP (FSxN). By using FSxN, Openshift gets more stability and redundancy across Availability Zones in the cloud.

#### Architecture
﻿![grafik](https://github.com/project-epicshit/project-epicshit.github.io/assets/36699674/6c80b9b6-c0c0-415a-9606-178ce4f9b743)


### Why ROSA with FSxN
The combination of RedHat and NetApp products provides an HA solution for running Kubernetes workloads across availability zones (AZs). From a container perspective, there is always the possibility of restarting on another node in another zone. With NetApp Storage for Persistent Data there is an HA solution which managed the failover.

NetApp Astra Trident is used to connect the storage to ROSA. As a CSI driver, Trident is responsible for provisioning the PVC on FSxN and mounting it directly into the container

### Prerequisites
	- AWS credentials that provides the necessary permissions to create the resources (VPC, Compute, etc)
	- AWS account has been linked with RedHat (see ROSA documentation)

### Connectivity
The most important part of this solution is that connectivity can be established between ROSA nodes and FSxN SVM. It is necessary to pay attention to VPC, subnet, routing table etc..
There are many ways to accomplish this, but for the sake of simplicity, in this guide everything is configured in the same VPC and subnets.

## Steps in this guide
1. Creating ROSA
2. Deploying  FSx for NetApp ONTAP
3. Installing and configuring Trident

### Creating a  ROSA Cluster with 
There minumum of three ways to deploy a ROSA cluster. 
	- On the CLI with the command "rosa"
	- WebUI => https://console.redhat.com
	- Terraform

This guide will use the rosa CLI [^1]

```bash
rosa create cluster
```

following inputs are required - for the most questions I used the default settings!

```bash
I: Enabling interactive mode
? Cluster name: **democluster**
? Deploy cluster with Hosted Control Plane (optional): **No**
? Create cluster admin user: **Yes**
? Username: **demoadmin**
? Password: [? for help] **************
? Deploy cluster using AWS STS: **Yes**
W: In a future release STS will be the default mode.
W: --sts flag will not be necessary if you wish to use STS.
W: --non-sts/--mint-mode flag will be necessary if you do not wish to use STS.
? OpenShift version: **4.12.35**
? Configure the use of IMDSv2 for ec2 instances optional/required (optional): 
I: Using arn:aws:iam::XXXXXXXXXXXX:role/ManagedOpenShift-Installer-Role for the Installer role
I: Using arn:aws:iam::XXXXXXXXXXXX:role/ManagedOpenShift-ControlPlane-Role for the ControlPlane role
I: Using arn:aws:iam::XXXXXXXXXXXX:role/ManagedOpenShift-Worker-Role for the Worker role
I: Using arn:aws:iam::XXXXXXXXXXXX:role/ManagedOpenShift-Support-Role for the Support role
? External ID (optional): 
? Operator roles prefix: **democluster-s4i6**
? Deploy cluster using pre registered OIDC Configuration ID: **No**
W: No OIDC Configuration found; will continue with the classic flow.
? Tags (optional): 
? Multiple availability zones (optional): **Yes**
? AWS region: **us-east-1**
? PrivateLink cluster (optional): **No**
? Machine CIDR: **10.0.0.0/16**
? Service CIDR: **172.30.0.0/16**
? Pod CIDR: **10.128.0.0/14**
? Install into an existing VPC (optional): **No**
? Select availability zones (optional): **No**
? Enable Customer Managed key (optional): **No**
? Compute nodes instance type: **m5.xlarge**
? Enable autoscaling (optional): **No**
? Compute nodes: **3**
? Default machine pool labels (optional): 
? Host prefix: **23**
? Enable FIPS support (optional): **No**
? Encrypt etcd data (optional): **No**
? Disable Workload monitoring (optional): **Yes**

I: Creating cluster 'democluster'
```

After filling out all the questions you will get an overview what will be deployed now!

```
I: To create this cluster again in the future, you can run:
   rosa create cluster ...
I: To view a list of clusters and their status, run 'rosa list clusters'
I: Cluster 'democluster' has been created.
I: Once the cluster is installed you will need to add an Identity Provider before you can login into the cluster. See 'rosa create idp --help' for more information.

Name:                       democluster
ID:                         26m2gh2eplvr1m7ffaaku4mlmtkr63v6
External ID:                
Control Plane:              Customer Hosted
OpenShift Version:          
Channel Group:              stable
DNS:                        Not ready
AWS Account:                XXXXXXXXXXX
API URL:                    
Console URL:                
Region:                     us-east-1
Multi-AZ:                   true
Nodes:
 - Control plane:           3
 - Infra:                   3
 - Compute:                 3
Network:
 - Type:                    OVNKubernetes
 - Service CIDR:            172.30.0.0/16
 - Machine CIDR:            10.0.0.0/16
 - Pod CIDR:                10.128.0.0/14
 - Host Prefix:             /23
STS Role ARN:               arn:aws:iam::XXXXXXXXXXX:role/ManagedOpenShift-Installer-Role
Support Role ARN:           arn:aws:iam::XXXXXXXXXXX:role/ManagedOpenShift-Support-Role
Instance IAM Roles:
 - Control plane:           arn:aws:iam::XXXXXXXXXXX:role/ManagedOpenShift-ControlPlane-Role
 - Worker:                  arn:aws:iam::XXXXXXXXXXX:role/ManagedOpenShift-Worker-Role
Operator IAM Roles:
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-machine-api-aws-cloud-credentials
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-cloud-credential-operator-cloud-crede
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-image-registry-installer-cloud-creden
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-ingress-operator-cloud-credentials
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-cluster-csi-drivers-ebs-cloud-credent
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-cloud-network-config-controller-cloud
Managed Policies:           No
State:                      waiting (Waiting for OIDC configuration)
Private:                    No
Created:                    Oct  5 2023 08:39:36 UTC
User Workload Monitoring:   disabled
Details Page:               https://console.redhat.com/openshift/details/s/2WKxfWyuNMhPGuwp0SqdW95Q5zk
OIDC Endpoint URL:          https://rh-oidc.s3.us-east-1.amazonaws.com/26m2gh2eplvr1m7ffaaku4mlmtkr63v6 (Classic)

I: 
Run the following commands to continue the cluster creation:

	rosa create operator-roles --cluster democluster
	rosa create oidc-provider --cluster democluster

I: To determine when your cluster is Ready, run 'rosa describe cluster -c democluster'.
I: To watch your cluster installation logs, run 'rosa logs install -c democluster --watch'.

```

For creating the operator-roles and the oidc-provider I used the "auto" creation mode!

﻿![grafik](https://github.com/project-epicshit/project-epicshit.github.io/assets/36699674/9850ce16-6c30-45e5-bfcc-d4501ddbe18f)

It takes a while for the cluster to be deployed. RedHat gives a time range of 30-60 minutes!


## Creating FSx

### Links / Footnotes
[ROSA - Creating Cluster Guide](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa-sts-creating-a-cluster-quickly.html)

[^1]: ROSA CLI



