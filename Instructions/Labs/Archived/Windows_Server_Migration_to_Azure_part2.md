---
lab:
    title: 'Migrate Windows Servers workloads by using Azure Migrate'
    module: 'TW-1002.1-2024: Migrating Windows Server to Microsoft Azure'
---

# TW-1002.1-2024: Migrating Windows Server to Microsoft Azure
# Workshop: Migrating Windows Server to Microsoft Azure
# Lab 2: Migrate Windows Servers workloads by using Azure Migrate

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

## Lab 2: Migrate Windows Servers workloads by using Azure Migrate

Estimated Time: 90 minutes

> **Note:** This scenario, for practical reasons, is implemented by leveraging the Hyper-V nesting support in Azure VMs. The on-premises environment is simulated by using nested Windows Server VMs running in an Azure VM with the Hyper-V server role installed. This minimizes time it takes to complete individual stages of the migration process. However, the migration process illustrated in this lab would not differ (from the functional standpoint) if performed by using virtual machines running on the Hyper-V platform in an actual on-premises environment. 

> **Important:** Before you start working on this lab, make sure that the deployment to the resource group **nested-hyper-v-RG** has successfully completed. You can determine this by navigating to the **nested-hyper-v-RG** resource group page and reviewing the entry below the **Deployments** label in the **Essential** section. This entry should display **1 succeeded**. The deployment was launched at the beginning of the first lab of this workshop and it takes about 45 minutes.

## Objectives

In this lab, you will:

- Discover and assess on-premises Windows Servers by using Azure Migrate
- Migrate on-premises Windows Servers by using Azure Migrate

### Exercise 1: Discover and assess on-premises Windows Servers by using Azure Migrate

In this exercise, you will use set up Azure Migrate in the on-premises environment and then use it to discover and assess Windows Servers assets. The exercise will involve the following on-premises servers:

- HYPERVHOST
- DC01
- FS01
- WEB01

The exercise consists of the following tasks:

- Task 1: Set up prerequisites for the migration
- Task 2: Implement hybrid network connectivity
- Task 3: Import data about Windows Server assets into Azure Migrate
- Task 4: Implement the Azure Migrate appliance
- Task 5: Assess manually imported data about Windows Server assets
- Task 6: Discover Windows Server assets by using the Azure Migrate appliance
- Task 7: Assess Windows Server assets discovered by using the Azure Migrate appliance

#### Task 1: Set up prerequisites for the migration

In this task, you will set up prerequisites for the migration, including setting up the on-premises environment and creating an Azure Migrate project.

