---
lab:
    title: 'Migrate Windows Server file services by using Storage Migration Service'
    module: 'TW-1002.1-2024: Migrating Windows Server to Microsoft Azure'
---

# TW-1002.1-2024: Migrating Windows Server to Microsoft Azure
# Workshop: Migrating Windows Server to Microsoft Azure
# Lab 3: Migrate Windows Server file services by using Storage Migration Service

Total workshop estimated time: 210 minutes

## Objectives

After completing this workshop, you will be able to:

- Migrate on-premises Windows Server resources to Azure-based management model by using Azure Arc and Azure Policy
- Migrate on-premises Windows Server resources to Azure-based unified monitoring model by using Azure Monitor
- Migrate on-premises Windows Server resources to Azure-based security model by using Microsoft Defender for Cloud
- Migrate on-premises Windows Server resources to Azure-based patch management model by using Azure Update Manager
- Plan and assess migration of Server resources by using Azure Migrate
- Migrate Windows Server resources by using Azure Migrate
- Migrate on-premises Windows Server-based file servers to Azure VMs by using Storage Migration Service

## Lab 3: Migrate Windows Server file services by using Storage Migration Service

Estimated Time: 60 minutes

> **Note:** This scenario, for practical reasons, is implemented by leveraging the Hyper-V nesting support in Azure VMs. The on-premises environment is simulated by using nested Windows Server VMs running in an Azure VM with the Hyper-V server role installed. This minimizes time it takes to complete individual stages of the migration process. However, the migration process illustrated in this lab would not differ (from the functional standpoint - leaving the latency and bandwidth implications aside) if performed by using an actual on-premises environment.

## Objectives

In this lab, you will:

- Prepare for the migration
- Perform migration and cutover by using Storage Migration Service

> **Note:** You will leverage the virtual network peering and routing configuration which you implemented prior to carrying migration by using Azure Migrate in the previous lab. This will allow you to migrate a file share from the **FS01** Hyper-V virtual machine to an automatically provisioned Azure VM.

### Exercise 1: Prepare for the migration

In this exercise, you will 

. The exercise will involve the following on-premises servers:

- An Azure Migrate appliance serving the role of the orchestration server
- FS01 serving the role of the source server

> **Note:** Azure Migrate Appliance was chosen strictly for the sake of simplicity and convenience. It is a Hyper-V VM running Windows Server 2022, which you installed in earlier in this workshop. While, in general, this appliance would be used specifically for Azure Migrate-based purposes, it is suitable to serve as the Storage Migration Service orchestrator in a lab scenario. More importantly, its configuration and usage in this context are precisely the same as they would be in case of a dedicated orchestrator. 

> **Note:** The target server will be provisioned automatically during the migration by leveraging the built-in functionality of Storage Migration Service.

The exercise consists of the following tasks:

- Task 1: Configure the orchestrator server
- Task 2: Configure the source server

#### Task 1: Configure the orchestrator server

In this task, you will configure the Azure Migrate appliance as the Storage migration Service orchestration server. You will start by joining the appliance to the domain hosting the server you intend to migrate. Following the domain join, you will install on the virtual appliance Windows Admin Center and register it with Azure.

> **Note:** Storage Migration Service is a Windows Admin Center-based tool. The reason you need to register it with Azure in this case is because you will leverage the feature that automatically provisions the target Azure VM to which on-premises file shares will be migrated. 

1. If needed, connect to the server SEA-ADM1 and sign in as TAILWINDTRADERS\\Administrator and re-establish Remote Desktop session to the **hypervhost** Azure VM.
1. Within the Remote Desktop session to **hypervhost**, switch to the **Server Manager** window, select the **Tools** menu header, and, in the drop-down menu, select **Hyper-V Manager**. 
1. In the **Hyper-V Manager** console, in the list of virtual machines, select the Azure Migrate Appliance VM and, in the lower section of the **Actions** pane, select **Connect**.
1. If needed, in the **Virtual Machine Connection** window, in the **Actions** menu, select **Ctrl+Alt+Delete** and sign in by using the local **Administrator** user name and the password you set when setting up the appliance. 
1. In the **Virtual Machine Connection** window, start a Windows PowerShell ISE session and open the script pane.
1. In the **Virtual Machine Connection** window, right-click the start menu icon and select **Run**.
1. In the **Run** text box, enter **sysdm.cpl** and select **OK**.
1. In the **System Properties** window, on the **Computer Name** tab, select **Change**.
1. In the **Computer name** text box, enter **AZMIG01**.
1. Select the **Domain** option, in the text box enter **tailwindtraders.org**, and then select **OK**.
1. When prompted, in the **Computer Name/Domain Changes** dialog box, specify the credentials of the **TAILWINDTRADERS\\Administrator** account and then select **OK**.
1. When presented with a dialog box confirming the changes, select **OK**, and then select **OK** again.
1. In the **System Properties** window, select **Close** and then select **Restart Now**.
1. Following restart, sign in back by using the **TAILWINDTRADERS\\Administrator** credentials.

   > **Note:** Next, you will install Windows Admin Center.

