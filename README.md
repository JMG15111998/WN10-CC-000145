# WN10-CC-000145


# 🛡️ Vulnerability Management Lab – WN10-CC-000145

**Title:** Users must be prompted for a password on resume from sleep (on battery)  
**STIG ID:** WN10-CC-000145  
**Compliance Frameworks:** DISA STIG, HIPAA, NIST 800-53, PCI DSS  
**Lab Type:** Vulnerability Simulation, Detection, Remediation, and Verification  
**Tools Used:** Azure, Windows 10, PowerShell, Tenable.sc / Nessus

---

## 📋 Lab Objectives

This lab provides a hands-on walkthrough for managing a Windows 10 vulnerability that violates STIG control `WN10-CC-000145`. You will:

- Deploy a vulnerable Windows 10 VM in Azure  
- Simulate a STIG violation using PowerShell  
- Detect the issue via authenticated Tenable scan  
- Remediate the finding using command-line controls  
- Verify the fix via post-remediation scanning  

This README is intended for cybersecurity analysts, blue team engineers, or students in vulnerability management training.

---

## 📁 Table of Contents

1. [Azure VM Setup](#azure-vm-setup)  
2. [Vulnerability Simulation](#vulnerability-simulation)  
3. [Tenable Scan Configuration](#tenable-scan-configuration)  
4. [Initial Vulnerability Scan](#initial-vulnerability-scan)  
5. [Remediation via PowerShell](#remediation-via-powershell)  
6. [Post-Remediation Verification](#post-remediation-verification)  
7. [Security Rationale](#security-rationale)  
8. [Appendix: PowerShell Commands](#appendix-powershell-commands)

---

## ☁️ Azure VM Setup

### 🔹 1. Provision the Windows 10 VM

| Setting              | Value                    |
|----------------------|--------------------------|
| VM Name              | `Win10-STIGLab-01`       |
| OS Image             | Windows 10 Pro (Gen 2)   |
| Size                 | Standard D2s v3          |
| Region               | Closest to analyst       |
| Resource Group       | `vm-lab-w10`             |

### 🔹 2. Secure Admin Credentials

> ⚠️ **Do NOT use default lab credentials** like `labuser` / `Cyberlab123!`.  
Use a strong, unique password that meets complexity requirements and is stored securely.

### 🔹 3. Network Security Group (NSG)

- Allow Inbound:
  - RDP (TCP 3389)
  - WinRM (TCP 5985) *(for remote PowerShell, if used)*
- Deny all other public ports by default.

### 🔹 4. Local Configuration on the VM

#### Disable Windows Firewall:
```powershell
# Open wf.msc and disable all profiles
```

#### Enable Remote PowerShell Access:
```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord -Force
```

---

## ⚙️ Vulnerability Simulation

### 🔸 Objective
Create a non-compliant state by configuring the system to **not require** a password when resuming from sleep on battery power.

### 🔸 PowerShell Command
```powershell
# Simulate vulnerability: disable password prompt on resume (battery)
powercfg /setdcvalueindex SCHEME_CURRENT SUB_NONE CONSOLELOCK 0
```

### 🔸 Description
- `0` = No password required (non-compliant)
- `setdcvalueindex` modifies power behavior while on **battery (DC mode)**

📸 **Screenshot Placeholder:** `Screenshot_01_Vulnerability_Config_PowerShell.png`

---

## 🔍 Tenable Scan Configuration

### 🔸 Scan Template: Advanced Network Scan

Configure a new scan in **Tenable.sc** or **Nessus** as follows:

#### ✅ Basic Settings
- Name: `Windows 10 STIG Compliance`
- Target: IP of the Azure VM

#### ✅ Discovery Settings
- Ping remote host
- TCP full connect scan
- Enable NetBIOS and SMB probing

#### ✅ Assessment Settings
- Use **authenticated scan** with local admin credentials
- Enable:
  - Remote Registry
  - Admin Shares
  - Server Service

#### ✅ Compliance Settings
- Import and assign:  
  `DISA STIG – Microsoft Windows 10 v3r4.audit`

---

## 🧪 Initial Vulnerability Scan

After the scan completes, review the results:

| Finding ID | WN10-CC-000145 |
|------------|----------------|
| Description | Resume from sleep (on battery) does not prompt for password |
| Status      | ❌ **Fail**     |
| Expected    | `CONSOLELOCK = 1` (require password) |
| Actual      | `CONSOLELOCK = 0` (no password required) |

📸 **Screenshot Placeholder:** `Screenshot_02_Tenable_Vuln_Finding_BeforeFix.png`

---

## 🛠️ Remediation via PowerShell

### 🔸 Secure Configuration Commands

```powershell
# Require password on resume from sleep (battery mode)
powercfg /setdcvalueindex SCHEME_CURRENT SUB_NONE CONSOLELOCK 1
powercfg /setactive SCHEME_CURRENT
```

> `1` = Require password  
> Changes apply immediately; no reboot required.

📸 **Screenshot Placeholder:** `Screenshot_03_PowerShell_Remediation_Applied.png`

---

## ✅ Post-Remediation Verification

### 🔸 Option 1: PowerShell Confirmation

```powershell
# Check console lock status
powercfg /query SCHEME_CURRENT SUB_NONE CONSOLELOCK
```

**Expected Output:**
```
Current DC Power Setting Index: 0x00000001
```

### 🔸 Option 2: Tenable Rescan

Re-run the same scan. The vulnerability should now be marked as:

| STIG ID  | WN10-CC-000145 |
|----------|----------------|
| Status   | ✅ **Pass**     |

📸 **Screenshot Placeholder:** `Screenshot_04_Tenable_AfterRemediation_Pass.png`

---

## 🔐 Security Rationale

Allowing a system to resume from sleep without authentication poses a **physical security risk**, especially in mobile or remote work environments.

### Consequences of Non-Compliance:
- Unauthorized access to data if the laptop is left unattended
- Violation of regulatory standards such as HIPAA, PCI-DSS, and NIST 800-53
- Increased risk of data exfiltration or misuse

### Compliance Mapping:
| Framework | Control |
|-----------|---------|
| **HIPAA** | §164.312(a)(2)(iii) – Automatic logoff |
| **NIST**  | AC-11 – Session Lock |
| **PCI DSS** | 8.1.8 – Session Timeout |
| **DISA STIG** | WN10-CC-000145 |

---

## 📎 Appendix: PowerShell Commands

| Purpose         | Command |
|----------------|---------|
| **Simulate Vulnerability** | `powercfg /setdcvalueindex SCHEME_CURRENT SUB_NONE CONSOLELOCK 0` |
| **Remediate** | `powercfg /setdcvalueindex SCHEME_CURRENT SUB_NONE CONSOLELOCK 1` |
| **Activate Scheme** | `powercfg /setactive SCHEME_CURRENT` |
| **Verify** | `powercfg /query SCHEME_CURRENT SUB_NONE CONSOLELOCK` |

---

## 🧼 Cleanup (Optional)

After testing:

- Delete the Azure resource group to remove all assets:
```bash
az group delete --name vm-lab-w10 --yes --no-wait
```

- Revoke any stored credentials or authentication artifacts

---

## 📌 Notes

- This lab is reusable with other vulnerabilities by adjusting the registry or power settings.
- Swap out PowerShell and compliance IDs to simulate other findings (e.g., SMBv1, missing patches, etc.)

---

✅ **README Complete**  
For more vulnerability guides, see the `/labs/` folder or contact your security lab coordinator.

