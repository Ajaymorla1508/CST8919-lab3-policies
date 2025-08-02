# 🌐 Azure Policy Lab – Cloud Governance Gone Rogue  
**Course**: CST8919 – DevOps Security and Compliance  
**Company**: MapleTech Solutions  
**Resource Group**: `MTS-project1-RG`  
**Role**: Cloud Security Engineer  

---

## 📘 Lab Overview

At MapleTech Solutions, developers were deploying resources across various regions without proper governance. Our task was to bring structure using **Azure Policy** by enforcing:

- Region restriction to `Canada Central`
- Mandatory tagging using `ProjectName`
- Blocking public IP creation

This document outlines policy definitions, assignment steps, testing commands, and results.

---

## 🔐 Policies Implemented

### 1. 🗺️ Region Lockdown – Only Canada Central

- **Name**: `Only-CanadaCentral`
- **Effect**: `Deny`
- **Description**: Prevents resource deployment in regions other than `Canada Central`.

    ```json
    {
    "properties": {
        "displayName": "Only allow resources in Canada Central",
        "description": "Deny creation of resources not in Canada Central",
        "mode": "All",
        "policyRule": {
        "if": {
            "field": "location",
            "notEquals": "canadacentral"
        },
        "then": {
            "effect": "deny"
        }
        }
    }
    }
    ```
### 2. 🏷️ Require ProjectName Tag

- Name: Require-ProjectName-Tag

- Effect: Deny

- Description: Ensures all resources have a ProjectName tag.

    ```json
    {
    "properties": {
        "displayName": "Require ProjectName Tag",
        "description": "Deny resources without a ProjectName tag",
        "mode": "All",
        "parameters": {
        "tagName": {
            "type": "String",
            "defaultValue": "ProjectName",
            "metadata": {
            "description": "Tag name to enforce",
            "displayName": "Tag Name"
            }
        }
        },
        "policyRule": {
        "if": {
            "field": "[concat('tags[', parameters('tagName'), ']')]",
            "exists": "false"
        },
        "then": {
            "effect": "deny"
        }
        }
    }
    }
    ```
### 3. 🌐 Block Public IPs
- Name: Deny-Public-IP

- Effect: Deny

- Description: Prevents creation of public IP addresses.

    ```json
    {
    "properties": {
        "displayName": "Deny Public IP",
        "description": "Prevent creation of Public IP addresses",
        "mode": "All",
        "policyRule": {
        "if": {
            "field": "type",
            "equals": "Microsoft.Network/publicIPAddresses"
        },
        "then": {
            "effect": "deny"
        }
        }
    }
    }
    ```
### 🧩 Policy Initiative: MapleTech Secure Foundation
    - Name: MapleTech Secure Foundation

    - Policies Included:

    - Only-CanadaCentral

    - Require-ProjectName-Tag

    - Deny-Public-IP

    - Category: Security

    - Assigned To: MTS-project1-RG

    - Enforcement Mode: Enabled

### 🛠️ Policy Assignment & Testing (VS Code)
- Prerequisites
```bash
    az login
    az account set --subscription "<your-subscription-id>"
```
### Test Cases and Commands
- 1. ❌ Deploy VM in East US – Should Fail
    ```bash
    az vm create \
    --resource-group MTS-project1-RG \
    --name eastus-vm \
    --image UbuntuLTS \
    --location eastus \
    --admin-username azureuser \
    --generate-ssh-keys
    ```
Expected: ❌ Denied – Not Canada Central

2. ❌ Create Storage Account Without ProjectName Tag – Should Fail
    ```bash
    az storage account create \
    --name mtslabstorage \
    --resource-group MTS-project1-RG \
    --location canadacentral \
    --sku Standard_LRS
    ```
Expected: ❌ Denied – Missing required tag

3. ❌ Create Public IP Address – Should Fail
    ```bash
    az network public-ip create \
    --resource-group MTS-project1-RG \
    --name public-ip-test \
    --location canadacentral
    ```
Expected: ❌ Denied – Public IP creation blocked

4. ✅ Deploy VM in Canada Central with ProjectName Tag – Should Succeed
    ```bash
    az vm create \
    --resource-group MTS-project1-RG \
    --name secure-vm \
    --image UbuntuLTS \
    --location canadacentral \
    --tags ProjectName=PolicyLab \
    --public-ip-address "" \
    --admin-username azureuser \
    --generate-ssh-keys
    ```
Expected: ✅ Allowed – Region and tag compliant, no public IP

📂 Folder Structure for Submission
```pgsql
/policy-lab
│
├── README.md                     # This file
├── screenshots/
│   ├── denied-eastus.png
│   ├── no-tag-storage.png
│   ├── public-ip-denied.png
│   ├── allowed-vm.png
│
├── policy-definitions/
│   ├── only-canadacentral.json
│   ├── require-projectname-tag.json
│   ├── deny-public-ip.json
│
└── video-demo.txt         
```   
### Demo Video (10 mins)
[Watch the 5-minute demo on YouTube](https://www.youtube.com/watch?v=QrMHR35nZAk)

### 🧠 Lessons Learned
- Azure Policy is powerful for organization-wide governance.

- JSON policies must be well-formed and specific (especially field usage).

- Initiatives simplify enforcement by grouping policies.

- Even with Deny effects, policy testing is safe and non-destructive.

- Tags and location enforcement help control costs and data residency.

---