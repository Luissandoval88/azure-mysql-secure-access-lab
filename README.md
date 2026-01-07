# azure-mysql-secure-access-lab

Deploying and securing a MySQL workload in Microsoft Azure using Bastion, Managed Identity, Azure Key Vault, and RBAC, following a zero-trust, identity-first access model.

---

## Why This Lab Exists

This lab was created to migrate a locally hosted MySQL database into Azure while intentionally designing it to be **secure by default**.

Rather than just deploying infrastructure, the focus was on:
- Avoiding hardcoded credentials
- Limiting network exposure
- Using identity-based access controls
- Making realistic security tradeoffs

Azure was selected to gain hands-on experience with **cloud security, identity, and secrets management**.

---

## Step 1: Azure Environment Setup

The following Azure resources were created:

- Linux-based Azure Virtual Machine
- Virtual Network (VNet)
- Network Security Group (NSG)
- Azure Bastion
- Azure Key Vault

At this stage:
- MySQL was installed on the VM
- MySQL was bound to `127.0.0.1`
- No database ports were publicly exposed

---

## Step 2: Secure Administrative Access with Azure Bastion

Azure Bastion was implemented first to enable:
- Browser-based SSH access
- No public SSH exposure
- No inbound port 22 required

This aligns with a **zero-trust administrative model** and removes reliance on open management ports.

---

## Step 3: Practical Requirement & Network Adjustment

To manage the database visually, **MySQL Workbench** was chosen as the client tool.

### The Challenge
MySQL Workbench requires **SSH tunneling**, which Azure Bastion does not directly support.

### The Security Tradeoff
To enable Workbench access **without significantly increasing risk**:

- SSH (port 22) was enabled
- Access was restricted to **a single trusted source IP**
- Bastion remained enabled for administrative access
- No unrestricted public SSH access was allowed

This reflects a real-world security decision balancing usability and risk.

---

## Network Security Summary

- Azure Bastion for secure admin access
- SSH (port 22) restricted to a trusted IP
- MySQL (3306) not exposed publicly
- MySQL bound to localhost only

---

## Step 4: MySQL Workbench Connection & Validation

Before adding additional security layers, connectivity was validated.

### MySQL Workbench Configuration

- **Connection Method:** Standard TCP/IP over SSH
- **SSH Host:** Azure VM public IP
- **SSH User:** Azure VM user
- **SSH Key:** Private SSH key
- **MySQL Host:** `127.0.0.1`
- **MySQL Port:** `3306`
- **MySQL User:** `wb_user`

### Result

- SSH tunnel established successfully
- MySQL connection validated
- Database accessible without public MySQL exposure

This step confirmed:
- Network rules were correctly configured
- The database was functional
- Further security hardening would not break access

---

## Step 5: Identity & Access Management (Removing Hardcoded Credentials)

To eliminate embedded credentials, **Azure Managed Identity** was introduced.

### What Was Done

- System-assigned managed identity enabled on the VM
- Azure RBAC used to control access
- VM identity granted **Key Vault Secrets User** role
- No credentials stored on disk or in scripts

### Why This Matters

- Authentication is handled by Azure Entra ID
- Secrets are accessed securely at runtime
- Access can be revoked instantly
- All access is logged and auditable

This removes the need for **password-based authentication in code**.

---

## Step 6: Secrets Management with Azure Key Vault

Azure Key Vault was used to securely store MySQL connection details.

### Secrets Stored

- `mysql-admin-username`
- `mysql-admin-password`
- `mysql-host`
- `mysql-port`

### Runtime Access Example

```bash
az login --identity

MYSQL_USER=$(az keyvault secret show \
  --vault-name kv-mysql-secure-lab \
  --name mysql-admin-username \
  --query value -o tsv)

MYSQL_PASS=$(az keyvault secret show \
  --vault-name kv-mysql-secure-lab \
  --name mysql-admin-password \
  --query value -o tsv)
