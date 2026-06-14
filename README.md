## Project Architecture Overview

The final architecture consists of:
1. **Two Production Web Hubs**: Located in **Canada Central** (Region 1) and **Spain Central** (Region 2).
2. **Local High Availability**: A Standard Layer-4 Public Load Balancer per region managing an isolated pool of two virtual web server nodes configured in a stateless Round-Robin rotation.
3. **Global Traffic Optimization**: An Azure Traffic Manager profile leveraging the **Performance** routing method to minimize latency by dynamically directing users to their closest regional cluster.
4. **Independent Validation Sandbox**: A standalone Windows 11 client deployment inside **South Africa North** (Region 3) to accurately test cross-continental failover and geolocation routing.

---

## Subnetting Plan & Address Space Calculations

To adhere to the absolute isolation constraints required for your deployment, we must operate entirely within the assigned private address block **`192.168.30.0/24`**. This provides a total pool of **256 continuous IP addresses** ($2^8 = 256$).

To prevent network overlapping across our three global regions, the `/24` block is divided into **four distinct subnets** using a `/26` variable-length subnet mask (VLSM). Each subnet provides **64 total address slots** ($2^6 = 64$):

### 🔢 Core Network Partition Matrix

| Subnet Name | Purpose / Workload Target | Network Base Range | CIDR Suffix | Usable IP Scope Range |
| :--- | :--- | :--- | :--- | :--- |
| `Canada-Subnet` | Region 1 Web Servers (Canada Central) | `192.168.30.0` | `/26` | `192.168.30.1` to `192.168.30.62` |
| `Spain-Subnet` | Region 2 Web Servers (Spain Central) | `192.168.30.64` | `/26` | `192.168.30.65` to `192.168.30.126` |
| `Testing-Subnet` | Region 3 Validation Client (South Africa) | `192.168.30.128` | `/26` | `192.168.30.129` to `192.168.30.190` |
| *Reserved Pool* | Unassigned / Future Infrastructure Expansion | `192.168.30.192` | `/26` | `192.168.30.193` to `192.168.30.254` |

> ⚠️ **CRITICAL AZURE SYSTEM NOTE:** Azure automatically reserves **5 IP addresses** within *every* custom subnet created:
> * `x.x.x.0`: Network Base Address.
> * `x.x.x.1`: Default Subnet Gateway Routing Interface.
> * `x.x.x.2` / `x.x.x.3`: Azure Internal DNS Mapping & Address Resolution Services.
> * `x.x.x.63` (or final index): Subnet Broadcast Address.

---

## Execution Phases: Step-by-Step Implementation

### Phase 1: Resource Group Provisioning

The resource group establishes the unified management boundary for our global assets.

