# MST400 – Project 1
# Azure Multi-Region Website Deployment with Global Load Distribution

**Student:** Alireza  
**Course:** MST400 – Cloud Administration  
**Semester:** Summer 2026  
**Institution:** Seneca Polytechnic  

---

## Project Overview

In this project, I deployed a company website across multiple Microsoft Azure regions with global traffic routing, regional load balancing, and automatic failover. The solution ensures high availability and routes users to the geographically closest region automatically.

---

## Architecture Diagram

![Architecture Diagram](screenshots/00-diagram.png)

---

## What Was Built

| Component | Details |
|-----------|---------|
| Regions | Canada Central, Spain Central, South Africa North |
| Web Servers | 4 VMs (2 per region) running IIS |
| Load Balancers | 2 Azure Standard Load Balancers (Round Robin) |
| Traffic Routing | Azure Traffic Manager (Performance routing) |
| Testing Client | Windows VM in South Africa North |
| FQDN | alireza-tm.trafficmanager.net |

---

## IP Address Plan

| Region | VNet | Subnet | CIDR |
|--------|------|--------|------|
| Canada Central (R1) | Alireza-VNet-R1 | Alireza-Subnet-R1 | 192.168.18.0/26 |
| Spain Central (R2) | Alireza-VNet-R2 | Alireza-Subnet-R2 | 192.168.18.64/26 |
| South Africa North (R3) | Alireza-VNet-R3 | Alireza-Subnet-R3 | 192.168.18.128/26 |

---

## Phase 1 — Resource Group

Created one resource group `Alireza-S64P-RG` in Canada Central to contain all project resources.

![Resource Group](screenshots/01-resource-group.png)

---

## Phase 2 — Virtual Networks

Created 3 Virtual Networks, one per region. Each VNet uses the assigned IP space `192.168.18.0/24` with a custom `/26` subnet. The default subnet was deleted and replaced with a named subnet.

![Canada VNet](screenshots/02canada-vmnet.png)
![Spain VNet](screenshots/03-spain-vmnet.png)
![South Africa VNet](screenshots/04-south-africa-vmnet.png)

---

## Phase 3 — Virtual Machines

Deployed 4 web server VMs (2 in Canada, 2 in Spain) and 1 testing client VM in South Africa. All VMs use Windows Server. Web servers have no public IP — traffic goes through the load balancer.

![All VMs](screenshots/09-all-vms.png)

![Canada WebSrv1](screenshots/05-ca-websrv1.png)
![Canada WebSrv2](screenshots/06-ca-websrv2.png)
![Spain WebSrv1](screenshots/07-sp-websrv1.png)
![Spain WebSrv2](screenshots/08-sp-websrv2.png)

---

## Phase 4 — IIS Web Server Configuration

Installed IIS on all 4 web servers and customized the default page to show the student name, region, and server name. HTTP Keep-Alive was disabled to allow Round Robin to be visible.

---

## Phase 5 — Load Balancers

Created 2 Azure Standard Load Balancers (one per region). Each load balancer uses Round Robin with no session persistence.

### Canada Load Balancer (Alireza-LB-R1)

![Canada LB Overview](screenshots/10-lb-canada.png)
![Canada LB Frontend](screenshots/11-lb-canada-frontend.png)
![Canada LB Backend Pool](screenshots/12-lb-canada-backend-pool.png)
![Canada LB Rule](screenshots/13-lb-canada-rule.png)
![Canada LB Health Probe](screenshots/14-lb-canada-health.png)

### Spain Load Balancer (Alireza-LB-R2)

![Spain LB Overview](screenshots/15-lb-spain.png)
![Spain LB Frontend](screenshots/16-lb-spain-frontend.png)
![Spain LB Backend Pool](screenshots/17-lb-spain-backend-pool.png)
![Spain LB Rule](screenshots/18-lb-spain-rule.png)
![Spain LB Health Probe](screenshots/19-lb-spain-health.png)

---

## Phase 6 — Traffic Manager

Created an Azure Traffic Manager profile with **Performance** routing. This routes each user to the closest Azure region based on network latency.

**Why Performance routing?**
- Automatically sends users to the nearest region
- Monitors endpoint health with probes
- Fails over to another region if one goes down
- Provides a free public FQDN: `alireza-tm.trafficmanager.net`

![Traffic Manager Profile](screenshots/20-traffic-manager-profile.png)
![Traffic Manager Endpoints](screenshots/21-traffic-manager-endpoints.png)

---

## Phase 7 — South Africa Testing VM

Deployed a Windows VM in South Africa North to simulate a remote user accessing the website from a different geographic location.

![South Africa VM](screenshots/17-south-africa-vm.png)

---

## Demo Results

### Traffic Routing to Nearest Region

Users are automatically routed to the closest region:
- **From Canada** → routed to **Canada Central**
- **From South Africa** → routed to **Spain Central** (geographically closer)

![Connect from Canada](screenshots/22-connect-from-canada.png)
![Connect from South Africa](screenshots/23-connect-from-south-africa.png)

---

### Round Robin Load Balancing

Each request is distributed evenly between the 2 web servers in the region. HTTP Keep-Alive was disabled on IIS so each request creates a new TCP connection, making Round Robin visible.

PowerShell was used to demonstrate Round Robin:

```powershell
for($i=1; $i -le 10; $i++) {
    $result = Invoke-WebRequest -Uri "http://alireza-tm.trafficmanager.net" -UseBasicParsing -DisableKeepAlive
    Write-Host "Request $i :" ($result.Content | Select-String "WebSrv\d")
    Start-Sleep -Milliseconds 500
}
```

![PowerShell Round Robin](screenshots/24-power-shell.png)

---

### Global Availability — Failover Test

Both Canada VMs were stopped. Traffic Manager detected the failure and automatically rerouted traffic from Canada to Spain Central — the website remained fully accessible.

---

## Key Learnings

- How to design and deploy multi-region infrastructure on Microsoft Azure
- How Azure Traffic Manager routes users using Performance-based routing
- How Azure Load Balancer distributes traffic with Round Robin across VMs
- Subnetting a /24 IP space into /26 subnets for 3 separate regions

## Challenges Faced

| Challenge | Solution |
|-----------|----------|
| Round Robin not visible in browser | Disabled HTTP Keep-Alive in IIS on all VMs |
| Traffic Manager endpoints greyed out | Added DNS name label to each Load Balancer public IP |
| Browser caching preventing server switching | Used PowerShell with -DisableKeepAlive flag |

---

## Azure Resources Used

- Resource Group: `Alireza-S64P-RG`
- Virtual Networks: `Alireza-VNet-R1`, `Alireza-VNet-R2`, `Alireza-VNet-R3`
- Virtual Machines: `Alireza-Canada-WebSrv1`, `Alireza-Canada-WebSrv2`, `Alireza-Spain-WebSrv1`, `Alireza-Spain-WebSrv2`, `Alireza-SouthAfrica-Client`
- Load Balancers: `Alireza-LB-R1`, `Alireza-LB-R2`
- Traffic Manager: `Alireza-TM`

---

*All Azure resources were deleted after project completion to avoid unwanted charges.*