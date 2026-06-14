markdown_guide = """# Comprehensive Implementation Guide: Multi-Region Azure Website Deployment with Global Performance Routing

This repository provides an expert-level, step-by-step technical manual for implementing a high-availability, fault-tolerant website architecture across multiple geographic regions in Microsoft Azure.

---

## 🗺️ Project Architecture Overview

The final architecture consists of:
1. **Two Production Web Hubs**: Located in **Canada Central** (Region 1) and **Spain Central** (Region 2).
2. **Local High Availability**: A Standard Layer-4 Public Load Balancer per region managing an isolated pool of two virtual web server nodes configured in a stateless Round-Robin rotation.
3. **Global Traffic Optimization**: An Azure Traffic Manager profile leveraging the **Performance** routing method to minimize latency by dynamically directing users to their closest regional cluster.
4. **Independent Validation Sandbox**: A standalone Windows 11 client deployment inside **South Africa North** (Region 3) to accurately test cross-continental failover and geolocation routing.

---

## 🧮 Subnetting Plan & Address Space Calculations

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

## 🚀 Execution Phases: Step-by-Step Implementation

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