1. In the **Virtual Machine Connection** window, start a Windows PowerShell ISE session and open the script pane.
1. In the Windows PowerShell ISE script pane, enter the following script that installs Windows Admin Center and select the green arrow icon in the toolbar to execute it:

   ```powershell
   Invoke-WebRequest 'https://aka.ms/WACDownload' -OutFile "$pwd\WAC.msi"
   $msiArgs = @("/i", "$pwd\WAC.msi", "/qn", "/L*v", "log.txt", "SME_PORT=443", "SSL_CERTIFICATE_OPTION=generate")
   Start-Process msiexec.exe -Wait -ArgumentList $msiArgs
   ```

   > **Note:** Wait for the installation of Windows Admin Center to complete. The installation should take less than 1 minute. It provisions the Windows Admin Center gateway component accessible via `https://localhost`, secured by a self-signed certificate valid for 60 days.

   > **Note:** Next, you need to register Windows Admin Center with Azure.

1. Within the Remote Desktop session, start a web browser and navigate to the `https://localhost` page.

   > **Note:** Ensure that you use the **localhost** name, rather than the actual server name.

1. When presented with the warning **Your connection isn't private**, select **Advanced** and then select **Continue to localhost (unsafe)**.
1. If prompted to authenticate, sign in as Administrator.
1. If needed, close the pane confirming the successful installation, wait for the updates of the Windows Admin Center extensions to complete, and acknowledge their completion.
1. In Windows Admin Center, on the **All connections** page, select the cogwheel icon in the upper right corner of the page.
1. On the **Settings \| Account** page, in the **Azure Account** section, select **Register with Azure** and then, on the **Register with Azure** pane, select **Register**.
1. On the **Get started with Azure in Windows Admin Center** pane, in step 2 of the registration process, select **Copy** to copy the registration code into Clipboard.
1. Select the link next to the **Enter the code** text in step 3 of the registration process.
 
   > **Note:** This will open another tab in the web browser window displaying the Enter code page.

1. In the **Enter code** pane, paste the code you copied into Clipboard and select **Next**.
1. When prompted to sign in, provide the same Microsoft Entra ID credentials you have been using so far in the lab to authenticate access to the Azure subscription.
1. When prompted to confirm the question **Are you trying to sign in to Windows Admin Center?**, select **Continue**.
1. Verify that the sign in was successful and close the newly opened tab of the web browser window.
1. Back on the **Get started with Azure in Windows Admin Center** pane, in step 4 of the registration process, ensure that the **Use existing** option is selected and, in the **Microsoft Entra application ID** text box, enter the **App ID** string listed on the **Resources** tab of the lab web interface and select **Connect**.

   > **Note:** This application has been pre-provisioned in the Microsoft Entra ID tenant associated with the Azure subscription you are using in this lab.

1. On the Get started with Azure in Windows Admin Center blade, in step 5 of the registration process, select **Sign in**.
1. If prompted to sign in, provide the same Microsoft Entra ID credentials you have been using so far in the lab to authenticate access to the Azure subscription.
1. If prompted, in the **Permissions requested** pop-up window, review the permissions required by the application and select **Accept**.
1. On the **Register with Azure** pane of the Windows Admin Center page, verify that the registration was successful.

#### Task 2: Configure the source server

In this task, you will implement prerequisites for the Storage Migration Service-based migration by using Windows Admin Center. This will include installing Windows Management Framework (WMF) 5.1 on the FS01 VM running Windows Server 2012 R2 (which is required to manage it by using Windows Admin Center) and using Windows Admin Center to configure its Windows Firewall with Advanced Security rules.

