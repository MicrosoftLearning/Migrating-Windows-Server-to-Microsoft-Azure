---
lab:
    title: 'Migrate on-premises Windows Servers to Azure-based management model by using Azure Arc'
    module: 'TW-1002.1-2024: Windows Server Migration to Azure Technical Workshop'
---

# TW-1002.1-2024: Windows Server Migration to Azure
# Workshop: Windows Server Migration to Azure
# Lab 1: Migrate on-premises Windows Servers to Azure-based management model by using Azure Arc

Total workshop estimated time: 210 minutes

## Objectives

After completing this workshop, you will be able to:

- Migrate on-premises Windows Server resources to Azure-based management model by using Azure Arc and Azure Policy
- Migrate on-premises Windows Server resources to Azure-based unified monitoring model by using Azure Monitor
- Migrate on-premises Windows Server resources to Azure-based security model by using Microsoft Defender for Cloud
- Migrate on-premises Windows Server resources to Azure-based update management model by using Azure Update Manager
- Plan and assess migration of Server resources by using Azure Migrate
- Migrate Windows Server resources by using Azure Migrate
- Migrate on-premises Windows Server-based file servers to Azure VMs by using Storage Migration Service

## Lab 1: Migrate on-premises Windows Servers to Azure-based management model by using Azure Arc

Estimated Time: 60 minutes

> **Note:** While Azure Arc does not implement server migration in the traditional sense, it constitutes an essential part of the overall Windows Server migration strategy offered by Microsoft. It extends Azure services and management to any infrastructure, including on-premises servers, multi-cloud environments, and edge locations. As the result, it enables organizations to manage their on-premises Windows Server operating systems and their workloads in the same manner as Azure-hosted resources. This is crucial for optimizing operational model in organizations that follow a hybrid or multi-cloud strategy. With Azure Arc, organizations can leverage Azure services such as Azure Policy, Azure Monitor, and Microsoft Defender for Cloud for servers hosted in their own datacenters. This ensures consistent application of policies, monitoring, and security measures across all of the organization's technology assets. In addition, Azure Arc-related features, such as dependency maps of Azure Monitor for Arc-enabled machines provide information which tends to be particularly valuable in preparation for cloud migrations.

> **Note:** It's important to note that, while our focus is on Windows Server, the scope of Azure Arc extends to Kubernetes clusters, Azure data services, SQL Server, and virtualized platforms such as Azure Stack HCI and VMware vSphere.

## Objectives

In this lab, you will:

- Connect on-premises Windows Server resources to Azure Arc
- Manage Azure Arc-enabled Windows servers by using Azure Policy
- Monitor Azure Arc-enabled Windows servers by using Azure Monitor
- Enhance security of Azure Arc-enabled Windows servers by using Microsoft Defender for Cloud

### Exercise 1: Connect on-premises Windows Server resources to Azure Arc

In this exercise, you will use different methods to connect on-premises Windows Server resources to Azure Arc. The exercise will involve the following on-premises servers:

- SEA-ADM1
- SEA-SVR1
- SEA-SVR2
- SEA-DC1

The exercise consists of the following tasks:

- Task 1: Prepare for connecting on-premises Windows Server resources to Azure Arc
- Task 2: Connect on-premises Windows Server resources to Azure Arc by using Windows Admin Center
- Task 3: Connect on-premises Windows Server resources to Azure Arc non-interactively at scale

