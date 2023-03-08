# Lab Deployment in Azure

Within this repo you will find an ARM template that deploys a virtual machine within Azure and then helps you build out a small lab environment within that virtual machine that can be used to replicate an on-prem solution you can use to set up Azure Backup, Azure Site Recovery, Azure Migrate, etc. 

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fweeyin83%2FLab-Deployment-in-Azure%2Fmaster%2FVMdeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fweeyin83%2FLab-Deployment-in-Azure%2Fmaster%2FVMdeploy.json" target="_blank">
    <img src="http://armviz.io/visualizebutton.png"/>
</a>

<!-- TOC start -->
- [Lab Deployment in Azure](#lab-deployment-in-azure)
  * [Azure VM Details](#azure-vm-details)
  * [Lab Details](#lab-details)
  * [Setup - IP Configuration](#setup-ip-configuration)
  * [Setup - Azure VM Host Credentials](#setup-azure-vm-host-credentials)
  * [Setup - Windows Updates](#setup-windows-updates)
  * [Credits](#credits)
<!-- TOC end -->

## Azure VM Details
This lab is all hosted within an Azure VM.  The Azure VM allows for nested virtualisation. 

The VM has Windows Server 2022 installed and Hyper-V enabled. The template deploys the lab as a Standard D8s v3 (8 vcpus, 32 GiB memory) VM.  The recommendation would be that once you have deployed the lab to scale the Azure VM to a size that makes sense for your intended purpose.  If you are you going to deploy more virtual machines to it then you need to make it larger. 

## Lab Details

The ARM template will deploy a virtual machine within Azure and then install Hyper-V within that virtual machine.  It will also download some VHD files and deploy five servers onto that Hyper-V environment. 

The servers are all joined to the domain **tailwindtraders.org**. The login name for the admin of the domain is **tailwindtraders\administrator** and the password is: **Password**: demo@pass123

|  VM Name  | Operating System   | Purpose   |  Processor | Memory | Comments |
|---|---|---|---|---|---|
|  AD01 |  Windows Server 2008 R2 | Domain Controller, DHCP, DNS   |  1 | 2GB | |
|  FS01 | Windows Server 2012 R2   | File Server   |   1 | 2GB | The file share is on the C drive, there are some sample files and folders. You can use this to lab out some Azure File shares. |
| SQL01  | Windows Server 2016   | SQL Server  |  2 | 8GB | The SQL installation is on the C drive, not best practice, but okay for a lab and maybe identifying improvements that can be made.  |
| WEB01  | Windows Server 2016   | Web front end server  |   1 | 2GB | IIS is installed on this server. |
| WEB02  | Ubuntu Server 22.04.2   | ?? |   1 | 4GB | |

FS01, SQL01, WEB01 and WEB02 were all patched at the start of March 2023.  AD01 wouldn't patch. 

The AD01 server is the domain controller, DHCP and DNS server.  It should give out IP addresses to the servers when imported, but if you have any issues there are details on how to set static IPs to them below. 

The FS01 server is a file server.  It has the file server role installed on it, it also has the FIle Server Resource Manager (FSRM) installed.  It's not an ideal setup as the files are all stored within the C drive but, there are files can you can use it to assess with Azure Migrate or look to set up an Azure File sync demo. 

SQL01 is the database server, it has the SQL server role installed and the SQL server management tools installed. 

WEB01 is an IIS Server. 

WEB02 is a Ubuntu server. 

None of the servers are activated with licenses, if you have an MSDN subscription you can get product keys to activate the servers or run them as is with a trial license. 
 
## Setup - IP Configuration

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

- Log into SQL01 and set the server to have a static IP configuration as follows:
    - IP Address: 192.168.0.4
    - Subnet Mask: 255.255.255.0
    - Default Gateway: 192.168.0.1
    - Preferred DNS: 192.168.0.2
    - Alternative DNS: 8.8.8.8
    
- Log into WEB01 and set the server to have a static IP configuration as follows:
    - IP Address: 192.168.0.5
    - Subnet Mask: 255.255.255.0
    - Default Gateway: 192.168.0.1
    - Preferred DNS: 192.168.0.2
    - Alternative DNS: 8.8.8.8
    
- Within WEB02 run the following commands:
    - sudo apt-get update
    - sudo apt-get upgrade -y
    - sudo apt-get install "linux-cloud-tools-$(uname -r)" -y
    - sudo apt-get install --install-recommends linux-tools-virtual-lts -y 

## Setup - Azure VM Host Credentials

To log onto the Azure VM the credentials are: 

**Username**: mcwadmin
**Password**: demo@pass123

_It is recommend that you change this._

## Setup - Windows Updates

The VMs that are deployed during this lab environment were patched in May 2021, if you are deploying after this time and you want to get your system up to date there is a script on each VM that you can use to do that, C:\Scripts\patches.ps1 will install a PowerShell module and then check for updates that are missing and install them.  You can run this script and let it run while you carry out other tasks. 


## Credits
Written by: Sarah Lean

Find me on:

* My Blog: https://www.techielass.com
* Twitter: https://twitter.com/techielass
* LinkedIn: http://uk.linkedin.com/in/sazlean
* Github: https://github.com/weeyin83