1. Within the Remote Desktop session to **hypervhost**, switch to the **Hyper-V Manager** console, in the list of virtual machines, select **FS01** and, in the lower section of the **Actions** pane, select **Connect**.
1. In the **Virtual Machine Connection** window, in the **Actions** menu, select **Ctrl+Alt+Delete**.
1. Sign in to **FS01** by using **TAILWINDTRADERS\\Administrator** user name and **Pa55w.rd1234** password. 
1. In the **Virtual Machine Connection** window, start a Windows PowerShell ISE session and open the script pane.
1. In the Windows PowerShell ISE script pane, enter the following script that re-enables the Windows Update service, starts it, downloads the WMF 5.1 installer, and invokes its installation.

   ```powershell
   $serviceName = 'wuauserv'
   Set-Service -Name $serviceName -StartupType Automatic
   Start-Service -Name $serviceName

   $url = 'https://download.microsoft.com/download/6/F/5/6F5FF66C-6775-42B0-86C4-47D41F2DA187/Win8.1AndW2K12R2-KB3191564-x64.msu'
   $tempDir = 'C:\Temp\'
   $msuPath = "$tempDir\WMF5.1.msu"
   
   [Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12

   New-Item -Path $tempDir -ItemType Directory -Force
   Invoke-WebRequest -Uri $url -OutFile $msuPath
   Start-Process -FilePath "wusa.exe" -ArgumentList "$msuPath /quiet" -Wait
   ```

   > **Note:** Wait until the installation completes. This might take about 5 minutes. Following the installation, the operating system will automatically reboot. 

1. Following the restart of the FS01 operating system, in the Remote Desktop session, switch to the Virtual Machine Connection to the Azure Migrate Appliance VM, in the Virtual Machine Connection window, in the web browser window displaying the Windows Admin Center interface, in the upper-left corner of the page, select **Settings** and, in the drop-down menu, select **All connections**.
1. On the **All connections** page, select **+ Add**.
1. In the **Add or create resources** pane, in the **Servers** box, select **Add**.
1. On the **Add one** tab, in the **Server name** text box, enter **fs01.tailwindtraders.org**
1. When prompted for credentials, select the **Use another account for this connection**, in the **Username** and **Password** text boxes, enter **TAILWINDTRADERS\\Administrator** and **Pa55w.rd1234**, respectively, and then select **Add with credentials**.
1. Once the name of the server appears below the **Server name** text box with a green check mark next to it, select **Add**.

   > **Note:** To complete implementing the prerequisites, you will adjust the Windows Firewall with Advanced Security rules. The source, destination, and orchestrator computer must have the File and Printer Sharing (SMB-In) firewall rule enabled inbound. In addition, the source and destination computers must have the following firewall rules enabled inbound:
   - Netlogon Service (NP-In)
   - Windows Management Instrumentation (DCOM-In)
   - Windows Management Instrumentation (WMI-In)

1. Back on the **All connections** page of the Windows Admin Center, select **fs01.tailwindtraders.org**.
1. On the **fs01.tailwindtraders.org** page, in the vertical **Tools** menu, scroll down, select **Firewall** and, on the **Firewall** pane, select **Incoming rules**.
1. In the list of incoming rules, check the status of the following rules and, if needed, enable them:

   - File and Printer Sharing (SMB-In)
   - Netlogon Service (NP-In)
   - Windows Management Instrumentation (DCOM-In)
   - Windows Management Instrumentation (WMI-In)

1. Navigate back to the **All connections** page of the Windows Admin Center and select **azmig01.tailwindtraders.org (Gateway)**.
1. On the **azmig01.tailwindtraders.org (Gateway)** page, in the vertical **Tools** menu, scroll down, select **Firewall** and, on the **Firewall** pane, select **Incoming rules**.
1. In the list of incoming rules, check the status of the following rules and, if needed, enable them:

   - File and Printer Sharing (SMB-In)
   - Netlogon Service (NP-In)
   - Windows Management Instrumentation (DCOM-In)
   - Windows Management Instrumentation (WMI-In)

### Exercise 2: Perform migration and cutover by using Storage Migration Service

In this exercise, you will migrate file shares from the on-premises server FS01 to an automatically provisioned Azure VM by using Storage Migration Service. The exercise will involve the following on-premises servers:

- Azure Migrate Appliance serving the role of the orchestration server
- FS01 serving the role of the source server

The exercise consists of the following tasks:

- Task 1: Inventory servers
- Task 2: Transfer data
- Task 3: Cut over to the new server
- Task 4: Validate the outcome 

#### Task 1: Inventory servers

In this task, you will register the Hyper-V Server host with Azure Migrate.

