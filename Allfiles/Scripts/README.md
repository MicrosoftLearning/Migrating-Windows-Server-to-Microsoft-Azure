# Lab Deployment in Azure

Within this repo you will find an ARM template that deploys a virtual machine within Azure and then helps you build out a small lab environment within that virtual machine that can be used to replicate an on-prem solution you can use to set up Azure Backup, Azure Site Recovery, Azure Migrate, etc. 

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fpolichtm%2FNested_Hyper-V_in_Azure%2Fmain%2FVMdeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fpolichtm%2FNested_Hyper-V_in_Azure%2Fmain%2FVMdeploy.json" target="_blank">
    <img src="http://armviz.io/visualizebutton.png"/>
</a>

# Table of contents

- [Lab Deployment in Azure](#lab-deployment-in-azure)
  - [Azure VM Details](#azure-vm-details)
  - [Azure VM Host Credentials](#azure-vm-host-credentials)
  - [Lab Details](#lab-details)
  - [Lab virtual machines IP Information](#lab-virtual-machines-ip-information)
  - [Lab VM Windows Updates](#lab-vm-windows-updates)
  - [Lab Use Cases](#lab-use-cases)
  - [Tutorials](#tutorials)
  - [Credits](#credits)

## Azure VM Details
This lab is all hosted within an Azure VM.  The Azure VM allows for nested virtualisation. 

The VM has Windows Server 2022 installed and Hyper-V enabled. The template deploys the lab as a Standard D8s v3 (8 vcpus, 32 GiB memory) VM.  The recommendation would be that once you have deployed the lab to scale the Azure VM to a size that makes sense for your intended purpose.  If you are you going to deploy more virtual machines to it then you need to make it larger. 

## Azure VM Host Credentials

To log onto the Azure VM the credentials are: 

**Username**: rootadmin
**Password**: demo@pass123

_It is recommend that you change this._

## Lab Details

The ARM template will deploy a virtual machine within Azure and then install Hyper-V within that virtual machine.  It will also download some VHD files and deploy five servers onto that Hyper-V environment. 

The servers are all joined to the domain **tailwindtraders.org**. The login name for the admin of the domain is **tailwindtraders\administrator** and the password is: **Password**: Pa55w.rd1234

| VM Name | Operating System | Purpose | Processor | Memory | Comments |
|---|---|---|---|---|---|
| AD01 | Windows Server 2008 R2 | Domain Controller, DHCP, DNS |  1 | 2GB | |
| FS01 | Windows Server 2012 R2 | File Server | 1 | 2GB | The file share is on the C drive, there are some sample files and folders. You can use this to lab out some Azure File shares |
| WEB01 | Windows Server 2016 | Web front end server | 1 | 2GB | IIS is installed on this server |

FS01 and WEB01 were all patched at the start of March 2023. Patches for AD01 are no longer available.

The AD01 server is the domain controller, DHCP and DNS server. It should give out IP addresses to the servers when imported, but if you have any issues there are details on how to set static IPs to them below. 

The FS01 server is a file server.  It has the file server role installed on it, it also has the FIle Server Resource Manager (FSRM) installed.  It's not an ideal setup as the files are all stored within the C drive but, there are files can you can use it to assess with Azure Migrate or look to set up an Azure File sync demo. 

None of the servers are activated with licenses, if you have an MSDN subscription you can get product keys to activate the servers or run them as is with a trial license. 
 
## Lab virtual machines IP Information

Once the servers are deployed you need to carry out the following configuration within the servers manually: 

- Log into AD01 and set the server to have a static IP configuration as follows: 
    - IP Address: 192.168.0.2
    - Subnet Mask: 255.255.255.0
    - Default Gateway: 192.168.0.1
    - Preferred DNS: 127.0.0.1
    - Alternative DNS: 8.8.8.8

- Log into FS01 and set the server to have a static IP configuration as follows:
    - IP Address: 192.168.0.3
    - Subnet Mask: 255.255.255.0
    - Default Gateway: 192.168.0.1
    - Preferred DNS: 192.168.0.2
    - Alternative DNS: 1.1.1.1

- Log into WEB01 and set the server to have a static IP configuration as follows:
    - IP Address: 192.168.0.5
    - Subnet Mask: 255.255.255.0
    - Default Gateway: 192.168.0.1
    - Preferred DNS: 192.168.0.2
    - Alternative DNS: 8.8.8.8
    
## Lab VM Windows Updates

If you are deploying this lab after March 2023 and want to update patches, you can initial this manually.  Alternatively there is a script on the file share **\\FS01\TT-Files\ITScripts\Updates.ps1** that can be ran and force patching.

## Lab Use Cases
This lab has been designed to try and simulate an on-prem infrastructure, with common servers you'd encounter or need.  A domain controller, file server, SQL database, and some web servers. 

You can spin up this lab and try out some of the following:
* Install Azure Arc on the servers and try to extend Azure services to them
* Configure Azure File Sync and leverage that within a "production" like environment
* Deploy Azure Migrate and assess the workloads _(note you will have to treat the servers like physical servers as you don't have access to assess the top Hyper-V layer)_
* Any other use cases you might have... 

## Tutorials

Some tutorials on how to use this lab have been created: 

* [Lab deployment steps](Tutorials/lab-deployment.md)
* [Use Azure File Shares with the lab](Tutorials/file-server.md)

## Credits
The orginal content written by: Sarah Lean