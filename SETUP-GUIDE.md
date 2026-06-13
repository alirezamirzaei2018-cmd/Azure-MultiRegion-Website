# MST400 – Project 1 Setup Guide
# Azure Multi-Region Website Deployment with Global Load Distribution

**Student:** Alireza  
**Course:** MST400 – Cloud Administration  
**Semester:** Summer 2026  
**Institution:** Seneca Polytechnic

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [IP Address Plan](#ip-address-plan)
3. [Phase 1 — Create Resource Group](#phase-1--create-resource-group)
4. [Phase 2 — Create Virtual Networks](#phase-2--create-virtual-networks)
5. [Phase 3 — Create Virtual Machines](#phase-3--create-virtual-machines)
6. [Phase 4 — Install and Configure IIS](#phase-4--install-and-configure-iis)
7. [Phase 5 — Create Load Balancers](#phase-5--create-load-balancers)
8. [Phase 6 — Create Traffic Manager](#phase-6--create-traffic-manager)
9. [Phase 7 — Create South Africa Testing VM](#phase-7--create-south-africa-testing-vm)
10. [Phase 8 — Demo and Testing](#phase-8--demo-and-testing)
11. [Troubleshooting](#troubleshooting)
12. [Cleanup](#cleanup)

---

## Prerequisites

Before starting this project, make sure you have:

- Access to Microsoft Azure portal at [portal.azure.com](https://portal.azure.com)
- An active **odl_user** account (Seneca Labs subscription)
- Your assigned IP address space: `192.168.18.0/24`
- A local computer with RDP client (built into Windows)

> ⚠️ **Important:** Always use your **odl_user** account, not a personal Microsoft account. Using the wrong account will result in a grade of 0.

---

## IP Address Plan

Before creating any resources, plan your subnets:

| Region | VNet Name | Subnet Name | Subnet CIDR | Usable Hosts |
|--------|-----------|-------------|-------------|--------------|
| Canada Central (R1) | Alireza-VNet-R1 | Alireza-Subnet-R1 | 192.168.18.0/26 | 62 hosts |
| Spain Central (R2) | Alireza-VNet-R2 | Alireza-Subnet-R2 | 192.168.18.64/26 | 62 hosts |
| South Africa North (R3) | Alireza-VNet-R3 | Alireza-Subnet-R3 | 192.168.18.128/26 | 62 hosts |

**VM IP Assignments:**

| VM Name | Private IP | Region |
|---------|------------|--------|
| Alireza-Canada-WebSrv1 | 192.168.18.4 | Canada Central |
| Alireza-Canada-WebSrv2 | 192.168.18.5 | Canada Central |
| Alireza-Spain-WebSrv1 | 192.168.18.68 | Spain Central |
| Alireza-Spain-WebSrv2 | 192.168.18.69 | Spain Central |
| Alireza-SouthAfrica-Client | 192.168.18.132 | South Africa North |

---

## Phase 1 — Create Resource Group

A Resource Group is a container that holds all Azure resources for this project.

**Step 1** — Sign in to [portal.azure.com](https://portal.azure.com) using your **odl_user** account

**Step 2** — In the top search bar, type `Resource groups` → click on it

**Step 3** — Click **+ Create**

**Step 4** — Fill in the fields:

| Field | Value |
|-------|-------|
| Subscription | *(your odl subscription — leave as default)* |
| Resource group name | `Alireza-S64P-RG` |
| Region | **(Canada) Canada Central** |

**Step 5** — Click **Review + Create** → Click **Create**

**Step 6** — Wait for deployment to complete → Click **Go to resource group**

> 📸 Take a screenshot of the resource group overview page showing the name and location.

---

## Phase 2 — Create Virtual Networks

You need 3 Virtual Networks — one for each region. Repeat these steps 3 times with different values.

### VNet for Region 1 — Canada Central

**Step 1** — In the top search bar, type `Virtual networks` → click on it

**Step 2** — Click **+ Create**

**Step 3** — On the **Basics** tab, fill in:

| Field | Value |
|-------|-------|
| Subscription | *(default)* |
| Resource group | `Alireza-S64P-RG` |
| Name | `Alireza-VNet-R1` |
| Region | **(Canada) Canada Central** |

**Step 4** — Click the **IP Addresses** tab at the top

**Step 5** — Delete the default address space by clicking the **trash icon** next to it

**Step 6** — Click **+ Add an IP address space** and enter:
- Address space: `192.168.18.0/24`

**Step 7** — Delete the default subnet by clicking the **three dots (...)** next to it → **Delete**

**Step 8** — Click **+ Add a subnet** and fill in:

| Field | Value |
|-------|-------|
| Subnet name | `Alireza-Subnet-R1` |
| Starting address | `192.168.18.0` |
| Subnet size | `/26` |

**Step 9** — Click **Add** → Click **Review + Create** → Click **Create**

> 📸 Take a screenshot: Open VNet-R1 → click **Subnets** on the left → screenshot showing `Alireza-Subnet-R1` with `192.168.18.0/26`

---

### VNet for Region 2 — Spain Central

Repeat all steps above with these different values:

| Field | Value |
|-------|-------|
| Name | `Alireza-VNet-R2` |
| Region | **Spain Central** |
| Subnet name | `Alireza-Subnet-R2` |
| Starting address | `192.168.18.64` |
| Subnet size | `/26` |

> 📸 Take a screenshot of VNet-R2 Subnets tab showing `Alireza-Subnet-R2` with `192.168.18.64/26`

---

### VNet for Region 3 — South Africa North

Repeat all steps above with these different values:

| Field | Value |
|-------|-------|
| Name | `Alireza-VNet-R3` |
| Region | **South Africa North** |
| Subnet name | `Alireza-Subnet-R3` |
| Starting address | `192.168.18.128` |
| Subnet size | `/26` |

> 📸 Take a screenshot of VNet-R3 Subnets tab showing `Alireza-Subnet-R3` with `192.168.18.128/26`

---

## Phase 3 — Create Virtual Machines

You need to create 4 web server VMs (2 in Canada, 2 in Spain). Repeat the steps below for each VM using the values in the table.

### Common Settings for All Web Server VMs

| Field | Value |
|-------|-------|
| OS Image | Windows Server 2022 Datacenter — Gen2 |
| Size | Standard_B1s or Standard_D2s_v3 |
| Administrator username | `Alireza` |
| Password | *(create a strong password — use the same for all VMs)* |
| Public inbound ports | Allow selected ports |
| Inbound ports | RDP (3389), HTTP (80) |
| Public IP | **None** *(traffic goes through the load balancer)* |

---

### Canada-WebSrv1

**Step 1** — Search `Virtual machines` → Click **+ Create** → **Azure virtual machine**

**Step 2** — On the **Basics** tab:

| Field | Value |
|-------|-------|
| Resource group | `Alireza-S64P-RG` |
| Virtual machine name | `Alireza-Canada-WebSrv1` |
| Region | **(Canada) Canada Central** |
| Availability options | **Availability set** |
| Availability set | Click **Create new** → Name: `Alireza-Canada-AS` → OK |
| Security type | **Standard** |
| Image | **Windows Server 2022 Datacenter — Gen2** |
| Size | **Standard_D2s_v3** |
| Username | `Alireza` |
| Password | *(your password)* |
| Public inbound ports | **Allow selected ports** |
| Select ports | **RDP (3389)** and **HTTP (80)** |

**Step 3** — Click the **Networking** tab:

| Field | Value |
|-------|-------|
| Virtual network | `Alireza-VNet-R1` |
| Subnet | `Alireza-Subnet-R1` |
| Public IP | **None** |
| NIC network security group | **Basic** |
| Inbound ports | **HTTP (80)** and **RDP (3389)** |

**Step 4** — Click **Review + Create** → Click **Create**

**Step 5** — Wait for deployment to finish (2–3 minutes)

> 📸 Take a screenshot of the VM overview page showing **Running** status

---

### Canada-WebSrv2

Repeat all steps for Canada-WebSrv1 but change:

| Field | Value |
|-------|-------|
| Virtual machine name | `Alireza-Canada-WebSrv2` |
| Availability set | Select **existing** `Alireza-Canada-AS` |
| All other settings | Same as Canada-WebSrv1 |

> 📸 Take a screenshot of the VM overview page showing **Running** status

---

### Spain-WebSrv1

**Step 1** — Create VM with these settings on the **Basics** tab:

| Field | Value |
|-------|-------|
| Resource group | `Alireza-S64P-RG` |
| Virtual machine name | `Alireza-Spain-WebSrv1` |
| Region | **Spain Central** |
| Availability options | **Availability set** |
| Availability set | Click **Create new** → Name: `Alireza-Spain-AS` → OK |
| Security type | **Standard** |
| Image | **Windows Server 2022 Datacenter — Gen2** |
| Size | **Standard_D2s_v3** |
| Username | `Alireza` |
| Password | *(your password)* |
| Public inbound ports | **Allow selected ports** |
| Select ports | **RDP (3389)** and **HTTP (80)** |

**Step 2** — On the **Networking** tab:

| Field | Value |
|-------|-------|
| Virtual network | `Alireza-VNet-R2` |
| Subnet | `Alireza-Subnet-R2` |
| Public IP | **None** |
| NIC network security group | **Basic** |
| Inbound ports | **HTTP (80)** and **RDP (3389)** |

**Step 3** — Click **Review + Create** → Click **Create**

> 📸 Take a screenshot showing **Running** status

---

### Spain-WebSrv2

Repeat all steps for Spain-WebSrv1 but change:

| Field | Value |
|-------|-------|
| Virtual machine name | `Alireza-Spain-WebSrv2` |
| Availability set | Select **existing** `Alireza-Spain-AS` |
| All other settings | Same as Spain-WebSrv1 |

> 📸 Take a screenshot of all 4 VMs in the Virtual Machines list showing **Running** status

---

## Phase 4 — Install and Configure IIS

Do this on all 4 web server VMs. Since the VMs have no public IP, you need to temporarily add one to RDP in.

### Step A — Temporarily Add a Public IP to Each VM

**Step 1** — Go to the VM in Azure portal (e.g. `Alireza-Canada-WebSrv1`)

**Step 2** — Click **Networking** on the left menu → Click the **Network Interface** link

**Step 3** — Click **IP configurations** on the left → Click **ipconfig1**

**Step 4** — Under Public IP address, click **Associate** → Click **Create new**
- Name: `TempIP-WebSrv1`
- Click **OK**

**Step 5** — Click **Save** → Go back to VM overview → Copy the **Public IP address**

### Step B — RDP Into the VM

**Step 1** — On your local computer press **Windows + R** → type `mstsc` → press Enter

**Step 2** — In the Computer field type the **Public IP** of the VM → Click **Connect**

**Step 3** — Enter:
- Username: `Alireza`
- Password: *(your VM password)*

**Step 4** — Click **OK** → Click **Yes** to accept the certificate warning

### Step C — Install IIS

**Step 1** — Inside the VM, right-click the **Start** button → Click **Windows PowerShell (Admin)**

**Step 2** — Run this command:

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

**Step 3** — Wait 1–2 minutes for installation to complete

### Step D — Disable HTTP Keep-Alive on IIS

This is required for Round Robin to be visible in testing. Run this in the same PowerShell window:

```powershell
Import-Module WebAdministration
Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/httpProtocol" -name "allowKeepAlive" -value "False"
Restart-Service W3SVC
```

### Step E — Customize the IIS Default Page

**Step 1** — Open **File Explorer** → Navigate to `C:\inetpub\wwwroot\`

**Step 2** — Right-click `iisstart.htm` → **Open with** → **Notepad**

**Step 3** — Select all text (Ctrl+A) → Delete it

**Step 4** — Paste the HTML below (change VM name and region for each server):

**For Alireza-Canada-WebSrv1:**
```html
<html>
<head><title>Alireza Website</title></head>
<body style="font-family:Arial; text-align:center; margin-top:100px; background-color:#f0f8ff;">
  <h1>Welcome to Alireza's Website</h1>
  <h2>Region: Canada Central</h2>
  <h2>Server: Alireza-Canada-WebSrv1</h2>
</body>
</html>
```

**For Alireza-Canada-WebSrv2:** change the last `<h2>` to `Alireza-Canada-WebSrv2`

**For Alireza-Spain-WebSrv1:**
```html
<html>
<head><title>Alireza Website</title></head>
<body style="font-family:Arial; text-align:center; margin-top:100px; background-color:#fff8f0;">
  <h1>Welcome to Alireza's Website</h1>
  <h2>Region: Spain Central</h2>
  <h2>Server: Alireza-Spain-WebSrv1</h2>
</body>
</html>
```

**For Alireza-Spain-WebSrv2:** change the last `<h2>` to `Alireza-Spain-WebSrv2`

**Step 5** — Save the file (Ctrl+S) → Close Notepad

**Step 6** — Open **Edge** browser inside the VM → type `http://localhost` → verify your custom page appears

> 📸 Take a screenshot of the browser showing your custom page (do this for all 4 VMs)

### Step F — Remove the Temporary Public IP

**Step 1** — Close the RDP window

**Step 2** — Go back to Azure portal → VM → Networking → Network Interface → IP configurations → ipconfig1

**Step 3** — Click **Dissociate** next to the public IP → Click **Save**

**Step 4** — Go to **Public IP addresses** → find and **Delete** the temporary IP

> ⚠️ Repeat Phase 4 Steps A–F for all 4 web server VMs

---

## Phase 5 — Create Load Balancers

### Load Balancer for Canada — Alireza-LB-R1

**Step 1** — Search `Load balancers` → Click **+ Create**

**Step 2** — On the **Basics** tab:

| Field | Value |
|-------|-------|
| Resource group | `Alireza-S64P-RG` |
| Name | `Alireza-LB-R1` |
| Region | **(Canada) Canada Central** |
| SKU | **Standard** |
| Type | **Public** |
| Tier | **Regional** |

**Step 3** — Click the **Frontend IP configuration** tab → Click **+ Add a frontend IP**:

| Field | Value |
|-------|-------|
| Name | `Alireza-LB-R1-Frontend` |
| Public IP address | Click **Create new** |
| Public IP name | `Alireza-LB-R1-PublicIP` |
| Availability zone | **Zone-redundant** |
| Routing preference | **Microsoft network** |

Click **OK** → Click **Add**

**Step 4** — Click the **Backend pools** tab → Click **+ Add a backend pool**:

| Field | Value |
|-------|-------|
| Name | `Alireza-LB-R1-BackendPool` |
| Virtual network | `Alireza-VNet-R1` |
| Backend Pool Configuration | **NIC** |

Click **+ Add** → Select `Alireza-Canada-WebSrv1` → Select its NIC → Click **Add**  
Click **+ Add** again → Select `Alireza-Canada-WebSrv2` → Select its NIC → Click **Add**  
Click **Save**

**Step 5** — Click the **Inbound rules** tab → Click **+ Add a load balancing rule**:

| Field | Value |
|-------|-------|
| Name | `Alireza-LB-R1-Rule` |
| Frontend IP address | `Alireza-LB-R1-Frontend` |
| Backend pool | `Alireza-LB-R1-BackendPool` |
| Protocol | **TCP** |
| Port | `80` |
| Backend port | `80` |
| Session persistence | **None** *(this enables Round Robin)* |
| Health probe | Click **Create new** |

Health probe settings:

| Field | Value |
|-------|-------|
| Name | `Alireza-LB-R1-HealthProbe` |
| Protocol | **HTTP** |
| Port | `80` |
| Path | `/` |
| Interval | `5` |

Click **Save** → Click **Add**

**Step 6** — Click **Review + Create** → Click **Create**

> 📸 Take a screenshot of LB-R1 overview showing Frontend IP, Backend Pool (2 VMs), and Load Balancing Rule

---

### Add DNS Name Label to the Public IP

> ⚠️ This step is required before Traffic Manager can use this IP as an endpoint.

**Step 1** — Search `Public IP addresses` → Click on `Alireza-LB-R1-PublicIP`

**Step 2** — Click **Configuration** on the left menu

**Step 3** — In the **DNS name label** field type: `alireza-lb-r1`

**Step 4** — Click **Save**

---

### Load Balancer for Spain — Alireza-LB-R2

Repeat all steps for Alireza-LB-R1 but use these values:

| Field | Value |
|-------|-------|
| Name | `Alireza-LB-R2` |
| Region | **Spain Central** |
| Frontend name | `Alireza-LB-R2-Frontend` |
| Public IP name | `Alireza-LB-R2-PublicIP` |
| Backend pool name | `Alireza-LB-R2-BackendPool` |
| Virtual network | `Alireza-VNet-R2` |
| VMs in backend pool | `Alireza-Spain-WebSrv1` and `Alireza-Spain-WebSrv2` |
| Rule name | `Alireza-LB-R2-Rule` |
| Health probe name | `Alireza-LB-R2-HealthProbe` |
| DNS name label | `alireza-lb-r2` |

> 📸 Take a screenshot of LB-R2 showing Frontend IP, Backend Pool, and Load Balancing Rule

---

## Phase 6 — Create Traffic Manager

Traffic Manager routes users to the nearest region using DNS-based Performance routing.

**Step 1** — Search `Traffic Manager profiles` → Click **+ Create**

**Step 2** — Fill in:

| Field | Value |
|-------|-------|
| Name | `Alireza-TM` |
| Routing method | **Performance** |
| Subscription | *(default)* |
| Resource group | `Alireza-S64P-RG` |

**Step 3** — Click **Create**

**Step 4** — Once created, open `Alireza-TM`

> 📸 Take a screenshot of the Traffic Manager overview showing the DNS name `alireza-tm.trafficmanager.net`

**Step 5** — On the left menu click **Endpoints** → Click **+ Add**

**Step 6** — Add the Canada endpoint:

| Field | Value |
|-------|-------|
| Type | **Azure endpoint** |
| Name | `Alireza-Endpoint-Canada` |
| Target resource type | **Public IP address** |
| Target resource | `Alireza-LB-R1-PublicIP` |
| Enable Endpoint | ✅ Checked |

Click **Add**

**Step 7** — Click **+ Add** again for the Spain endpoint:

| Field | Value |
|-------|-------|
| Type | **Azure endpoint** |
| Name | `Alireza-Endpoint-Spain` |
| Target resource type | **Public IP address** |
| Target resource | `Alireza-LB-R2-PublicIP` |
| Enable Endpoint | ✅ Checked |

Click **Add**

**Step 8** — Wait 2–3 minutes then refresh the page

> 📸 Take a screenshot of the Endpoints page showing both endpoints with **Monitor Status: Online**

> ⚠️ **If endpoints are greyed out:** The Public IP has no DNS name. Go to Public IP addresses → select the IP → Configuration → add a DNS name label → Save. Then try adding the endpoint again.

---

## Phase 7 — Create South Africa Testing VM

This VM simulates a remote user in a different geographic location for testing traffic routing.

**Step 1** — Search `Virtual machines` → Click **+ Create** → **Azure virtual machine**

**Step 2** — On the **Basics** tab:

| Field | Value |
|-------|-------|
| Resource group | `Alireza-S64P-RG` |
| Virtual machine name | `Alireza-SouthAfrica-Client` |
| Region | **South Africa North** |
| Availability options | **No infrastructure redundancy required** |
| Security type | **Standard** |
| Image | **Windows Server 2022 Datacenter — Gen2** |
| Size | **Standard_D2s_v3** |
| Username | `Alireza` |
| Password | *(your password)* |
| Public inbound ports | **Allow selected ports** |
| Select ports | **RDP (3389)** |

**Step 3** — On the **Networking** tab:

| Field | Value |
|-------|-------|
| Virtual network | `Alireza-VNet-R3` |
| Subnet | `Alireza-Subnet-R3` |
| Public IP | **Leave as default (Create new)** — needed for RDP access |
| NIC network security group | **Basic** |
| Inbound ports | **RDP (3389)** |

**Step 4** — Click **Review + Create** → Click **Create**

> 📸 Take a screenshot of the South Africa VM overview showing **Running** status and its public IP

---

## Phase 8 — Demo and Testing

### Test 1 — Traffic Routing to Nearest Region (20 Points)

#### From Your Local Computer (Canada):

**Step 1** — Open your browser (Chrome or Edge)

**Step 2** — In the address bar type:
```
http://alireza-tm.trafficmanager.net
```

**Step 3** — Press Enter — you should see the page showing **Canada Central** and one of the Canada server names

> 📸 Screenshot: Browser showing Canada Central with the FQDN visible in the URL bar

---

#### From the South Africa VM:

**Step 1** — In Azure portal, open `Alireza-SouthAfrica-Client` → Click **Connect** → **RDP** → **Download RDP file**

**Step 2** — Open the downloaded RDP file → Enter username `Alireza` and password → Connect

**Step 3** — Inside the South Africa VM, open **Microsoft Edge**

**Step 4** — Type in the address bar:
```
http://alireza-tm.trafficmanager.net
```

**Step 5** — You should see **Spain Central** — Traffic Manager routed to the closest region

> 📸 Screenshot: Browser inside South Africa VM showing Spain Central with FQDN in the URL bar

---

### Test 2 — Round Robin Load Balancing (25 Points)

#### From Your Local Computer:

**Step 1** — Open **PowerShell** on your local computer

**Step 2** — Run this command:

```powershell
for($i=1; $i -le 10; $i++) {
    $result = Invoke-WebRequest -Uri "http://alireza-tm.trafficmanager.net" -UseBasicParsing -DisableKeepAlive
    Write-Host "Request $i :" ($result.Content | Select-String "WebSrv\d")
    Start-Sleep -Milliseconds 500
}
```

**Step 3** — Watch the output alternate between `WebSrv1` and `WebSrv2`

> 📸 Screenshot: PowerShell output showing requests alternating between WebSrv1 and WebSrv2

> **Note:** Browsers reuse TCP connections (HTTP Keep-Alive) which prevents Round Robin from being visible. PowerShell with `-DisableKeepAlive` creates a new connection per request, accurately demonstrating load balancer behavior. IIS Keep-Alive was also disabled on all VMs for browser testing.

---

### Test 3 — Global Availability / Failover (20 Points)

**Step 1** — Go to Azure portal → **Virtual machines**

**Step 2** — Click on `Alireza-Canada-WebSrv1` → Click **Stop** at the top → Click **Yes**

**Step 3** — Go back → Click on `Alireza-Canada-WebSrv2` → Click **Stop** → Click **Yes**

**Step 4** — Wait 2–3 minutes for Traffic Manager health probes to detect the failure

> 📸 Screenshot: Virtual Machines list showing both Canada VMs with **Stopped (deallocated)** status

**Step 5** — On your local computer, open the browser → go to:
```
http://alireza-tm.trafficmanager.net
```

**Step 6** — The page should now show **Spain Central** — Traffic Manager automatically failed over

> 📸 Screenshot: Browser on local computer now showing Spain Central (failover confirmed)

**Step 7** — After the demo, go back and **Start** both Canada VMs again

---

## Troubleshooting

### Problem: Round Robin not working in browser

**Cause:** Browsers reuse TCP connections (HTTP Keep-Alive), always going to the same server.

**Solution 1 — Disable Keep-Alive on IIS (run on each VM):**
```powershell
Import-Module WebAdministration
Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/httpProtocol" -name "allowKeepAlive" -value "False"
Restart-Service W3SVC
```

**Solution 2 — Use PowerShell with -DisableKeepAlive flag:**
```powershell
for($i=1; $i -le 10; $i++) {
    $result = Invoke-WebRequest -Uri "http://alireza-tm.trafficmanager.net" -UseBasicParsing -DisableKeepAlive
    Write-Host "Request $i :" ($result.Content | Select-String "WebSrv\d")
    Start-Sleep -Milliseconds 500
}
```

**Solution 3 — Flush DNS between browser tests:**
```cmd
ipconfig /flushdns
```

---

### Problem: Traffic Manager endpoints are greyed out (No FQDN)

**Cause:** Traffic Manager requires Public IPs to have a DNS name label assigned.

**Solution:**

**Step 1** — Search `Public IP addresses` in Azure portal

**Step 2** — Click on `Alireza-LB-R1-PublicIP`

**Step 3** — Click **Configuration** on the left

**Step 4** — In **DNS name label** type: `alireza-lb-r1`

**Step 5** — Click **Save**

**Step 6** — Repeat for `Alireza-LB-R2-PublicIP` with label: `alireza-lb-r2`

**Step 7** — Go back to Traffic Manager → try adding endpoints again

---

### Problem: VM agent status is not ready

**Cause:** The VM was just created and Windows is still initializing.

**Solution:** Wait 3–5 minutes and refresh the page. The warning disappears automatically once Windows finishes starting up. The VM is safe to use as long as status shows **Running**.

---

### Problem: IIS page not showing after installation

**Cause:** IIS service may not be running or firewall is blocking port 80.

**Solution:**
```powershell
# Check IIS service status
Get-Service W3SVC

# Start IIS if not running
Start-Service W3SVC

# Verify page loads locally
Invoke-WebRequest http://localhost -UseBasicParsing
```

---

### Problem: Website not accessible from browser after VMs are stopped (failover test takes too long)

**Cause:** Traffic Manager health probe interval is 30 seconds by default, plus DNS TTL.

**Solution:** Wait 3–5 minutes after stopping VMs before testing. Traffic Manager needs time to detect the failure and update DNS.

---

## Cleanup

> ⚠️ **Always delete resources after completing the project** to avoid charges on your Azure credit.

**Step 1** — Go to Azure portal → Search `Resource groups`

**Step 2** — Click on `Alireza-S64P-RG`

**Step 3** — Click **Delete resource group** at the top

**Step 4** — In the confirmation box, type the resource group name: `Alireza-S64P-RG`

**Step 5** — Click **Delete**

This deletes ALL resources in the group at once including:
- All 5 Virtual Machines
- All 3 Virtual Networks and Subnets
- Both Load Balancers and their Public IPs
- The Traffic Manager profile

> Note: The Traffic Manager profile may need to be deleted separately if it was created in a different resource group.

---

## Resource Naming Reference

| Resource | Name Used |
|----------|-----------|
| Resource Group | `Alireza-S64P-RG` |
| VNet Region 1 | `Alireza-VNet-R1` |
| VNet Region 2 | `Alireza-VNet-R2` |
| VNet Region 3 | `Alireza-VNet-R3` |
| Subnet Region 1 | `Alireza-Subnet-R1` |
| Subnet Region 2 | `Alireza-Subnet-R2` |
| Subnet Region 3 | `Alireza-Subnet-R3` |
| Canada WebSrv1 | `Alireza-Canada-WebSrv1` |
| Canada WebSrv2 | `Alireza-Canada-WebSrv2` |
| Spain WebSrv1 | `Alireza-Spain-WebSrv1` |
| Spain WebSrv2 | `Alireza-Spain-WebSrv2` |
| South Africa VM | `Alireza-SouthAfrica-Client` |
| Availability Set Canada | `Alireza-Canada-AS` |
| Availability Set Spain | `Alireza-Spain-AS` |
| Load Balancer Canada | `Alireza-LB-R1` |
| Load Balancer Spain | `Alireza-LB-R2` |
| LB Canada Frontend IP | `Alireza-LB-R1-Frontend` |
| LB Spain Frontend IP | `Alireza-LB-R2-Frontend` |
| LB Canada Public IP | `Alireza-LB-R1-PublicIP` |
| LB Spain Public IP | `Alireza-LB-R2-PublicIP` |
| LB Canada Backend Pool | `Alireza-LB-R1-BackendPool` |
| LB Spain Backend Pool | `Alireza-LB-R2-BackendPool` |
| LB Canada Rule | `Alireza-LB-R1-Rule` |
| LB Spain Rule | `Alireza-LB-R2-Rule` |
| LB Canada Health Probe | `Alireza-LB-R1-HealthProbe` |
| LB Spain Health Probe | `Alireza-LB-R2-HealthProbe` |
| Traffic Manager | `Alireza-TM` |
| TM Endpoint Canada | `Alireza-Endpoint-Canada` |
| TM Endpoint Spain | `Alireza-Endpoint-Spain` |

---

*Project completed as part of MST400 — Cloud Administration, Summer 2026, Seneca Polytechnic.*