1. Within the Remote Desktop session to the Hyper-V server hosting the virtual machines to be migrated, in the Virtual Machine Connection window to the Azure Migrate Appliance VM, in the web browser displaying the Windows Admin Center connection to **azmig01.tailwindtraders.com**, in the vertical **Tools** menu, scroll down and select **Storage Migration Service**.
1. In the **Storage Migration Service** window, select **Install* and wait for the installation to complete.
1. Review the information presented in the **Migrate storage in three steps** pane and select **Close**.
1. On the **Storage Migration Service** pane, scroll down and select **+ New job**.
1. On the **New job** pane, in the **Job name** text box, enter **smsjob1**, verify that the **Windows servers and clusters** **Source devices** option is selected, and then select **OK**.
1. On the **Inventory servers** tab of the **smsjob1** pane, review the **Check prerequisites** section and select **Next**.
1. In the **Enter credentials** section, enter the credentials of the TAILWINDTRADERS\\Administrator account, de-select the **Migrate from failover clusters** checkbox, and select **Next**.
1. In the **Install required features** section, select **Next**.
1. On the **Add and scan devices** pane, select **+ Add a device**.
1. On the **Add source device** pane, ensure that the **Device name** option is selected, in the **Name** text box, enter **fs01.tailwindtraders.org**, and select **Add*.
1. Select the newly added entry representing the fs01.tailwindtraders.org server and select **Validate** in the toolbar.

   > **Note:** You might need to select the ellipsis symbol directly above the server entry in order to display the **Validate** toolbar entry.

   > **Note:** Ensure that the validation results in the **Pass** result. In case the validation process fails, select the **Fail** link, review the validation results, and address any misconfiguration issues.

1. Select the newly added entry representing the fs01.tailwindtraders.org server and select **Start scan** in the toolbar.

   > **Note:** You might need to select the ellipsis symbol directly above the server entry in order to display the **Start scan** toolbar entry.

1. Review the scan results, focusing on the lower part of the **Add and scan devices** pane, ensure that they include the entry representing the **IT-Files** share on the fs01.tailwindtraders.org server, and select **Next**.

#### Task 2: Transfer data

In this task, you will step through provisioning the destination server that will host the file shares migrated from the source server and transfer of these shares.

1. While connected to the Azure Migrate Appliance VM, in the web browser window displaying the Storage Migration Service smsjob1 page, on the **Transfer data** tab, in the **Enter credentials** section, enter the credentials of the TAILWINDTRADERS\\Administrator account and select **Next**.
1. On the **Specify the destination for: fs01.tailwindtraders.org** pane, select the **Create a new Azure VM** option and then select **Create VM**.
1. On the **Create an Azure VM** pane, on the **Basics** tab, specify the following information and then select **Next: Size**.

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**migrate-target-01-RG**|
    |VM name|**FS02**|
    |Region|the name of the Azure region you used as target for Azure Migrate|
    |Operating system image|**2019-Datacenter**|
    |Local admin account on the VM|**smsadmin**|
    |Password|**Pa55w.rd1234**|

1. On the **Create an Azure VM** pane, on the **Size** tab, select **B2s** and then select **Next: Disks**.
1. On the **Create an Azure VM** pane, on the **Disks** tab, ensure that the **Premium SSD** entry is displayed in the **Disk type** drop-down list and then select **Next: Domain**.
1. On the **Create an Azure VM** pane, on the **Domain** tab, specify the following information and then select **Next: Networking**.

    |Setting|Value|
    |---|---|
    |Account with domain join permissions|**TAILWINDTRADERS\\Administrator**|
    |Password|**Pa55w.rd1234**|
    |Add this account to the local Administrators group|Enabled|

1. On the **Create an Azure VM** pane, on the **Networking** tab, specify the following information and then select **Next: Review + Create**.

    |Setting|Value|
    |---|---|
    |Network interface card name|**FS02-NIC**|
    |Virtual network|**MigrateTargetVNET**|
    |Subnet|**INFRA**|