> **Note:** Azure Migrate supports a wide range of scenarios, including VMware-based virtualization and bare metal servers. For more information, refer to [Azure Migrate support matrix](https://learn.microsoft.com/en-us/azure/migrate/migrate-support-matrix).

1. If needed, connect to the server SEA-ADM1 and sign in as CONTOSO\\Administrator.
1. If prompted **Do you want to allow your PC to be discoverable by other PCs and devices on this network?**, select **No**.
1. Once signed to the server SEA-ADM1, start a web browser and navigate to the Azure portal at `https:\\portal.azure.com`.
1. When prompted to authenticate, sign in by using the Microsoft Entra ID credentials listed on the **Resources** tab of the lab web interface.
1. In the Azure portal, in the **Search** text box, search for and select **Virtual machines**.
1. On the **Virtual machines** page, in the list of virtual machines, select **hypervhost**.
1. On the **hypervhost** virtual machine page, note its **Location** property in order to determine the Azure region where the Azure VM resides.

   > **Note:** You will use this information later in this lab.

1. On the **hypervhost** virtual machine page, select **Connect** and, in the drop-down menu, select **Connect**.
1. On the **hypervhost \| Connect** page, select **Download RDP file**.
1. Open the downloaded RDP file, in the **Remote Desktop Connection** pop-up window, select **Connect**.
1. When prompted to enter credentials, authenticate by using the **migadmin** user name and the password consisting of the **mig** prefix followed by the password of the user account you used to access the Azure subscription used in this lab (listed in the **Resources** tab of the lab web interface).
1. Within the Remote Desktop session to **hypervhost**, if prompted **Do you want to allow your PC to be discoverable by other PCs and devices on this network?**, select **No**.
1. Within the Remote Desktop session to **hypervhost**, switch to the Server Manager window, select the **Tools** menu header, and, in the drop-down menu, select **Hyper-V Manager**. 
1. In the **Hyper-V Manager** console, in the list of virtual machines, select **DC01** and then, in the lower section of the **Actions** pane, select **Connect**.
1. In the Virtual Machine Connection window, in the top menu, select **Action** and, in the drop-down menu, select **Ctrl+Alt+Delete**.
1. When prompted, authenticate as **TAILWINDTRADERS\\Administrator** using the password **Pa55w.rd1234**.
1. If prompted for Windows activation, select **Ask me later** and then, select **OK**.
1. In the Virtual Machine Connection window, select **Windows PowerShell** icon in the taskbar.
1. From the **Administrator: Windows PowerShell** window, run the following commands:

   > **Note:** This is required to address an issue unrelated to the migration scenario, which affects the IP configuration of the virtual machine representing the on-premises Active Directory domain controller.

   ```powershell
   $networkConfig = Get-WmiObject Win32_NetworkAdapterConfiguration -Filter "IPEnabled='true'"
   $networkConfig.EnableStatic('192.168.0.2','255.255.255.0')
   $networkConfig.SetGateways('192.168.0.1', 1)
   $networkConfig.SetDNSServerSearchOrder('127.0.0.1')

   Start-Service -Name "DNS"

   Enable-PSRemoting -Force

   $registryPath = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters"
   $registryName = "DisabledComponents"
   Set-ItemProperty -Path $registryPath -Name $registryName -Type dword -Value 0xff

   Restart-Computer -Force
   ```

1. Switch back to the Remote Desktop session, start Windows PowerShell ISE.
1. From the Windows PowerShell ISE console pane, run the following command restarts the remaining two virtual machines **FS01** and **WEB01**:

   > **Note:** This is required to address an issue unrelated to the migration scenario, which affects the IP configuration of the virtual machine representing the on-premises Active Directory domain controller.

   ```powershell
   Restart-VM -Name 'FS01','WEB01' -Force
   ```

   > **Note:** Next, you will configure the Azure Migrate project.

1. Within the Remote Desktop session to **hypervhost**, start a web browser and navigate to the Azure portal at `https:\\portal.azure.com`.
1. When prompted to authenticate, sign in by using the Microsoft Entra ID credentials listed on the **Resources** tab of the lab web interface.
1. In the Azure portal, in the **Search** text box, search for and select **Azure Migrate**.
1. On the **Azure Migrate \| Get started** page, in the **Migrate and modernize your datacenter** pane, in the **Servers, databases, and web apps** section, select **Discover, assess, and migrate**.
1. On the **Azure Migrate \| Servers, databases, and web apps** page, select **Create project**.
1. On the **Create project** page, specify the following settings and then select **Create**:

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|The name of a **new** resource group **migrate-project-01-RG**|
   |Project|**migrate-project-01**|
   |Geography|**United States**|
   |Connectivity method|**Public endpoint**|

1. On the **Review + Create** tab, wait for the validation to complete and then select **Create**.

   > **Note:** Do not wait for the project to be provisioned but instead proceed to the next task. The project creation might take about 1 minute.

#### Task 2: Implement hybrid network connectivity

In this task, you will implement network connectivity between the on-premises network hosting the Hyper-V host and a pre-created Azure virtual network that will host migrated servers. This will include the following activities:

- Configuring peering between the two networks
- Configuring routing within the peered Azure virtual network representing the migration target environment
- Configuring DNS server settings in the Azure virtual network representing the migration target environment

> **Note:** In general, Azure Migrate does not require private network connectivity to replicate on-premises servers to Azure VMs. In our scenario, this is required in order to allow migrated domain member servers to reach the on-premises Active Directory domain controller. While in our lab, for practical reasons, the on-premises environment is simulated by using nested VMs running in an Azure VM with the Hyper-V server role installed, the underlying concept is applicable to real-life migrations. The primary difference is the way you will establish connectivity to the target Azure virtual network (via virtual network peering rather than by using hybrid technologies such as ExpressRoute or Site-to-Site VPN). However, the migration process illustrated in this lab does not differ (from the functional standpoint - leaving the latency and bandwidth implications aside) from one that involves an actual on-premises environment. 

1. Within the Remote Desktop session to **hypervhost**, in the web browser window displaying the Azure portal, in the **Search** text box, search for and select **Virtual networks**.
1. On the **Virtual networks** page, select **MigrateTargetVNET**.
1. On the **MigrateTargetVNET** page, in the vertical navigation menu on the left side, in the **Settings** section, select **Peerings**.
1. On the **MigrateTargetVNET \| Peerings** page, select **+ Add**.
1. On the **Add peering** page, in the **This virtual network** section, specify the following settings (leave others with their defaults):

   |Setting|Value|
   |---|---|
   |Peering link name|**MigrateTargetVNET-OnPremVNET**|
   |Allow 'MigrateTargetVNET' to access the peered virtual network|Enabled|
   |Allow 'MigrateTargetVNET' to receive forwarded traffic from the peered virtual network|Enabled|

1. On the **Add peering** page, in the **Remote virtual network** section, specify the following settings (leave others with their defaults):

   |Setting|Value|
   |---|---|
   |Peering link name|**OnPremVNET-MigrateTargetVNET**|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Virtual network|**OnPremVNET**|
   |Allow 'OnPremVNET' to access the peered virtual network|Enabled|
   |Allow 'OnPremVNET' to receive forwarded traffic from the peered virtual network|Enabled|

1. Select **Add**.

   > **Note:** Wait for the peering to be provisioned. The peering should be created within a few seconds. 

> **Note:** The required routing configuration within the Hyper-V host has been already set up as part of the initial provisioning of the lab environment (you can review them if you are interested by running the `route print` command within Remote Desktop session to the Hyper-V host. However, to allow bi-directional routing between the Hyper-V internal network to which its nested virtual machines are connected and the Azure virtual networks, you will also need to implement user-defined routes in the latter. 

1. Within the Remote Desktop session to **hypervhost**, in the web browser window displaying the Azure portal, in the **Search** text box, search for and select **Route tables**.
1. On the **Route tables** page, select **+ Create**.
1. On the **Basics** tab of the **Create Route table** page, specify the following settings and then select **Review + create**:

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|**migrate-target-01-RG**|
   |Region|the name of the same Azure region you identified earlier in this task|
   |Virtual network name|**MigrateTargetRT**|
   |Propagate gateway routes|**No**|

1. On the **Review + create** tab, wait for the validation process to complete and then select **Create**.
1. On the **Your deployment is complete** page, select **Go to resource**.
1. On the **MigrateTargetRT** page, in the vertical navigation menu on the left side, in the **Settings** section, select **Routes**.
1. On the **MigrateTargetRT \| Routes** page, select **+ Add**.
1. In the **Add route** pane, specify the following settings and then select **Add**:

   |Setting|Value|
   |---|---|
   |Route name|**OnPremVNETNATRoute**|
   |Destination type|**IP Addresses**|
   |Destination IP addresses/CIDR ranges|**192.168.0.0/24**|
   |Next hop type|**Virtual appliance**|
   |Next hop address|**10.0.2.4**|

1. Back on the **MigrateTargetRT** page, in the vertical navigation menu on the left side, in the **Settings** section, select **Subnets**.
1. On the **MigrateTargetRT \| Subnets** page, select **+ Associate**.
1. In the **Associate subnet** pane, specify the following settings and then select **OK**:

   |Setting|Value|
   |---|---|
   |Virtual network|**MigrateTargetVNET**|
   |Subnet|**INFRA**|

1. In the **Associate subnet** pane, specify the following settings and then select **OK**:

   |Setting|Value|
   |---|---|
   |Virtual network|**OnPremVNET**|
   |Subnet|**DNAT**|

   > **Note:** Finally, you will configure DNS settings in the pre-created Azure virtual network that will be used to host target Azure VMs.

1. Within the Remote Desktop session to **hypervhost**, in the web browser window displaying the Azure portal, in the **Search** text box, search for and select **Virtual networks**.
1. On the **Virtual networks** page, select **MigrateTargetVNET**.
1. On the **MigrateTargetVNET** virtual network page, in the vertical navigation menu on the left side, in the **Settings** section, select **DNS servers**.
1. On the **MigrateTargetVNET \| DNS servers** page, in the **DNS servers** section, select the **Custom** option, in the **IP Address** text box, enter **192.168.0.2**, and then select **Save**.

   > **Note:** This is the IP address assigned to the on-premises Active Directory domain controller **DC01**. This needs to be set in order for servers migrated to Azure VMs to be able to communicate with their Active Directory environment. 

#### Task 3: Import data about Windows Server assets into Azure Migrate

In this task, you will import data about the servers you intend to migrate to Azure into the project you created in the previous task. 

> **Note:** In this exercise, you will perform the discovery using two different methods - by relying on the functionality provided by an Azure Migrate appliance and by using a pre-created CSV file that contains information similar to the one generated by the appliance (to time it in the optimal manner, we will actually start with a pre-created CSV file). This approach will serve two main purposes:

- It will help you become familiar with both methods.
- It will help minimize the time required for the Azure Migrate Appliance discovery to complete.

1. Within the Remote Desktop session to **hypervhost**, in the web browser displaying the Azure portal, navigate to the **Azure Migrate \| Servers, databases, and web apps** page and select **Discover**.
1. On the **Discover** page, select the **Import using CSV** option and review the corresponding instructions.

   > **Note:** Typically, you would download a template **CSV** file and populate it with data applicable to the servers you intend to migrate. In this case, you will use a pre created CSV file that contains the relevant information for **DC01**, **FS01**, and **WEB01** servers. 

1. Open File Explorer and, in the **Documents** folder, create a file named **AzureMigrateImport-01.csv**. Copy into the file the content available at `https://raw.githubusercontent.com/MicrosoftLearning/Migrating-Windows-Server-to-Microsoft-Azure/master/Allfiles/Labs/AzureMigrateimport-01.csv` and save the change.

   > **Note:** Review the content and verify that it contains information about the three virtual machines representing the on-premises environment.

1. Next to the **Upload the CSV file** text box, select the folder icon, in the **Open** dialog box, navigate to the location of the file, select it, select **Open**, and, once back on the **Discover* page, select **Import**.

   > **Note:** Do not wait for the import to complete but instead proceed to the next task. The discover might take up to 10 minutes to complete.

#### Task 4: Implement the Azure Migrate appliance

In this task, you will download, import, and configure the Azure Migrate virtual appliance on the Hyper-V host.

1. Within the Remote Desktop session to **hypervhost**, in the web browser displaying the Azure portal, navigate back to the **Azure Migrate \| Servers, databases and web apps** page and, in the **Assessment tools** section, select **Discover**.
1. On the **Discover** page, ensure that the **Discover using appliance** option is selected and, in the **Are your servers virtualized?** drop-down list, select **Yes, with Hyper-V**.
1. In the **1. Generate project key** section, in the **Name your appliance** text box, enter **azmig01** and then select **Generate key**.

   > **Note:** Wait for the creation of the project-related Azure resources to complete. This might take about 3 minutes.

1. Record the project key. You will need it later in this lab.
1. In the **2. Download Azure Migrate appliance** section, ensure that the **.VHD file. 12 GB** option is selected and then select **Download**.

   > **Note:** Wait for the download to complete and that the downloaded file is approximately 9.3 GB in size. If not, repeat this step. The download might take about 10 minutes.

   > **Note:** Consider, in the meantime, starting the next task in order to minimize an idle wait time and then, once you complete that task, returning back to this step.

1. Within the Remote Desktop session to **hypervhost**, switch to the Windows PowerShell ISE window and open the script pane.
1. In the Windows PowerShell ISE script pane, enter the following script that extracts the content of the downloaded file into the **C:\\VM\\AZMIG01** directory and then select the green arrow icon in the toolbar to execute it:

   ```powershell
   $zipFilePath = "$env:USERPROFILE\DOWNLOADS\AzureMigrateAppliance.zip"
   $extractPath = 'C:\VM\AZMIG01'

   Unblock-File -Path $zipFilePath
   New-Item -ItemType Directory -Path $extractPath -Force

   $shell = New-Object -ComObject Shell.Application
   $zipFolder = $shell.Namespace($zipFilePath + '\AzureMigrateAppliance')
   $destinationFolder = $shell.Namespace($extractPath)
   $destinationFolder.CopyHere($zipFolder.Items(), 16)
   ```

   > **Note:** Wait for the script to complete. This might take about 3 minutes. At that point, you will be able to create a virtual machine that will host the Azure Migrate appliance.

1. Within the Remote Desktop session to **hypervhost**, switch to the **Hyper-V Manager** console.
1. In the **Hyper-V Manager** console, in the **Actions** pane, select **Import Virtual Machine**.
1. On the **Before You Begin** page of the **Import Virtual Machine** wizard select **Next**.
1. On the **Locate Folder** page, in the **Folder** text box, enter **C:\\VM\\AZMIG01\\** and select **Next**.
1. On the **Select Virtual Machine** page, accept the default selection and select **Next**.
1. On the **Choose Import Type** type, ensure that the **Register the virtual machine in-place (use the existing unique ID)** is selected and then select **Next**.
1. On the **Connect Network** page, in the **Connection** drop-down list, select **NatSwitch** and then select **Next**.
1. On the **Summary** page, select **Finish**.

   > **Note:** Next, you will configure the Azure Migrate virtual appliance.

1. Back in the **Hyper-V Manager**, select the newly imported virtual machine and, in the lower section of the **Actions** pane, select **Start**. 
1. Once the virtual machine is running, in the lower section of the **Actions** pane, select **Connect**.
1. In the Virtual Machine Connection window, on the **License terms** page, select **Accept**.
1. On the **Customize settings** page, use the **Password** and **Reenter password** text boxes to set the password for the local Administrator account to **Pa55w.rd1234** and then select **Finish**.
1. In the Virtual Machine Connection window, in the top menu, select **Action** and, in the drop-down menu, select **Ctrl+Alt+Delete**.
1. When prompted, authenticate as the local Administrator using the newly set password.
1. If prompted **Do you want to allow your PC to be discoverable by other PCs and devices on this network?**, select **No**.
1. Once you sign in, wait for a web browser window to open.

   > **Note:** The web browser will automatically display a page hosted by the locally running web server on port 44368. 

1. When prompted to accept **Terms of use**, select **I agree**.
1. On the **Appliance Configuration Manager** page, verify that the **Check connectivity to Azure** and **Check time is in sync with Azure** steps successfully completed. Next, in the second **Check latest updates and register appliance** section, in the **Register Hyper-V appliance by passing the key here** text box, enter the project key you recorded earlier in this task and select **Verify**.

   > **Note:** Use the **Type clipboard text** action in the **Clipboard** menu of the Virtual Machine Connection window to paste the product key value.

   > **Note:** The appliance will verify the key and start the auto-update service, which updates all the services on the appliance to their latest versions. Wait for the appliance update to complete. This might take about 5 minutes. 

1. In the **New update installed** pane, select **Refresh**.

   > **Note:** Once the web page refreshes, you might need to select **Verify** again.

1. Back on the **Appliance Configuration Manager** page, in the **Azure user login and appliance registration status** section, select **Login**.
1. In the **Continue with Azure Login** pane, select **Copy code & Login**. This will automatically open another tab in the same web browser window.
1. When prompted, enter the code you just copied and select **Next**.
1. When prompted, sign in by using the Microsoft Entra ID credentials listed on the **Resources** tab of the lab web interface.
1. Switch back to the web browser tab displaying the **Appliance Configuration Manager** page.

   > **Note:** Do not wait the registration to complete but instead proceed to the next task. The registration might take up to 10 minutes, although it typically completes within 5 minutes.

#### Task 5: Assess manually imported data about Windows Server assets

In this task, you will use Azure Migrate to assess data about Windows Server assets you imported into Azure Migrate earlier in this exercise.

1. Within the Remote Desktop session to **hypervhost**, in the web browser displaying the Azure portal, navigate back to the **Azure Migrate \| Servers, databases, and web apps** page.
1. On the **Azure Migrate \| Servers, databases, and web apps** page, in the **Assessment tools** section, verify that the summary lists **3** discovered servers.
1. In the **Assessment tools** section, select **Assess** menu header and, in the drop-down menu, select **Azure VM**.
1. On the **Basics** tab of the **Create assessment** page, ensure that the **Assessment type** is set to **Azure VM** and **Discovery source** is set to **Imported servers** and then, on the right side of **Assessment settings**, select **Edit**.
1. On the **Assessment settings** page, specify the following settings and then select **Save**:

   |Setting|Value|
   |---|---|
   |Target location|The same Azure region where the Azure VM hosting the Hyper-V host resides|
   |Storage type|**Standard SSD managed disks**|
   |Saving options (Compute)|**None**|
   |Sizing criteria|**As on-premises**|
   |VM series|**Dsv5_series**|
   |Comfort factor|**1**|
   |Offer/Licensing program|**Pay-As-You-Go**|
   |Currency|**US Dollar ($)**|
   |Discount|**0**|
   |VM uptime Day(s) per month|**31**|
   |VM uptime Hour(s) per day|**24**|
   |Already have a Windows Server license|**No**|
   |Security|**Yes, with Microsoft Defender for Cloud**|

   > **Note:** The choice of **Sizing criteria** in our case is driven by the lack of performance data. In general, you might want to consider using the **Performance-based** option, since this would take into account potential cost savings if the resources allocated to on-premises servers are underutilized.

   > **Note:** The choice of VM series would be dependent on the Windows Server workloads.

1. Back on the **Basics** tab of the **Create assessment** page, select **Next: Select servers to assess >**.
1. On the **Select servers to assess** tab, perform the following actions:

   1. In the **Assessment name** text box, enter **migrate-project-01-assessment-imported**
   1. In the **Select or create a group** section, ensure that **Create new** is selected and, in the **Group name** text box, enter **migrate-project-01-assessment-imported-group-01**
   1. In the **Add machines to the group** section, review the list of the servers and select the checkboxes next to **DC01**, **FS01**, and **WEB01**.

      > **Note:** If there are no servers listed at this point, re-import the same file again. 

      > **Note:** As you review the list of the servers, note the **Out of support** links appearing in the **Operating system support** column in the rows corresponding to the **DC01** and **FS01** servers. Select the links and review the information provided in the **Operating system license support status** pane. As per the information provided there, one (and, in some cases, the only) way to remediate this issue is to migration to Azure, which provides the benefit of the free, extended support for operating system security updates.

1. Select **Next: Review + create assessment >**.
1. On the **Next: Review + create assessment** tab, select **Create assessment**.

   > **Note:** Wait for the assessment to complete. The assessment preparation should take less than 1 minute.

1. To view the outcome of the assessment you initiated in the previous exercise, navigate back to the **Azure Migrate \| Servers, databases, and web apps** page and, in the **Assessment tools** section, in the **Assessment** subsection, select **1** next to the **Azure VM** entry.

   > **Note:** Use the **Refresh** button at the top of the page to update the assessment status.

1. On the **Azure Migrate: Discovery and assessment \| Assessments** page, verify that the **migrate-project-01-assessment-imported** entry is listed with the **Ready** status, and select it.
1. On the **migrate-project-01-assessment-imported** page, review the graphs representing such information for all three Windows Server virtual machines as **Azure readiness**, **Monthly cost estimates**, **Storage - Monthly cost estimates**, **Distribution by OS License support status**, and **Distribution by Windows Server version**. The page also contains **Saving options**, which provide the estimate of savings with Azure Hybrid benefit.

   > **Note:** To display more detailed information, select **Azure readiness** and **Cost details** items in the vertical navigation menu on the left side of the **migrate-project-01-assessment-imported** page.

#### Task 6: Discover Windows Server assets by using the Azure Migrate appliance

In this task, you will use the Azure Migrate virtual appliance you implemented earlier in this exercise to discover Windows Server assets. 

1. Within the Remote Desktop session to **hypervhost**, switch to the Virtual Machine Connection window to the Azure Migrate virtual appliance.
1. In the web browser window displaying the **Appliance Configuration Manager** page, scroll down to the section titled **2. Manage credentials and discovery sources** and, in **Step 1: Provide Hyper-V credentials for discovery of Hyper-V VMs**, select **Add credentials**.
1. In the **Add credentials** pane, specify the following settings and then select **Save**:

   |Setting|Value|
   |---|---|
   |Source type|**Hyper-V Host/Cluster**|
   |Friendly name|**HyperVAdmin**|
   |Username|The name of the Hyper-V admin account you used to sign in to the Azure VM|
   |Password|The password of the Hyper-V admin account you used to sign in to the Azure VM|

1. Scroll down to **Step 2: Provide Hyper-V host/cluster details** and select **Add discovery source**.
1. In the **Add discovery source** pane, select **Add single item**, specify the following settings, and then select **Save**:

   |Setting|Value|
   |---|---|
   |Discovery source|**Hyper-V Host/Cluster**|
   |IP Address/FQDN|**192.168.0.1**|
   |Map credentials|**HyperVAdmin**|

   > **Note:** **192.168.0.1** is the IP address assigned to the Hyper-V host network interface connected to the **NatSwitch** virtual switch, to which all of the nested virtual machines are connected. You can verify this by reviewing the output of the **ipconfig** command ran within the Hyper-V host and the virtual appliance.

1. Ensure that the validation status of the discovery source is successful. 
1. Scroll down to **Step 3: Provide server credentials to perform software inventory, agentless dependency analysis, discovery of SQL Server instances and databases and discovery of web apps in your Hyper-V environment**, ensure that the slider is enabled, select **Add credentials**, specify the following settings, and then select **Save**:

   |Setting|Value|
   |---|---|
   |Credentials type|**Domain credentials**|
   |Friendly name|**DomainAdmin**|
   |Domain name|**tailwindtraders.org**|
   |Username|**Administrator**|
   |Password|**Pa55w.rd1234**|

1. In the same section, select **Add credentials** again, specify the following settings, and then select **Save**:

   |Setting|Value|
   |---|---|
   |Credentials type|**Windows (Non-domain)**|
   |Friendly name|**LocalAdmin**|
   |Username|**Administrator**|
   |Password|**Pa55w.rd1234**|

   > **Note:** After the Hyper-V Server discovery, the automatically initiates software inventory, followed by discovery of SQL Server instances and databases, and discovery of web apps. This might take up to 24 hours to complete, so it will not be covered here. 

   > **Note:** In general, you should follow the principle of least privilege when deciding which accounts to use for discovery. The account you specify should be a member of the local Administrators group on the servers you intend to migrate. We rely on the built-in domain Administrator account strictly for the sake of simplicity. 

1. Select **Start discovery**.

   > **Note:** Wait for about 5 minutes for the discovery to be completed. Note, however, the completion of the discovery might take longer if there are any issues with connectivity or access to the servers being discovered. 

#### Task 7: Assess Windows Server assets discovered by using the Azure Migrate appliance

In this task, you will use Azure Migrate to assess Windows Server assets discovered by using Azure Migrate Appliance.

1. Within the Remote Desktop session to **hypervhost**, in the web browser displaying the Azure portal, navigate back to the **Azure Migrate \| Servers, databases, and web apps** page.
1. On the **Azure Migrate \| Servers, databases, and web apps** page, in the vertical navigation menu on the left side, in the **Manage** section, select **Discovered items**.
1. On the **Azure Migrate \| Discovered items** page, ensure that the **Azure Migrate appliance** tab is selected and verify that all four virtual machines appear on the list.

   > **Note:** Some of the data might not be available and some of the validation tasks might be failing.

   > **Note:** If you see the four entries on the list of virtual machines, proceed to the next step in this task. If not, either wait a bit longer or proceed directly to the next exercise.

1. On the **Azure Migrate \| Discovered items** page, in the vertical navigation menu on the left side, in the **Migration goals** section, select **Servers, databases, and web apps**.
1. On the **Azure Migrate \| Servers, databases, and web apps** page, in the **Assessment tools** section, select **Assess** menu header and, in the drop-down menu, select **Azure VM**.
1. On the **Basics** tab of the **Create assessment** page, ensure that the **Assessment type** is set to **Azure VM** and **Discovery source** is set to **Servers discovered from Azure Migrate appliance** and then, on the right side of **Assessment settings**, select **Edit**.
1. On the **Assessment settings** page, specify the following settings and then select **Save**:

   |Setting|Value|
   |---|---|
   |Target location|The same Azure region where the Azure VM hosting the Hyper-V host resides|
   |Storage type|**Premium managed disks**|
   |Saving options (Compute)|**None**|
   |Sizing criteria|**As on-premises**|
   |VM series|**Dsv5_series**|
   |Comfort factor|**1**|
   |Offer/Licensing program|**Pay-As-You-Go**|
   |Currency|**US Dollar ($)**|
   |Discount|**0**|
   |VM uptime Day(s) per month|**31**|
   |VM uptime Hour(s) per day|**24**|
   |Already have a Windows Server license|**No**|
   |Security|**Yes, with Microsoft Defender for Cloud**|

   > **Note:** The choice of **Sizing criteria** in our case is driven by the lack of performance data. In general, you might want to consider using the **Performance-based** option, since this would take into account potential cost savings if the resources allocated to on-premises servers are underutilized.

   > **Note:** The choice of VM series would be dependent on the Windows Server workloads.

1. Back on the **Basics** tab of the **Create assessment** page, select **Next: Select servers to assess >**.
1. On the **Select servers to assess** tab, perform the following actions:

   1. In the **Assessment name** text box, enter **migrate-project-01-assessment-appliance**
   1. In the **Select or create a group** section, ensure that **Create new** is selected and, in the **Group name** text box, enter **migrate-project-01-assessment-appliance-group-01**
   1. In the **Add machines to the group** section, review the list of the servers and select the checkboxes next to **DC01**, **FS01**, and **WEB01**.

      > **Note:** Here as well you have the **Out of support** links appearing in the **Operating system support** column in the rows corresponding to the **DC01** and **FS01** servers. Select the links and review the information provided in the **Operating system license support status** pane. As per the information provided there, one (and, in some cases, the only) way to remediate this issue is to migration to Azure, which provides the benefit of the free, extended support for operating system security updates.

1. Select **Next: Review + create assessment >**.
1. On the **Next: Review + create assessment** tab, select **Create assessment**.

   > **Note:** Wait for the assessment to complete. The assessment preparation should take less than 1 minute.

1. As with import-based assessment, to view the outcome, navigate back to the **Azure Migrate \| Servers, databases, and web apps** page and, in the **Assessment tools** section, in the **Assessment** subsection, select **2** next to the **Azure VM** entry.

   > **Note:** Use the **Refresh** button at the top of the page to update the assessment status.

1. On the **Azure Migrate: Discovery and assessment \| Assessments** page, verify that the **migrate-project-01-assessment-appliance** entry is listed with the **Ready** status, and select it.
1. On the **migrate-project-01-assessment-appliance** page, review the graphs representing such information for both Windows Server virtual machines as **Azure readiness**, **Monthly cost estimates**, **Storage - Monthly cost estimates**, **Distribution by OS License support status**, and **Distribution by Windows Server version**. The page also contains **Saving options**, which provide the estimate of savings with Azure Hybrid benefit.

   > **Note:** To display more detailed information, select **Azure readiness** and **Cost details** items in the vertical navigation menu on the left side of the **migrate-project-01-assessment-appliance** page.

### Exercise 2: Migrate on-premises Windows Servers by using Azure Migrate

In this exercise, you will migrate one of the on-premises Windows Servers to an Azure VM by using Azure Migrate, following its assessment. The exercise will involve the following on-premises servers:

- HYPERVHOST
- WEB01

> **Note:** The primary reason for using a single server only (rather than all three) is to minimize the time required for the migration to complete. However, the migration steps would be practically identical when migrating multiple servers. Another reason is that FS01-hosted file shares will be migrated by using a different approach in the upcoming lab.

The exercise consists of the following tasks:

- Task 1: Register the Hyper-V Server host with Azure Migrate
- Task 2: Prepare for replication of Windows Server virtual machines to Azure VMs
- Task 3: Replicate Windows Server virtual machines to Azure VMs

#### Task 1: Register the Hyper-V Server host with Azure Migrate

In this task, you will register the Hyper-V Server host with Azure Migrate.

1. Within the Remote Desktop session to **hypervhost**, in the web browser displaying the Azure portal, navigate back to the **Azure Migrate \| Servers, databases, and web apps** page and, in the **Migration tools** section, select **Discover**.

   > **Note:** Make sure that **migrate-project-01** appears in the **Project** filter at the top of the page.

1. On the **Discover** page, in the **Are your servers virtualized?** drop-down list, select **Yes, with Hyper-V**.
1. In the **Target region** drop-down list, select the same Azure region where the Azure VM hosting the Hyper-V Server resides

   > **Note:** Make sure that you set the **Target location** as described.

1. Select the checkbox confirming that the target region for the migration is the one you selected and then select the **Crete resources** button.

   > **Note:** Wait for the resource provisioning to complete. This might take about 1 minute.

1. On the **Discover** page, in the section **1. Prepare Hyper-V host servers**, select the **Download** button to download credentials file for the Azure Recovery Services vault automatically provisioned by Azure Migrate.
1. Open another tab in the same browser window and navigate to `https://download.microsoft.com/download/b/d/b/bdba49ae-06d6-4a18-9b91-b49bcf048fd9/AzureSiteRecoveryProvider.exe` to download the Hyper-V replication provider (AzureSiteRecoveryProvider.exe) installer.

   > **Important:** Do **not** use the **Download** *link* on the **Discover** page to download the Hyper-V replication provider (AzureSiteRecoveryProvider.exe) software installer.

1. Launch the downloaded **AzureSiteRecoveryProvider.exe** installer. This will start the **Azure Site Recovery Provider Setup (Hyper-V server)** wizard.
1. On the **Microsoft Update** tab of the **Azure Site Recovery Provider Setup (Hyper-V server)** wizard, select **Off** and then select **Next**.
1. On the **Installation** tab of the wizard, accept the default installation location and select **Install**. 

   > **Note:** Wait for the installation to complete. This might take about 1 minute.

1. Once the installation completes, select **Register**. This will start the **Microsoft Azure Site Recovery Registration Wizard**.
1. On the **Vault Settings** tab of the **Microsoft Azure Site Recovery Registration Wizard** window, select **Browse*, in the **Open** dialog box, navigate to the **Downloads** folder, select the newly downloaded vault credentials file, and then select **Open**.
1. Back on the **Vault Settings** tab, select **Next**.
1. On the **Proxy Settings** tab, select **Next**. This will trigger registration in the Azure Recovery Services vault.

   > **Note:** Wait for registration to complete. This might take about 3 minutes.

   > **Note:** In case you are presented with a message informing about an internal error during registration, delete the originally downloaded credentials file, use the **Download** button on the **Discover** page to download a new one, and rerun the **Azure Site Recovery Provider Setup (Hyper-V server)**. 

1. On the **Registration** tab, ensure that the server was successfully registered and then select **Finish**.
1. Back in the web browser window, refresh the **Discover** page of **Azure Migrate**, in the **Are your servers virtualized?** drop-down list, select again **Yes, with Hyper-V**, and then select **Finalize registration**.

   > **Note:** Wait for the registration to be finalized. Finalizing the registration might take about 15 minutes, although typically completes much faster.

#### Task 2: Prepare for replication of Windows Server virtual machines to Azure VMs

In this task, you will perform additional configuration tasks that should be completed prior to migrating systems running Windows Server to Azure VMs. Most of these tasks are not specific to Azure Migrate, but apply to migration scenarios in general. You will also implement an Azure Storage account used for storing replicated content of disks attached to on-premises servers (which is specific to Azure Migrate).

> **Note:** For more information, refer to the Microsoft Learn article [Prepare to connect to Azure Windows VMs](https://learn.microsoft.com/en-us/azure/migrate/prepare-for-migration#prepare-to-connect-to-azure-windows-vms). This article also lists changes automatically applied by Azure Migrate, which would need to be implemented separately if other migration tools were used. 

1. Within the Remote Desktop session to **hypervhost**, switch to the **Hyper-V Manager** console.
1. In the **Hyper-V Manager** console, in the list of virtual machines, select **WEB01** and then, in the lower section of the **Actions** pane, select **Connect**.
1. In the Virtual Machine Connection window, in the top menu, select **Action** and, in the drop-down menu, select **Ctrl+Alt+Delete**.
1. When prompted, authenticate as **TAILWINDTRADERS\\Administrator** using the password **Pa55w.rd1234**.
1. If prompted for Windows activation, select **Ask me later** and then, select **OK**.
1. In the Virtual Machine Connection window, select **Windows PowerShell ISE** icon in the Start menu.
1. From the **Administrator: Windows PowerShell ISE** window, run the following commands which enable Remote Desktop functionality (in case it is currently disabled) and configure it:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server' -Name fDenyTSConnections -Value 0 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name fDenyTSConnections -Value 0 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name PortNumber -Value 3389 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name UserAuthentication -Value 0
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name LanAdapter -Value 0 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name KeepAliveEnable -Value 1  -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name KeepAliveInterval -Value 1  -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name KeepAliveTimeout -Value 1 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name fDisableAutoReconnect -Value 0 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name fInheritReconnectSame -Value 1 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name fReconnectSame -Value 0 -Type DWord -Force
   Restart-Service -Name TermService -Force
   ```

1. Run the following commands which enables Windows Firewall with Advanced Security (in case it is currently disabled) and configures it:

   ```powershell
   Set-NetFirewallProfile -Profile Domain, Public, Private -Enabled True
   Get-NetFirewallRule -DisplayGroup 'Remote Desktop' | Set-NetFirewallRule -Enabled True
   Set-NetFirewallRule -Name FPS-ICMP4-ERQ-In -Enabled True
   New-NetFirewallRule -DisplayName AzurePlatform -Direction Inbound -RemoteAddress 168.63.129.16 -Profile Any -Action Allow -EdgeTraversalPolicy Allow
   New-NetFirewallRule -DisplayName AzurePlatform -Direction Outbound -RemoteAddress 168.63.129.16 -Profile Any -Action Allow
   ```

1. Run the following command which enables PowerShell Remoting:

   ```powershell
   Enable-PSRemoting -Force
   ```

   > **Note:** The tasks provided here do not represent the full extent of changes that are recommended as part of migration. For details, refer to the Microsoft Learn article [Prepare to connect to Azure Windows VMs](https://learn.microsoft.com/en-us/azure/migrate/prepare-for-migration#prepare-to-connect-to-azure-windows-vms) mentioned earlier.

   > **Note:** Next, you will implement the Azure Storage account used for storing replicated content of disks attached to on-premises servers. While you might have the option to rely on Azure Migrate to automatically set it up for you, this functionality does not work consistently, so to avoid any potential issues, you will perform this task yourself.

1. Within the Remote Desktop session to **hypervhost**, switch to the web browser window displaying the Azure portal, in the **Search** text box and then search for and select **Storage accounts**.
1. On the **Storage accounts** page, select **+ Create**.
1. On the **Basics** tab of the **Create a storage account** page, specify the following settings and select **Next: Advanced >**.

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group| **migrate-target-01-RG**|
   |Storage account name|any globally unique name between 3 and 24 in length consisting of letters and digits|
   |Region|the name of the same Azure region you used earlier in this task|
   |Performance|**Standard**|
   |Redundancy|**Locally-redundant storage (LRS)**|

1. On the **Advanced** tab, review the available options, accept the defaults, and select **Next: Networking >**.
1. On the **Networking** tab, review the available options, accept the defaults, and select **Next: Data protection >**.
1. On the **Data protection** tab of the **Create storage account** page, clear the **Enable soft delete for blobs** and **Enable soft delete for containers** checkboxes and select **Review**.

   > **Note:** These settings must be disabled when the storage account is used for hosting content replicated by Azure Site Recovery.

1. On the **Review** tab, wait for the validation process to complete and select **Create**.

   > **Note:** Wait for the provisioning to complete. The storage account should be created within 2 minutes.

   > **Note:** You also need to ensure that Azure Migrate will be able to access the newly created storage account. To accomplish this, you will use the system assigned managed identity of the Azure Recovery Services vault, which Azure Migrate provisioned automatically for you. This Azure Recovery Services vault is actually the resource that orchestrates the replication activities.

1. In the web browser window displaying the Azure portal, in the **Search** text box and then search for and select **Recovery Services vaults**.
1. On the **Recovery Services vaults** page, select the vault name which starts with the **migrate-project-01** prefix.
1. On the Recovery Services vault page, in the vertical navigation menu on the left side, in the **Settings** section, select **Identity**.
1. On the vault's **Identity** page, ensure that the **System assigned** tab is selected, set the **Status** switch to **On** and select **Save**.
1. When prompted, in the **Enable system assigned managed identity** pop-up window, select **Yes**.
1. On the vault's **Identity** page, select **Azure role assignments**.
1. On the **Azure role assignments** page, select **+ Add role assignment (Preview)**.
1. In the **Add role assignment (Preview)** pane, specify the following settings and then select **Save**:

   |Setting|Value|
   |---|---|
   |Scope|**Storage**|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource|The name of the storage account you created earlier in this task|
   |Role|**Contributor**|

1. Back on the **Azure role assignments** page, select **+ Add role assignment (Preview)**.
1. In the **Add role assignment (Preview)** pane, specify the following settings and then select **Save**:

   |Setting|Value|
   |---|---|
   |Scope|**Storage**|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource|The name of the storage account you created earlier in this task|
   |Role|**Storage Blob Data Contributor**|

#### Task 3: Replicate Windows Server virtual machines to Azure VMs

In this task, you will use the Azure Migrate to configure replication of one of the Hyper-V virtual machines running Windows Server to an Azure VM.

1. Within the Remote Desktop session to **hypervhost**, in the web browser displaying the Azure portal, navigate back to the **Azure Migrate \| Servers, databases, and web apps** page and, in the **Migration tools** section, verify that there are **4** discovered servers and then select the **4** link.
1. On the **Discovered servers** page, select **Replicate**.
1. On the **Specify intent** page, ensure that the **What do you want to migrate?** drop-down list contains the entry **Servers or virtual machines (VM)**.
1. Next, ensure that the **Where do you want to migrate to?** drop-down list contains the entry **Azure VM** and select **Continue**.
1. On the **Basics** tab of the **Replicate** page, in the **Are your machines virtualized?** drop-down list, select **Yes, with Hyper-V** and then select **Next**.
1. On the **Virtual machines** tab of the **Replicate** page, perform the following actions:

   1. If you completed the Azure Migrate appliance discovery-based assessment, in the **Import migration settings from an assessment** drop-down list, select **Yes, apply migration settings from an Azure Migrate assessment**. Otherwise, select **No, I'll specify the migration settings manually**.
   1. If you selected **Yes, apply migration settings from an Azure Migrate assessment**, in the **Select group** drop-down list, select **migrate-project-01-assessment-appliance-group-01** and, in the **Select assessment** drop-down list, select **migrate-project-01-assessment-appliance**.
   1. In the list of virtual machines, select the checkbox next to **WEB01**.

   > **Note:** You will migrate **FS01** in the last part of this workshop.

1. Select **Next**.
1. On the **Target settings** tab of the **Replicate** page, perform the following actions:

   1. Ensure that the **Subscription** drop-down list displays the name of the Azure subscription you are using in this lab.
   1. In the **Resource group** section, select **migrate-target-01-RG**.
   1. Clear the checkbox **Register with SQL IaaS extension**.
   1. Leave the checkbox **I have a Windows Server license** cleared.
   1. Ensure that **Auto-create (default)** appears in the **Cache storage account** drop-down list. 

   > **Note:** If this option is not available but instead you are presented with the **Replication storage account** drop-down list, select the storage account you created in the previous task. 

   1. In the **Virtual network** drop-down list, select **MigrateTargetVNET**.
   1. Ensure that **INFRA** appears in the **Subnet** drop-down list.
   1. Ensure that **No infrastructure redundancy required** appears in the **Availability options** drop-down list.

1. Select **Next**.
1. On the **Compute** tab of the **Replicate** page, in the **OS Type** drop-down list, select **Windows**.

   > **Note:** If you successfully completed the Azure Migrate appliance discovery-based assessment, the Azure VM size assigned by default will be based on the recommended size from that assessment. You can change it at this point if needed.

1. Select **Next**.
1. On the **Disks** tab of the **Replicate** page, in the **Disk Type** drop-down list, select **Standard SSD**.
1. Select **Next**.
1. On the **Tags** tab of the **Replicate** page, select **Next**.
1. On the **Review + Start replication** tab of the **Replicate** page, select **Replicate**.

   > **Note:** Wait for replication to complete. This might take about 10 minutes.

   > **Note:** At this point, to minimize the idle wait time, consider starting to work on the next lab of this workshop. Changes you apply in the next lab should not affect the replication of **WEB01** virtual machine you started in this lab.

1. To monitor the progress of replication, in the web browser displaying the Azure portal, navigate back to the **Azure Migrate \| Servers, databases, and web apps** page and, in the **Migration tools** section, select **Overview**.
1. On the **Azure Migrate: Migration and modernization** page, in the vertical navigation menu on the left side, in the **Manage** section, select **Jobs**. 
1. On the **Azure Migrate: Migration and modernization \| Jobs** page, track the progress of jobs, looking in particular for **Finalize protection on the primary virtual machine**, followed by **Finalize protection on the recovery virtual machine**.
1. Once the job **Finalize protection on the recovery virtual machine** is listed with the **Successful** status, navigate back to the **Azure Migrate: Migration and modernization** page and, in the vertical navigation menu on the left side, select **Replicating machines**.
1. Ensure that the **WEB01** virtual machines is listed with the **Protected** replication status (rather than **Waiting for first recovery point**).

   > **Note:** Use the **Refresh** button in the toolbar to update the replication status.

1. Once **WEB01** is listed with the **Protected** replication status, select the **WEB**.
1. On the **WEB01** page, in the toolbar, select **Test migration**. 
1. From the **Test migration** page, perform the following tasks:

   1. Select **Check for upgrade** next to the operating system label. 
   1. In the **OS Upgrade** pane, review the upgrade options available to you.

      > **Note:** In this lab, you will not step through an operating system upgrade since this would introduce extra wait time. 

   1. In the **OS Upgrade** pane, select **Close**.
   1. Back on the **Test migration** page, in the **Virtual network** drop-down list, select **MigrateTargetVNET**.
   1. Select **Test migration**.

      > **Note:** Typically, you would use a separate, fully isolated virtual network for test migrations. You will use the same virtual network for migration for the sake of simplicity.

      > **Note:** Wait for the test migration to complete. This might take about 10 minutes.

1. To monitor the progress of replication, on the **WEB01** page, select the **Initiating test failover** link to display the **Test failover** page.

   > **Note:** Use the information on the **Test failover** page to keep track of the failover progress. The test migration completes with a successful start of the test migrated virtual machine. 

1. To validate the successful test migration, within the Remote Desktop session to **hypervhost**, in the web browser window displaying the Azure portal, in the **Search** text box, search for and select **Virtual machines**.
1. On the **Virtual machines** page, in the list of virtual machines, select **WEB01-test**.
1. On the **WEB01-test** virtual machine page, select **Connect** and, in the drop-down menu, select **Connect**.
1. On the **WEB01-test \| Connect** page, select **Download RDP file**.
1. In the pop-up message in the web browser window, select **Keep** and then select **Open file**.
1. In the **Remote Desktop Connection** pop-up window, select **Connect**.
1. When prompted, sign in as **TAILWINDTRADERS\\Administrator** with the **Pa55w.rd1234** password.
1. Within the Remote Desktop session to the **WEB01-test** Azure VM, start Command Prompt, from the Command Prompt window, run `ping 192.168.0.2`, verify that you receive replies, and then sign out.
1. Within the Remote Desktop session to **hypervhost**, in the Azure portal, navigate back to the **Azure Migrate: Migration and modernization \| Replicating machines** page.
1. On the **Azure Migrate: Migration and modernization \| Replicating machines** page, select **WEB01**.
1. On the **WEB01** page, select **Clean up test migration**.
1. On the **Test migrate cleanup** page, select the checkbox **Testing is complete. Delete test virtual machine** and then select **Cleanup Test**.

   > **Note:** Wait for the test migrate cleanup to complete. This might take about 2 minutes. As before, you can monitor the progress of the **Test failover cleanup** job from its page accessible via the **Cleanup test failover pending** link on the **WEB01** page.

   > **Note:** To complete the process, you will perform migration. It is important to realize that the migration step is considered non-reversible. 

1. Navigate back to the **Azure Migrate: Migration and modernization \| Replicating machines** page and, in the list of replicated virtual machines, select **WEB01**.
1. On the **WEB01** page, select **Migrate**.
1. On the **Migrate** page, ensure that the **Shutdown virtual machines and perform a planned migration with no data loss?** drop-down list entry is set to **Yes** and then select **Migrate**.

   > **Note:** Wait for the migration to complete. This should take about 10 minutes. As before, you can monitor the progress of the **Planned failover** job from its page accessible from the **Azure Migrate: Migration and modernization \| Jobs** page. Once the migration completes, you might want to use the same procedure you stepped through following the test migration in order to validate that the migration was successful.