> **Note:** There are other methods of connecting Windows Server resources to Azure Arc in addition to these presented here. For their comprehensive listing, refer to [Azure Connected Machine agent deployment options](https://learn.microsoft.com/en-us/azure/azure-arc/servers/deployment-options).

#### Task 1: Prepare for connecting on-premises Windows Server resources to Azure Arc

In this task, you will register Azure resource providers required to implement Azure Arc-enabled servers, install Windows Admin Center, and register it with Azure.

> **Note:** To implement Azure Arc-enabled servers, you need to ensure that the following resource providers are registered in the target Azure subscription:

- Microsoft.HybridCompute
- Microsoft.GuestConfiguration
- Microsoft.HybridConnectivity
- Microsoft.Compute 

1. Connect to the server SEA-ADM1 and, if needed, sign in as CONTOSO\\Administrator.
1. If prompted *Do you want to allow your PC to be discoverable by other PCs and devices on this network?*, select **No**.
1. Once signed to the server SEA-ADM1, start a web browser and navigate to the Azure portal at `https://portal.azure.com`.
1. When prompted to authenticate, sign in by using the Microsoft Entra ID credentials listed on the **Resources** tab of the lab web interface.
1. In the web browser window displaying the Azure portal, select the **Cloud Shell** icon to open the Cloud Shell pane. If needed, select **PowerShell** to start a PowerShell session and select **Confirm** to confirm your choice. 

   > **Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, ensure that the target subscription is listed in the **Subscription** text box, click **Create storage**, and wait until the PowerShell prompt is displayed in the Cloud Shell pane. This might take about 2 minutes.

1. In the Cloud Shell pane, run the following commands to register the Azure resource providers required to implement Azure Arc-enabled servers:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.HybridCompute
   Register-AzResourceProvider -ProviderNamespace Microsoft.GuestConfiguration
   Register-AzResourceProvider -ProviderNamespace Microsoft.HybridConnectivity
   Register-AzResourceProvider -ProviderNamespace Microsoft.AzureArcData
   ```

   > **Note**: Do not wait for the registration to complete but instead proceed to the next step. The registration should complete within the next few minutes.

   > **Note:** Since the next task will involve connecting to Azure Arc by using Windows Admin Center, you need to install it first.

1. While signed to the server SEA-ADM1, start Windows PowerShell ISE and open the script pane.
1. In the Windows PowerShell ISE script pane, enter the following script that installs Windows Admin Center and select the green arrow icon in the toolbar to execute it:

   ```powershell
   Invoke-WebRequest 'https://aka.ms/WACDownload' -OutFile "$pwd\WAC.msi"
   $msiArgs = @("/i", "$pwd\WAC.msi", "/qn", "/L*v", "log.txt", "SME_PORT=443", "SSL_CERTIFICATE_OPTION=generate")
   Start-Process msiexec.exe -Wait -ArgumentList $msiArgs
   ```

   > **Note:** Wait for the installation of Windows Admin Center to complete. The installation should take about 1 minute. It provisions the Windows Admin Center gateway component accessible via `https://SEA-ADM1.contoso.com` or `https://localhost`, secured by a self-signed certificate valid for 60 days.

   > **Note:** Next, you need to register Windows Admin Center with Azure.

1. While signed to the server SEA-ADM1, start a web browser and navigate to the `https://localhost` page.

   > **Note:** Ensure that you use the **localhost** name, rather than the actual server name.

1. When presented with the warning **Your connection isn't private**, select **Advanced** and then select **Continue to localhost (unsafe)**.

   > **Note:** The warning is expected since the target site is using a self-signed certificate.

1. If prompted to authenticate, sign in as CONTOSO\\Administrator.
1. If needed, close the pane confirming the successful installation, wait for the updates of the Windows Admin Center extensions to complete, and acknowledge their completion.
1. In Windows Admin Center, on the **All connections** page, select the cogwheel icon in the upper right corner of the page.
1. On the **Settings \| Account** page, in the **Azure Account** section, select **Register with Azure** and then, on the **Register with Azure** pane, select **Register**.
1. On the **Get started with Azure in Windows Admin Center** pane, in step 2 of the registration process, select **Copy** to copy the registration code into Clipboard.
1. Select the link next to the **Enter the code** text in step 3 of the registration process.
 
   > **Note:** This will open another tab in the Microsoft Edge window displaying the Enter code page.

1. In the **Enter code** pane, paste the code you copied into Clipboard and select **Next**.
1. When prompted to sign in, provide the same Microsoft Entra ID credentials you used earlier in this exercise to authenticate your access to the Azure subscription.
1. When prompted to confirm the question **Are you trying to sign in to Windows Admin Center?**, select **Continue**.
1. Verify that the sign in was successful and close the newly opened tab of the Microsoft Edge window.
1. Back on the **Get started with Azure in Windows Admin Center** pane, in step 4 of the registration process, ensure that the **Use existing** option is selected, in the **Microsoft Entra application ID** text box, enter the **App ID** string listed in the lab instructions, and select **Connect**.

   > **Note:** This application has been pre-provisioned in the Microsoft Entra ID tenant associated with the Azure subscription you are using in this lab.

1. On the Get started with Azure in Windows Admin Center blade, in step 5 of the registration process, select **Sign in**.
1. If prompted to sign in, provide the same Microsoft Entra ID credentials you have been using so far in the lab to authenticate access to the Azure subscription.
1. If prompted, in the **Permissions requested** pop-up window, review the permissions required by the application and select **Accept**.
1. On the **Register with Azure** pane of the Windows Admin Center page, verify that the registration was successful.

#### Task 2: Connect on-premises Windows Server resources to Azure Arc by using Windows Admin Center

In this task, you will use Windows Admin Center you installed in the previous task to connect one of your on-premises Windows Server resources to Azure Arc.

1. While signed to the server SEA-ADM1, in the web browser window displaying the Windows Admin Center interface, select **Settings**, select **All connections**, and, in the list of connections, select the **sea-adm1.contoso.com (Gateway)** entry.
1. In the Windows Admin Center interface, on the **sea-adm1.contoso.com** page, in the navigation menu on the left side, select **Azure hybrid center**.
1. In the **Azure hybrid center** pane, on the **Available Services** tab, in the **Set up Azure Arc** section, select **Set up**.
1. In the **Set up Azure Arc** pane, perform the following actions:

   1. Ensure that the **Subscription** drop-down list displays the name of the Azure subscription you are using in this lab.
   1. In the **Resource group** section, ensure that the **Create new** option is selected and then enter **arc-01-RG**.
   1. In the **Azure region** drop-down list, select the name of the Azure region close to the lab location. 

   > **Note:** This Azure region will store metadata of your on-premises servers.

1. Select **Set up** to proceed with configuring **SEA-ADM1** as an Azure Arc-enabled server.

   > **Note:** The server will connect to Azure, download the Connected Machine agent, install it and register with Azure Arc. To track the progress, select the **Notifications** icon (a gear wheel) in the upper right corner of the Windows Admin Center toolbar. The installation will trigger a display of a Command Prompt window, providing execution context for the **azcmagent.exe** installer of the Connected Machine agent. The installation might take about 5 minutes.

1. To validate the successful outcome, switch to the web browser window displaying the Azure portal, in the **Search** text box, search for and select **Azure Arc**.
1. On the **Azure Arc** page, in the navigation menu on the left side, in the **Infrastructure** section, select **Machines**.
1. On the **Azure Arc \| Machines ** page, verify that the entry named **SEA-ADM1** appears in the list of Azure Arc-enabled machines with the Arc agent status of **Connected**.

#### Task 3: Connect on-premises Windows Server resources to Azure Arc non-interactively at scale

In this task, you will connect Windows Server resources to Azure Arc by using a service principal, which illustrates one of the non-interactive methods for connecting servers to Azure Arc that facilitate at scale scenarios. This service principal can be used instead of your identity (which was used in the previous task) to provide access to the target Azure subscription.

> **Note:** In this case, the service principal has been pre-created for you and is listed in the lab instructions.

> **Note:** The process of connecting to Azure Arc is automated by using a template script available from the Azure portal. You will generate it first.

1. While signed to the server SEA-ADM1, in the web browser window displaying the Azure portal at `https://portal.azure.com`, on the **Azure Arc \| Machines** page, select **Add/Create** and then select **Add a machine** from the drop-down menu.
1. On the **Add servers with Azure Arc** page, in the **Add multiple servers** tile, select **Generate script**.
1. On the **Basics** tab of the **Add multiple servers with Azure Arc** page, specify the following settings:

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|*arc-01-RG**|
   |Region|The name of the same Azure region you selected in the previous task|
   |Operating system|**Windows**|
   |Connectivity method|**Public endpoint**|
   |Service principal|The name of the service principal listed in the lab instructions|

   > **Note:** The service principal must have the **Azure Connected Machine Onboarding** role-based access control (RBAC) role at the scope of the resource group or subscription that will host metadata of your on-premises servers. In this lab, you will leverage the same service principal that you used in the first task of this exercise to register Windows Admin Center with Azure, although in real-world scenarios, you might want to designate a separate service account for it.

1. Select **Download and run script**.
1. On the **Download and run script** tab of the **Add multiple servers with Azure Arc** page, ensure that the **Deployment method** option is set to **Basic** and in the section labeled **2. Download the script and add service principal credentials**, select **Download**.
1. If prompted, in the upper right corner of the web browser window, select **Keep** to continue with the download.

   > **Note:** For Windows, the script is named **OnboardingScript.ps1**.

   > **Note:** The script establishes a connection to Azure Arc by using the **azcmagent** command and referencing the service principal you designated. To successfully execute the command, you need to replace the `<ENTER SECRET HERE>` placeholder in line 4 of the script with the value of the service principal secret, which you can find listed in the lab instructions.

1. Switch to the **Administrator: Windows PowerShell ISE** window and open the downloaded **OnboardingScript.ps1** file.
1. In the script pane of the **Administrator: Windows PowerShell ISE** window, replace the `<ENTER SECRET HERE>` placeholder in line 4 of the script with the value of the service principal secret, which you can find listed in the lab instructions, and save the change.
1. From the console pane of the **Administrator: Windows PowerShell ISE** window, run the following command to allow running the downloaded script in environments where the RemoteSigned PowerShell execution policy is in place:

   ```powershell
   Unblock-File -Path $env:USERPROFILE\DOWNLOADS\OnboardingScript.ps1
   ```

1. Open another tab in the script pane of the **Administrator: Windows PowerShell ISE** window and use it to run the following script to execute the script on three servers **SEA-SVR1**, **SEA-SVR2**, and **SEA-DC1** that are part of the lab environment and, as the result, connect them to Azure Arc:

   ```powershell
   $scriptName = 'OnboardingScript.ps1'
   $scriptPath = "$env:USERPROFILE\Downloads\$scriptName"
   $remoteDirectoryName = 'Temp'
   $servers = 'SEA-SVR1','SEA-SVR2','SEA-DC1'

   $servers | ForEach-Object {New-Item -ItemType Directory -Path (Join-Path "\\$_\c`$\" "$remoteDirectoryName") -Force}
   $servers | ForEach-Object {Copy-Item -Path $scriptPath -Destination (Join-Path "\\$_\c`$\" "$remoteDirectoryName")}

   Invoke-Command -ComputerName $servers -ScriptBlock {
      param($directory, $file)
      & $("c:\$directory\$file")
   } -ArgumentList ($remoteDirectoryName, $scriptName)
   ```

   > **Note:** The script creates the C:\Temp directory on the target servers, copies to it the script, and uses PowerShell Remoting to launch the script execution.

   > **Note:** Wait for the script to complete. This might take about 3 minutes.

1. As in the previous task, to validate the successful outcome, in the web browser window displaying the Azure portal, navigate to the **Azure Arc \| Machines** page and verify that the entries for **SEA-SVR1** and **SEA-SVR2** appear in the list of Azure Arc-enabled machines with the Arc agent status of **Connected**.

### Exercise 2: Manage Azure Arc-enabled Windows servers by using Azure Policy

In this exercise, you will use Azure Policy in order to configure and assess the status of Arc-enabled Windows Servers. The exercise will involve the following on-premises servers:

- SEA-ADM1
- SEA-SVR1
- SEA-SVR2
- SEA-DC1

The exercise consists of the following tasks:

- Task 1: Create a policy assignment
- Task 2: Evaluate policy results

> **Note:** Azure Policy supports a wide range of scenarios that deal with compliance assessment of Arc-enabled Windows Severs. For an example of such scenario, refer to [Tutorial: Create a policy assignment to identify non-compliant resources](https://learn.microsoft.com/en-us/azure/azure-arc/servers/learn/tutorial-assign-policy-portal).

#### Task 1: Create a policy assignment

In this task, you will assign a built-in policy to the resource group containing your Arc-enabled Windows servers you implemented in the previous exercise of this lab. The policy will result in automatic installation of the Azure Monitor agent, which you will facilitate a review of the monitoring functionality of Arc-enabled machines in the next exercise of this lab.

1. While signed to the server SEA-ADM1, in the web browser window displaying the Azure portal at `https://portal.azure.com`, in the **Search** text box, search for and select **Policy**.
1. On the **Policy** page, in the vertical navigation menu on the left side, in the **Authoring** section, select **Definitions**.
1. On the **Policy \| Definitions** page, in the **Search box, enter **Configure Windows Arc-enabled machines** and, in the list of results, select **Configure Windows Arc-enabled machines to run Azure Monitor Agent**.
1. On the **Configure Windows Arc-enabled machines to run Azure Monitor Agent** policy definition page, review the policy definition and available effects and then select **Assign**.
1. On the **Basics** tab of the **Configure Windows Arc-enabled machines to run Azure Monitor Agent** policy assignment page, on the right side of the **Scope** text box, select the ellipsis button, in the **Scope** pane, in the **Subscription** drop-down list, select the Azure subscription you are using in this lab, in the **Resource Group** drop-down list, select **arc-01-RG**, followed by the **Select** button.
1. Back on the **Basics** tab of the **Configure Windows Arc-enabled machines to run Azure Monitor Agent** policy assignment page, note that **Policy enforcement** is enabled by default and select **Next**.
1. On the **Advanced** tab, review the available options without modifying them and then select **Next**.
1. On the **Parameters** tab, select **Next**, since this policy definition does not include any parameters that need input or review.
1. On the **Remediation** tab, select the **Create a remediation task* checkbox, ensure that **System assigned managed identity** option is selected, and then select **Next**.

   > **Note:** This is necessary in order to apply the assignment to the existing resources. In general, a policy assignment will apply only to resources created afterwards. However, policies with the **deployIfNotExist** effect (or **Modify**) will apply automatically to existing resources if remediation task is configured.

   > **Note:** All Azure Arc-enabled machines have, by default, system assigned managed identity enabled.

1. On the **Non-compliance message** tab, select **Next**.
1. On the **Review + create** tab, select **Create**.

#### Task 2: Evaluate policy results

In this task, you will review the results of the policy assignment.

1. While signed to the server SEA-ADM1, in the web browser window displaying the Azure portal at `https://portal.azure.com`, navigate back to the **Policy \| Definitions** page and, in the vertical menu on the left side, select **Compliance**.
1. On the **Policy \| Compliance** page, in the list of policy assignments, locate the **Configure Windows Arc-enabled machines to run Azure Monitor Agent** entry. Most likely, at this point, the policy will likely list the **Compliance** status as *Not started** or **Non-compliant**, with all of the four resources targeted by the policy assignment as non-compliant.

   > **Note:** If needed, wait until the compliance status changes to **Non-compliant**. This could take about 3 minutes. You might need to refresh the page to display the newly created policy assignment and its updated status.

1. On the **Policy \| Compliance** page, select the **Configure Windows Arc-enabled machines to run Azure Monitor Agent** entry.
1. On the **Configure Windows Arc-enabled machines to run Azure Monitor Agent** policy compliance page, review the detailed listing of non-compliant resources and note the **Create remediation task** button in the toolbar.

   > **Note:** You could use this functionality to explicitly invoke the remediation task. In our case, this is not required since the remediation task should already be running. 

1. To verify whether the remediation task is running, navigate back to the **Policy \| Compliance** page, select **Remediation** and, on the **Policy \| Remediation** page, select the **Remediation tasks** tab.
1. In the list remediation tasks, note the existing entry representing the remediation task associated with the newly created policy assignment.

   > **Note:** The remediation task should transition from the **Evaluating** state through **In Progress** to **Complete**. This might take about 15 minutes. Do not wait for the task to complete but instead proceed to the next task. Consider revisiting this task later on.

   > **Note:** Alternatively, to verify the status of the remediation task, you can navigate back to the **Configure Windows Arc-enabled machines to run Azure Monitor Agent** policy compliance page, select the **Activity Logs** button in the toolbar, and review the log entries. You can use the log to track progress of the remediation activities. At some point, the log should display the entries referencing invocation of the **deployIfNotExists** policy action and installation of an Azure Arc extension.

1. Once the remediation task completes, navigate back to the **Configure Windows Arc-enabled machines to run Azure Monitor Agent** policy compliance page and verify that all four resources are listed as compliant.

   > **Note:** You might need to refresh the page to display the updated compliance status.

1. To validate the results in another way, in the Azure portal, navigate back to the **Azure Arc \| Machines** page and select the **SEA-ADM1** entry.
1. On the **SEA-ADM1** page, in the vertical navigation menu on the left side, select **Extensions** and verify that the **AzureMonitorWindowsAgent** appears on the list of installed extensions.

### Exercise 3: Monitor Azure Arc-enabled Windows servers by using Azure Monitor

In this exercise, you will configure monitoring of Azure Arc-enabled on-premises Windows Servers. The exercise will involve the following on-premises server:

- SEA-DC1

The exercise consists of the following tasks:

- Task 1: Configure VM Insights for Azure Arc-enabled Windows Servers
- Task 2: Review the monitoring capabilities of Azure Arc-enabled Windows Servers

#### Task 1: Configure VM Insights for Azure Arc-enabled Windows Servers

In this task, you will configure VM Insights for one of the four Windows Server resources you onboarded to Azure Arc earlier in this lab. Note that you could alternatively manage this configuration by using Azure Policy for multiple servers, similarly to how you deployed the Azure Monitor agent in the previous exercise.

> **Note:** To start, you will create an Azure Log Analytics workspace dedicated to the use of telemetry generated by Arc-enabled Windows Servers.

1. While signed to the server SEA-ADM1, in the web browser window displaying the Azure portal at `https://portal.azure.com`, in the **Search** text box, search for and select **Log Analytics workspaces**.
1. On the **Log Analytics workspaces** page, select **+ Create**.
1. On the **Basics** tab of the **Create Log Analytics workspace** page, specify the following settings and then select **Review + Create**:

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|**arc-01-RG**|
   |Name|**arc-01-laworkspace**|
   |Region|The name of the same Azure region you were using in the previous exercises of this lab|

1. On the **Review + Create** tab, wait for the validation to complete and then select **Create**.

   > **Note:** Wait for the provisioning process to complete. This should take about 1 minute. 

1. While signed to the server SEA-ADM1, in the web browser window displaying the Azure portal at `https://portal.azure.com`, from the **Azure Arc \| Machines** page select the **SEA-DC1** entry.
1. On the **SEA-DC1** page, in the vertical navigation menu on the left side, in the **Monitoring** section, select **Insights**.
1. On the **SEA-DC1 \| Insights** page, select **Enable**. 
1. In the **Monitoring configuration** pane, below the **Data collection rule** drop-down list, select **Create New**.

   > **Note:** You will create a new data collection rule that includes processes and dependencies in order to enable the Map functionality.

1. In the **Create new rule** pane, specify the following settings:

   |Setting|Value|
   |---|---|
   |Data collection rule name|**arc-01-datamap-DCR**|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Enable processes and dependencies (Map)|enabled|
   |Log Analytics workspaces|**arc-01-laworkspace**|

1. Select **Create**.
1. Back in the **Monitoring configuration** pane, select **Configure**.

   > **Note:** Do not wait for the completion of monitoring configuration (you can use the notification area accessible via the bell icon in the toolbar of the Azure portal page to track the progress) but instead proceed to the next task. The monitoring configuration process might take about 10 minutes to complete.

#### Task 2: Review monitoring capabilities of Azure Arc-enabled Windows Servers

In this task, you will review the resulting monitoring capabilities of Azure Arc-enabled Windows servers provided by VM Insights.

> **Note:** The completion of the monitoring configuration might take several minutes. However, while you won't likely be able to view map data during that time, you should be able to access performance data. To determine whether this is the case, refresh the web browser window displaying the **SEA-DC1 \| Insights** page every minute or so until the **Enable** button no longer appears and then select the **Performance tab**.

> **Note:** To minimize the idle time, consider switching back to the task which involved implementing Azure Policy remediation task and then returning back to this task.

1. While on the **SEA-DC1 \| Insights** page, refresh the web browser page to display its updated interface that includes **Get started**, **Performance**, and **Map** tabs.
1. On the **SEA-DC1 \| Insights** page, select the **Performance** tab and review the charts displaying CPU, memory, disk, and network telemetry.
1. On the **SEA-DC1 \| Insights** page, select the **Map** tab. This interface provides data about processes running on the monitored server, along with their incoming and outgoing connections. 

   > **Note:** The **Map** functionality should become available once the monitoring configuration you initiated in the previous task completes. 

1. Expand the list of processes for the monitored server. Select one of the processes to review its details and dependencies.
1. Select **SEA-DC1** to display the server properties and, in the properties pane, select **Log Events**. This will display a table summarizing event types and their corresponding counts.
1. To view actual logged events, select any of the event type entries. You will be redirected to the Log Analytics workspace where the events are collected. From this interface, you can examine individual log entries in detail. 

### Exercise 4: Enhance security of Azure Arc-enabled Windows servers by using Microsoft Defender for Cloud

In this exercise, you will configure extra protection of Azure Arc-enabled on-premises Windows Server resources by using Microsoft Defender for Cloud. The exercise will involve the following on-premises server:

- SEA-DC1

The exercise consists of the following tasks:

- Task 1: Configure Microsoft Defender for Cloud-based protection of Azure Arc-enabled Windows servers 
- Task 2: Review the Microsoft Defender for Cloud-based protection of Azure Arc-enabled Windows servers 
- Task 3: Implement Azure-based update management of Azure Arc-enabled Windows servers 

#### Task 1: Configure Microsoft Defender for Cloud-based protection of Azure Arc-enabled Windows servers 

In this task, you will enable protection of Arc-enabled Windows servers by Microsoft Defender for Cloud. You will use for this purpose the Log Analytics workspace you created in the previous exercise.

1. While signed to the server SEA-ADM1, in the web browser window displaying the Azure portal at `https://portal.azure.com`, in the **Search** text box, search for and select **Microsoft Defender for Cloud**.
1. On the **Microsoft Defender for Cloud \| Overview** page, in the navigation menu on the left side, select **Getting started**.
1. On the **Microsoft Defender for Cloud \| Getting started** page, on the **Upgrade** tab, in the **Enable Defender for Cloud on 1 subscription** section, ensure that the checkbox next to the Azure subscription entry is selected and then select **Upgrade**.

   > **Note:** This will initiate a 30-day trial of Microsoft Defender for Cloud.

1. Once redirected to the **Install agents** tab, select **Install agents**.
1. On the **Microsoft Defender for Cloud \| Getting started** page, in the navigation menu on the left side, in the **Management** section, select **Environment settings**.
1. On the **Microsoft Defender for Cloud \| Environment settings** page, expand the hierarchy consisting of Azure management groups, your Azure subscription, and the newly created Log Analytics workspace, and then select the entry representing your Azure subscription. 
1. On the **Settings \| Defender plans** page, in the **Cloud Workload Protection (CWP)** section, for the **Servers** plan, if needed, set the **Status** to **On** and then select the **Settings** link.
1. On the **Settings & monitoring** page, if needed, set **Vulnerability assessment for machines** to **On** and select **Edit configuration**.
1. In the **Extension deployment configuration** pane, set the **Vulnerability assessment for machines** option to **Off** and then select **Continue**.

   > **Note:** In general, you should consider enabling this option, however, you'll examine another method of implementing it later in this exercise.

1. Back on the **Settings \| Defender plans** page, select **Save**.
1. Navigate back to the **Microsoft Defender for Cloud \| Environment settings** page and, in the hierarchy displaying the management groups, the Azure subscription, and the Log Analytics workspace, select the **arc-01-laworkspace** entry.
1. On the **Settings \| Defender plans** page, set the **Plan** for **Servers** to **On** and then select **Save**.
1. In the **Search** text box, search for and select **Log Analytics workspaces**.
1. On the **Log Analytics workspaces** page, select **arc-01-laworkspace**.
1. On the **arc-01-laworkspace \| Overview** page, in the navigation menu on the left side, in the **Settings** section, select **Agents**.
1. On the **arc-01-laworkspace \| Agents** page, on the **Windows servers** tab, select **Data Collection Rules**.
1. On the **Data collection rules** page, select the **MSVMI-arc-01-datamap-DCR** entry.
1. On the **MSVMI-arc-01-datamap-DCR** page, in the navigation menu on the left side, in the **Configuration** section, select **Data sources**.
1. On the **MSVMI-arc-01-datamap-DCR \| Data sources** page, select **+ Add**.
1. In the **Add data source** pane, on the **Data source** tab, in the **Data source type** drop-down list, select **Windows Event Logs**.
1. Keep the **Basic** level of event log collection selected, in the **Security** section, select **Audit success** and **Audit failure**, and then select the **Destination** tab.
1. Verify that the **Destination type** is set to **Azure Monitor Logs** in the **arc-01-laworkspace (arc-01-RG)** and then select **Add data source**.

#### Task 2: Review the Microsoft Defender for Cloud-based protection of Azure Arc-enabled Windows servers 

In this task, you will review the Microsoft Defender for Cloud-based protection capabilities available to Azure Arc-enabled Windows servers .

1. While signed to the server SEA-ADM1, in the web browser window displaying the Azure portal, navigate back to the **Microsoft Defender for Cloud \| Overview** page.
1. On the **Microsoft Defender for Cloud \| Overview** page, in the navigation menu on the left side, in the **General** section, select **Inventory**.
1. On the **Microsoft Defender for Cloud \| Inventory** page, in the list of servers, select the **sea-dc1** entry.
1. On the **Resource health** page of the **sea-dc1** server, review the list of recommendations.
1. Select the high severity recommendation **Machines should have a vulnerability assessment solution**.
1. On the **Machines should have a vulnerability assessment solution** page, expand the **Remediation steps** section, select **Quick fix logic**, review the **Automatic remediation script content**, select **Close**, and then select **Fix**.
1. On the **Machines should have a vulnerability assessment solution** page, ensure that the option **Microsoft Defender vulnerability management (included with Microsoft Defender for servers)** is selected and then select **Proceed**.
1. On the **Fixing resources** page, select **Fix 1 resource**.

   > **Note:** Do not wait for the remediation to complete successfully. This should take less than 1 minute, however, after the process completes, it may take up to 24 hours before the corresponding resource is marked as healthy. 

#### Task 3: Implement Azure-based update management of Azure Arc-enabled Windows servers 

In this task, you will implement update management of Azure Arc-enabled Windows servers by using Azure Update Manager.

> **Note:** For update management to work, you have to first enable and start the Windows Update service on the target Windows systems. The service is intentionally disabled on the lab virtual machines to prevent from interruptions due to updates. You will temporarily enable it on **SEA-DC1**.

1. While signed to the server SEA-ADM1, switch to the Windows PowerShell ISE window.
1. Open another tab in the Windows PowerShell ISE script pane and run the following script to enable and start the Windows Update service:

   ```powershell
   $serviceName = 'wuauserv'
   $servers = 'SEA-DC1'

   Invoke-Command -ComputerName $servers -ScriptBlock {
      param($serviceName)
      Set-Service -Name $serviceName -StartupType Automatic
      Start-Service -Name $serviceName
   } -ArgumentList ($serviceName)
   ```

1. While signed to the server SEA-ADM1, switch to the web browser window displaying the Azure portal at `https://portal.azure.com`, from the **Azure Arc \| Machines** page select the **SEA-DC1** entry.
1. On the **SEA-DC1** page, in the vertical navigation menu on the left side, in the **Operations** section, select **Updates**.
1. On the **SEA-DC1 \| Updates** page, in the **Guest OS updates** section, select **Go to Updates by using Azure Update Manager**.
1. On the **SEA-DC1 \| Updates** page, select **Check for updates** and, in the **Trigger assess now**, select **OK**.

   > **Note:** Do not wait for the assessment to complete, but instead proceed to the next step. The assessment might take about 4 minutes to complete.

1. On the **SEA-DC1 \| Updates** page, select **Update settings**, on the **Change update settings** page, review the existing settings and then select **Cancel**.

   > **Note:** Hotpatching is available only for Azure VMs running Windows Server Datacenter Azure Edition. Patch orchestration is not appliable to Arc-enabled servers.

1. On the **SEA-DC1 \| Updates** page, select **Schedule updates**. 
1. On the **Basics** tab of the **Create a maintenance configuration** page, specify the following settings:

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|**arc-01-RG**|
   |Configuration name|**arc-01-um-configuration**|
   |Region|The name of the same Azure region you were using in the previous exercises of this lab|
   |Maintenance scope|**Guest (Azure VM, Arc-enabled VMs/servers**|
   |Reboot setting|**Reboot if required**|

1. Select **Add a schedule**, in the **Add/Modify schedule** pane, specify the following settings and then select **Save**:

   |Setting|Value|
   |---|---|
   |Start on|next Saturday's date at **9:00 PM** of your local time zone|
   |Maintenance window|**3** Hours and **0** Minutes|
   |Repeats|**1 Week** on **Saturday**|
   |Add end date|disabled|

1. Back on the **Basics** tab of the **Create a maintenance configuration** page, select **Next: Dynamic scopes >**:
1. On the **Dynamic scopes** tab, select **Next: Resources >**.

   > **Note:** Dynamic scopes allow you to narrow down the scope of configuration by using such criteria as resource groups, locations, operating system types, or tags.

1. On the **Resources tab, verify that the **DC1** appears in the list of resources and select **Next: Updates >**.
1. On the **Updates** tab, review the existing settings without making any changes and select **Review + create**.

   > **Note:** You have the option of including update classifications as well as including and excluding individual KB ID/packages.

1. On the **Review + create** tab, wait for the validation to complete and then select **Create**.

   > **Note:** Do not wait for the maintenance configuration setup to complete but instead proceed to the next step. The setup should complete within 1 minute. 

1. Navigate back to the **SEA-DC1 \| Updates** page and select **One-time update**.
1. On the **Machines** tab of the **Install one-time updates** page, select the checkbox next to the **SEA-DC1** entry and then select **Next**.
1. On the **Updates** tab, review the selected updates to install and select **Next**. 

   > **Note:** You have the option of including and excluding individual KB ID/packages.

1. On the **Properties** tab, in the **Reboot option** drop-down list, select **Reboot if required** and select **Next**. 
1. On the **Review + install** tab, review the resulting settings and select **Install**.

   > **Note:** Do not wait for the update installation to complete. You can review it later on by reviewing the **History** tab on the **SEA-DC1 \| Updates** page in the Azure portal or by triggering another assessment against the **SEA-DC1** from the same page. 