1. Authenticate to the [Azure Portal](https://portal.azure.com) using your assigned student credential context (`odl_user`).
2. Click into the global **Search bar** at the top center of the portal workspace.
3. Type **Resource groups** and click on it from the dropdown choices.
4. On the top-left command toolbar, click the **`+ Create`** button.
5. In the **Basics** tab, configure the following explicit parameters:
   * **Subscription**: Retain your default lab subscription.
   * **Resource group**: Enter `YourName-S64P-RG` (Replace `YourName` with your actual name/initials, ensuring zero spaces).
   * **Region**: Select **(US) East US**.
6. Click **`Review + create`** at the bottom left.
7. Wait for validation to pass, then click **`Create`**.

---

### Phase 2: Region 1 Infrastructure — Canada Central

#### Task 1: Virtual Network (VNet) Deployment
1. Type **Virtual networks** in the top global search bar and select it.
2. Click **`+ Create`** on the top toolbar.
3. In the **Basics** tab, provide the following precise parameters:
   * **Resource Group**: Select your container `YourName-S64P-RG`.
   * **Name**: Enter `YourName-VNet-R1`.
   * **Region**: Select **Canada Central**.
4. Click **`Next: Security >`** and bypass to proceed to the **`IP Addresses`** workspace sheet.
5. Delete any auto-populated default IP entries by clicking the trash icon next to the address line.
6. In the primary IPv4 address space text block, enter: **`192.168.30.0/24`**.
7. Click the **`+ Add a subnet`** button and declare the following values inside the flyout panel:
   * **Subnet Name**: `Canada-Subnet`
   * **Starting Address**: `192.168.30.0`
   * **Subnet Size**: Select `/26 (64 addresses)`.
8. Click **`Add`** at the bottom of the flyout panel.
9. Click **`Review + create`**, verify validation, and then click **`Create`**.

#### Task 2: Provisioning Web Servers (Canada-WebSrv1 & Canada-WebSrv2)
*Execute this complete sequence **twice** to create two identical virtual machines.*

1. Search for **Virtual machines** in the global lookup bar and select it.
2. Click **`+ Create`** and select **`Azure virtual machine`**.
3. In the **Basics** tab, map the following system values carefully:
   * **Resource Group**: `YourName-S64P-RG`.
   * **Virtual machine name**: `Canada-WebSrv1` *(Use `Canada-WebSrv2` for the second loop)*.
   * **Region**: **Canada Central**.
   * **Availability options**: `No infrastructure redundancy required`.
   * **Security type**: `Standard`.
   * **Image**: `Windows Server 2025 Datacenter: x64 Gen2`.
   * **Size**: `Standard_D2s_v3` (2 vCPUs, 8 GiB Memory).
   * **Username**: `localadmin`.
   * **Password**: Create a secure, complex password matching platform requirements.
   * **Public inbound ports**: Select **`Allow selected ports`**.
   * **Select inbound ports**: Check both **`RDP (3389)`** and **`HTTP (80)`** options.
4. Click **`Next: Disks >`**. Under the **OS disk type** parameter line, change the selection to **`Standard HDD`**.
5. Click **`Next: Networking >`**. Verify that the interface has mapped dynamically to:
   * **Virtual Network**: `YourName-VNet-R1`
   * **Subnet**: `Canada-Subnet (192.168.30.0/26)`
   * **Public IP**: Retain the auto-generated parameter.
6. Click **`Next: Management >`** and move to the **`Monitoring`** tab. Change **Boot diagnostics** to **`Disable`**.
7. Click **`Review + create`**, verify validation parameters, and click **`Create`**.

---

### Phase 3: Region 2 Infrastructure — Spain Central

#### Task 1: Virtual Network (VNet) Deployment
1. Navigate to **Virtual networks** from the center search utility window.
2. Click **`+ Create`** on the top command bar.
3. In the **Basics** configuration layout page, define these fields:
   * **Resource Group**: `YourName-S64P-RG`.
   * **Name**: `YourName-VNet-R2`.
   * **Region**: **Spain Central**.
4. Move directly to the **`IP Addresses`** section panel. Delete the default range using the trash bin utility icon.
5. Under the IPv4 address string input container box, enter: **`192.168.30.0/24`**.
6. Click **`+ Add a subnet`** and enter these exact metrics inside the properties panel layout:
   * **Subnet Name**: `Spain-Subnet`
   * **Starting Address**: `192.168.30.64`
   * **Subnet Size**: Select `/26 (64 addresses)`.
7. Click **`Add`**, select **`Review + create`**, pass validation checks, and click **`Create`**.

#### Task 2: Provisioning Web Servers (Spain-WebSrv1 & Spain-WebSrv2)
*Execute this complete sequence **twice** to create two identical virtual machines.*

1. Navigate to **Virtual machines** and select **`+ Create -> Azure virtual machine`**.
2. In the **Basics** profile grid layout, configure the system properties matrix:
   * **Resource Group**: `YourName-S64P-RG`.
   * **Virtual machine name**: `Spain-WebSrv1` *(Use `Spain-WebSrv2` for the second loop)*.
   * **Region**: **Spain Central**.
   * **Availability options**: `No infrastructure redundancy required`.
   * **Security type**: `Standard`.
   * **Image**: `Windows Server 2025 Datacenter: x64 Gen2`.
   * **Size**: `Standard_D2s_v3`.
   * **Username**: `localadmin`.
   * **Password**: Input your chosen secure complex password credentials string.
   * **Public inbound ports**: Select **`Allow selected ports`**.
   * **Select inbound ports**: Check both **`RDP (3389)`** and **`HTTP (80)`**.
3. Click **`Next: Disks >`** and adjust the OS disk storage parameter selector to **`Standard HDD`**.
4. Click **`Next: Networking >`** and map to these specific configuration layers:
   * **Virtual Network**: `YourName-VNet-R2`
   * **Subnet**: `Spain-Subnet (192.168.30.64/26)`
5. Click **`Next: Management >`**, select **`Monitoring`**, and set **Boot diagnostics** to **`Disable`**.
6. Click **`Review + create`**, pass the validation check, and click **`Create`**.

---

### Phase 4: Web Hub Environment Customization

This sequence must be executed on **all four deployed VM nodes** individually to transform them into active web server instances displaying your specific student metadata.

1. Navigate to the resource blade page of your target running virtual machine (e.g., `Canada-WebSrv1`).
2. Scroll down the left settings blade sidebar until reaching the **`Help`** section group; click on **`Run command`**.
3. Select **`RunPowerShellScript`** from the action layout template listing.
4. Copy and paste the following PowerShell script block into the command code editor terminal box, replacing the uppercase variables with your matching data parameters:
---

### Phase 5: Regional High Availability Load Balancers

*Execute this design lifecycle configuration cycle **twice**: once for Region 1 (Canada) and once for Region 2 (Spain).*

1. Enter **Load balancers** inside the top lookup component engine interface and click on it.
2. Select **`+ Create`** on the top actions task utility header.
3. Under the **Basics** configuration tab grid, enter the parameters exactly:
   * **Resource Group**: `YourName-S64P-RG`.
   * **Name**: `YourName-LB-R1` *(Use `YourName-LB-R2` for Spain)*.
   * **Region**: **Canada Central** *(Use **Spain Central** for the second LB)*.
   * **SKU**: `Standard`.
   * **Type**: `Public`.
   * **Tier**: `Regional`.
4. Click **`Next: Frontend IP configuration >`**. Select **`+ Add a frontend IP configuration`**:
   * **Name**: `YourName-FE-R1` *(Use `YourName-FE-R2` for Spain)*.
   * **IP type**: `IP address`.
   * **Public IP address**: Select **`Create new`**.
     * *Flyout Panel Config*: Name it `YourName-LBPip-R1` *(or `YourName-LBPip-R2`)*. Retain Standard and Static option controls as default. Click **`OK`**.
   * Click **`Add`** to commit the frontend mapping properties.
5. Click **`Next: Backend pools >`**. Select **`+ Add a backend pool`**:
   * **Name**: `YourName-BE-R1` *(Use `YourName-BE-R2` for Spain)*.
   * **Virtual network**: `YourName-VNet-R1` *(Use `YourName-VNet-R2` for Spain)*.
   * **Backend Pool Configuration**: Select **`NIC`**.
   * Under the Virtual Machines section line, click **`+ Add`**. Check both matching local computing target nodes (e.g., both Canada VMs for LB1). Click **`Add`**.
   * Click **`Save`** / **`Add`** to store the logical asset cluster.
6. Click **`Next: Inbound rules >`**. Select **`+ Add a load balancing rule`**:
   * **Name**: `YourName-LBRule-R1` *(Use `YourName-LBRule-R2` for Spain)*.
   * **IP Version**: `IPv4`.
   * **Frontend IP address**: Select your created frontend tracking string from the list.
   * **Backend pool**: Select your matching backend pool tracking string from the list.
   * **Protocol / Port / Backend Port**: Select **`TCP`** and enter **`80`** for both port settings fields.
   * **Health probe**: Select **`Create new`**. Set Name to `YourName-HP-R1` *(or `YourName-HP-R2`)*, Protocol to `TCP`, Port to `80`, Interval to `5`. Click **`Save`**.
   * **Session persistence**: Select **`None`** *(Mandatory for verifying clean round-robin distribution during screen captures)*.
7. Click **`Add`**, then choose **`Review + create`**, pass validation verification, and click **`Create`**.

---

### Phase 6: Global Traffic Manager Profile Performance Engine

#### Task 1: Setting up the Traffic Routing Profile
1. Search for **Traffic Manager profiles** in the global centralized portal search container tool and click it.
2. Click **`+ Create`** on the action operations context panel grid line.
3. Fill out the creation pane fields as specified:
   * **Name**: Choose a unique, lowercase URL prefix, such as `yournamemulti-regionwebsite` (This registers your public global URL routing node: `yournamemulti-regionwebsite.trafficmanager.net`).
   * **Routing method**: Select **`Performance`** *(This choice forces geographical latency routing to send clients to their closest active region automatically)*.
   * **Subscription**: Choose your active subscription framework container.
   * **Resource Group**: Select your container group `YourName-S64P-RG`.
4. Click **`Create`**.

#### Task 2: Linking Endpoints
1. Open your newly deployed Traffic Manager profile resource dashboard.
2. Click **`Endpoints`** inside the left side menu under the **Settings** group categorization.
3. Click **`+ Add`** from the upper command bar interface to link your first region.
4. Complete the entry parameters exactly:
   * **Type**: `Azure endpoint`.
   * **Name**: `Canada-Endpoint`.
   * **Target resource type**: `Public IP address`.
   * **Target public IP address**: Select your Canada Load Balancer Public IP identity context string (**`YourName-LBPip-R1`**).
5. Click **`Add`**.
6. Repeat steps 3–5 to link your second region, naming it `Spain-Endpoint` and linking it to **`YourName-LBPip-R2`**.

---

### Phase 7: Region 3 Testing Client Workspace Sandbox

#### Task 1: Setting up the Client VNet Architecture
1. Navigate to **Virtual networks** and click **`+ Create`**.
2. On the **Basics** tab, apply these settings:
   * **Resource Group**: `YourName-S64P-RG`.
   * **Name**: `YourName-VNet-R3`.
   * **Region**: **South Africa North**.
3. Move to the **`IP Addresses`** page, wipe out the default address entries, and type **`192.168.30.0/24`** inside the primary box container.
4. Click **`+ Add a subnet`** and provision the following boundary values:
   * **Subnet Name**: `Testing-Subnet`
   * **Starting Address**: `192.168.30.128`
   * **Subnet Size**: Select `/26 (64 addresses)`.
5. Click **`Add`**, select **`Review + create`**, pass verification, and click **`Create`**.

#### Task 2: Provisioning the South Africa Windows 11 VM
1. Navigate to **Virtual machines** and select **`+ Create -> Azure virtual machine`**.
2. In the **Basics** workspace pane configuration layer, apply these fields:
   * **Resource Group**: `YourName-S64P-RG`.
   * **Virtual machine name**: `South-AfricaVM`.
   * **Region**: **South Africa North**.
   * **Image**: `Windows 11 Pro, version 23H2 - x64 Gen2`.
   * **Size**: `Standard_D2s_v3`.
   * **Username**: `localadmin`.
   * **Password**: Supply your standardized, secure password matrix string.
   * **Public inbound ports**: Select **`Allow selected ports`** and check **`RDP (3389)`** from the option selection box.
3. Click **`Next: Disks >`** and adjust the storage line profile to **`Standard HDD`**.
4. Click **`Next: Networking >`** and verify the mapping bounds:
   * **Virtual Network**: `YourName-VNet-R3`
   * **Subnet**: `Testing-Subnet (192.168.30.128/26)`
5. Go to **Management** -> **`Monitoring`** and set **Boot diagnostics** to **`Disable`**.
6. Click **`Review + create`**, pass validation checks, and click **`Create`**.

---

## Documentation Capture & Verification Workflow

To secure full point metrics for your project evaluation checklist, follow these validation execution procedures precisely and document the required screen state captures.

### Step 1: Physical Local Machine Validation Context
1. On your physical desktop computer interface environment, clear out cached local DNS lookup tracking blocks. To execute this on Windows 11, open the **Start Menu**, search for `cmd`, select **Command Prompt**, input the exact command string line: `ipconfig /flushdns` inside the terminal workspace area, and press **Enter**.
2. Launch a new web browser software window, enter your specific global Traffic Manager routing FQDN URL link path mapping into the browser locator container box: `http://yournamemulti-regionwebsite.trafficmanager.net/` and press **Enter**.
3. Verify that the webpage safely loads and echoes your specific student identification details matched with the regional point located closest to your actual physical geolocation coordinates (e.g., resolving directly to the Canada Central hub if you are situated inside North America).

> 📸 **SCREENSHOT REMINDER:** Capture a full-window crop layout rendering of your web browser indicating successful endpoint display. Your unique administrative identity login context tag `odl_user` situated in the top right profile header band of your Azure dashboard page context must be visible and highlighted clearly for verification.

### Step 2: Cross-Continental Geolocation Client Mapping Test
1. In the Azure Portal, open the page for your active `South-AfricaVM` and copy its **Public IP address**.
2. On your local machine, open the native **Remote Desktop Connection** utility (click Start, search for the tool, and open it).
3. Input the copied Public IP address context block matching your South Africa instance and click **`Connect`**. Provide the access credentials (`localadmin` / your secure password) when prompted.
4. Inside the interactive Remote Desktop window session layout, launch the internal **Microsoft Edge** browser software package.
5. In the navigation tracking bar, enter your root Traffic Manager FQDN path link sequence string: `http://yournamemulti-regionwebsite.trafficmanager.net/` and press **Enter**.
6. Verify that the routing logic directs you to the closest node from a South African perspective—meaning it will route traffic to the European node cluster (**Spain-WebSrv1** or **Spain-WebSrv2**).

> 📸 **SCREENSHOT REMINDER:** Take a full screenshot capture encompassing your entire active Remote Desktop window workspace field. The browser inside the virtual machine must display web server content from the Spain Central computing cluster to confirm that regional performance routing functions as intended.

### Step 3: Layer-4 Round-Robin & Outage Failover Verification Checklist
1. Within the browser window running inside your remote desktop session to `South-AfricaVM`, select the address entry bar.
2. Click the refresh button **10 consecutive times** slowly without changing browser tabs or window Focus contexts.
3. Confirm that the node instance line changes (alternating between displaying web server content from `Spain-WebSrv1` and `Spain-WebSrv2` dynamically). This proves that stateless Round-Robin rotation rules are executing correctly inside the local load balancers.
4. Return to your primary cloud management dashboard screen, navigate to **Virtual machines**, and select the checkboxes next to both Region 1 nodes (**Canada-WebSrv1** and **Canada-WebSrv2**).
5. Click **`Stop`** on the command ribbon toolbar to turn off these systems and simulate a complete regional facility failure.
6. Return to your local machine web browser tab from Step 1. Click the reload button.
7. Verify that the website stays online. The Traffic Manager engine must automatically route your connection request to the active Spain Central web server nodes instead, proving global resilience.

> 📸 **SCREENSHOT REMINDER:** Capture a full-window screen clip of your local computer browser showing that the website remains active after the complete shutdown of Region 1 systems. This screenshot must explicitly verify that your traffic was redirected to the Region 2 (Spain Central) backup infrastructure cluster seamlessly.
