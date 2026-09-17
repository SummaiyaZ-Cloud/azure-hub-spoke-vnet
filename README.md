# Enterprise Hub-and-Spoke Virtual Network with VNet Peering & NSGs

## Project Overview

This project demonstrates the design and deployment of a hub-and-spoke network architecture in Microsoft Azure.

I created a centralized Hub Virtual Network and two separate Spoke Virtual Networks to isolate application and database workloads. The networks were connected using Azure VNet Peering, while Network Security Groups (NSGs) were used to control traffic between the workloads.

Azure Bastion was used to securely access the Linux virtual machines through the Azure Portal and perform private-network connectivity testing.

This lab provided hands-on experience with Azure networking, network segmentation, VNet peering, NSG rules, Azure Bastion, and workload isolation.

## Architecture

The environment consisted of:

- **Hub VNet:** `vnet-hub-prod` — `10.0.0.0/16`
- **Management Subnet:** `10.0.1.0/24`
- **AzureBastionSubnet:** `10.0.2.0/24`
- **Application Spoke VNet:** `vnet-spoke-app` — `10.1.0.0/16`
- **Database Spoke VNet:** `vnet-spoke-db` — `10.2.0.0/16`
- **Application VM:** `vm-spoke-app`
- **Database VM:** `vm-spoke-db`
- Hub-to-spoke VNet peering
- Network Security Groups (NSGs)
- Azure Bastion for administrative access