1. On the **Create an Azure VM** pane, on the **Review + Create** tab, review the summary of the configuration settings and select **Create**.
1. On the **Creating your Azure VM** pane, monitor the progress of the Azure VM provisioning process and, once completed, select **Close**.

   > **Note:** The provisioning process should take about 5 minutes. The process will include creating disks, network interface, and the Azure VM, joining the server to the domain, customizing it to satisfy the Storage Migration Service prerequisites for a destination server, and configuring SMS transfer mapping.

   > **Note:** In case the provisioning process fails during the customization stage, on the **Creating your Azure VM** page select **Close**, back in the **Specify the destination for: fs1.tailwindtraders.org** pane, in the **Virtual machine deployment failed** section, select **Abandon VM**, in the **You're about to abandon an Azure VM**, select **Abandon** again, then,  in the **Specify the destination for: fs01.tailwindtraders.org** pane, select the option **Use an existing server or VM**. Then select **Browse**, in the **Select destination device**, enter **fs02.tailwindtraders.org**, select **Search**, in the list of results, select **fs02.tailwindtraders.org**, and then select **Add**. Next, open another tab in the web browser window, navigate to **https://localhost**, add a connection to **fs02.tailwindtraders.org** and enable the same firewall rules you enabled for **fs01.tailwindtraders.org** at the end of the previous exercise. Finally, switch back to the first browser tab and, in the **Specify the destination for: fs01.tailwindtraders.org** pane, with the **fs02.tailwindtraders.org** entry in the **Destination device** text box, select **Scan**.

1. On the **Transfer data** tab of the **smslabjob2** pane, in the **Specify the destination for: fs01.tailwindtraders.org** section, verify that the **Destination volume** is set to **C:** and select **Next**.
1. In the **Adjust transfer settings** section, review the available settings, in the **Migrate users and groups** section, select the **Don't transfer users and groups** option, and select **Next**.
1. Once you reach the **Install required features** section, wait for the **Storage Migration Service Proxy** feature to be installed the destination server and then select **Next**.
1. In the **Validate source and destination device** section, select **Validate**. 
1. Review the results of the validation process, verify there are no errors (warnings are acceptable), and select **Next**.
1. In the **Start the transfer** section, select **Start transfer**.
1. Wait for the transfer to complete. This should take just a few seconds since the source server contains relatively small number of files.
1. Select **Next**.

#### Task 3: Cut over to the new server

In this task, you will complete the migration by cutting over to the new server.

1. Within the Remote Desktop session to the Hyper-V server hosting the virtual machines to be migrated, in the Virtual Machine Connection window to the Azure Migrate Appliance VM, in the web browser displaying the Windows Admin Center page, on the **Cut over to the new servers** tab, in the **Enter credentials** section, ensure that the **TAILWINDTRADERS\\Administrator** credentials are listed as **Stored credentials** for both the source and destination devices, and then select **Next**.
1. In the **Configure cutover from fs01.tailwindtraders.org to fs02.tailwindtraders.org** section, specify the following information and select **Next**:

    |Setting|Value|
    |---|---|
    |Migrate network settings|Enabled|
    |All network adapters migrated|Enabled|
    |Source network adapter - Use DHCP|Enabled|
    |Destination network adapter|**Ethernet**|
    |Rename the source device after cutover - Choose a new name|**fs01-old**|

1. In the **Adjust cutover settings** section, in the **Enter AD credentials** section, select **Enter new credentials**, enter the credentials of the **TAILWINDTRADERS\\Administrator** account, and then select **Next**.
1. In the **Validate source and destination devices** section, select **Validate**. 
1. Review the results of the validation process, verify there are no errors, and select **Next**.
1. In the **Cut over to the new servers** section, select **Start cutover**.
1. Monitor the progress of cutover. The cutover process should complete within a few minutes.

   > **Note:** You have the option of initiating Azure File Sync setup directly from the same interface.

1. In the **Cut over to the new servers** section, select **Finish**.
1. Back on the **Storage Migration Service** pane, close the **Migrate storage in three steps** pop-up window and review the list of jobs, which, at this point, should include the newly completed **smsjob1** with the status of **Completed**.

#### Task 4: Validate the outcome 

In this task, you will validate the outcome of the migration of file shares from the FS01 server.

1. Use the Virtual Machine Connection to connect to the console of the **FS01** virtual machine and sign in to it as TAILWINDTRADERS\\Administrator.
1. In the **Virtual Machine Connection** window, right-click the start menu icon and select **Run**.
1. In the **Run** text box, enter **sysdm.cpl** and select **OK**.
1. In the **System Properties** window, on the **Computer Name** tab, note that the computer name changed to **FS01-OLD**.
1. Use the Start menu to shut down the operating system.
1. Switch to the Virtual Machine Connection to **AZMIG01** and launch Windows PowerShell.
1. In the **Administrator: Windows PowerShell** window, run the following command to validate access to the migrated file server content.

   ```powershell
   Get-SmbShare -CimSession FS01.tailwindtraders.org
   ```

1. Review the output and verify that it includes the migrated share named **IT-Share**.
