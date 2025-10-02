# Data-Breach-Incident-Response-and-Remediation
Google Cloud Cybersecurity Capstone Project

## Objective

Conducted end-to-end incident response for a simulated enterprise data breach scenario involving compromised cloud infrastructure and exposed payment card data. Identified and remediated critical security misconfigurations including publicly accessible storage buckets, malware-infected virtual machines with excessive API privileges, and internet-exposed SSH/RDP ports. Utilized Google Cloud Security Command Center to analyze PCI DSS 3.2.1 compliance violations, implement security controls following defense-in-depth principles, and validate complete remediation of all critical vulnerabilities affecting cloud compute, storage, and network resources.

### Skills Learned

Incident Response & Management:

- Complete incident response lifecycle execution (Detection → Containment → Eradication → Recovery → Post-Incident)
- Security breach investigation and root cause analysis
- Risk assessment and vulnerability prioritization based on severity
- Incident documentation and stakeholder communication

Cloud Security Operations:

- Google Cloud Security Command Center (SCC) operations and analysis
- Compliance assessment and validation (PCI DSS 3.2.1)
- Multi-resource security remediation across cloud infrastructure
- Security posture assessment and improvement

Technical Remediation:

- Compute Engine security hardening and VM recovery from snapshots
- Cloud Storage access control and IAM policy configuration
- VPC firewall rule creation, modification, and security optimization
- Network segmentation and access control implementation

Compliance & Governance:

- PCI DSS compliance framework application
- Security findings interpretation and remediation mapping
- Compliance report generation and validation
- Security control implementation verification

### Tools Used

- Google Cloud Security Command Center (SCC) - Centralized vulnerability detection and compliance monitoring
- Google Compute Engine - Virtual machine management and secure boot configuration
- Google Cloud Storage - Object storage security and IAM policy management
- VPC Firewall Rules - Network security and access control
- Cloud IAM - Identity and access management
- PCI DSS 3.2.1 Framework - Payment card industry compliance standards
- Cloud Console & Cloud Shell - Resource management and CLI operations
- VM Snapshots - System recovery and malware remediation

## Steps

### Phase 1: Detection & Analysis

#### Security Command Center Overview
*Ref 1: Security Command Center showing active vulnerabilities by resource type*

<img width="2552" alt="Security Command Center Overview" src="https://github.com/user-attachments/assets/e1175f3d-4ac3-4bd4-ba5b-79bd0ab838d9" />

Conducted comprehensive security assessment using Security Command Center to identify breach scope and affected resources.

**Findings Summary:**
- **3 Resource Types Compromised**: Cloud Storage bucket, Compute Engine VM (cc-app-01), VPC Firewall
- **High Severity Findings**: 5 critical vulnerabilities
- **Medium Severity Findings**: 4 moderate vulnerabilities
- **Compliance Violations**: Multiple PCI DSS 3.2.1 non-compliance issues

---

#### PCI DSS 3.2.1 Compliance Analysis

Reviewed PCI DSS 3.2.1 compliance report to identify security control failures:

| **Finding Category** | **Severity** | **PCI DSS Rule Violated** | **Security Impact** |
|---------------------|--------------|---------------------------|---------------------|
| Public Bucket ACL | HIGH | Cloud Storage buckets should not be anonymously or publicly accessible | Sensitive data exposed to internet |
| Open SSH Port | HIGH | Firewall rules should not allow connections from all IPs on port 22 | Unrestricted remote access vulnerability |
| Open RDP Port | HIGH | Firewall rules should not allow connections from all IPs on port 3389 | Windows remote desktop exposed globally |
| Public IP Address | HIGH | VMs should not be assigned public IP addresses | Direct internet exposure of compromised VM |
| Firewall Logging Disabled | MEDIUM | Firewall rule logging should be enabled | No audit trail for network access |
| Full API Access | MEDIUM | Default service account with full API access | Excessive privileges on compromised VM |
| Bucket Policy Disabled | MEDIUM | Uniform bucket-level access not enforced | Inconsistent access controls |

---

#### Cloud Storage Bucket Analysis
*Ref 2: Security findings filtered by Cloud Storage bucket*

<img width="2271" alt="Cloud Storage Bucket Vulnerabilities" src="https://github.com/user-attachments/assets/8110cd5c-8403-4290-a07a-1489d65aff7d" />

**Active vulnerabilities identified:**
- **Public Bucket ACL** - Anyone on the internet can read stored files
- **Bucket Policy Only Disabled** - No explicit bucket policy controlling access
- **Bucket Logging Disabled** - No audit trail of data access attempts

**File Compromised**: `myfile.csv` containing sensitive customer credit card data

---

#### Compute Engine VM Analysis
*Ref 3: Findings filtered by Compute Instance showing cc-app-01 vulnerabilities*

<img width="2212" alt="Compute Engine VM Vulnerabilities" src="https://github.com/user-attachments/assets/3247038f-d363-4a48-9972-a8762765de15" />

