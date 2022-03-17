---
title: "Build your DevOps Environment!"
date:  2022-02-08T00:00:00+02:00
draft: true
categories: ["NetApp","Docker","Kubernetes","Trident","DevOps","Azure","GCP","AstraControl"]
banner: /na_devops.png
layout: post
---

The LinkedIn post "NetApp in a Cloud DevOps Context" showed how an optimized Kubernetes environment can look in the cloud. This series of articles now show how to build such an environment in an automated way.

There are or will be four parts to this series.

   Part 1: Planning
   Part 2: Setting up the required infrastructure
   Part 3: Activation of the backup
   Part 4: Monitoring and optimization of the environment
   
The cloud providers Microsoft Azure and Google Cloud Platform are used to set up the test environments. However, this is only because NetApp's Astra Control Service does not support Amazon Web Service. As soon as it is available, this will be made up for immediately.

Before we start, a short note: The scripts used here can be used for testing purposes. However, I assume no liability for the use in production environments. In addition, these also generate cloud resources, which will generate costs!

Deploying scripts is easy. However, you need to think about several points in advance:

   Setup of the network in the cloud - IP addresses, firewall rules, etc.
   Access rights to cloud resources
   Licenses
To set up a test environment, you need to register the NetApp resource provider in Azure. As soon as the registration is completed, a NetApp account must be created in Azure, in which the capacity pool and volumes will later be created.

ˋˋˋ
az account set --subscription <subscriptionId
az provider register --namespace Microsoft.NetApp --wait
ˋˋˋ

As soon as Azure NetApp Files has been released, a NetApp account is created in a resource group that can be used for the test environment. This resource group is then also used later on.

ˋˋˋ
az netappfiles account create --resource-group <rg-name> --location <location> --account-name <anf-account-name>
ˋˋˋ

With Google Cloud, you select the Cloud Volume Service in the Market Place and accept the EULA. Before you can provision storage, however, you have to establish communication with the managed NetApp. This can be done conveniently via the GCP Cloud Console.

ˋˋˋ
## CVS "default"
gcloud --project=<gcp-project> compute addresses create netapp-addresses-sds-default --global --purpose=VPC_PEERING --prefix-length=25 --network=default --no-user-output-enable

gcloud --project=<gcp-project> services vpc-peerings connect --service=cloudvolumesgcp-sds-api-network.netapp.com --ranges=netapp-addresses-sds-default --network=default --no-user-output-enabled

gcloud --project=<gcp-project> compute networks peerings update netapp-sds-nw-customer-peer --network=default --import-custom-routes --export-custom-routes


## CVS Performance
gcloud --project=<gcp-project> compute addresses create netapp-addresses-default --global --purpose=VPC_PEERING --prefix-length=24 --network=default --no-user-output-enabled

gcloud --project=<gcp-project> services vpc-peerings connect --service=cloudvolumesgcp-api-network.netapp.com --ranges=netapp-addresses-default --network=default --no-user-output-enabled

gcloud --project=<gcp-project> compute networks peerings update netapp-cv-nw-customer-peer --network=default --import-custom-routes --export-custom-routesd

ˋˋˋ

Now that the most important preparations have been made, you can start provisioning storage. But this will be described in my next article first.