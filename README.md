# Azure Hub-and-Spoke Network Architecture & Security Implementation Lab

## 📌 Project Overview
This project demonstrates the design, deployment, and configuration of an **Azure Hub-and-Spoke VNet Topology** with secure inter-VNet peering, Network Security Group (NSG) traffic rules, and Azure Bastion access. 

The Hub-and-Spoke architecture provides a scalable framework to isolate workloads (App and Database tiers) in dedicated spoke VNets while centralizing shared services (Management and Bastion) in a core hub VNet.

---

## 📐 Architecture Topology

```
                         +-----------------------------------+
                         |          vnet-hub-prod            |
                         |           10.0.0.0/16             |
                         |-----------------------------------|
                         |  - Subnet-Management (10.0.1.0/24) |
                         |  - AzureBastionSubnet (10.0.2.0/24)|
                         +-----------------+-----------------+
                                           |
                    +----------------------+----------------------+
                    | VNet Peering               VNet Peering     |
                    v                                             v
  +-----------------------------------+         +-----------------------------------+
  |          vnet-spoke-app           |         |          vnet-spoke-db            |
  |           10.1.0.0/16             |         |           10.2.0.0/16             |
  |-----------------------------------|         |-----------------------------------|
  |  - default subnet                 |         |  - Subnet-DB (10.2.1.0/24)        |
  |  - Workload: vm-spoke-app         |         |  - Workload: vm-spoke-db          |
  +-----------------------------------+         +-----------------------------------+
```

---

## ⚙️ Resource & IP Addressing Scheme

| Resource Name | Type | IP Address Space / Range | Purpose / Subnet |
| :--- | :--- | :--- | :--- |
| `rg-hubspoke-lab` | Resource Group | N/A | Central resource container |
| `vnet-hub-prod` | Virtual Network | `10.0.0.0/16` | Hub VNet for central management |
| ↳ `Subnet-Management` | Subnet | `10.0.1.0/24` | Central management subnet |
| ↳ `AzureBastionSubnet`| Subnet | `10.0.2.0/24` | Secure management access host |
| `vnet-spoke-app` | Virtual Network | `10.1.0.0/16` | Application Tier Spoke VNet |
| ↳ `default` | Subnet | `10.1.0.0/24` | Application server workloads |
| `vnet-spoke-db` | Virtual Network | `10.2.0.0/16` | Database Tier Spoke VNet |
| ↳ `Subnet-DB` | Subnet | `10.2.1.0/24` | Database server workloads |
| `vm-spoke-app` | Virtual Machine | Private IP (App Tier) | Ubuntu Linux Workload |
| `vm-spoke-db` | Virtual Machine | Private IP (`10.2.1.4`) | MySQL / Database Workload |
| `nsg-spoke-db` | NSG | Security Boundary | Restricts DB inbound traffic |

---

## 🛠️ Step-by-Step Implementation Guide

### Step 1: Resource Group & Hub VNet Setup
1. Create Resource Group: `rg-hubspoke-lab` (Region: `East US`).
2. Create Hub VNet `vnet-hub-prod` with address space `10.0.0.0/16`.
3. Configure subnets inside `vnet-hub-prod`:
   - `Subnet-Management`: `10.0.1.0/24`
   - `AzureBastionSubnet`: `10.0.2.0/24`

### Step 2: Spoke VNets Configuration
1. **Application Spoke (`vnet-spoke-app`)**:
   - Address Space: `10.1.0.0/16`
   - Subnet: `default` (`10.1.0.0/24`)
2. **Database Spoke (`vnet-spoke-db`)**:
   - Address Space: `10.2.0.0/16`
   - Subnet: `Subnet-DB` (`10.2.1.0/24`)

### Step 3: VNet Peering Setup
Configure bidirectional VNet peerings between Hub and Spokes:
* **Hub to App Spoke**: `spoke-app-to-hub` ↔ `hub-to-spoke-app` (**State**: *Connected*, **Sync**: *Fully Synchronized*)
* **Hub to DB Spoke**: `spoke-db-to-hub` ↔ `hub-to-spoke-db` (**State**: *Connected*, **Sync**: *Fully Synchronized*)

### Step 4: Compute & Network Security Group Deployment
1. Deploy `vm-spoke-app` into `vnet-spoke-app`.
2. Deploy `vm-spoke-db` into `vnet-spoke-db`.
3. Configure Network Security Group `nsg-spoke-db` to enforce strict tier isolation:
   - **Rule Name**: `Allow-App-Subnet-To-DB`
   - **Priority**: `100`
   - **Source**: IP Range `10.1.0.0/16` (App Subnet/VNet)
   - **Destination**: Any (`10.2.1.0/24` DB Subnet)
   - **Destination Port**: `3306` (MySQL)
   - **Protocol**: Any / TCP
   - **Action**: `Allow`

### Step 5: Verification & Bastion Connectivity
1. Connect securely to `vm-spoke-app` via **Azure Bastion**.
2. Verify cross-VNet connectivity and route propagation by initiating ICMP/network diagnostics from `vm-spoke-app` to target DB workload (`10.2.1.4`):
   ```bash
   azureuser@vm-spoke-app:~$ ping -c 4 10.2.1.4
   ```

---

## 🔒 Security Best Practices Implemented
* **Private Subnet Enforcement**: Outbound direct internet access disabled on internal subnets via private subnet flags.
* **Least Privilege Access**: Dedicated NSG inbound rule permitting traffic strictly from the App subnet (`10.1.0.0/16`) to MySQL port `3306`.
* **Management Isolation**: Native Azure Bastion utilized for secure SSH access without exposing public IP addresses directly to the VMs.