**Active vulnerabilities identified:**
- **Malware: Bad Domain** (LOW) - VM accessed malware-associated domain, **indicating compromise**
- **Public IP Address** (HIGH) - VM directly exposed to internet
- **Compute Secure Boot Disabled** (MEDIUM) - Allows unauthorized boot code
- **Default Service Account Used** (MEDIUM) - Overly permissive default account
- **Full API Access** (MEDIUM) - Complete access to all Google Cloud APIs

**Critical Finding**: The "Malware: Bad Domain" finding confirmed the VM was actively compromised and communicating with malicious infrastructure.

---

#### VPC Firewall Analysis
*Ref 4: Findings filtered by Firewall showing overly permissive rules*

<img width="2223" alt="VPC Firewall Vulnerabilities" src="https://github.com/user-attachments/assets/60758008-25e0-4626-8364-d3f19857f3aa" />

**Active vulnerabilities identified:**
- **Open SSH Port** (HIGH) - Port 22 accessible from entire internet (0.0.0.0/0)
- **Open RDP Port** (HIGH) - Port 3389 accessible from entire internet (0.0.0.0/0)
- **Firewall Rule Logging Disabled** (MEDIUM) - No record of firewall traffic

**Root Cause**: Default firewall rules (`default-allow-ssh`, `default-allow-rdp`) were never restricted from internet-wide access.

---

### Phase 2: Containment

#### Isolate Compromised VM
*Ref 5: VM instances page with cc-app-01 stopped*

<img width="2107" alt="Stopping Compromised VM" src="https://github.com/user-attachments/assets/4cfe35dd-69ba-49af-ac22-83d8e3d383b5" />

**Immediate Containment Actions:**
- Stopped compromised VM `cc-app-01` to prevent further malicious activity
- Prevented lateral movement within cloud environment
- Halted ongoing data exfiltration

**Containment Impact**: Immediately stopped malicious actor's access to compromised system and prevented further damage.

---

### Phase 3: Eradication

#### VM Recovery and Hardening
*Ref 6: Create VM instance page showing cc-app-02 configuration*

<img width="2268" alt="Creating Hardened VM" src="https://github.com/user-attachments/assets/8abb9191-5511-4bc9-a45e-398b529003b2" />

*Ref 7: VM Security settings with Secure Boot enabled*

<img width="974" alt="Secure Boot Configuration" src="https://github.com/user-attachments/assets/f3b67fc7-d355-401a-9eaf-1dd93dfd596b" />

Created replacement VM `cc-app-02` from clean pre-infection snapshot with security hardening:

**Configuration Details:**
- **Source**: `cc-app01-snapshot` (pre-malware backup)
- **Machine Type**: e2-medium (Shared-core)
- **Network Tag**: `cc` (for targeted firewall rules)
- **External IP**: None (removed public internet exposure)
- **Service Account**: Qwiklabs User Service Account (removed default full access)
- **Secure Boot**: Enabled (prevents unauthorized boot code)

**Security Improvements Over Original VM:**

| **Security Control** | **cc-app-01 (Compromised)** | **cc-app-02 (Hardened)** |
|---------------------|----------------------------|--------------------------|
| Public IP Address | ✗ Exposed | ✓ None (Private only) |
| Secure Boot | ✗ Disabled | ✓ Enabled |
| Service Account | ✗ Default (Full API) | ✓ Limited permissions |
| Network Tag | ✗ None | ✓ `cc` (controlled access) |
| Malware Status | ✗ Infected | ✓ Clean (from snapshot) |

---

#### Decommission Compromised System
*Ref 8: VM instances showing cc-app-02 running and cc-app-01 deleted*

<img width="2104" alt="Compromised VM Deleted" src="https://github.com/user-attachments/assets/30e51d2e-7e7f-4e6a-a613-e311c255b54a" />

**Decommissioned Compromised System:**
- Permanently deleted `cc-app-01` to eliminate malware source
- Verified only secure `cc-app-02` remained operational

---

#### Cloud Storage Security Remediation
*Ref 9: Cloud Storage bucket permissions showing public access*

<img width="2208" alt="Cloud Storage Bucket Permissions" src="https://github.com/user-attachments/assets/20deb71c-33b2-4f28-adbf-d2ab0c861450" />

*Ref 10: Preventing public access and switching to uniform bucket-level access*

<img width="1941" alt="Uniform Bucket Access Configuration" src="https://github.com/user-attachments/assets/2d6427e4-4f38-4fc4-8a1b-3b5f642f6eef" />

**Step 1: Remove Public Access**
- Clicked "Prevent public access" in Public access tile
- Immediately revoked internet-wide read access to sensitive data

**Step 2: Enforce Uniform Bucket-Level Access**
- Switched from fine-grained ACLs to uniform IAM-based permissions
- Selected "Add project role ACLs to bucket IAM policy" to preserve legitimate access
- Removed `allUsers` principal from all bucket permissions

**Security Impact:**
- Eliminated public data exposure
- Centralized permission management through IAM
- Prevented ACL-based permission bypasses
- Maintained legitimate user access through project roles










