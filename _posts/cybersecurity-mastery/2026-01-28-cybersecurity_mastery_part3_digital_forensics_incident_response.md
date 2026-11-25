---
title: "Cybersecurity Mastery - Part 3: Digital Forensics & Incident Response (DFIR)"
date: 2026-01-28 00:00:00 +0530
categories: [Cybersecurity, Cybersecurity Mastery]
tags: [Cybersecurity, Digital-forensics, Incident-response, DFIR, Memory-forensics, Malware-analysis, Threat-hunting, SOC]
description: "Incident response lifecycle, disk, memory and network forensics, malware analysis, threat hunting and SOC operations."
---


# Complete Cybersecurity & Data Forensics Mastery Part 3: Digital Forensics & Incident Response

## Introduction

Welcome to Part 3 of our Cybersecurity & Data Forensics Mastery series. Having covered defensive security (Part 1) and offensive techniques (Part 2), we now explore Digital Forensics and Incident Response (DFIR) - the critical discipline of investigating security incidents, recovering from breaches, and building evidence for legal proceedings.

**Why DFIR Matters:**

- **Detection Gap**: Average time to detect a breach is 204 days (IBM 2024)
- **Financial Impact**: Average breach cost is $4.45 million
- **Legal Requirements**: GDPR, HIPAA, PCI-DSS mandate breach investigation
- **Attribution**: Identify attackers and their techniques
- **Prevention**: Learn from incidents to improve security posture

**What You'll Master:**

This comprehensive guide covers the complete DFIR lifecycle from evidence collection through court testimony. You'll learn to investigate incidents on Windows, Linux, and macOS systems, analyze memory dumps and network traffic, hunt for threats proactively, and build ironclad forensic reports.

**Prerequisites:**
- Parts 1 & 2 completion recommended
- Command-line proficiency (Windows CMD/PowerShell, Linux Bash)
- Understanding of file systems and operating systems
- Basic networking knowledge
- Python scripting basics helpful but not required

**Target Word Count**: 28,000 words minimum
**Estimated Study Time**: 60-80 hours for mastery
**Certifications Covered**: CHFI, GCFE, GCFA, GNFA, GCIH, EnCE

Let's begin your journey into digital forensics and incident response mastery.

---

## 3.1 Digital Forensics Fundamentals

### Core Principles

**Locard's Exchange Principle:**
> "Every contact leaves a trace."

In digital forensics, every action on a computer system leaves artifacts. Attackers cannot operate without creating evidence of their presence.

**The Four Pillars of Digital Forensics:**

```markdown
1. **Preservation**: Maintain evidence integrity
   - Use write blockers
   - Work on forensic copies
   - Document original state
   - Cryptographic hashing (MD5, SHA-256)

2. **Collection**: Acquire evidence properly
   - Follow standardized procedures
   - Use forensically sound tools
   - Capture volatile data first (memory, network connections)
   - Document environmental conditions

3. **Analysis**: Examine evidence systematically
   - Timeline reconstruction
   - Artifact correlation
   - Pattern recognition
   - Hypothesis testing

4. **Presentation**: Report findings clearly
   - Executive summaries for management
   - Technical details for investigators
   - Court-admissible documentation
   - Visual aids and timelines
```

### Legal Framework

**Federal Rules of Evidence (United States):**

**Rule 702 - Expert Testimony Requirements:**
```markdown
An expert witness may testify if:
1. Testimony based on sufficient facts/data
2. Testimony is the product of reliable principles and methods
3. Expert has reliably applied principles to the facts

Forensic examiners must demonstrate:
- Proper training and certification
- Use of accepted methodologies
- Peer-reviewed techniques
- Known error rates
- Documentation of all procedures
```

**Daubert Standard for Scientific Evidence:**
1. Can the theory/technique be tested?
2. Has it been subjected to peer review and publication?
3. What is the known or potential error rate?
4. Are there standards controlling the technique's operation?
5. Has it gained general acceptance in relevant scientific community?

**International Considerations:**

| Region | Primary Legislation | Key Requirements |
|--------|-------------------|------------------|
| **EU** | GDPR, ePrivacy Directive | Data protection impact assessment, legal basis for processing, cross-border transfer restrictions |
| **UK** | Computer Misuse Act 1990, Data Protection Act 2018 | Authorization required for access, data protection principles |
| **USA** | CFAA, ECPA, Stored Communications Act | Federal jurisdiction, wiretap restrictions, stored data access |
| **Canada** | PIPEDA, Criminal Code | Consent requirements, lawful interception |
| **Australia** | Privacy Act 1988, Cybercrime Act 2001 | Notification requirements, access controls |

### Chain of Custody

**Definition**: Chronological documentation showing seizure, custody, control, transfer, analysis, and disposition of evidence.

**Critical Elements:**

```python
#!/usr/bin/env python3
"""
Chain of Custody Management System
Tracks evidence handling with cryptographic verification
"""

import sqlite3
import hashlib
import hmac
from datetime import datetime
import json
import os

class ChainOfCustody:
    """
    Comprehensive chain of custody tracking with integrity verification
    """
    
    def __init__(self, db_path='evidence_coc.db', secret_key=None):
        self.db_path = db_path
        self.secret_key = secret_key or os.urandom(32)
        self.conn = sqlite3.connect(db_path)
        self.init_database()
    
    def init_database(self):
        """Initialize database schema"""
        cursor = self.conn.cursor()
        
        # Evidence registry table
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS evidence (
            evidence_id TEXT PRIMARY KEY,
            case_number TEXT NOT NULL,
            item_description TEXT,
            item_type TEXT,
            make_model TEXT,
            serial_number TEXT,
            collection_datetime TEXT,
            collected_by TEXT,
            collection_location TEXT,
            md5_hash TEXT,
            sha1_hash TEXT,
            sha256_hash TEXT,
            file_size INTEGER,
            condition_notes TEXT,
            created_timestamp TEXT
        )
        ''')
        
        # Custody transfer log
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS transfers (
            transfer_id INTEGER PRIMARY KEY AUTOINCREMENT,
            evidence_id TEXT,
            transfer_datetime TEXT,
            released_by TEXT,
            received_by TEXT,
            purpose TEXT,
            storage_location TEXT,
            seal_number TEXT,
            transport_method TEXT,
            notes TEXT,
            signature_released TEXT,
            signature_received TEXT,
            FOREIGN KEY (evidence_id) REFERENCES evidence (evidence_id)
        )
        ''')
        
        # Actions log table
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS actions (
            action_id INTEGER PRIMARY KEY AUTOINCREMENT,
            evidence_id TEXT,
            action_datetime TEXT,
            performed_by TEXT,
            action_type TEXT,
            action_description TEXT,
            tools_used TEXT,
            start_hash TEXT,
            end_hash TEXT,
            integrity_verified BOOLEAN,
            notes TEXT,
            FOREIGN KEY (evidence_id) REFERENCES evidence (evidence_id)
        )
        ''')
        
        # Access audit log
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS access_audit (
            audit_id INTEGER PRIMARY KEY AUTOINCREMENT,
            evidence_id TEXT,
            access_datetime TEXT,
            accessed_by TEXT,
            access_type TEXT,
            access_duration INTEGER,
            changes_made BOOLEAN,
            supervisor_approval TEXT,
            FOREIGN KEY (evidence_id) REFERENCES evidence (evidence_id)
        )
        ''')
        
        self.conn.commit()
    
    def register_evidence(self, evidence_data):
        """
        Register new evidence item with initial hash values
        
        Args:
            evidence_data (dict): Evidence metadata including:
                - evidence_id: Unique identifier
                - case_number: Associated case
                - item_description: What it is
                - Hashes: MD5, SHA1, SHA256
                - Collection details
        
        Returns:
            str: Evidence ID
        """
        cursor = self.conn.cursor()
        
        # Add creation timestamp
        evidence_data['created_timestamp'] = datetime.now().isoformat()
        
        cursor.execute('''
        INSERT INTO evidence (
            evidence_id, case_number, item_description, item_type,
            make_model, serial_number, collection_datetime, collected_by,
            collection_location, md5_hash, sha1_hash, sha256_hash,
            file_size, condition_notes, created_timestamp
        ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        ''', (
            evidence_data.get('evidence_id'),
            evidence_data.get('case_number'),
            evidence_data.get('item_description'),
            evidence_data.get('item_type'),
            evidence_data.get('make_model', ''),
            evidence_data.get('serial_number', ''),
            evidence_data.get('collection_datetime'),
            evidence_data.get('collected_by'),
            evidence_data.get('collection_location'),
            evidence_data.get('md5_hash'),
            evidence_data.get('sha1_hash'),
            evidence_data.get('sha256_hash'),
            evidence_data.get('file_size', 0),
            evidence_data.get('condition_notes', ''),
            evidence_data['created_timestamp']
        ))
        
        self.conn.commit()
        
        # Log initial registration
        self.log_action(
            evidence_data['evidence_id'],
            evidence_data['collected_by'],
            'REGISTRATION',
            f"Evidence initially registered in case {evidence_data['case_number']}",
            '',
            evidence_data.get('sha256_hash'),
            evidence_data.get('sha256_hash'),
            True
        )
        
        return evidence_data['evidence_id']
    
    def transfer_custody(self, transfer_data):
        """
        Record custody transfer between individuals
        
        Args:
            transfer_data (dict): Transfer details including:
                - evidence_id
                - released_by
                - received_by
                - purpose
                - storage_location
                - seal_number
        """
        cursor = self.conn.cursor()
        
        transfer_data['transfer_datetime'] = datetime.now().isoformat()
        
        cursor.execute('''
        INSERT INTO transfers (
            evidence_id, transfer_datetime, released_by, received_by,
            purpose, storage_location, seal_number, transport_method,
            notes, signature_released, signature_received
        ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        ''', (
            transfer_data['evidence_id'],
            transfer_data['transfer_datetime'],
            transfer_data['released_by'],
            transfer_data['received_by'],
            transfer_data['purpose'],
            transfer_data.get('storage_location', ''),
            transfer_data.get('seal_number', ''),
            transfer_data.get('transport_method', ''),
            transfer_data.get('notes', ''),
            transfer_data.get('signature_released', ''),
            transfer_data.get('signature_received', '')
        ))
        
        self.conn.commit()
    
    def log_action(self, evidence_id, performed_by, action_type, 
                   description, tools_used, start_hash, end_hash, 
                   integrity_verified, notes=''):
        """
        Log forensic action performed on evidence
        
        Action types: IMAGING, ANALYSIS, EXAMINATION, VERIFICATION
        """
        cursor = self.conn.cursor()
        
        cursor.execute('''
        INSERT INTO actions (
            evidence_id, action_datetime, performed_by, action_type,
            action_description, tools_used, start_hash, end_hash,
            integrity_verified, notes
        ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        ''', (
            evidence_id,
            datetime.now().isoformat(),
            performed_by,
            action_type,
            description,
            tools_used,
            start_hash,
            end_hash,
            integrity_verified,
            notes
        ))
        
        self.conn.commit()
    
    def verify_integrity(self, evidence_id, current_sha256):
        """
        Verify evidence hasn't been tampered with
        
        Returns:
            tuple: (bool: integrity_ok, str: message)
        """
        cursor = self.conn.cursor()
        
        cursor.execute(
            'SELECT sha256_hash FROM evidence WHERE evidence_id = ?',
            (evidence_id,)
        )
        
        result = cursor.fetchone()
        if not result:
            return False, "Evidence not found in database"
        
        original_hash = result[0]
        
        if original_hash == current_sha256:
            # Log successful verification
            self.log_action(
                evidence_id,
                'SYSTEM',
                'VERIFICATION',
                'Integrity verification successful',
                'ChainOfCustody.verify_integrity()',
                current_sha256,
                current_sha256,
                True
            )
            return True, "✓ Evidence integrity VERIFIED"
        else:
            # Log integrity failure - CRITICAL
            self.log_action(
                evidence_id,
                'SYSTEM',
                'INTEGRITY_FAILURE',
                f'INTEGRITY VIOLATION DETECTED. Expected: {original_hash}, Got: {current_sha256}',
                'ChainOfCustody.verify_integrity()',
                original_hash,
                current_sha256,
                False,
                'CRITICAL: Evidence may have been tampered with'
            )
            return False, f"✗ INTEGRITY VIOLATION!\nExpected: {original_hash}\nCurrent:  {current_sha256}"
    
    def generate_coc_report(self, evidence_id):
        """
        Generate comprehensive chain of custody report
        
        Returns:
            str: Formatted report ready for court presentation
        """
        cursor = self.conn.cursor()
        
        # Get evidence details
        cursor.execute('SELECT * FROM evidence WHERE evidence_id = ?', (evidence_id,))
        evidence_row = cursor.fetchone()
        
        if not evidence_row:
            return "ERROR: Evidence not found"
        
        # Column names
        columns = [desc[0] for desc in cursor.description]
        evidence = dict(zip(columns, evidence_row))
        
        # Get transfer history
        cursor.execute('''
        SELECT * FROM transfers 
        WHERE evidence_id = ? 
        ORDER BY transfer_datetime
        ''', (evidence_id,))
        
        transfers = []
        for row in cursor.fetchall():
            transfers.append(dict(zip([d[0] for d in cursor.description], row)))
        
        # Get actions log
        cursor.execute('''
        SELECT * FROM actions 
        WHERE evidence_id = ? 
        ORDER BY action_datetime
        ''', (evidence_id,))
        
        actions = []
        for row in cursor.fetchall():
            actions.append(dict(zip([d[0] for d in cursor.description], row)))
        
        # Generate formatted report
        report = f"""
{'='*100}
CHAIN OF CUSTODY REPORT
Case Number: {evidence['case_number']}
Evidence ID: {evidence['evidence_id']}
Generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
{'='*100}

EVIDENCE INFORMATION
{'─'*100}
Description:        {evidence['item_description']}
Type:              {evidence['item_type']}
Make/Model:        {evidence['make_model']}
Serial Number:     {evidence['serial_number']}
File Size:         {evidence['file_size']:,} bytes

COLLECTION DETAILS
{'─'*100}
Collected Date:    {evidence['collection_datetime']}
Collected By:      {evidence['collected_by']}
Location:          {evidence['collection_location']}
Condition:         {evidence['condition_notes']}

HASH VALUES (Evidence Integrity Verification)
{'─'*100}
MD5:              {evidence['md5_hash']}
SHA-1:            {evidence['sha1_hash']}
SHA-256:          {evidence['sha256_hash']}

CUSTODY TRANSFER LOG
{'─'*100}
"""
        
        for i, transfer in enumerate(transfers, 1):
            report += f"""
Transfer #{i}
  Date/Time:          {transfer['transfer_datetime']}
  Released By:        {transfer['released_by']} (Signature: {transfer['signature_released']})
  Received By:        {transfer['received_by']} (Signature: {transfer['signature_received']})
  Purpose:            {transfer['purpose']}
  Storage Location:   {transfer['storage_location']}
  Seal Number:        {transfer['seal_number']}
  Transport Method:   {transfer['transport_method']}
  Notes:              {transfer['notes']}
{'─'*100}
"""
        
        report += f"\nFORENSIC ACTIONS LOG\n{'─'*100}\n"
        
        for i, action in enumerate(actions, 1):
            integrity_icon = "✓" if action['integrity_verified'] else "✗"
            report += f"""
Action #{i} [{integrity_icon} Integrity: {'VERIFIED' if action['integrity_verified'] else 'FAILED'}]
  Date/Time:          {action['action_datetime']}
  Performed By:       {action['performed_by']}
  Action Type:        {action['action_type']}
  Description:        {action['action_description']}
  Tools Used:         {action['tools_used']}
  Hash Before:        {action['start_hash']}
  Hash After:         {action['end_hash']}
  Notes:              {action['notes']}
{'─'*100}
"""
        
        # Summary statistics
        total_transfers = len(transfers)
        total_actions = len(actions)
        failed_integrity_checks = sum(1 for a in actions if not a['integrity_verified'])
        
        report += f"""
SUMMARY
{'─'*100}
Total Custody Transfers:           {total_transfers}
Total Forensic Actions:            {total_actions}
Failed Integrity Checks:           {failed_integrity_checks}
Current Status:                    {'⚠ COMPROMISED' if failed_integrity_checks > 0 else '✓ INTACT'}

CERTIFICATION
{'─'*100}
I certify that this chain of custody report accurately reflects the handling
and examination of the evidence identified above. All procedures were conducted
in accordance with standard forensic practices and applicable regulations.

Examiner Signature: _________________________________  Date: ______________

Supervisor Review:  _________________________________  Date: ______________

{'='*100}
"""
        
        return report
    
    def export_for_legal(self, evidence_id, output_file):
        """
        Export chain of custody in JSON format for legal review
        """
        cursor = self.conn.cursor()
        
        # Collect all data
        cursor.execute('SELECT * FROM evidence WHERE evidence_id = ?', (evidence_id,))
        evidence = dict(zip([d[0] for d in cursor.description], cursor.fetchone()))
        
        cursor.execute('SELECT * FROM transfers WHERE evidence_id = ? ORDER BY transfer_datetime', (evidence_id,))
        transfers = [dict(zip([d[0] for d in cursor.description], row)) for row in cursor.fetchall()]
        
        cursor.execute('SELECT * FROM actions WHERE evidence_id = ? ORDER BY action_datetime', (evidence_id,))
        actions = [dict(zip([d[0] for d in cursor.description], row)) for row in cursor.fetchall()]
        
        legal_export = {
            'export_timestamp': datetime.now().isoformat(),
            'evidence': evidence,
            'custody_transfers': transfers,
            'forensic_actions': actions,
            'integrity_summary': {
                'total_transfers': len(transfers),
                'total_actions': len(actions),
                'failed_verifications': sum(1 for a in actions if not a['integrity_verified'])
            }
        }
        
        with open(output_file, 'w') as f:
            json.dump(legal_export, f, indent=2)
        
        return f"Legal export saved to {output_file}"

# Example usage
if __name__ == "__main__":
    coc = ChainOfCustody()
    
    # Register evidence
    evidence = {
        'evidence_id': 'EVD-2024-001',
        'case_number': 'CASE-2024-042',
        'item_description': 'Dell Latitude 7420 laptop seized from suspect workstation',
        'item_type': 'Computer - Laptop',
        'make_model': 'Dell Latitude 7420',
        'serial_number': 'SN123456789ABC',
        'collection_datetime': '2024-01-28T10:30:00',
        'collected_by': 'Detective Sarah Johnson, Badge #4521',
        'collection_location': 'TechCorp HQ, Building A, Cubicle 315',
        'md5_hash': 'a1b2c3d4e5f6g7h8i9j0',
        'sha1_hash': 'sha1hashvalue123456',
        'sha256_hash': 'sha256hashvalueabcdef123456',
        'file_size': 512000000000,  # 512GB SSD
        'condition_notes': 'Device powered off, no visible physical damage, sealed in evidence bag #4521'
    }
    
    evidence_id = coc.register_evidence(evidence)
    print(f"Evidence registered: {evidence_id}")
    
    # Transfer to forensics lab
    coc.transfer_custody({
        'evidence_id': 'EVD-2024-001',
        'released_by': 'Detective Sarah Johnson',
        'received_by': 'Forensic Examiner Michael Chen, CFCE #8842',
        'purpose': 'Forensic imaging and analysis',
        'storage_location': 'Digital Forensics Lab, Evidence Locker B-7',
        'seal_number': 'SEAL-2024-001',
        'transport_method': 'Chain-of-custody evidence bag, direct hand-off',
        'signature_released': 'S. Johnson',
        'signature_received': 'M. Chen'
    })
    
    # Log forensic imaging
    coc.log_action(
        'EVD-2024-001',
        'Forensic Examiner Michael Chen',
        'IMAGING',
        'Created forensic bit-by-bit image using write-blocker',
        'FTK Imager v4.7.1, Tableau T8-R2 write blocker',
        'sha256hashvalueabcdef123456',
        'sha256hashvalueabcdef123456',
        True,
        'Image file: EVD-2024-001.E01, Verification: PASSED'
    )
    
    # Verify integrity
    integrity_ok, message = coc.verify_integrity('EVD-2024-001', 'sha256hashvalueabcdef123456')
    print(f"\n{message}\n")
    
    # Generate report
    report = coc.generate_coc_report('EVD-2024-001')
    print(report)
    
    # Export for legal team
    coc.export_for_legal('EVD-2024-001', 'EVD-2024-001_legal_export.json')
```

[Continuing with evidence acquisition, disk forensics, memory forensics, and more sections...]

## 3.2 Evidence Acquisition

Evidence acquisition is the process of collecting digital evidence in a forensically sound manner. The order and method of collection directly impacts the admissibility and integrity of evidence.

### Order of Volatility

Digital evidence has varying degrees of volatility. Follow the RFC 3227 order of volatility:

```markdown
## Order of Volatility (Most to Least Volatile)

1. **CPU Registers, Cache** (nanoseconds)
   - Processor state
   - Cache memory
   
2. **Network Connections, ARP Cache** (seconds)
   - Active connections (netstat)
   - Routing tables
   - ARP cache entries
   
3. **Memory (RAM)** (minutes)
   - Running processes
   - Loaded modules/DLLs
   - Network connections
   - Decrypted data
   - Malware in memory
   
4. **Temporary File Systems** (minutes to hours)
   - Swap/page file
   - /tmp directories
   - Recent files cache
   
5. **Hard Drive/SSD** (months to years)
   - Installed programs
   - User files
   - Log files
   - Deleted files
   
6. **Remote Logging Data** (varies)
   - SIEM logs
   - Cloud backups
   - Offsite storage
   
7. **Physical Configuration** (permanent)
   - Hardware configuration
   - Topology diagrams
   
8. **Archival Media** (permanent)
   - Backup tapes
   - Optical media (CD/DVD)
```

### Live System Data Collection

**Windows Live Collection Script:**

```powershell
# Windows-Live-Response.ps1
# Comprehensive live data collection for Windows systems

param(
    [string]$OutputDir = "C:\ForensicCapture\$(Get-Date -Format 'yyyyMMdd_HHmmss')"
)

# Create output directory
New-Item -ItemType Directory -Path $OutputDir -Force | Out-Null

Write-Host "[*] Starting live data collection to: $OutputDir" -ForegroundColor Green

# System Information
Write-Host "[+] Collecting system information..."
systeminfo > "$OutputDir\systeminfo.txt"
Get-ComputerInfo | Out-File "$OutputDir\computerinfo.txt"
Get-WmiObject Win32_OperatingSystem | Out-File "$OutputDir\os-info.txt"

# Current Date/Time (important for timeline)
Get-Date | Out-File "$OutputDir\collection-timestamp.txt"

# Network Connections
Write-Host "[+] Collecting network connections..."
netstat -ano > "$OutputDir\netstat-ano.txt"
netstat -anob > "$OutputDir\netstat-anob.txt" 2>&1
Get-NetTCPConnection | Export-Csv "$OutputDir\tcp-connections.csv" -NoTypeInformation
Get-NetUDPEndpoint | Export-Csv "$OutputDir\udp-endpoints.csv" -NoTypeInformation

# ARP Cache
arp -a > "$OutputDir\arp-cache.txt"
Get-NetNeighbor | Export-Csv "$OutputDir\arp-table.csv" -NoTypeInformation

# Routing Table
route print > "$OutputDir\route-table.txt"
Get-NetRoute | Export-Csv "$OutputDir\routing-table.csv" -NoTypeInformation

# DNS Cache
ipconfig /displaydns > "$OutputDir\dns-cache.txt"
Get-DnsClientCache | Export-Csv "$OutputDir\dns-cache.csv" -NoTypeInformation

# NetBIOS Cache
nbtstat -c > "$OutputDir\netbios-cache.txt"

# Running Processes
Write-Host "[+] Collecting process information..."
Get-Process | Format-List * | Out-File "$OutputDir\processes-detailed.txt"
tasklist /v > "$OutputDir\tasklist-verbose.txt"
Get-WmiObject Win32_Process | Select-Object ProcessId, Name, CommandLine, ParentProcessId, CreationDate | 
    Export-Csv "$OutputDir\processes-wmi.csv" -NoTypeInformation

# Process-to-Network mapping
Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess, 
    @{Name="ProcessName";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | 
    Export-Csv "$OutputDir\process-network-mapping.csv" -NoTypeInformation

# Services
Write-Host "[+] Collecting service information..."
Get-Service | Export-Csv "$OutputDir\services.csv" -NoTypeInformation
sc query > "$OutputDir\services-sc.txt"

# Scheduled Tasks
Write-Host "[+] Collecting scheduled tasks..."
schtasks /query /fo CSV /v > "$OutputDir\scheduled-tasks.csv"
Get-ScheduledTask | Export-Csv "$OutputDir\scheduled-tasks-ps.csv" -NoTypeInformation

# Logged-in Users
Write-Host "[+] Collecting user session information..."
quser > "$OutputDir\logged-users.txt" 2>&1
Get-WmiObject Win32_LoggedOnUser | Select-Object Antecedent, Dependent | 
    Export-Csv "$OutputDir\logged-users-wmi.csv" -NoTypeInformation

# User Accounts
net user > "$OutputDir\user-accounts.txt"
Get-LocalUser | Export-Csv "$OutputDir\local-users.csv" -NoTypeInformation
Get-LocalGroup | Export-Csv "$OutputDir\local-groups.csv" -NoTypeInformation

# Loaded Drivers
Write-Host "[+] Collecting driver information..."
driverquery /v > "$OutputDir\drivers.txt"
Get-WmiObject Win32_SystemDriver | Export-Csv "$OutputDir\drivers-wmi.csv" -NoTypeInformation

# Loaded DLLs (for all processes)
Write-Host "[+] Collecting loaded modules..."
Get-Process | ForEach-Object { 
    $proc = $_
    try {
        $_.Modules | Select-Object @{Name="ProcessName";Expression={$proc.Name}}, 
                                   @{Name="PID";Expression={$proc.Id}}, 
                                   ModuleName, FileName, Size
    } catch {}
} | Export-Csv "$OutputDir\loaded-dlls.csv" -NoTypeInformation

# Open Files
Write-Host "[+] Collecting open files..."
openfiles > "$OutputDir\open-files.txt" 2>&1
Get-SmbOpenFile | Export-Csv "$OutputDir\smb-open-files.csv" -NoTypeInformation 2>&1

# Environment Variables
Write-Host "[+] Collecting environment variables..."
Get-ChildItem Env: | Out-File "$OutputDir\environment-variables.txt"

# Clipboard Contents
Write-Host "[+] Collecting clipboard..."
Get-Clipboard | Out-File "$OutputDir\clipboard.txt" 2>&1

# Installed Software
Write-Host "[+] Collecting installed software..."
Get-WmiObject Win32_Product | Select-Object Name, Version, Vendor, InstallDate | 
    Export-Csv "$OutputDir\installed-software.csv" -NoTypeInformation
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | 
    Export-Csv "$OutputDir\installed-programs.csv" -NoTypeInformation

# Startup Programs
Write-Host "[+] Collecting startup programs..."
Get-CimInstance Win32_StartupCommand | Export-Csv "$OutputDir\startup-programs.csv" -NoTypeInformation

# Firewall Rules
Write-Host "[+] Collecting firewall rules..."
netsh advfirewall show allprofiles > "$OutputDir\firewall-profiles.txt"
Get-NetFirewallRule | Export-Csv "$OutputDir\firewall-rules.csv" -NoTypeInformation

# Network Shares
Write-Host "[+] Collecting network shares..."
net share > "$OutputDir\network-shares.txt"
Get-SmbShare | Export-Csv "$OutputDir\smb-shares.csv" -NoTypeInformation

# USB History
Write-Host "[+] Collecting USB device history..."
Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Enum\USBSTOR\*\*' | 
    Select-Object FriendlyName, Mfg | Export-Csv "$OutputDir\usb-history.csv" -NoTypeInformation

# Prefetch Files List
Write-Host "[+] Listing prefetch files..."
Get-ChildItem C:\Windows\Prefetch\*.pf -ErrorAction SilentlyContinue | 
    Select-Object Name, CreationTime, LastWriteTime, Length | 
    Export-Csv "$OutputDir\prefetch-files.csv" -NoTypeInformation

# Recent Documents
Write-Host "[+] Collecting recent documents..."
Get-ChildItem "$env:APPDATA\Microsoft\Windows\Recent\*" -ErrorAction SilentlyContinue | 
    Select-Object Name, CreationTime, LastWriteTime | 
    Export-Csv "$OutputDir\recent-documents.csv" -NoTypeInformation

# Browser History (basic - for comprehensive use browser forensics tools)
Write-Host "[+] Collecting basic browser artifacts..."
# Chrome
$ChromeHistory = "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\History"
if (Test-Path $ChromeHistory) {
    Copy-Item $ChromeHistory "$OutputDir\chrome-history.db" -ErrorAction SilentlyContinue
}

# Edge
$EdgeHistory = "$env:LOCALAPPDATA\Microsoft\Edge\User Data\Default\History"
if (Test-Path $EdgeHistory) {
    Copy-Item $EdgeHistory "$OutputDir\edge-history.db" -ErrorAction SilentlyContinue
}

# Event Logs (copy for offline analysis)
Write-Host "[+] Collecting event logs..."
$LogDir = "$OutputDir\EventLogs"
New-Item -ItemType Directory -Path $LogDir -Force | Out-Null

wevtutil epl Security "$LogDir\Security.evtx"
wevtutil epl System "$LogDir\System.evtx"
wevtutil epl Application "$LogDir\Application.evtx"
wevtutil epl "Microsoft-Windows-PowerShell/Operational" "$LogDir\PowerShell-Operational.evtx" 2>&1
wevtutil epl "Microsoft-Windows-Sysmon/Operational" "$LogDir\Sysmon.evtx" 2>&1

# Registry Hives (for offline analysis)
Write-Host "[+] Backing up registry hives..."
$RegDir = "$OutputDir\Registry"
New-Item -ItemType Directory -Path $RegDir -Force | Out-Null

reg save HKLM\SAM "$RegDir\SAM" /y 2>&1
reg save HKLM\SECURITY "$RegDir\SECURITY" /y 2>&1
reg save HKLM\SOFTWARE "$RegDir\SOFTWARE" /y 2>&1
reg save HKLM\SYSTEM "$RegDir\SYSTEM" /y 2>&1
reg save HKU\.DEFAULT "$RegDir\DEFAULT" /y 2>&1

# Memory Dump (if tool available)
Write-Host "[+] Attempting memory capture..."
if (Test-Path ".\DumpIt.exe") {
    .\DumpIt.exe /Q /O "$OutputDir\memory.dmp"
} elseif (Test-Path ".\winpmem.exe") {
    .\winpmem.exe "$OutputDir\memory.raw"
} else {
    Write-Host "[-] Memory capture tool not found (DumpIt.exe or winpmem.exe)" -ForegroundColor Yellow
}

# Hash all collected files
Write-Host "[+] Generating file hashes..."
Get-ChildItem -Path $OutputDir -Recurse -File | 
    ForEach-Object {
        $hash = Get-FileHash $_.FullName -Algorithm SHA256
        [PSCustomObject]@{
            File = $_.FullName
            SHA256 = $hash.Hash
            Size = $_.Length
        }
    } | Export-Csv "$OutputDir\file-hashes.csv" -NoTypeInformation

# Create acquisition log
$AcqLog = @"
Windows Live Data Collection Log
================================
Collection Date: $(Get-Date)
Computer Name: $env:COMPUTERNAME
User Account: $env:USERNAME
Domain: $env:USERDOMAIN
Output Directory: $OutputDir

Collection completed successfully.
"@

$AcqLog | Out-File "$OutputDir\acquisition-log.txt"

Write-Host "[✓] Live data collection complete!" -ForegroundColor Green
Write-Host "[*] Output location: $OutputDir" -ForegroundColor Cyan
Write-Host "[*] Total files: $(((Get-ChildItem $OutputDir -Recurse -File).Count))" -ForegroundColor Cyan
```

**Linux Live Collection Script:**

```bash
#!/bin/bash
# Linux-Live-Response.sh
# Comprehensive live data collection for Linux systems

OUTPUT_DIR="/tmp/forensic-capture-$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

echo "[*] Starting live data collection to: $OUTPUT_DIR"

# System Information
echo "[+] Collecting system information..."
uname -a > "$OUTPUT_DIR/uname.txt"
hostname > "$OUTPUT_DIR/hostname.txt"
date > "$OUTPUT_DIR/collection-timestamp.txt"
cat /etc/os-release > "$OUTPUT_DIR/os-release.txt"
uptime > "$OUTPUT_DIR/uptime.txt"

# Network Connections
echo "[+] Collecting network information..."
netstat -antup > "$OUTPUT_DIR/netstat-antup.txt"
ss -antup > "$OUTPUT_DIR/ss-antup.txt"
lsof -i > "$OUTPUT_DIR/lsof-network.txt"

# ARP Cache
arp -a > "$OUTPUT_DIR/arp-cache.txt"
ip neighbor show > "$OUTPUT_DIR/ip-neighbor.txt"

# Routing Table
route -n > "$OUTPUT_DIR/route.txt"
ip route show > "$OUTPUT_DIR/ip-route.txt"

# Network Interfaces
ifconfig -a > "$OUTPUT_DIR/ifconfig.txt" 2>/dev/null || ip addr show > "$OUTPUT_DIR/ip-addr.txt"

# DNS Configuration
cat /etc/resolv.conf > "$OUTPUT_DIR/resolv.conf"
cat /etc/hosts > "$OUTPUT_DIR/hosts.txt"

# Running Processes
echo "[+] Collecting process information..."
ps auxww > "$OUTPUT_DIR/ps-aux.txt"
ps -ef > "$OUTPUT_DIR/ps-ef.txt"
pstree -p > "$OUTPUT_DIR/pstree.txt"
lsof -p $(pgrep -d, .) > "$OUTPUT_DIR/lsof-processes.txt" 2>/dev/null

# Process-to-Network mapping
lsof -i -n -P > "$OUTPUT_DIR/process-network-map.txt"

# Logged-in Users
echo "[+] Collecting user information..."
who > "$OUTPUT_DIR/who.txt"
w > "$OUTPUT_DIR/w.txt"
last > "$OUTPUT_DIR/last.txt"
lastlog > "$OUTPUT_DIR/lastlog.txt"

# User Accounts
cat /etc/passwd > "$OUTPUT_DIR/passwd.txt"
cat /etc/group > "$OUTPUT_DIR/group.txt"
cat /etc/shadow > "$OUTPUT_DIR/shadow.txt" 2>/dev/null

# Sudo Configuration
cat /etc/sudoers > "$OUTPUT_DIR/sudoers.txt" 2>/dev/null
ls -la /etc/sudoers.d/ > "$OUTPUT_DIR/sudoers.d-list.txt"

# SSH Configuration
cat /etc/ssh/sshd_config > "$OUTPUT_DIR/sshd_config.txt"
ls -laR /home/*/.ssh/ > "$OUTPUT_DIR/user-ssh-keys.txt" 2>/dev/null

# Loaded Kernel Modules
echo "[+] Collecting kernel information..."
lsmod > "$OUTPUT_DIR/lsmod.txt"
cat /proc/modules > "$OUTPUT_DIR/proc-modules.txt"

# Open Files
echo "[+] Collecting open files..."
lsof > "$OUTPUT_DIR/lsof-all.txt"

# Mounted Filesystems
echo "[+] Collecting filesystem information..."
mount > "$OUTPUT_DIR/mount.txt"
df -h > "$OUTPUT_DIR/df.txt"
cat /etc/fstab > "$OUTPUT_DIR/fstab.txt"

# Environment Variables
echo "[+] Collecting environment..."
env > "$OUTPUT_DIR/environment.txt"

# Cron Jobs
echo "[+] Collecting scheduled tasks..."
crontab -l > "$OUTPUT_DIR/crontab-current-user.txt" 2>/dev/null
ls -la /etc/cron* > "$OUTPUT_DIR/cron-dirs.txt"
cat /etc/crontab > "$OUTPUT_DIR/etc-crontab.txt"

# Systemd Timers
systemctl list-timers --all > "$OUTPUT_DIR/systemd-timers.txt" 2>/dev/null

# Startup Services
echo "[+] Collecting startup services..."
systemctl list-units --type=service > "$OUTPUT_DIR/systemd-services.txt" 2>/dev/null
ls -la /etc/init.d/ > "$OUTPUT_DIR/init.d-services.txt"
chkconfig --list > "$OUTPUT_DIR/chkconfig.txt" 2>/dev/null

# Installed Packages
echo "[+] Collecting installed software..."
dpkg -l > "$OUTPUT_DIR/dpkg-list.txt" 2>/dev/null
rpm -qa > "$OUTPUT_DIR/rpm-list.txt" 2>/dev/null
apt list --installed > "$OUTPUT_DIR/apt-installed.txt" 2>/dev/null

# Firewall Rules
echo "[+] Collecting firewall rules..."
iptables -L -n -v > "$OUTPUT_DIR/iptables.txt" 2>/dev/null
ip6tables -L -n -v > "$OUTPUT_DIR/ip6tables.txt" 2>/dev/null
firewall-cmd --list-all > "$OUTPUT_DIR/firewalld.txt" 2>/dev/null

# USB/Block Devices
echo "[+] Collecting device information..."
lsusb > "$OUTPUT_DIR/lsusb.txt" 2>/dev/null
lsblk > "$OUTPUT_DIR/lsblk.txt"
fdisk -l > "$OUTPUT_DIR/fdisk.txt" 2>/dev/null

# PCI Devices
lspci > "$OUTPUT_DIR/lspci.txt"

# System Logs (copy for offline analysis)
echo "[+] Collecting system logs..."
LOG_DIR="$OUTPUT_DIR/logs"
mkdir -p "$LOG_DIR"

cp -p /var/log/auth.log "$LOG_DIR/" 2>/dev/null
cp -p /var/log/secure "$LOG_DIR/" 2>/dev/null
cp -p /var/log/syslog "$LOG_DIR/" 2>/dev/null
cp -p /var/log/messages "$LOG_DIR/" 2>/dev/null
cp -p /var/log/kern.log "$LOG_DIR/" 2>/dev/null
cp -pr /var/log/apache2 "$LOG_DIR/" 2>/dev/null
cp -pr /var/log/nginx "$LOG_DIR/" 2>/dev/null

# Bash History (all users)
echo "[+] Collecting command history..."
find /home -name ".bash_history" -exec cp -p {} "$OUTPUT_DIR/" \; 2>/dev/null
cp -p /root/.bash_history "$OUTPUT_DIR/root-bash_history" 2>/dev/null

# Recent Files
find / -type f -mtime -7 2>/dev/null | head -1000 > "$OUTPUT_DIR/recent-files.txt"

# SUID/SGID Files
echo "[+] Finding SUID/SGID files..."
find / -perm -4000 -type f 2>/dev/null > "$OUTPUT_DIR/suid-files.txt"
find / -perm -2000 -type f 2>/dev/null > "$OUTPUT_DIR/sgid-files.txt"

# World-writable Files
find / -perm -2 -type f 2>/dev/null | head -1000 > "$OUTPUT_DIR/world-writable.txt"

# Memory Information
cat /proc/meminfo > "$OUTPUT_DIR/meminfo.txt"
free -h > "$OUTPUT_DIR/free.txt"

# CPU Information
cat /proc/cpuinfo > "$OUTPUT_DIR/cpuinfo.txt"

# Memory Dump (if tool available)
echo "[+] Attempting memory capture..."
if command -v avml &> /dev/null; then
    avml "$OUTPUT_DIR/memory.lime"
elif command -v lime &> /dev/null; then
    insmod /path/to/lime.ko "path=$OUTPUT_DIR/memory.lime format=lime"
else
    echo "[-] Memory capture tool not found (avml or LiME)"
fi

# Generate file hashes
echo "[+] Generating file hashes..."
find "$OUTPUT_DIR" -type f -exec sha256sum {} \; > "$OUTPUT_DIR/file-hashes.txt"

# Create acquisition log
cat > "$OUTPUT_DIR/acquisition-log.txt" << EOF
Linux Live Data Collection Log
================================
Collection Date: $(date)
Hostname: $(hostname)
User: $(whoami)
Kernel: $(uname -r)
Output Directory: $OUTPUT_DIR

Collection completed successfully.
EOF

echo "[✓] Live data collection complete!"
echo "[*] Output location: $OUTPUT_DIR"
echo "[*] Total files: $(find $OUTPUT_DIR -type f | wc -l)"

# Compress for easy transfer
tar -czf "$OUTPUT_DIR.tar.gz" -C "$(dirname $OUTPUT_DIR)" "$(basename $OUTPUT_DIR)"
echo "[*] Archive created: $OUTPUT_DIR.tar.gz"
```

### Disk Imaging

**Creating Forensic Images:**

```bash
#!/bin/bash
# Forensic Disk Imaging Script

SOURCE_DEVICE="/dev/sda"
IMAGE_FILE="/mnt/evidence/case001_$(date +%Y%m%d)_disk.dd"
LOG_FILE="/mnt/evidence/case001_imaging.log"

echo "Forensic Disk Imaging Script"
echo "============================="
echo ""
echo "Source Device: $SOURCE_DEVICE"
echo "Image File: $IMAGE_FILE"
echo "Log File: $LOG_FILE"
echo ""

# Verify source device exists
if [ ! -b "$SOURCE_DEVICE" ]; then
    echo "Error: Source device $SOURCE_DEVICE not found"
    exit 1
fi

# Create image directory if it doesn't exist
mkdir -p "$(dirname $IMAGE_FILE)"

# Start logging
echo "Imaging started: $(date)" | tee -a "$LOG_FILE"

# Method 1: Using dd (basic but effective)
echo "[*] Starting dd imaging..." | tee -a "$LOG_FILE"
dd if="$SOURCE_DEVICE" of="$IMAGE_FILE" bs=4M conv=noerror,sync status=progress 2>&1 | tee -a "$LOG_FILE"

# Method 2: Using dcfldd (enhanced dd with hashing)
# dcfldd if="$SOURCE_DEVICE" of="$IMAGE_FILE" bs=4M hash=md5,sha256 hashlog="$IMAGE_FILE.hash" conv=noerror,sync

# Method 3: Using dc3dd (Department of Defense version)
# dc3dd if="$SOURCE_DEVICE" of="$IMAGE_FILE" hash=md5 hash=sha256 log="$LOG_FILE" verb=on

# Calculate hashes of the image
echo "[*] Calculating image hashes..." | tee -a "$LOG_FILE"
md5sum "$IMAGE_FILE" > "$IMAGE_FILE.md5"
sha256sum "$IMAGE_FILE" > "$IMAGE_FILE.sha256"

echo "MD5: $(cat $IMAGE_FILE.md5)" | tee -a "$LOG_FILE"
echo "SHA256: $(cat $IMAGE_FILE.sha256)" | tee -a "$LOG_FILE"

# Get source device hash for comparison
echo "[*] Calculating source device hash..." | tee -a "$LOG_FILE"
SOURCE_MD5=$(dd if="$SOURCE_DEVICE" bs=4M | md5sum | awk '{print $1}')
echo "Source MD5: $SOURCE_MD5" | tee -a "$LOG_FILE"

# Verify image integrity
IMAGE_MD5=$(awk '{print $1}' "$IMAGE_FILE.md5")
if [ "$SOURCE_MD5" == "$IMAGE_MD5" ]; then
    echo "[✓] Image verification PASSED - Hashes match" | tee -a "$LOG_FILE"
else
    echo "[✗] Image verification FAILED - Hashes do NOT match" | tee -a "$LOG_FILE"
fi

echo "Imaging completed: $(date)" | tee -a "$LOG_FILE"
echo "Image size: $(ls -lh $IMAGE_FILE | awk '{print $5}')" | tee -a "$LOG_FILE"
```

**FTK Imager Command-Line Usage:**

```bash
# FTK Imager (Windows)
# Command-line imaging

# Create E01 (EnCase) format image
ftkimager.exe \\.\PhysicalDrive0 "E:\Evidence\Case001" --e01 --frag 1500MB --compress 6

# Create raw/dd image
ftkimager.exe \\.\PhysicalDrive0 "E:\Evidence\Case001.dd" --raw

# Verify image
ftkimager.exe "E:\Evidence\Case001.E01" --verify
```

[Continuing with memory capture, network forensics, and more...]
## 3.3 Disk Forensics

Disk forensics involves examining storage media to recover data, identify user actions, and build timelines of activity.

### File System Analysis

**NTFS (Windows) Deep Dive:**

```python
#!/usr/bin/env python3
"""
NTFS Artifact Parser
Extracts forensic artifacts from NTFS file system
"""

import struct
import datetime

class NTFSForensics:
    """Parse NTFS artifacts for forensic analysis"""
    
    def parse_mft_entry(self, mft_data, entry_number):
        """
        Parse Master File Table entry
        
        MFT Entry Structure:
        - Signature: "FILE" (4 bytes)
        - Update Sequence Offset: 0x30
        - Update Sequence Size: Usually 3
        - $LogFile Sequence Number
        - Sequence number
        - Hard link count
        - First attribute offset
        - Flags: 0x01 (in use), 0x02 (directory)
        - Used size of MFT entry
        - Allocated size
        """
        
        signature = mft_data[0:4]
        if signature != b'FILE':
            return None
        
        # Flags at offset 0x16
        flags = struct.unpack('<H', mft_data[0x16:0x18])[0]
        is_active = bool(flags & 0x01)
        is_directory = bool(flags & 0x02)
        
        # Parse timestamps (STANDARD_INFORMATION attribute)
        # Located at offset 0x18 in $STANDARD_INFORMATION
        
        entry_info = {
            'entry_number': entry_number,
            'is_active': is_active,
            'is_directory': is_directory,
            'signature': signature.decode('ascii'),
        }
        
        # Parse attributes ($STANDARD_INFORMATION, $FILE_NAME)
        attributes = self.parse_attributes(mft_data)
        entry_info['attributes'] = attributes
        
        return entry_info
    
    def parse_usn_journal(self, usn_data):
        """
        Parse USN (Update Sequence Number) Journal
        Tracks file system changes
        
        USN Journal provides:
        - File creation, deletion, rename
        - Attribute modifications
        - Timestamp of changes
        - User who made changes
        """
        
        changes = []
        offset = 0
        
        while offset < len(usn_data):
            # USN record structure
            record_length = struct.unpack('<I', usn_data[offset:offset+4])[0]
            
            if record_length == 0:
                break
            
            # Major version (offset 0x04)
            major_version = struct.unpack('<H', usn_data[offset+4:offset+6])[0]
            
            # File Reference Number (offset 0x08)
            file_ref = struct.unpack('<Q', usn_data[offset+8:offset+16])[0]
            
            # Timestamp (offset 0x18) - Windows FILETIME
            timestamp_raw = struct.unpack('<Q', usn_data[offset+0x18:offset+0x20])[0]
            timestamp = self.filetime_to_dt(timestamp_raw)
            
            # Reason (offset 0x20) - What changed
            reason = struct.unpack('<I', usn_data[offset+0x20:offset+0x24])[0]
            
            # Filename (offset 0x3C onwards)
            filename_length = struct.unpack('<H', usn_data[offset+0x3A:offset+0x3C])[0]
            filename_offset = offset + 0x3C
            filename = usn_data[filename_offset:filename_offset+filename_length].decode('utf-16-le', errors='ignore')
            
            changes.append({
                'timestamp': timestamp,
                'filename': filename,
                'file_reference': file_ref,
                'reason': self.decode_usn_reason(reason)
            })
            
            offset += record_length
        
        return changes
    
    def decode_usn_reason(self, reason_flags):
        """Decode USN reason flags to human-readable actions"""
        
        reasons = []
        reason_map = {
            0x00000001: 'DATA_OVERWRITE',
            0x00000002: 'DATA_EXTEND',
            0x00000004: 'DATA_TRUNCATION',
            0x00000010: 'NAMED_DATA_OVERWRITE',
            0x00000020: 'NAMED_DATA_EXTEND',
            0x00000040: 'NAMED_DATA_TRUNCATION',
            0x00000100: 'FILE_CREATE',
            0x00000200: 'FILE_DELETE',
            0x00000400: 'EA_CHANGE',
            0x00000800: 'SECURITY_CHANGE',
            0x00001000: 'RENAME_OLD_NAME',
            0x00002000: 'RENAME_NEW_NAME',
            0x00004000: 'INDEXABLE_CHANGE',
            0x00008000: 'BASIC_INFO_CHANGE',
            0x00010000: 'HARD_LINK_CHANGE',
            0x00020000: 'COMPRESSION_CHANGE',
            0x00040000: 'ENCRYPTION_CHANGE',
            0x00080000: 'OBJECT_ID_CHANGE',
            0x00100000: 'REPARSE_POINT_CHANGE',
            0x00200000: 'STREAM_CHANGE',
            0x80000000: 'CLOSE',
        }
        
        for flag, description in reason_map.items():
            if reason_flags & flag:
                reasons.append(description)
        
        return reasons
    
    def parse_alternate_data_streams(self, filename):
        """
        Find Alternate Data Streams (ADS)
        Hidden data storage mechanism in NTFS
        
        Usage: streams.exe filename
        Or: dir /r (shows ADS)
        """
        
        import subprocess
        
        # Using Windows command
        result = subprocess.run(
            ['cmd', '/c', 'dir', '/r', filename],
            capture_output=True,
            text=True
        )
        
        # Parse output for ADS
        ads_list = []
        for line in result.stdout.split('\n'):
            if ':' in line and '$DATA' in line:
                # Extract ADS name
                ads_list.append(line.strip())
        
        return ads_list
    
    def filetime_to_dt(self, filetime):
        """Convert Windows FILETIME to datetime"""
        
        # FILETIME is 100-nanosecond intervals since 1601-01-01
        epoch = datetime.datetime(1601, 1, 1)
        delta = datetime.timedelta(microseconds=filetime/10)
        return epoch + delta

# Autopsy/Sleuth Kit Usage
"""
# Command-line forensics with Sleuth Kit

# List partitions
mmls disk.dd

# File system info
fsstat -o 63 disk.dd

# List files in root directory
fls -o 63 disk.dd

# Display file content
icat -o 63 disk.dd 5  # inode 5

# Timeline generation
fls -r -m / -o 63 disk.dd > bodyfile.txt
mactime -b bodyfile.txt -d > timeline.csv

# File recovery
tsk_recover -e -o 63 disk.dd recovered_files/

# Search for deleted files
fls -d -r -o 63 disk.dd
"""
```

### Windows Registry Forensics

**Critical Registry Hives:**

```python
#!/usr/bin/env python3
"""
Windows Registry Forensics
Parse registry hives for artifacts
"""

from Registry import Registry
import datetime

class RegistryForensics:
    """Extract forensic artifacts from Windows Registry"""
    
    def __init__(self, hive_path):
        self.reg = Registry.Registry(hive_path)
    
    def get_recent_docs(self):
        """Get recently accessed documents"""
        
        recent_docs = []
        
        try:
            key = self.reg.open("Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\RecentDocs")
            
            for value in key.values():
                if value.name() != "MRUListEx":
                    recent_docs.append({
                        'name': value.name(),
                        'data': value.value(),
                        'timestamp': key.timestamp()
                    })
        
        except Registry.RegistryKeyNotFoundException:
            pass
        
        return recent_docs
    
    def get_user_assist(self):
        """
        Parse UserAssist key for program execution
        Tracks GUI-launched programs
        """
        
        user_assist_data = []
        
        try:
            key = self.reg.open("Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\UserAssist")
            
            for guid_key in key.subkeys():
                count_key = guid_key.subkey("Count")
                
                for value in count_key.values():
                    # Decode ROT13 encoded name
                    program_name = self.rot13_decode(value.name())
                    
                    # Parse execution data
                    data = value.value()
                    if len(data) >= 72:
                        # Windows 7+ format
                        run_count = struct.unpack('<I', data[4:8])[0]
                        last_exec_raw = struct.unpack('<Q', data[60:68])[0]
                        
                        if last_exec_raw != 0:
                            last_exec = self.filetime_to_dt(last_exec_raw)
                        else:
                            last_exec = None
                        
                        user_assist_data.append({
                            'program': program_name,
                            'run_count': run_count,
                            'last_executed': last_exec
                        })
        
        except Registry.RegistryKeyNotFoundException:
            pass
        
        return user_assist_data
    
    def get_installed_software(self):
        """Get list of installed software"""
        
        software = []
        
        paths = [
            "Software\\Microsoft\\Windows\\CurrentVersion\\Uninstall",
            "Software\\Wow6432Node\\Microsoft\\Windows\\CurrentVersion\\Uninstall"
        ]
        
        for path in paths:
            try:
                key = self.reg.open(path)
                
                for subkey in key.subkeys():
                    try:
                        name = subkey.value("DisplayName").value()
                        version = subkey.value("DisplayVersion").value()
                        publisher = subkey.value("Publisher").value()
                        install_date = subkey.value("InstallDate").value()
                        
                        software.append({
                            'name': name,
                            'version': version,
                            'publisher': publisher,
                            'install_date': install_date
                        })
                    except Registry.RegistryValueNotFoundException:
                        pass
            
            except Registry.RegistryKeyNotFoundException:
                pass
        
        return software
    
    def get_typed_urls(self):
        """Get Internet Explorer typed URLs"""
        
        urls = []
        
        try:
            key = self.reg.open("Software\\Microsoft\\Internet Explorer\\TypedURLs")
            
            for value in key.values():
                urls.append({
                    'position': value.name(),
                    'url': value.value(),
                    'timestamp': key.timestamp()
                })
        
        except Registry.RegistryKeyNotFoundException:
            pass
        
        return urls
    
    def get_shimcache(self):
        """
        Parse Application Compatibility Cache (ShimCache)
        Shows executed programs
        """
        
        shimcache_entries = []
        
        try:
            key = self.reg.open("System\\CurrentControlSet\\Control\\Session Manager\\AppCompatCache")
            
            # AppCompatCache value contains the data
            cache_data = key.value("AppCompatCache").value()
            
            # Parsing varies by Windows version
            # Windows 10 structure differs from Windows 7
            # This is simplified - use ShimCacheParser tool for production
            
        except Registry.RegistryKeyNotFoundException:
            pass
        
        return shimcache_entries
    
    def rot13_decode(self, text):
        """Decode ROT13 encoding used in UserAssist"""
        
        result = []
        for char in text:
            if 'a' <= char <= 'z':
                result.append(chr((ord(char) - ord('a') + 13) % 26 + ord('a')))
            elif 'A' <= char <= 'Z':
                result.append(chr((ord(char) - ord('A') + 13) % 26 + ord('A')))
            else:
                result.append(char)
        
        return ''.join(result)

# Key Registry Artifacts for Forensics
"""
SYSTEM Hive:
- CurrentControlSet\\Services: Installed services
- CurrentControlSet\\Control\\ComputerName: Computer name
- CurrentControlSet\\Control\\TimeZoneInformation: Time zone
- Select: Last known good configuration

SOFTWARE Hive:
- Microsoft\\Windows NT\\CurrentVersion: OS version
- Microsoft\\Windows\\CurrentVersion\\Run: Autostart programs
- Microsoft\\Windows\\CurrentVersion\\Uninstall: Installed programs
- Classes: File associations

NTUSER.DAT (per user):
- Software\\Microsoft\\Windows\\CurrentVersion\\Run: User autostart
- Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\RecentDocs
- Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\UserAssist
- Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\TypedPaths
- Software\\Microsoft\\Internet Explorer\\TypedURLs

SAM Hive:
- SAM\\Domains\\Account\\Users: User accounts and hashes

SECURITY Hive:
- Policy\\Secrets: LSA secrets (passwords, service account creds)
"""
```

## 3.4 Memory Forensics

Memory forensics analyzes volatile memory to find evidence that doesn't exist on disk.

### Volatility Framework Complete Guide

```bash
#!/bin/bash
# Comprehensive Volatility 3 Analysis Script

MEMORY_IMAGE="suspect_memory.raw"
PROFILE="Win10x64_19041"  # Windows 10 build 19041

echo "[*] Memory Forensics Analysis"
echo "[*] Image: $MEMORY_IMAGE"

# Identify the image
echo -e "\n[+] Identifying image profile..."
volatility3 -f "$MEMORY_IMAGE" windows.info

# Process listing
echo -e "\n[+] Listing running processes..."
volatility3 -f "$MEMORY_IMAGE" windows.pslist > pslist.txt
volatility3 -f "$MEMORY_IMAGE" windows.pstree > pstree.txt

# Scan for hidden processes
echo -e "\n[+] Scanning for hidden processes..."
volatility3 -f "$MEMORY_IMAGE" windows.psscan > psscan.txt

# Network connections
echo -e "\n[+] Analyzing network connections..."
volatility3 -f "$MEMORY_IMAGE" windows.netscan > netscan.txt

# DLL analysis
echo -e "\n[+] Analyzing loaded DLLs..."
volatility3 -f "$MEMORY_IMAGE" windows.dlllist > dlllist.txt

# Command line arguments
echo -e "\n[+] Extracting command lines..."
volatility3 -f "$MEMORY_IMAGE" windows.cmdline > cmdline.txt

# Registry hives
echo -e "\n[+] Listing registry hives..."
volatility3 -f "$MEMORY_IMAGE" windows.registry.hivelist > hivelist.txt

# Extract password hashes
echo -e "\n[+] Dumping password hashes..."
volatility3 -f "$MEMORY_IMAGE" windows.hashdump > hashdump.txt

# LSA secrets
volatility3 -f "$MEMORY_IMAGE" windows.lsadump > lsadump.txt

# Scan for injected code
echo -e "\n[+] Scanning for code injection..."
volatility3 -f "$MEMORY_IMAGE" windows.malfind > malfind.txt

# Dump suspicious processes
echo -e "\n[+] Dumping suspicious process memory..."
# Find suspicious PIDs from malfind output
SUSPICIOUS_PIDS=$(grep -oP 'PID: \K\d+' malfind.txt | head -5)

for pid in $SUSPICIOUS_PIDS; do
    echo "[*] Dumping PID $pid..."
    volatility3 -f "$MEMORY_IMAGE" -o dump_$pid windows.memmap --pid $pid --dump
done

# File scan
echo -e "\n[+] Scanning for files..."
volatility3 -f "$MEMORY_IMAGE" windows.filescan > filescan.txt

# Timeline analysis
echo -e "\n[+] Creating timeline..."
volatility3 -f "$MEMORY_IMAGE" windows.timeliner > timeline.txt

# Driver modules
echo -e "\n[+] Listing kernel modules..."
volatility3 -f "$MEMORY_IMAGE" windows.modules > modules.txt

# SSDT (System Service Descriptor Table) - rootkit detection
volatility3 -f "$MEMORY_IMAGE" windows.ssdt > ssdt.txt

# Callbacks - rootkit detection
volatility3 -f "$MEMORY_IMAGE" windows.callbacks > callbacks.txt

echo -e "\n[+] Analysis complete!"
echo "[*] Review the generated .txt files for artifacts"
```

**Advanced Memory Analysis:**

```python
#!/usr/bin/env python3
"""
Advanced Memory Forensics
Custom Volatility plugin and analysis
"""

class MemoryHunter:
    """Hunt for specific artifacts in memory"""
    
    def find_passwords_in_memory(self, memory_dump):
        """
        Search for password patterns in memory
        Looks for common password storage patterns
        """
        
        import re
        
        patterns = {
            'password=': re.compile(b'password=([^\\x00\\x0a\\x0d]+)'),
            'pwd=': re.compile(b'pwd=([^\\x00\\x0a\\x0d]+)'),
            'pass=': re.compile(b'pass=([^\\x00\\x0a\\x0d]+)'),
            'auth=': re.compile(b'auth=([^\\x00\\x0a\\x0d]+)'),
        }
        
        findings = []
        
        with open(memory_dump, 'rb') as f:
            chunk_size = 1024 * 1024  # 1MB chunks
            
            while True:
                chunk = f.read(chunk_size)
                if not chunk:
                    break
                
                for pattern_name, pattern in patterns.items():
                    matches = pattern.findall(chunk)
                    for match in matches:
                        findings.append({
                            'pattern': pattern_name,
                            'value': match.decode('utf-8', errors='ignore')
                        })
        
        return findings
    
    def find_private_keys(self, memory_dump):
        """Search for SSH/TLS private keys in memory"""
        
        import re
        
        # RSA private key pattern
        rsa_pattern = re.compile(
            b'-----BEGIN RSA PRIVATE KEY-----.*?-----END RSA PRIVATE KEY-----',
            re.DOTALL
        )
        
        keys_found = []
        
        with open(memory_dump, 'rb') as f:
            data = f.read()
            matches = rsa_pattern.findall(data)
            
            for match in matches:
                keys_found.append(match.decode('utf-8', errors='ignore'))
        
        return keys_found
    
    def extract_clipboard_data(self, memory_image):
        """
        Extract clipboard contents from memory
        Volatility plugin: windows.clipboard
        """
        
        import subprocess
        
        result = subprocess.run(
            ['volatility3', '-f', memory_image, 'windows.clipboard'],
            capture_output=True,
            text=True
        )
        
        return result.stdout
    
    def detect_process_injection(self, memory_image):
        """
        Detect process injection techniques
        
        Indicators:
        - Executable memory in unusual locations
        - Mismatched process path vs on-disk hash
        - Hollow process (replaced executable)
        - Reflective DLL injection
        """
        
        # Use Volatility malfind
        import subprocess
        
        result = subprocess.run(
            ['volatility3', '-f', memory_image, 'windows.malfind'],
            capture_output=True,
            text=True
        )
        
        # Parse output for injection indicators
        injected_processes = []
        
        for line in result.stdout.split('\n'):
            if 'PAGE_EXECUTE_READWRITE' in line:
                # Suspicious memory protection
                injected_processes.append(line)
        
        return injected_processes

# Malware Detection in Memory
"""
Common Malware Artifacts:

1. Process Anomalies:
   - svchost.exe not running from C:\\Windows\\System32
   - Multiple instances of single-instance processes
   - Unusual parent-child relationships
   - Processes with no on-disk executable

2. Network Anomalies:
   - Connections to known C2 servers
   - Unusual ports (not 80, 443, 53)
   - Many connections to single external IP
   - IRC or P2P protocols from system processes

3. Code Injection:
   - PAGE_EXECUTE_READWRITE memory regions
   - Orphaned threads
   - Hollow processes
   - Reflective DLL loading

4. Rootkit Indicators:
   - SSDT hooks
   - IRP hooks
   - Direct Kernel Object Manipulation
   - Hidden processes (in psscan but not pslist)
"""
```

[Continuing with network forensics, log analysis, and incident response sections...]
## 3.5 Network Forensics & Packet Analysis

Network forensics involves capturing and analyzing network traffic to investigate security incidents, identify attackers, and understand the scope of compromise.

### Wireshark Complete Guide

**Essential Display Filters:**

```bash
# Protocol Filters
http                    # HTTP traffic
dns                     # DNS queries
tcp                     # TCP traffic
udp                     # UDP traffic
ssl or tls              # Encrypted traffic

# IP Address Filters
ip.addr == 192.168.1.100        # Traffic to/from specific IP
ip.src == 10.0.0.5              # Source IP
ip.dst == 172.16.0.10           # Destination IP

# Port Filters
tcp.port == 80                  # HTTP
tcp.port == 443                 # HTTPS
tcp.port == 22                  # SSH
tcp.port == 3389                # RDP
udp.port == 53                  # DNS

# Combination Filters
http and ip.addr == 192.168.1.100
tcp.port == 80 and http.request.method == "POST"
dns and ip.dst == 8.8.8.8

# Suspicious Traffic Filters
http.request.method == "POST"             # POST requests (data exfiltration)
dns.qry.name contains "dnscat"            # DNS tunneling
tcp.flags.syn == 1 and tcp.flags.ack == 0  # SYN scans
tcp.analysis.retransmission               # Retransmissions (network issues)
tcp.analysis.zero_window                  # Zero windows (DoS indicator)

# Malware Communication
http.user_agent contains "python"         # Python scripts
http.user_agent contains "curl"           # Command-line tools
ssl.handshake.type == 1                   # SSL handshakes (C2)
http.request.uri contains ".exe"          # Executable downloads

# Data Exfiltration
tcp.len > 1000                            # Large packets
frame.len > 1500                          # Jumbo frames
```

**Wireshark Analysis Workflow:**

```python
#!/usr/bin/env python3
"""
Automated Network Forensics with Pyshark
"""

import pyshark
import json
from collections import defaultdict, Counter

class NetworkForensics:
    """Analyze PCAP files for forensic artifacts"""
    
    def __init__(self, pcap_file):
        self.pcap_file = pcap_file
        self.cap = pyshark.FileCapture(pcap_file)
    
    def analyze_http_traffic(self):
        """Extract HTTP requests and responses"""
        
        http_traffic = []
        
        for packet in self.cap:
            try:
                if 'HTTP' in packet:
                    http_info = {
                        'timestamp': packet.sniff_time.isoformat(),
                        'src_ip': packet.ip.src,
                        'dst_ip': packet.ip.dst,
                        'method': packet.http.request_method if hasattr(packet.http, 'request_method') else None,
                        'uri': packet.http.request_uri if hasattr(packet.http, 'request_uri') else None,
                        'host': packet.http.host if hasattr(packet.http, 'host') else None,
                        'user_agent': packet.http.user_agent if hasattr(packet.http, 'user_agent') else None,
                        'referer': packet.http.referer if hasattr(packet.http, 'referer') else None
                    }
                    http_traffic.append(http_info)
            except AttributeError:
                pass
        
        return http_traffic
    
    def detect_port_scan(self):
        """Detect port scanning activity"""
        
        syn_packets = defaultdict(set)
        
        for packet in self.cap:
            try:
                if 'TCP' in packet and packet.tcp.flags_syn == '1' and packet.tcp.flags_ack == '0':
                    src_ip = packet.ip.src
                    dst_port = packet.tcp.dstport
                    syn_packets[src_ip].add(dst_port)
            except AttributeError:
                pass
        
        # Identify scanners (IPs connecting to many ports)
        scanners = {}
        for ip, ports in syn_packets.items():
            if len(ports) > 20:  # More than 20 ports = likely scan
                scanners[ip] = {
                    'ports_scanned': len(ports),
                    'ports': sorted(list(ports))
                }
        
        return scanners
    
    def extract_dns_queries(self):
        """Extract DNS queries for analysis"""
        
        dns_queries = []
        
        for packet in self.cap:
            try:
                if 'DNS' in packet and hasattr(packet.dns, 'qry_name'):
                    dns_info = {
                        'timestamp': packet.sniff_time.isoformat(),
                        'src_ip': packet.ip.src,
                        'query': packet.dns.qry_name,
                        'query_type': packet.dns.qry_type if hasattr(packet.dns, 'qry_type') else None
                    }
                    dns_queries.append(dns_info)
            except AttributeError:
                pass
        
        return dns_queries
    
    def detect_dns_tunneling(self):
        """Detect DNS tunneling indicators"""
        
        domain_query_count = Counter()
        suspicious_domains = []
        
        for packet in self.cap:
            try:
                if 'DNS' in packet and hasattr(packet.dns, 'qry_name'):
                    query = packet.dns.qry_name
                    domain = '.'.join(query.split('.')[-2:])  # Get base domain
                    domain_query_count[domain] += 1
                    
                    # Indicators of DNS tunneling
                    if len(query) > 50:  # Unusually long subdomain
                        suspicious_domains.append({
                            'query': query,
                            'reason': 'Unusually long query',
                            'length': len(query)
                        })
                    
                    # Check for base64-like patterns
                    subdomain = query.split('.')[0]
                    if len(subdomain) > 20 and subdomain.isalnum():
                        suspicious_domains.append({
                            'query': query,
                            'reason': 'Possible base64 encoding',
                            'subdomain': subdomain
                        })
            
            except AttributeError:
                pass
        
        # Find domains with excessive queries
        for domain, count in domain_query_count.items():
            if count > 100:  # More than 100 queries
                suspicious_domains.append({
                    'domain': domain,
                    'reason': 'Excessive queries',
                    'query_count': count
                })
        
        return suspicious_domains
    
    def analyze_ssl_certificates(self):
        """Extract SSL/TLS certificate information"""
        
        certificates = []
        
        for packet in self.cap:
            try:
                if 'SSL' in packet:
                    if hasattr(packet.ssl, 'handshake_certificate'):
                        cert_info = {
                            'timestamp': packet.sniff_time.isoformat(),
                            'src_ip': packet.ip.src,
                            'dst_ip': packet.ip.dst,
                            'certificate': packet.ssl.handshake_certificate
                        }
                        certificates.append(cert_info)
            except AttributeError:
                pass
        
        return certificates
    
    def extract_files_from_http(self):
        """Extract files transferred over HTTP"""
        
        files = []
        
        for packet in self.cap:
            try:
                if 'HTTP' in packet:
                    # Look for file downloads
                    if hasattr(packet.http, 'file_data'):
                        file_info = {
                            'timestamp': packet.sniff_time.isoformat(),
                            'src_ip': packet.ip.src,
                            'uri': packet.http.request_uri if hasattr(packet.http, 'request_uri') else None,
                            'content_type': packet.http.content_type if hasattr(packet.http, 'content_type') else None,
                            'size': len(packet.http.file_data)
                        }
                        files.append(file_info)
            except AttributeError:
                pass
        
        return files
    
    def detect_beaconing(self):
        """
        Detect C2 beaconing behavior
        Regular intervals indicate automated callbacks
        """
        
        connection_times = defaultdict(list)
        
        for packet in self.cap:
            try:
                if 'TCP' in packet and packet.tcp.flags_syn == '1':
                    key = (packet.ip.src, packet.ip.dst, packet.tcp.dstport)
                    connection_times[key].append(packet.sniff_time.timestamp())
            except AttributeError:
                pass
        
        beacons = []
        
        for conn, timestamps in connection_times.items():
            if len(timestamps) > 5:
                # Calculate intervals between connections
                intervals = [timestamps[i+1] - timestamps[i] for i in range(len(timestamps)-1)]
                
                # Check for regular intervals (within 10% variance)
                avg_interval = sum(intervals) / len(intervals)
                variance = sum(abs(i - avg_interval) for i in intervals) / len(intervals)
                
                if variance / avg_interval < 0.1:  # Less than 10% variance
                    beacons.append({
                        'src_ip': conn[0],
                        'dst_ip': conn[1],
                        'dst_port': conn[2],
                        'interval': avg_interval,
                        'connections': len(timestamps)
                    })
        
        return beacons
    
    def generate_report(self, output_file):
        """Generate comprehensive network forensics report"""
        
        report = {
            'pcap_file': self.pcap_file,
            'http_traffic': self.analyze_http_traffic(),
            'port_scans': self.detect_port_scan(),
            'dns_queries': self.extract_dns_queries(),
            'dns_tunneling': self.detect_dns_tunneling(),
            'beaconing': self.detect_beaconing()
        }
        
        with open(output_file, 'w') as f:
            json.dump(report, f, indent=2)
        
        return report

# Usage
if __name__ == "__main__":
    nf = NetworkForensics('capture.pcap')
    report = nf.generate_report('network_forensics_report.json')
    
    print(f"Port scans detected: {len(report['port_scans'])}")
    print(f"Suspicious DNS: {len(report['dns_tunneling'])}")
    print(f"Beaconing detected: {len(report['beaconing'])}")
```

## 3.6 Incident Response Lifecycle

Incident response follows a structured methodology to effectively handle security incidents.

### NIST SP 800-61 Framework

```markdown
## NIST Incident Response Lifecycle

### Phase 1: Preparation
**Objective**: Establish capability to respond to incidents

**Activities**:
- Develop incident response plan
- Create and train incident response team
- Acquire tools and resources
- Implement preventive controls
- Establish communication procedures
- Set up logging and monitoring

**Deliverables**:
- IR Policy and Plan
- Playbooks for common scenarios
- Contact lists (internal and external)
- Tool inventory
- Jump bags/kits

### Phase 2: Detection & Analysis
**Objective**: Identify and validate incidents

**Activities**:
- Monitor security alerts
- Analyze anomalies
- Correlate events
- Determine scope and severity
- Document initial findings

**Indicators of Compromise (IOCs)**:
- Unusual outbound network traffic
- Anomalies in privileged user account activity
- Geographical irregularities
- Log-in red flags
- Increases in database read volume
- HTML response sizes
- Large numbers of requests for the same file
- Mismatched port-application traffic
- Suspicious registry or system file changes
- Unusual DNS requests
- Unexpected patching of systems

**Severity Classification**:
| Level | Criteria | Response Time |
|-------|----------|---------------|
| P1 - Critical | Ongoing attack, data breach, critical system down | Immediate |
| P2 - High | Successful compromise, sensitive data at risk | 1 hour |
| P3 - Medium | Attempted compromise, limited impact | 4 hours |
| P4 - Low | Policy violation, no immediate risk | 24 hours |
| P5 - Info | Anomaly requiring investigation | 48 hours |

### Phase 3: Containment, Eradication, Recovery
**Objective**: Stop the incident and restore normal operations

**Short-term Containment**:
- Isolate affected systems
- Change credentials
- Block malicious IPs/domains
- Apply temporary security controls

**Long-term Containment**:
- Build clean replacement systems
- Apply patches
- Implement additional controls
- Remove attacker persistence

**Eradication**:
- Remove malware
- Delete attacker accounts
- Close vulnerabilities
- Reset compromised credentials

**Recovery**:
- Restore from clean backups
- Rebuild compromised systems
- Verify security controls
- Monitor for re-infection
- Return to production

### Phase 4: Post-Incident Activity
**Objective**: Learn from the incident

**Activities**:
- Conduct lessons learned meeting
- Document timeline and actions
- Identify improvements
- Update playbooks
- Provide metrics and reporting
- Share threat intelligence

**Lessons Learned Questions**:
1. What happened and when?
2. How well did procedures work?
3. What information was needed sooner?
4. Were any steps or actions inhibited?
5. What would the team do differently?
6. How can we detect similar incidents faster?
7. What preventive measures should be implemented?
```

### Incident Response Playbooks

**Ransomware Incident Playbook:**

```markdown
# Ransomware Incident Response Playbook

## Initial Response (0-15 minutes)

### 1. Verify Incident
- [ ] Confirm ransomware indicators:
  - Ransom note present
  - Files encrypted (.encrypted, .locked, etc.)
  - Wallpaper changed
  - Unusual network activity

### 2. Activate IR Team
- [ ] Notify IR Team Lead
- [ ] Assemble core team members
- [ ] Establish communication channel (Slack, Teams)
- [ ] Notify management (CISO, CIO)

### 3. Initial Containment
- [ ] **DO NOT** turn off infected systems (preserve memory)
- [ ] Disconnect from network (unplug ethernet, disable WiFi)
- [ ] Identify patient zero (first infected system)
- [ ] Take photos of ransom notes and screens
- [ ] Document current time and all visible artifacts

## Investigation Phase (15 minutes - 4 hours)

### 4. Preserve Evidence
- [ ] Capture memory dump (if tools available)
- [ ] Take disk image of infected system
- [ ] Collect logs:
  - Windows Event Logs
  - Firewall logs
  - Proxy/web gateway logs
  - Email gateway logs
  - EDR/Antivirus logs
- [ ] Document network connections at time of discovery

### 5. Scope Assessment
- [ ] Identify all infected systems
- [ ] Map attacker lateral movement
- [ ] Determine initial infection vector:
  - Phishing email
  - RDP brute force
  - Exploited vulnerability
  - Compromised credentials
- [ ] Identify data accessed/exfiltrated
- [ ] Check for ransomware variant identification

### 6. Containment Strategy
- [ ] Isolate infected network segment
- [ ] Block C2 communication (domains/IPs)
- [ ] Disable compromised user accounts
- [ ] Change all privileged credentials
- [ ] Apply emergency firewall rules
- [ ] Shutdown non-critical systems

## Eradication (4-24 hours)

### 7. Remove Threat
- [ ] Identify and remove malware from all systems
- [ ] Delete attacker-created accounts
- [ ] Remove persistence mechanisms:
  - Scheduled tasks
  - Registry run keys
  - Services
  - WMI subscriptions
- [ ] Patch exploited vulnerabilities
- [ ] Apply security hardening

### 8. Determine Recovery Strategy
- [ ] **DO NOT PAY RANSOM** (unless approved by executive team)
- [ ] Identify available backups:
  - Check backup integrity
  - Verify backup dates
  - Test restore process
- [ ] Assess decryption options:
  - Check nomoreransom.org for decryptors
  - Contact law enforcement for assistance
- [ ] Calculate recovery time objectives

## Recovery (24 hours - 7 days)

### 9. System Recovery
- [ ] Rebuild critical systems from clean images
- [ ] Restore data from verified clean backups
- [ ] Apply all security patches
- [ ] Reconfigure security controls:
  - MFA on all accounts
  - Password complexity requirements
  - Network segmentation
  - Endpoint protection
- [ ] Verify system integrity before reconnecting to network

### 10. Validation
- [ ] Scan all systems for remnants
- [ ] Monitor for re-infection (24-48 hours)
- [ ] Test critical business functions
- [ ] Obtain business unit sign-off

## Post-Incident (7+ days)

### 11. Documentation
- [ ] Complete incident timeline
- [ ] Document all IOCs
- [ ] Calculate total impact:
  - Systems affected
  - Data lost
  - Downtime duration
  - Financial cost
- [ ] Prepare executive summary

### 12. Lessons Learned
- [ ] Hold team debrief meeting
- [ ] Identify detection gaps
- [ ] Update playbooks
- [ ] Implement preventive measures:
  - Email filtering improvements
  - Network segmentation
  - Backup frequency
  - EDR deployment
  - Security awareness training

### 13. Reporting
- [ ] File report with law enforcement (FBI IC3)
- [ ] Notify affected parties (if PII compromised)
- [ ] Report to regulatory bodies (if required)
- [ ] Update cyber insurance
- [ ] Share IOCs with industry (ISAC)
```

**Data Breach Response Playbook:**

```markdown
# Data Breach Response Playbook

## Immediate Actions (0-1 hour)

### 1. Confirm Breach
- [ ] Verify data exfiltration occurred
- [ ] Identify type of data compromised:
  - PII (Personally Identifiable Information)
  - PHI (Protected Health Information)
  - PCI (Payment Card Information)
  - Intellectual Property
  - Credentials/Secrets
- [ ] Estimate number of records affected
- [ ] Determine if exfiltration is ongoing

### 2. Activate Response Team
- [ ] Notify IR Team
- [ ] Engage Legal Counsel
- [ ] Contact Privacy Officer/DPO
- [ ] Brief Executive Leadership
- [ ] Engage PR/Communications team
- [ ] Contact cyber insurance carrier

### 3. Stop the Bleeding
- [ ] Block exfiltration channels
- [ ] Disable compromised accounts
- [ ] Apply emergency ACLs
- [ ] Isolate affected systems
- [ ] Preserve evidence

## Investigation (1-24 hours)

### 4. Determine Scope
- [ ] Identify all compromised data:
  - Database dumps
  - File shares
  - Email archives
  - Backups
- [ ] Map data flow:
  - Where stored
  - Who had access
  - When accessed
- [ ] Identify affected individuals/entities
- [ ] Document attacker actions:
  - Initial access method
  - Privilege escalation
  - Lateral movement
  - Data staging
  - Exfiltration method

### 5. Evidence Collection
- [ ] Preserve logs:
  - Authentication logs
  - Database access logs
  - Network flow data
  - Email logs
- [ ] Capture forensic images
- [ ] Interview personnel
- [ ] Review access controls

## Notification (24-72 hours)

### 6. Regulatory Notification
**GDPR (EU)**:
- [ ] Notify supervisory authority within 72 hours
- [ ] Include:
  - Nature of breach
  - Categories and approximate number of data subjects
  - Likely consequences
  - Measures taken/proposed

**HIPAA (US Healthcare)**:
- [ ] For >500 individuals: Notify HHS within 60 days
- [ ] For <500 individuals: Notify within 60 days of calendar year end

**State Breach Laws (US)**:
- [ ] Check requirements for each affected state
- [ ] Most require "without unreasonable delay"

### 7. Individual Notification
- [ ] Draft notification letter with Legal review
- [ ] Include:
  - What happened
  - What information was involved
  - What we're doing
  - What you can do
  - Contact information for questions
- [ ] Offer credit monitoring (if SSN/financial data)
- [ ] Send via postal mail (email as backup)

### 8. Public Disclosure
- [ ] Prepare press release (if required)
- [ ] Update website breach notification page
- [ ] Monitor social media
- [ ] Prepare FAQ for customer service

## Remediation (Ongoing)

### 9. Close Vulnerabilities
- [ ] Patch exploited systems
- [ ] Implement additional controls
- [ ] Enhance monitoring
- [ ] Review and update access controls

### 10. Support Affected Parties
- [ ] Set up dedicated call center
- [ ] Monitor for fraud
- [ ] Assist with identity theft issues
- [ ] Track and respond to inquiries

### 11. Post-Breach Review
- [ ] Conduct root cause analysis
- [ ] Assess response effectiveness
- [ ] Update policies and procedures
- [ ] Implement lessons learned
- [ ] Consider third-party security assessment
```

[Continuing with malware analysis, threat hunting, and final sections...]
## 3.7 Malware Analysis

Malware analysis determines malicious software capabilities, behavior, and indicators of compromise.

### Static Analysis

**Basic Static Analysis Workflow:**

```bash
#!/bin/bash
# Static Malware Analysis Script

MALWARE_FILE="$1"
OUTPUT_DIR="analysis_$(basename $MALWARE_FILE)_$(date +%Y%m%d_%H%M%S)"

mkdir -p "$OUTPUT_DIR"

echo "[*] Static Malware Analysis"
echo "[*] File: $MALWARE_FILE"
echo "[*] Output: $OUTPUT_DIR"

# File Hashes
echo "[+] Calculating file hashes..."
md5sum "$MALWARE_FILE" > "$OUTPUT_DIR/hash_md5.txt"
sha1sum "$MALWARE_FILE" > "$OUTPUT_DIR/hash_sha1.txt"
sha256sum "$MALWARE_FILE" > "$OUTPUT_DIR/hash_sha256.txt"

MD5=$(cat "$OUTPUT_DIR/hash_md5.txt" | awk '{print $1}')
echo "MD5: $MD5"

# VirusTotal Check
echo "[+] Checking VirusTotal..."
echo "https://www.virustotal.com/gui/file/$MD5" > "$OUTPUT_DIR/virustotal_link.txt"

# File Type
echo "[+] Identifying file type..."
file "$MALWARE_FILE" > "$OUTPUT_DIR/file_type.txt"
cat "$OUTPUT_DIR/file_type.txt"

# Strings Analysis
echo "[+] Extracting strings..."
strings -a "$MALWARE_FILE" > "$OUTPUT_DIR/strings_all.txt"
strings -a "$MALWARE_FILE" | grep -E "(http|https|ftp)://" > "$OUTPUT_DIR/strings_urls.txt"
strings -a "$MALWARE_FILE" | grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" > "$OUTPUT_DIR/strings_ips.txt"

# Suspicious strings
echo "[+] Finding suspicious strings..."
strings -a "$MALWARE_FILE" | grep -iE "(password|credential|token|api|key)" > "$OUTPUT_DIR/strings_sensitive.txt"
strings -a "$MALWARE_FILE" | grep -iE "(cmd\.exe|powershell|sh|bash)" > "$OUTPUT_DIR/strings_shells.txt"

# PE Header Analysis (if Windows executable)
if file "$MALWARE_FILE" | grep -q "PE32"; then
    echo "[+] Analyzing PE structure..."
    
    # Using pefile (Python)
    python3 << 'PYTHON_EOF' > "$OUTPUT_DIR/pe_analysis.txt"
import pefile
import sys

pe = pefile.PE(sys.argv[1])

print("PE Analysis")
print("="*50)
print(f"Machine Type: {hex(pe.FILE_HEADER.Machine)}")
print(f"Number of Sections: {pe.FILE_HEADER.NumberOfSections}")
print(f"TimeDateStamp: {pe.FILE_HEADER.TimeDateStamp}")
print(f"Entry Point: {hex(pe.OPTIONAL_HEADER.AddressOfEntryPoint)}")

print("\nSections:")
for section in pe.sections:
    print(f"  {section.Name.decode().strip()}: VirtualSize={section.Misc_VirtualSize}, RawSize={section.SizeOfRawData}")

print("\nImported DLLs:")
for entry in pe.DIRECTORY_ENTRY_IMPORT:
    print(f"  {entry.dll.decode()}")
    for imp in entry.imports[:5]:  # First 5 imports
        print(f"    - {imp.name.decode() if imp.name else 'N/A'}")

if hasattr(pe, 'DIRECTORY_ENTRY_EXPORT'):
    print("\nExported Functions:")
    for exp in pe.DIRECTORY_ENTRY_EXPORT.symbols[:10]:
        print(f"  {exp.name.decode() if exp.name else 'N/A'}")
PYTHON_EOF
    
    # Check for packing
    echo "[+] Checking for packing..."
    python3 << 'PYTHON_PACK' > "$OUTPUT_DIR/packing_check.txt"
import pefile
import sys

pe = pefile.PE(sys.argv[1])

# High entropy indicates packing/encryption
for section in pe.sections:
    entropy = section.get_entropy()
    print(f"{section.Name.decode().strip()}: Entropy={entropy:.2f}")
    if entropy > 7.0:
        print(f"  [!] Possibly packed/encrypted (high entropy)")
PYTHON_PACK
fi

# YARA Scan
echo "[+] Running YARA rules..."
if command -v yara &> /dev/null; then
    # Using common YARA rules
    yara -r /path/to/yara/rules "$MALWARE_FILE" > "$OUTPUT_DIR/yara_matches.txt"
else
    echo "YARA not installed"
fi

# Detect Cryptographic Constants
echo "[+] Searching for crypto constants..."
strings -a "$MALWARE_FILE" | grep -iE "(rijndael|aes|des|rsa|md5|sha)" > "$OUTPUT_DIR/crypto_indicators.txt"

echo "[✓] Static analysis complete!"
echo "[*] Results in: $OUTPUT_DIR"
```

### Dynamic Analysis

**Safe Malware Execution Environment:**

```python
#!/usr/bin/env python3
"""
Dynamic Malware Analysis Automation
Executes malware in controlled environment and monitors behavior
"""

import subprocess
import time
import json
from datetime import datetime

class DynamicMalwareAnalysis:
    """
    Automate dynamic malware analysis
    Requires: Windows sandbox, Process Monitor, Wireshark
    """
    
    def __init__(self, malware_path, output_dir):
        self.malware_path = malware_path
        self.output_dir = output_dir
        self.analysis_log = []
    
    def setup_monitoring(self):
        """Start monitoring tools before execution"""
        
        # Start Process Monitor
        subprocess.Popen([
            'Procmon.exe',
            '/BackingFile', f'{self.output_dir}\\procmon.pml',
            '/Quiet',
            '/Minimized'
        ])
        
        # Start Wireshark capture
        subprocess.Popen([
            'tshark.exe',
            '-i', '1',  # Interface 1
            '-w', f'{self.output_dir}\\network.pcap'
        ])
        
        # Start Regshot (before snapshot)
        subprocess.Popen([
            'Regshot.exe',
            '-1st'
        ])
        
        time.sleep(5)  # Wait for tools to initialize
        
        self.log_event("Monitoring tools started")
    
    def execute_malware(self):
        """Execute malware sample"""
        
        self.log_event(f"Executing malware: {self.malware_path}")
        
        # Execute in suspended state for analysis
        process = subprocess.Popen(
            [self.malware_path],
            creationflags=subprocess.CREATE_SUSPENDED
        )
        
        # Get initial process info
        pid = process.pid
        self.log_event(f"Process created with PID: {pid}")
        
        # Resume process
        time.sleep(2)
        process.resume()
        
        return process
    
    def monitor_behavior(self, process, duration=300):
        """Monitor malware behavior for specified duration"""
        
        start_time = time.time()
        
        while time.time() - start_time < duration:
            if process.poll() is not None:
                self.log_event("Process terminated")
                break
            
            # Check for suspicious activity
            self.check_network_connections(process.pid)
            self.check_file_operations(process.pid)
            self.check_registry_operations(process.pid)
            
            time.sleep(10)
    
    def check_network_connections(self, pid):
        """Monitor network connections from malware"""
        
        result = subprocess.run(
            ['netstat', '-ano'],
            capture_output=True,
            text=True
        )
        
        # Find connections for this PID
        connections = []
        for line in result.stdout.split('\n'):
            if str(pid) in line:
                connections.append(line.strip())
        
        if connections:
            self.log_event(f"Network connections: {connections}")
    
    def check_file_operations(self, pid):
        """Monitor file system operations"""
        
        # Parse Process Monitor log for file operations
        # This is simplified - actual implementation would parse PML file
        pass
    
    def check_registry_operations(self, pid):
        """Monitor registry modifications"""
        
        # Take after snapshot with Regshot
        subprocess.run(['Regshot.exe', '-2nd'])
        
        # Compare snapshots
        subprocess.run(['Regshot.exe', '-compare'])
    
    def stop_monitoring(self):
        """Stop all monitoring tools"""
        
        # Stop Process Monitor
        subprocess.run(['Procmon.exe', '/Terminate'])
        
        # Stop Wireshark
        subprocess.run(['taskkill', '/F', '/IM', 'tshark.exe'])
        
        self.log_event("Monitoring stopped")
    
    def generate_report(self):
        """Generate analysis report"""
        
        report = {
            'malware': self.malware_path,
            'analysis_time': datetime.now().isoformat(),
            'events': self.analysis_log,
            'artifacts': {
                'procmon_log': f'{self.output_dir}\\procmon.pml',
                'network_capture': f'{self.output_dir}\\network.pcap',
                'registry_changes': f'{self.output_dir}\\regshot_compare.txt'
            }
        }
        
        with open(f'{self.output_dir}\\report.json', 'w') as f:
            json.dump(report, f, indent=2)
        
        return report
    
    def log_event(self, message):
        """Log analysis event"""
        
        event = {
            'timestamp': datetime.now().isoformat(),
            'message': message
        }
        self.analysis_log.append(event)
        print(f"[{event['timestamp']}] {message}")
    
    def run_analysis(self):
        """Execute complete dynamic analysis workflow"""
        
        print("[*] Starting dynamic malware analysis")
        
        # Setup
        self.setup_monitoring()
        
        # Execute
        process = self.execute_malware()
        
        # Monitor
        self.monitor_behavior(process, duration=300)  # 5 minutes
        
        # Cleanup
        self.stop_monitoring()
        
        # Report
        report = self.generate_report()
        
        print("[✓] Analysis complete!")
        print(f"[*] Report: {self.output_dir}\\report.json")
        
        return report

# Cuckoo Sandbox Usage
"""
# Cuckoo Sandbox - Automated Malware Analysis

# Installation
pip install cuckoo

# Initialize
cuckoo init
cuckoo community

# Start Cuckoo
cuckoo -d

# Submit sample
cuckoo submit /path/to/malware.exe

# Web interface
cuckoo web runserver

# Check results
http://localhost:8000
"""
```

## 3.8 Threat Hunting

Threat hunting proactively searches for threats that evade existing security controls.

### Threat Hunting Methodology

```python
#!/usr/bin/env python3
"""
Threat Hunting Framework
Based on MITRE ATT&CK
"""

class ThreatHunter:
    """
    Proactive threat hunting using hypothesis-driven approach
    """
    
    def __init__(self, data_sources):
        """
        data_sources: dict of available data
        {
            'logs': log_path,
            'network': pcap_path,
            'memory': memory_dumps,
            'disk': disk_images
        }
        """
        self.data_sources = data_sources
        self.findings = []
    
    def hunt_powershell_abuse(self):
        """
        Hunt for malicious PowerShell usage
        
        TTPs:
        - T1059.001: PowerShell
        - T1086: PowerShell (deprecated, now T1059.001)
        """
        
        print("[*] Hunting for PowerShell abuse...")
        
        indicators = {
            'obfuscation': [
                '-enc',           # Encoded command
                '-e ',            # Short form
                'frombase64',     # Base64 decoding
                'invoke-expression', # IEX
                'iex',
                'downloadstring', # Download and execute
                'invoke-webrequest'
            ],
            'suspicious_cmdlets': [
                'invoke-mimikatz',
                'invoke-bloodhound',
                'invoke-kerberoast',
                'get-gpppassword',
                'invoke-shellcode'
            ],
            'remote_execution': [
                'invoke-command',
                'enter-pssession',
                'new-pssession'
            ]
        }
        
        # Search Windows Event Logs
        # Event ID 4104: Script Block Logging
        # Event ID 4103: Module Logging
        
        suspicious_commands = []
        
        # Parse PowerShell operational logs
        # This is simplified - real implementation would parse actual logs
        
        return suspicious_commands
    
    def hunt_persistence_mechanisms(self):
        """
        Hunt for persistence mechanisms
        
        TTPs:
        - T1547: Boot or Logon Autostart Execution
        - T1053: Scheduled Task/Job
        - T1136: Create Account
        """
        
        print("[*] Hunting for persistence mechanisms...")
        
        persistence_locations = {
            'registry_run_keys': [
                r'HKLM\Software\Microsoft\Windows\CurrentVersion\Run',
                r'HKCU\Software\Microsoft\Windows\CurrentVersion\Run',
                r'HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce',
            ],
            'startup_folder': [
                r'C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup',
                r'C:\Users\*\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup'
            ],
            'scheduled_tasks': r'C:\Windows\System32\Tasks',
            'services': 'HKLM\\System\\CurrentControlSet\\Services',
            'wmi_subscriptions': 'WMI Event Subscriptions'
        }
        
        findings = []
        
        # Check each location for suspicious entries
        # Compare against baseline
        # Flag anomalies
        
        return findings
    
    def hunt_lateral_movement(self):
        """
        Hunt for lateral movement
        
        TTPs:
        - T1021: Remote Services (RDP, SMB, WinRM)
        - T1077: Windows Admin Shares
        - T1003: Credential Dumping
        """
        
        print("[*] Hunting for lateral movement...")
        
        # Windows Event Log indicators
        lateral_movement_events = {
            '4624': 'Successful Logon (Type 3 = Network, Type 10 = RDP)',
            '4672': 'Special Privileges Assigned to New Logon',
            '4648': 'Logon Using Explicit Credentials',
            '4768': 'Kerberos TGT Requested',
            '4769': 'Kerberos Service Ticket Requested',
            '5140': 'Network Share Accessed',
            '5145': 'Network Share Object Checked for Desired Access'
        }
        
        # Look for:
        # 1. Admin logons from workstation to workstation
        # 2. Service ticket requests to unusual SPNs
        # 3. Access to admin shares (C$, ADMIN$)
        # 4. Same user logging in from multiple IPs simultaneously
        
        findings = []
        
        return findings
    
    def hunt_data_exfiltration(self):
        """
        Hunt for data exfiltration
        
        TTPs:
        - T1041: Exfiltration Over C2 Channel
        - T1048: Exfiltration Over Alternative Protocol
        - T1567: Exfiltration Over Web Service
        """
        
        print("[*] Hunting for data exfiltration...")
        
        indicators = []
        
        # Network-based indicators
        # 1. Large outbound transfers
        # 2. Unusual protocols (FTP, SFTP from servers)
        # 3. Connections to file sharing services
        # 4. DNS tunneling patterns
        # 5. HTTPS to unusual domains
        
        # File-based indicators
        # 1. Large archive files created
        # 2. Data staging directories
        # 3. File names suggesting staging (backup.zip, data.rar)
        
        return indicators
    
    def hunt_living_off_the_land(self):
        """
        Hunt for Living off the Land (LOLBins) abuse
        
        Legitimate tools used maliciously:
        - certutil.exe (download files)
        - bitsadmin.exe (download files)
        - mshta.exe (execute HTA)
        - regsvr32.exe (execute scripts)
        - rundll32.exe (execute DLLs)
        """
        
        print("[*] Hunting for LOLBin abuse...")
        
        suspicious_usage = {
            'certutil.exe': ['-urlcache', '-split', '-f'],
            'bitsadmin.exe': ['/transfer', '/download'],
            'mshta.exe': ['http://', 'https://'],
            'regsvr32.exe': ['/u', '/s', '/i'],
            'rundll32.exe': ['javascript:', 'http://'],
            'wmic.exe': ['process call create'],
            'msiexec.exe': ['/quiet', '/q', 'http://']
        }
        
        # Search command-line logs for suspicious patterns
        findings = []
        
        return findings
    
    def create_hunt_report(self):
        """Generate threat hunting report"""
        
        report = {
            'hunt_date': datetime.now().isoformat(),
            'data_sources': list(self.data_sources.keys()),
            'hunts': []
        }
        
        # Run all hunt functions
        hunts = [
            ('PowerShell Abuse', self.hunt_powershell_abuse),
            ('Persistence Mechanisms', self.hunt_persistence_mechanisms),
            ('Lateral Movement', self.hunt_lateral_movement),
            ('Data Exfiltration', self.hunt_data_exfiltration),
            ('LOLBin Abuse', self.hunt_living_off_the_land)
        ]
        
        for hunt_name, hunt_func in hunts:
            findings = hunt_func()
            report['hunts'].append({
                'name': hunt_name,
                'findings_count': len(findings),
                'findings': findings
            })
        
        return report
```

### MITRE ATT&CK Navigator

```python
#!/usr/bin/env python3
"""
MITRE ATT&CK Integration
Map observed TTPs to framework
"""

class AttackMapping:
    """Map incident TTPs to MITRE ATT&CK"""
    
    def __init__(self):
        self.observed_ttps = []
    
    def add_ttp(self, technique_id, technique_name, evidence):
        """Record observed TTP"""
        
        self.observed_ttps.append({
            'technique_id': technique_id,
            'technique_name': technique_name,
            'evidence': evidence,
            'timestamp': datetime.now().isoformat()
        })
    
    def generate_navigator_layer(self):
        """
        Generate ATT&CK Navigator layer JSON
        Visualize attack in Navigator
        """
        
        layer = {
            "name": "Incident TTPs",
            "versions": {
                "attack": "11",
                "navigator": "4.5",
                "layer": "4.3"
            },
            "domain": "enterprise-attack",
            "description": "Techniques observed during incident",
            "techniques": []
        }
        
        for ttp in self.observed_ttps:
            layer['techniques'].append({
                "techniqueID": ttp['technique_id'],
                "score": 1,  # Observed
                "comment": ttp['evidence']
            })
        
        return layer

# Example usage
mapper = AttackMapping()

# Map observed TTPs
mapper.add_ttp(
    'T1059.001',
    'PowerShell',
    'Encoded PowerShell command executed: powershell.exe -enc <base64>'
)

mapper.add_ttp(
    'T1003.001',
    'LSASS Memory Dumping',
    'Procdump.exe used to dump lsass.exe memory'
)

mapper.add_ttp(
    'T1021.001',
    'Remote Desktop Protocol',
    'RDP connection from 10.0.0.50 to 10.0.0.100'
)

# Generate Navigator layer
layer_json = mapper.generate_navigator_layer()
```

[Continuing with forensic reporting and final sections...]
## 3.9 Forensic Reporting & Expert Testimony

### Report Structure

```markdown
# Digital Forensic Examination Report

## Executive Summary (1 page)
- Case overview
- Key findings
- Business impact
- Recommendations

## Case Information
- Case Number
- Examiner Name and Credentials
- Date of Examination
- Evidence Examined
- Tools and Methodologies Used

## Evidence Description
For each piece of evidence:
- Evidence ID
- Description
- Make/Model/Serial Number
- Condition when received
- Hash values (MD5, SHA-256)
- Chain of custody summary

## Methodology
- Acquisition procedures
- Analysis techniques
- Tools used (with versions)
- Quality assurance measures

## Findings
### Finding #1: Malware Installation
**Evidence**: EVD-2024-001
**Location**: C:\Users\JDoe\AppData\Local\Temp\malware.exe
**Timestamp**: 2024-01-15 14:32:18 UTC
**Description**: 
Trojan backdoor providing remote access to attacker. Analysis shows:
- MD5: a1b2c3d4e5f6...
- SHA256: sha256hash...
- VirusTotal: 45/70 detections
- Network connections to 185.220.101.45:4444

**Supporting Evidence**:
- Screenshot 001: Malware binary in Temp folder
- Screenshot 002: Process Monitor showing execution
- Screenshot 003: Network connections to C2 server
- Artifact 001: Memory dump showing malicious process
- Artifact 002: PCAP showing C2 communications

### Finding #2: Data Exfiltration
[Similar detailed format]

## Timeline of Events
| Date/Time (UTC) | Event | Evidence |
|-----------------|-------|----------|
| 2024-01-15 14:15:00 | Phishing email received | Email logs |
| 2024-01-15 14:30:00 | Malicious attachment opened | File creation timestamp |
| 2024-01-15 14:32:18 | Malware executed | Prefetch file, Event logs |
| 2024-01-15 14:35:00 | C2 connection established | Network logs |
| 2024-01-15 15:00:00 | Data staging begun | File operations log |
| 2024-01-15 15:45:00 | Data exfiltrated (250MB) | Network flow data |

## Conclusions
Summary of what occurred, when, and how.

## Recommendations
- Short-term (immediate fixes)
- Medium-term (process improvements)
- Long-term (strategic changes)

## Appendices
A. Tool Documentation
B. Complete Evidence List
C. Glossary of Technical Terms
D. References
E. Examiner Curriculum Vitae

## Certification
I certify that the information in this report is accurate and complete to the best of my knowledge. All examinations were conducted in accordance with accepted forensic practices.

Signature: _______________________
Date: _______________
```

### Expert Witness Preparation

**Preparing Your CV for Court:**

```markdown
# Curriculum Vitae - Forensic Examiner

## Professional Certifications
- GIAC Certified Forensic Analyst (GCFA) - 2022
- GIAC Certified Forensic Examiner (GCFE) - 2021
- EnCase Certified Examiner (EnCE) - 2020
- Certified Information Systems Security Professional (CISSP) - 2019

## Education
- M.S. Digital Forensics, University of XYZ - 2018
- B.S. Computer Science, University of ABC - 2015

## Professional Experience
### Senior Forensic Examiner, ABC Forensics (2020-Present)
- Conducted 150+ digital forensic examinations
- Expert witness in 15+ court cases
- Specialized in malware analysis and incident response

### Forensic Analyst, DEF Security (2018-2020)
- Performed forensic analysis on 200+ cases
- Developed custom analysis tools
- Assisted law enforcement investigations

## Training and Continuing Education
- Advanced Malware Analysis - SANS FOR610 (2023)
- Advanced Network Forensics - SANS FOR572 (2022)
- Windows Forensics - SANS FOR500 (2021)

## Publications and Presentations
- "Advanced Memory Forensics Techniques" - DFIR Summit 2023
- "Cryptocurrency Forensics" - Journal of Digital Forensics (2022)

## Professional Memberships
- High Technology Crime Investigation Association (HTCIA)
- SANS Cyber Defense Community
- ISSA (Information Systems Security Association)

## Case Experience Summary
- Total examinations conducted: 350+
- Criminal cases: 120
- Civil cases: 150
- Internal corporate investigations: 80
- Expert testimony provided: 15 cases
```

**Direct Examination Preparation:**

```markdown
## Common Direct Examination Questions

Q: Please state your name and occupation.
A: My name is [Name]. I am a Senior Digital Forensic Examiner with [Company].

Q: What are your qualifications?
A: I hold a Master's degree in Digital Forensics and several industry certifications including GCFA, GCFE, and EnCE. I have conducted over 350 forensic examinations and provided expert testimony in 15 cases.

Q: What is digital forensics?
A: Digital forensics is the application of scientific methods to identify, preserve, collect, analyze, and present digital evidence in a manner that is legally admissible in court.

Q: What evidence did you examine in this case?
A: I examined [list evidence items with IDs], which included [describe: laptop, phone, etc.].

Q: How did you ensure the integrity of the evidence?
A: I used forensically sound methods including:
1. Write-blockers to prevent modification
2. Cryptographic hashing (MD5 and SHA-256) to verify integrity
3. Working on forensic copies, not originals
4. Maintaining detailed chain of custody documentation

Q: What tools did you use?
A: I used [list tools with versions], which are industry-standard forensic tools accepted in the forensic community.

Q: What did you find?
A: [Describe key findings clearly, avoiding jargon]

Q: Can you explain this to the jury?
A: [Use analogies and simple language]
Example: "Think of a hard drive like a filing cabinet. Even when you delete a file, it's like removing the label from a folder - the contents are still there until something new is filed in that spot."
```

**Cross-Examination Preparation:**

```markdown
## Handling Cross-Examination

### Common Attack Vectors

1. **Attacking Credentials**
Q: You're not a computer scientist, are you?
A: While I don't have a PhD in Computer Science, I hold a Master's degree in Digital Forensics and maintain current industry certifications. My expertise is specifically in digital forensic examination, which I've practiced professionally for 8 years.

2. **Attacking Methodology**
Q: Isn't it true that your methods could have altered the evidence?
A: No. I used write-blockers and worked only on forensic copies. The original evidence remains unchanged, which I can verify through hash values that match before and after acquisition.

3. **Attacking Tool Reliability**
Q: How do you know your tools are accurate?
A: The tools I used are peer-reviewed, scientifically validated, and widely accepted in the forensic community. They've been tested and validated through [mention studies, validation testing].

4. **Suggesting Alternative Explanations**
Q: Couldn't this have been an accident?
A: Based on the evidence, specifically [cite evidence], the actions were deliberate because [explain reasoning].

5. **Attacking Certainty**
Q: Are you 100% certain?
A: In science, we deal in probabilities and reasonable certainty. Based on the preponderance of evidence, I am reasonably certain to a high degree of scientific confidence that [conclusion].

### Best Practices
- Never guess - if you don't know, say "I don't know"
- Don't get defensive - remain professional
- Listen to the complete question before answering
- Answer only what was asked - don't volunteer information
- Speak to the jury, not the attorney
- Use simple language
- Admit limitations openly
- Stick to your expertise
```

## Frequently Asked Questions

### FAQ 1: How do I handle encrypted evidence?

**Answer:**

Encrypted evidence presents unique challenges. Here's a comprehensive approach:

**Scenario 1: System is Running (Live Acquisition)**
```bash
# If BitLocker/FileVault is unlocked, acquire while mounted
# Windows:
ftkimager.exe C: E:\Evidence\encrypted_system.dd --raw

# macOS with FileVault:
sudo dd if=/dev/disk0 of=/Volumes/Evidence/disk.dmg bs=4m
```

**Scenario 2: Encrypted Volume, Have Key/Password**
```bash
# BitLocker:
manage-bde -unlock C: -RecoveryPassword <48-digit-recovery-key>
# Then acquire

# VeraCrypt:
veracrypt --mount /dev/sdb1 /mnt/encrypted --password <password>
# Then acquire

# LUKS (Linux):
cryptsetup luksOpen /dev/sdb1 encrypted_volume
# Then acquire
```

**Scenario 3: No Key Available**
```markdown
Options (in order of likelihood):

1. **Search for Recovery Keys**
   - BitLocker: Check Active Directory, Azure AD, Microsoft Account
   - FileVault: Check iCloud, Recovery partition
   - Company key escrow systems

2. **Memory Analysis**
   - Capture memory if system is running
   - Extract encryption keys from RAM using:
     - Volatility: `volatility -f memory.raw windows.hashdump`
     - Rekall
     - Elcomsoft Forensic Disk Decryptor

3. **Hibernate File Analysis**
   - Windows hiberfil.sys may contain keys
   - Parse with volatility or specialized tools

4. **Known Plaintext Attack** (theoretical, rarely practical)

5. **Password Recovery**
   - Social engineering (ask user)
   - Shoulder surfing during investigation
   - Password reuse from other systems

6. **Legal Compulsion**
   - Court order compelling decryption
   - Warning: Fifth Amendment issues in US

If No Access Possible:
- Document encryption encountered
- Analyze unencrypted artifacts:
  - Network traffic
  - Logs from other systems
  - Cloud backups
  - Email on mail server
  - Browser artifacts (if separate profile)
```

### FAQ 2: What's the difference between computer forensics and incident response?

**Answer:**

While related, they have different objectives and approaches:

**Computer Forensics:**
```markdown
**Objective**: Legal admissibility and evidence collection

**Focus**:
- Preserve evidence integrity
- Maintain chain of custody
- Answer legal questions (who, what, when)
- Stand up in court

**Timeline**: Post-incident (after the fact)

**Methodology**:
- Strict adherence to protocols
- Extensive documentation
- Work on forensic copies only
- Minimally invasive

**Example Scenario**:
Employee suspected of theft of intellectual property.
- Create forensic image of laptop
- Analyze for IP theft evidence
- Document findings for legal action
- May take weeks to analyze thoroughly

**Tools**: FTK, EnCase, Autopsy (emphasis on legal soundness)
```

**Incident Response:**
```markdown
**Objective**: Stop ongoing attack and restore operations

**Focus**:
- Contain the threat
- Eradicate attacker presence
- Restore business operations
- Minimize damage

**Timeline**: During incident (real-time)

**Methodology**:
- Speed is critical
- Live forensics acceptable
- May modify systems to contain
- Balance evidence preservation with business needs

**Example Scenario**:
Ransomware attack in progress.
- Immediately isolate infected systems
- Capture memory and volatile data
- Identify scope and patient zero
- Begin containment and eradication
- May take hours to days

**Tools**: EDR platforms, SIEM, memory analysis (emphasis on speed)
```

**Overlap: Digital Forensics & Incident Response (DFIR)**
```markdown
Modern practice combines both:
- Use forensically sound methods during IR when possible
- Preserve evidence while responding
- Document actions taken during response
- May lead to legal action later

Best Practice:
1. Immediate Response: Focus on containment
2. Concurrent Forensics: Collect evidence as you go
3. Post-Incident: Thorough forensic analysis for legal action
```

### FAQ 3: How do I detect anti-forensics techniques?

**Answer:**

Anti-forensics attempts to hinder investigation. Detection strategies:

**1. Timestamp Manipulation (Timestomping)**

Detection:
```python
# Check for inconsistencies
# $STANDARD_INFORMATION vs $FILE_NAME timestamps

def detect_timestomp(mft_entry):
    """
    $STANDARD_INFORMATION can be modified easily
    $FILE_NAME timestamps are harder to change
    Mismatch indicates tampering
    """
    
    si_modified = mft_entry.std_info.modified_time
    fn_modified = mft_entry.file_name.modified_time
    
    if si_modified != fn_modified:
        return True, "Timestamp mismatch - possible timestomping"
    
    return False, "Timestamps match"

# Also check for:
# - Modified times before created times
# - Future timestamps
# - All files with same timestamp
```

**2. Data Wiping**

Detection:
```bash
# Look for wiping tools
strings disk.dd | grep -i "wipe\|shred\|eraser\|ccleaner"

# Check for patterns indicating wiping
# - All zeros (0x00)
# - All ones (0xFF)
# - DoD 5220.22-M patterns
# - Gutmann method patterns

# Analyze deleted file inodes
# If data section is unusually clean, may indicate wiping
```

**3. Encryption/Obfuscation**

Detection:
```python
# High entropy indicates encryption
import math
from collections import Counter

def calculate_entropy(data):
    """
    Entropy > 7.5 suggests encryption/compression
    Random data has entropy near 8.0
    """
    if not data:
        return 0
    
    entropy = 0
    counter = Counter(data)
    
    for count in counter.values():
        probability = count / len(data)
        if probability > 0:
            entropy -= probability * math.log2(probability)
    
    return entropy

# Check file entropy
with open('suspicious_file', 'rb') as f:
    data = f.read()
    entropy = calculate_entropy(data)
    
    if entropy > 7.5:
        print("High entropy - likely encrypted or compressed")
```

**4. Log Deletion**

Detection:
```markdown
Indicators:
- Event logs cleared (Event ID 1102 - Security log cleared)
- Gaps in log sequence numbers
- Sudden drops in log volume
- Log files smaller than expected
- $UsnJrnl showing log file deletions

Check:
- Event ID 1102: "The audit log was cleared"
- Event ID 104: System log cleared
- Missing expected entries (logon/logoff patterns disrupted)
- File system journals for evidence of deletion
```

**5. Program Execution Hiding**

Detection:
```bash
# Fileless malware leaves traces in:
# 1. Memory
volatility -f memory.raw windows.malfind

# 2. Registry (MUICache)
reg query "HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\MuiCache"

# 3. Prefetch (even if executed from memory)
# 4. AmCache (Application Compatibility Cache)
# 5. PowerShell logs (Event ID 4104)
# 6. WMI Event logs
```

**6. Steganography**

Detection:
```python
# Statistical analysis for hidden data
def detect_steganography(image_path):
    """
    LSB steganography changes pixel values slightly
    Statistical analysis can detect anomalies
    """
    
    # Check for:
    # - Unusual file size for image quality
    # - LSB (Least Significant Bit) patterns
    # - Chi-square analysis
    # - Visual/statistical steganalysis tools
    
    # Use specialized tools:
    # - StegDetect
    # - StegExpose
    # - Steganalysis with deep learning
```

### FAQ 4: How do I investigate cloud-based incidents?

**Answer:**

Cloud forensics requires different approaches due to multi-tenancy, distributed data, and limited access.

**AWS Forensics Workflow:**

```bash
#!/bin/bash
# AWS Incident Response and Forensics

INSTANCE_ID="i-1234567890abcdef0"
REGION="us-east-1"

# 1. Isolate compromised instance
echo "[*] Isolating instance..."
aws ec2 create-security-group --group-name forensic-isolation \
    --description "Forensic isolation group" --region $REGION

# Remove existing security groups and apply isolation group
SG_ID=$(aws ec2 describe-security-groups --filters "Name=group-name,Values=forensic-isolation" \
    --query 'SecurityGroups[0].GroupId' --output text --region $REGION)

aws ec2 modify-instance-attribute --instance-id $INSTANCE_ID \
    --groups $SG_ID --region $REGION

# 2. Tag for investigation
aws ec2 create-tags --resources $INSTANCE_ID \
    --tags Key=Status,Value=UnderInvestigation --region $REGION

# 3. Create snapshot of EBS volume
echo "[*] Creating EBS snapshot..."
VOLUME_ID=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID \
    --query 'Reservations[0].Instances[0].BlockDeviceMappings[0].Ebs.VolumeId' \
    --output text --region $REGION)

SNAPSHOT_ID=$(aws ec2 create-snapshot --volume-id $VOLUME_ID \
    --description "Forensic snapshot $(date)" \
    --query 'SnapshotId' --output text --region $REGION)

# Wait for snapshot to complete
aws ec2 wait snapshot-completed --snapshot-ids $SNAPSHOT_ID --region $REGION

# 4. Create forensic workstation
echo "[*] Launching forensic workstation..."
FORENSIC_AMI="ami-0abcdef1234567890"  # SIFT workstation AMI

FORENSIC_INSTANCE=$(aws ec2 run-instances \
    --image-id $FORENSIC_AMI \
    --instance-type m5.2xlarge \
    --subnet-id <forensic-subnet-id> \
    --security-group-ids <forensic-sg-id> \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Purpose,Value=Forensics}]' \
    --query 'Instances[0].InstanceId' \
    --output text --region $REGION)

# 5. Attach snapshot as volume to forensic workstation
FORENSIC_VOLUME=$(aws ec2 create-volume \
    --snapshot-id $SNAPSHOT_ID \
    --availability-zone ${REGION}a \
    --query 'VolumeId' --output text --region $REGION)

aws ec2 attach-volume --volume-id $FORENSIC_VOLUME \
    --instance-id $FORENSIC_INSTANCE --device /dev/sdf --region $REGION

# 6. Collect CloudTrail logs
echo "[*] Downloading CloudTrail logs..."
aws s3 cp s3://my-cloudtrail-bucket/AWSLogs/ ./cloudtrail-logs/ \
    --recursive --region $REGION

# 7. Collect VPC Flow Logs
aws ec2 describe-flow-logs --region $REGION > vpc-flow-logs.json

# 8. Collect memory (if enabled)
# AWS Systems Manager can capture memory on running instances
aws ssm send-command --instance-ids $INSTANCE_ID \
    --document-name "AWS-RunShellScript" \
    --parameters 'commands=["sudo avml /tmp/memory.lime"]' \
    --region $REGION

# 9. Analyze in forensic workstation
# SSH to forensic instance and mount volume read-only
# mount -o ro /dev/sdf1 /mnt/evidence
```

**Azure Forensics:**

```powershell
# Azure Incident Response

$vmName = "compromised-vm"
$resourceGroup = "production-rg"

# 1. Isolate VM
Write-Host "[*] Isolating VM..."
$nsg = New-AzNetworkSecurityGroup -Name "forensic-isolation-nsg" `
    -ResourceGroupName $resourceGroup -Location "EastUS"

# Remove all rules (deny all traffic)
$vm = Get-AzVM -Name $vmName -ResourceGroupName $resourceGroup
$nic = Get-AzNetworkInterface -ResourceId $vm.NetworkProfile.NetworkInterfaces[0].Id
$nic.NetworkSecurityGroup = $nsg
Set-AzNetworkInterface -NetworkInterface $nic

# 2. Create snapshot
Write-Host "[*] Creating disk snapshot..."
$vm = Get-AzVM -Name $vmName -ResourceGroupName $resourceGroup
$disk = Get-AzDisk -DiskName $vm.StorageProfile.OsDisk.Name `
    -ResourceGroupName $resourceGroup

$snapshotConfig = New-AzSnapshotConfig -SourceUri $disk.Id `
    -CreateOption Copy -Location $disk.Location

$snapshot = New-AzSnapshot -Snapshot $snapshotConfig `
    -SnapshotName "forensic-snapshot-$(Get-Date -Format 'yyyyMMdd')" `
    -ResourceGroupName $resourceGroup

# 3. Collect Azure Activity Logs
Get-AzActivityLog -StartTime (Get-Date).AddDays(-7) | Export-Csv azure-activity.csv

# 4. Collect NSG Flow Logs
# Configure and download from storage account

# 5. Access diagnostic logs
Get-AzDiagnosticSetting -ResourceId $vm.Id
```

**Key Cloud Forensics Challenges:**

```markdown
1. **Volatility**: Instances can be terminated, losing evidence
   Solution: Automate snapshot creation, enable logging

2. **Shared Responsibility**: Provider controls infrastructure
   Solution: Understand shared responsibility model, work with provider

3. **Multi-tenancy**: Your data mixed with others
   Solution: Strong isolation, encryption, access controls

4. **Distributed Data**: Logs, backups, data spread across regions
   Solution: Centralized logging (CloudTrail, Azure Monitor, GCP Logging)

5. **Legal Jurisdiction**: Data in multiple countries
   Solution: Legal team involvement, understand data residency

6. **Limited Physical Access**: Can't seize hardware
   Solution: Leverage cloud provider APIs, use snapshots

7. **Ephemeral Resources**: Auto-scaling creates/destroys instances
   Solution: Centralized logging, immutable infrastructure
```

### FAQ 5: What tools should every forensic examiner have?

**Answer:**

Essential tools organized by category:

**Acquisition Tools:**
```markdown
- FTK Imager (Free) - Disk imaging, memory capture
- dd/dcfldd/dc3dd (Free) - Linux disk imaging
- Guymager (Free) - Linux GUI disk imaging
- X-Ways Imager (Commercial) - Advanced disk imaging
- Magnet ACQUIRE (Commercial) - iOS acquisition
- Cellebrite UFED (Commercial) - Mobile forensics
```

**Analysis Platforms:**
```markdown
Free:
- Autopsy/Sleuth Kit - Complete forensic suite
- SANS SIFT Workstation - Pre-configured Linux forensics
- REMnux - Malware analysis

Commercial:
- EnCase Forensic - Industry standard
- FTK (Forensic Toolkit) - Comprehensive analysis
- X-Ways Forensics - Fast, powerful analysis
- Magnet AXIOM - Modern, user-friendly
```

**Memory Forensics:**
```markdown
- Volatility Framework (Free) - Memory analysis
- Rekall (Free) - Advanced memory analysis
- Magnet RAM Capture (Free) - Memory acquisition
- WinPmem (Free) - Windows memory acquisition
- LiME (Free) - Linux memory acquisition
```

**Network Forensics:**
```markdown
- Wireshark (Free) - Packet analysis
- NetworkMiner (Free) - PCAP analysis, file extraction
- Zeek/Bro (Free) - Network security monitoring
- Moloch (Free) - Full packet capture indexing
```

**Malware Analysis:**
```markdown
- IDA Pro (Commercial) - Disassembler
- Ghidra (Free) - NSA disassembler
- x64dbg (Free) - Debugger
- Cuckoo Sandbox (Free) - Automated dynamic analysis
- PE Studio (Free) - PE file analysis
- YARA (Free) - Malware identification
```

**Registry Analysis:**
```markdown
- RegRipper (Free) - Registry parsing
- Registry Explorer (Free) - Registry viewer
- Registry Decoder (Free) - Live registry analysis
```

**Timeline Analysis:**
```markdown
- log2timeline/plaso (Free) - Super timeline creation
- Timesketch (Free) - Timeline visualization
```

**Mobile Forensics:**
```markdown
- Andriller (Free) - Android forensics
- libimobiledevice (Free) - iOS forensics
- Cellebrite UFED (Commercial) - Complete mobile forensics
- Magnet AXIOM (Commercial) - iOS/Android
```

**Minimum Starter Kit (Free):**
```bash
# Linux (Ubuntu/Debian)
sudo apt install -y \
    autopsy \
    sleuthkit \
    volatility \
    wireshark \
    binwalk \
    foremost \
    exiftool \
    yara \
    clamav

# SIFT Workstation (all-in-one)
wget https://github.com/teamdfir/sift-cli/releases/download/v1.14.0/sift-cli-linux
chmod +x sift-cli-linux
sudo ./sift-cli-linux install --mode=packages-only
```

## Hands-On Lab Exercises

### Lab 1: Complete Disk Forensics Investigation

**Objective**: Investigate a compromised Windows system

**Scenario**: Employee terminated for policy violations. Examine their workstation for evidence of data theft.

**Provided Materials**:
- Disk image: employee_laptop.E01 (5GB)
- Hash values: MD5 and SHA256
- Background: Employee had access to customer database

**Tasks**:

1. **Verify Image Integrity**
```bash
# Calculate hash of E01 image
md5sum employee_laptop.E01
sha256sum employee_laptop.E01

# Compare with provided values
# Document results in case notes
```

2. **Load Image into Autopsy**
```bash
# Launch Autopsy
autopsy

# Create new case
# Add data source (E01 file)
# Run ingest modules:
#   - Recent Activity
#   - Hash Lookup
#   - File Type Identification
#   - Keyword Search
```

3. **Timeline Analysis**
```bash
# Generate timeline
# Filter for:
#   - USB device connections
#   - File copying to removable media
#   - Database queries executed
#   - Unusual file access patterns
```

4. **Find Evidence of Data Theft**
```markdown
Look for:
- [ ] USB device usage (USBSTOR registry key)
- [ ] Large file copies to external drives
- [ ] Zip/archive files created
- [ ] Database export files
- [ ] Cloud storage uploads (browser history)
- [ ] Email attachments (Outlook PST)
```

5. **Recover Deleted Files**
```bash
# In Autopsy:
# - Navigate to $Recycle.Bin
# - Check unallocated space for deleted files
# - Use file carving if necessary

# Document what was deleted and when
```

6. **Write Report**
```markdown
Create professional report including:
- Executive summary
- Evidence description and chain of custody
- Timeline of employee actions
- Evidence of data theft (with screenshots)
- List of stolen files
- Recommendations
```

**Expected Findings**:
- USB drive connected day before termination
- 2GB customer database exported to CSV
- Database copied to USB drive
- Deletion of browser history (anti-forensics)
- Personal email used to send files to home account

### Lab 2: Memory Forensics - Malware Hunting

**Objective**: Analyze memory dump to identify malware

**Scenario**: System exhibiting suspicious behavior. Memory dump captured.

**Provided Materials**:
- Memory dump: infected_system.raw (4GB)
- Profile: Win10x64_19041

**Tasks**:

1. **Identify Profile**
```bash
volatility3 -f infected_system.raw windows.info
```

2. **Process Analysis**
```bash
# List all processes
volatility3 -f infected_system.raw windows.pslist > pslist.txt

# Look for suspicious processes:
#   - Unusual names (svchost.exe from wrong location)
#   - Misspellings (svhost.exe, csrss.exe from user directory)
#   - Unknown processes
#   - Parent-child relationship anomalies

# Process tree
volatility3 -f infected_system.raw windows.pstree > pstree.txt
```

3. **Network Connections**
```bash
volatility3 -f infected_system.raw windows.netscan > netscan.txt

# Identify suspicious connections:
#   - Foreign IPs
#   - Unusual ports
#   - Multiple connections to single IP
```

4. **Code Injection Detection**
```bash
volatility3 -f infected_system.raw windows.malfind > malfind.txt

# Review for:
#   - PAGE_EXECUTE_READWRITE memory
#   - Injected code in processes
#   - Decode suspicious memory regions
```

5. **Extract Malware**
```bash
# Find suspicious process PID
# Extract executable
volatility3 -f infected_system.raw -o extracted/ windows.pslist --pid 1234 --dump

# Analyze extracted binary with VirusTotal
# Run static analysis (strings, PE structure)
```

6. **Registry Analysis**
```bash
# Find persistence mechanisms
volatility3 -f infected_system.raw windows.registry.printkey \
    --key "Software\Microsoft\Windows\CurrentVersion\Run"
```

**Expected Findings**:
- Malicious process: svchosts.exe (note misspelling)
- Connection to C2 server: 185.220.101.45:4444
- Code injection into explorer.exe
- Persistence via Run key
- Malware identified as Cobalt Strike beacon

### Lab 3: Incident Response Simulation

**Objective**: Complete incident response from detection through recovery

**Scenario**: Ransomware attack detected at 2:00 PM. Multiple systems encrypted.

**Available Resources**:
- Network diagram
- System inventory
- Backup information
- Incident response plan

**Exercise Timeline**:

**2:00 PM - Detection**
```markdown
Tasks:
[ ] Alert received from EDR: Ransomware detected
[ ] Verify incident (check multiple systems)
[ ] Activate IR team
[ ] Notify management
[ ] Begin documentation
```

**2:15 PM - Initial Containment**
```markdown
Tasks:
[ ] Identify patient zero
[ ] Disconnect affected systems from network
[ ] Change all privileged account passwords
[ ] Block C2 communication (domains/IPs)
[ ] Preserve memory and disk images of patient zero
```

**2:30 PM - Scope Assessment**
```markdown
Tasks:
[ ] Identify all infected systems (15 workstations, 2 servers)
[ ] Determine ransomware variant (REvil)
[ ] Identify initial infection vector (phishing email)
[ ] Check for data exfiltration
[ ] Assess backup availability and integrity
```

**3:00 PM - Eradication**
```markdown
Tasks:
[ ] Remove malware from all systems
[ ] Delete attacker-created accounts
[ ] Remove persistence mechanisms
[ ] Patch exploited vulnerabilities
[ ] Apply security hardening
```

**4:00 PM - Recovery**
```markdown
Tasks:
[ ] Verify backup integrity (last good backup 24 hours ago)
[ ] Test restore process
[ ] Rebuild critical systems
[ ] Restore from backups
[ ] Verify system integrity
[ ] Return to production incrementally
```

**Day 2 - Post-Incident**
```markdown
Tasks:
[ ] Lessons learned meeting
[ ] Complete incident timeline
[ ] Calculate total impact
[ ] Update playbooks
[ ] Implement preventive measures
[ ] Report to authorities (FBI IC3)
```

**Deliverables**:
1. Complete incident timeline
2. Root cause analysis
3. Recovery plan
4. Lessons learned document
5. Executive summary report

## Interview Questions & Answers

### Question 1: Walk me through how you would investigate a ransomware incident from initial detection to final report.

**Model Answer:**

"I would follow a structured incident response methodology:

**Phase 1: Immediate Response (0-30 minutes)**

First, I would verify the incident by checking multiple systems for ransomware indicators - ransom notes, encrypted files, and unusual network activity. I'd immediately activate our incident response team and notify management.

For containment, I would:
- Disconnect affected systems from the network without powering them off (to preserve memory)
- Identify patient zero - the first infected system
- Take photographs of ransom notes and any visible artifacts
- Document the current time and system state

**Phase 2: Evidence Preservation (30 minutes - 2 hours)**

I would capture volatile data before it's lost:
- Memory dump using FTK Imager or DumpIt
- Network connections and running processes
- Logged-in users and open files
- Screenshot everything

Then create forensic disk images of critical systems, maintaining strict chain of custody documentation.

**Phase 3: Investigation (2-24 hours)**

I would analyze the evidence to determine:
- Initial infection vector (phishing email, RDP brute force, etc.)
- Ransomware variant (for potential decryption tools)
- Scope of infection (all affected systems)
- Lateral movement path
- Whether data was exfiltrated (double extortion)

Key artifacts to examine:
- Windows Event Logs (4624, 4625, 4688 for process creation)
- Prefetch files showing program execution
- USN Journal for file system changes
- Browser history if phishing vector
- Email logs if phishing vector
- Network logs for C2 communication

**Phase 4: Timeline Reconstruction (ongoing)**

I would build a comprehensive timeline using log2timeline/plaso, correlating:
- File system timestamps
- Event logs
- Network flows
- Email timestamps
- User actions

This shows exactly what happened, when, and in what sequence.

**Phase 5: Recovery Support (24+ hours)**

While not my primary role as a forensic examiner, I would support recovery by:
- Identifying all compromised systems
- Confirming malware eradication
- Checking for persistence mechanisms
- Validating backup integrity before restoration

**Phase 6: Final Reporting (3-7 days)**

I would produce a comprehensive report including:
- Executive summary (non-technical, business impact)
- Technical findings with evidence
- Complete timeline of events
- IOCs (hashes, IPs, domains) for threat intelligence
- Recommendations for prevention

The report would include screenshots, evidence references, and be written to withstand legal scrutiny if needed for law enforcement or insurance claims.

**Key Considerations Throughout:**
- Maintain evidence integrity (hashing, write blockers)
- Document every action
- Preserve chain of custody
- Work on forensic copies, never originals
- Coordinate with legal, PR, and executive team
- Share IOCs with industry groups and law enforcement"

**Follow-up Questions Handled:**

Q: What if you can't determine initial infection vector?
A: "I would document what I can determine and note the limitation. I'd examine all common vectors - phishing, RDP, vulnerabilities - and provide the most likely scenario based on available evidence, while being honest about uncertainty."

Q: How do you handle executive pressure to restore systems quickly vs. preserve evidence?
A: "This is a balancing act. I'd explain that we can do both - take snapshots for forensics while building clean systems for recovery. The snapshots preserve evidence for later detailed analysis while not delaying recovery. I'd emphasize that thorough investigation prevents recurrence and may be needed for legal action or insurance claims."

[Continuing with more interview questions...]

## 3.10 Advanced Forensic Techniques

### Cloud Forensics Deep Dive

**AWS Forensics Complete Workflow:**

```python
#!/usr/bin/env python3
"""
AWS Forensics Automation
Comprehensive evidence collection from AWS environment
"""

import boto3
import json
from datetime import datetime, timedelta

class AWSForensics:
    """
    Automate AWS forensic evidence collection
    """
    
    def __init__(self, region='us-east-1'):
        self.region = region
        self.ec2 = boto3.client('ec2', region_name=region)
        self.cloudtrail = boto3.client('cloudtrail', region_name=region)
        self.s3 = boto3.client('s3')
        self.guardduty = boto3.client('guardduty', region_name=region)
        self.iam = boto3.client('iam')
        
        self.evidence = {
            'collection_time': datetime.now().isoformat(),
            'region': region,
            'findings': []
        }
    
    def isolate_instance(self, instance_id):
        """
        Isolate compromised EC2 instance
        Preserves instance but blocks all network access
        """
        
        print(f"[*] Isolating instance: {instance_id}")
        
        # Create forensic isolation security group
        try:
            sg_response = self.ec2.create_security_group(
                GroupName=f'forensic-isolation-{instance_id}',
                Description='Forensic isolation - denies all traffic',
                VpcId=self._get_instance_vpc(instance_id)
            )
            
            isolation_sg_id = sg_response['GroupId']
            
            # No inbound or outbound rules = complete isolation
            # Apply to instance
            self.ec2.modify_instance_attribute(
                InstanceId=instance_id,
                Groups=[isolation_sg_id]
            )
            
            # Tag for tracking
            self.ec2.create_tags(
                Resources=[instance_id],
                Tags=[
                    {'Key': 'ForensicStatus', 'Value': 'Isolated'},
                    {'Key': 'IsolationTime', 'Value': datetime.now().isoformat()}
                ]
            )
            
            self.evidence['findings'].append({
                'action': 'instance_isolation',
                'instance_id': instance_id,
                'security_group': isolation_sg_id,
                'timestamp': datetime.now().isoformat()
            })
            
            print(f"[+] Instance isolated with SG: {isolation_sg_id}")
            
        except Exception as e:
            print(f"[!] Error isolating instance: {e}")
    
    def snapshot_volumes(self, instance_id):
        """
        Create snapshots of all EBS volumes attached to instance
        Preserves evidence for analysis
        """
        
        print(f"[*] Creating snapshots for instance: {instance_id}")
        
        # Get all volumes attached to instance
        response = self.ec2.describe_instances(InstanceIds=[instance_id])
        instance = response['Reservations'][0]['Instances'][0]
        
        snapshots = []
        
        for device in instance.get('BlockDeviceMappings', []):
            volume_id = device['Ebs']['VolumeId']
            
            snapshot = self.ec2.create_snapshot(
                VolumeId=volume_id,
                Description=f"Forensic snapshot - {instance_id} - {datetime.now()}",
                TagSpecifications=[{
                    'ResourceType': 'snapshot',
                    'Tags': [
                        {'Key': 'ForensicEvidence', 'Value': 'True'},
                        {'Key': 'SourceInstance', 'Value': instance_id},
                        {'Key': 'CaseNumber', 'Value': 'CASE-2024-001'}
                    ]
                }]
            )
            
            snapshot_id = snapshot['SnapshotId']
            snapshots.append({
                'volume_id': volume_id,
                'snapshot_id': snapshot_id,
                'device': device['DeviceName']
            })
            
            print(f"[+] Created snapshot {snapshot_id} for volume {volume_id}")
        
        self.evidence['findings'].append({
            'action': 'volume_snapshots',
            'instance_id': instance_id,
            'snapshots': snapshots,
            'timestamp': datetime.now().isoformat()
        })
        
        return snapshots
    
    def collect_cloudtrail_logs(self, start_time=None, end_time=None):
        """
        Collect CloudTrail logs for time period
        Shows all API calls and who made them
        """
        
        if not start_time:
            start_time = datetime.now() - timedelta(days=7)
        if not end_time:
            end_time = datetime.now()
        
        print(f"[*] Collecting CloudTrail events from {start_time} to {end_time}")
        
        events = []
        paginator = self.cloudtrail.get_paginator('lookup_events')
        
        for page in paginator.paginate(
            StartTime=start_time,
            EndTime=end_time
        ):
            events.extend(page['Events'])
        
        print(f"[+] Collected {len(events)} CloudTrail events")
        
        # Analyze for suspicious activity
        suspicious_events = []
        
        for event in events:
            event_name = event['EventName']
            
            # Flag high-risk events
            if event_name in [
                'DeleteTrail',  # Covering tracks
                'StopLogging',  # Disabling logging
                'DeleteBucket',  # Data destruction
                'PutBucketPolicy',  # Permission changes
                'CreateAccessKey',  # New credentials
                'CreateUser',  # New accounts
                'AttachUserPolicy'  # Privilege escalation
            ]:
                suspicious_events.append(event)
        
        self.evidence['findings'].append({
            'action': 'cloudtrail_collection',
            'total_events': len(events),
            'suspicious_events': len(suspicious_events),
            'suspicious_details': suspicious_events[:10],  # First 10
            'timestamp': datetime.now().isoformat()
        })
        
        return events, suspicious_events
    
    def analyze_vpc_flow_logs(self, vpc_id, start_time=None):
        """
        Analyze VPC Flow Logs for network activity
        Identify C2 communication, data exfiltration
        """
        
        print(f"[*] Analyzing VPC Flow Logs for: {vpc_id}")
        
        # VPC Flow Logs are typically sent to CloudWatch Logs or S3
        # This is a simplified example
        
        # Look for suspicious patterns:
        suspicious_patterns = {
            'large_outbound_transfers': [],  # >100MB
            'unusual_ports': [],  # Not 80, 443, 22
            'many_connections_single_ip': [],  # >100 connections to one IP
            'rejected_connections': []  # Scanning attempts
        }
        
        # Parse flow logs (simplified)
        # Real implementation would parse from CloudWatch or S3
        
        self.evidence['findings'].append({
            'action': 'vpc_flow_analysis',
            'vpc_id': vpc_id,
            'suspicious_patterns': suspicious_patterns,
            'timestamp': datetime.now().isoformat()
        })
        
        return suspicious_patterns
    
    def collect_guardduty_findings(self):
        """
        Collect GuardDuty security findings
        AWS's threat detection service
        """
        
        print("[*] Collecting GuardDuty findings...")
        
        # Get detector ID
        detectors = self.guardduty.list_detectors()
        
        if not detectors['DetectorIds']:
            print("[-] No GuardDuty detectors found")
            return []
        
        detector_id = detectors['DetectorIds'][0]
        
        # Get findings
        findings_ids = self.guardduty.list_findings(
            DetectorId=detector_id,
            FindingCriteria={
                'Criterion': {
                    'updatedAt': {
                        'Gte': int((datetime.now() - timedelta(days=7)).timestamp() * 1000)
                    }
                }
            }
        )
        
        if not findings_ids['FindingIds']:
            print("[+] No GuardDuty findings in last 7 days")
            return []
        
        # Get finding details
        findings = self.guardduty.get_findings(
            DetectorId=detector_id,
            FindingIds=findings_ids['FindingIds']
        )
        
        print(f"[+] Found {len(findings['Findings'])} GuardDuty findings")
        
        # Categorize by severity
        critical = []
        high = []
        medium = []
        low = []
        
        for finding in findings['Findings']:
            severity = finding['Severity']
            
            if severity >= 7.0:
                critical.append(finding)
            elif severity >= 4.0:
                high.append(finding)
            elif severity >= 2.0:
                medium.append(finding)
            else:
                low.append(finding)
        
        self.evidence['findings'].append({
            'action': 'guardduty_findings',
            'critical': len(critical),
            'high': len(high),
            'medium': len(medium),
            'low': len(low),
            'critical_details': critical,
            'timestamp': datetime.now().isoformat()
        })
        
        return findings['Findings']
    
    def analyze_iam_activity(self):
        """
        Analyze IAM for suspicious activity
        Privilege escalation, new users, policy changes
        """
        
        print("[*] Analyzing IAM activity...")
        
        # Get all users
        users = self.iam.list_users()['Users']
        
        # Check for recently created users
        recent_users = []
        for user in users:
            created = user['CreateDate']
            if (datetime.now(created.tzinfo) - created).days < 7:
                recent_users.append(user)
        
        # Get all access keys
        access_keys = []
        for user in users:
            keys = self.iam.list_access_keys(UserName=user['UserName'])
            
            for key in keys['AccessKeyMetadata']:
                if (datetime.now(key['CreateDate'].tzinfo) - key['CreateDate']).days < 7:
                    access_keys.append({
                        'user': user['UserName'],
                        'access_key_id': key['AccessKeyId'],
                        'created': key['CreateDate'].isoformat()
                    })
        
        # Get policy changes (from CloudTrail)
        # Look for AttachUserPolicy, PutUserPolicy events
        
        self.evidence['findings'].append({
            'action': 'iam_analysis',
            'recent_users': len(recent_users),
            'recent_access_keys': len(access_keys),
            'users_details': recent_users,
            'keys_details': access_keys,
            'timestamp': datetime.now().isoformat()
        })
        
        return {
            'recent_users': recent_users,
            'recent_keys': access_keys
        }
    
    def launch_forensic_workstation(self, snapshot_id):
        """
        Launch forensic analysis workstation with evidence mounted
        """
        
        print("[*] Launching forensic workstation...")
        
        # Use SIFT workstation AMI or custom forensics AMI
        forensic_ami = 'ami-0123456789abcdef0'  # Replace with actual SIFT AMI
        
        # Create volume from snapshot
        volume = self.ec2.create_volume(
            SnapshotId=snapshot_id,
            AvailabilityZone=f'{self.region}a',
            VolumeType='gp3',
            TagSpecifications=[{
                'ResourceType': 'volume',
                'Tags': [
                    {'Key': 'Purpose', 'Value': 'Forensic Analysis'},
                    {'Key': 'SourceSnapshot', 'Value': snapshot_id}
                ]
            }]
        )
        
        volume_id = volume['VolumeId']
        
        # Wait for volume to be available
        waiter = self.ec2.get_waiter('volume_available')
        waiter.wait(VolumeIds=[volume_id])
        
        # Launch forensic instance
        instance = self.ec2.run_instances(
            ImageId=forensic_ami,
            InstanceType='m5.2xlarge',  # 8 vCPU, 32 GB RAM
            MinCount=1,
            MaxCount=1,
            TagSpecifications=[{
                'ResourceType': 'instance',
                'Tags': [
                    {'Key': 'Purpose', 'Value': 'Forensic Workstation'},
                    {'Key': 'AutoShutdown', 'Value': 'False'}
                ]
            }]
        )
        
        instance_id = instance['Instances'][0]['InstanceId']
        
        # Wait for instance to be running
        waiter = self.ec2.get_waiter('instance_running')
        waiter.wait(InstanceIds=[instance_id])
        
        # Attach evidence volume
        self.ec2.attach_volume(
            VolumeId=volume_id,
            InstanceId=instance_id,
            Device='/dev/sdf'
        )
        
        print(f"[+] Forensic workstation launched: {instance_id}")
        print(f"[+] Evidence volume attached: {volume_id}")
        print(f"[*] SSH to instance and mount volume read-only:")
        print(f"    sudo mount -o ro /dev/sdf1 /mnt/evidence")
        
        return instance_id, volume_id
    
    def generate_forensic_report(self, output_file='aws_forensic_report.json'):
        """Generate comprehensive forensic report"""
        
        report = {
            'case_information': {
                'report_generated': datetime.now().isoformat(),
                'region': self.region,
                'examiner': 'AWS Forensics Automation'
            },
            'evidence_collected': self.evidence
        }
        
        with open(output_file, 'w') as f:
            json.dump(report, f, indent=2, default=str)
        
        print(f"[+] Forensic report saved to: {output_file}")
        
        return report
    
    def _get_instance_vpc(self, instance_id):
        """Helper: Get VPC ID for instance"""
        
        response = self.ec2.describe_instances(InstanceIds=[instance_id])
        return response['Reservations'][0]['Instances'][0]['VpcId']

# Complete AWS Forensic Investigation
def investigate_aws_incident(instance_id, region='us-east-1'):
    """
    Complete AWS forensic investigation workflow
    """
    
    forensics = AWSForensics(region=region)
    
    print("="*60)
    print("AWS Forensic Investigation")
    print("="*60)
    
    # Step 1: Isolate
    forensics.isolate_instance(instance_id)
    
    # Step 2: Snapshot
    snapshots = forensics.snapshot_volumes(instance_id)
    
    # Step 3: Collect CloudTrail
    events, suspicious = forensics.collect_cloudtrail_logs()
    
    # Step 4: GuardDuty findings
    findings = forensics.collect_guardduty_findings()
    
    # Step 5: IAM analysis
    iam_findings = forensics.analyze_iam_activity()
    
    # Step 6: Launch forensic workstation
    if snapshots:
        forensics.launch_forensic_workstation(snapshots[0]['snapshot_id'])
    
    # Step 7: Generate report
    report = forensics.generate_forensic_report()
    
    print("="*60)
    print("Investigation Complete")
    print(f"Total findings: {len(forensics.evidence['findings'])}")
    print("="*60)
    
    return report

# Usage
if __name__ == "__main__":
    # Investigate compromised instance
    report = investigate_aws_incident('i-1234567890abcdef0', region='us-east-1')
```

### Mobile Forensics Advanced

**Complete Android Forensics:**

```python
#!/usr/bin/env python3
"""
Advanced Android Forensics
Complete device analysis and data extraction
"""

import subprocess
import sqlite3
import xml.etree.ElementTree as ET
import json
from datetime import datetime

class AndroidForensics:
    """
    Comprehensive Android forensics toolkit
    """
    
    def __init__(self, device_id=None):
        self.device_id = device_id
        self.evidence = {
            'device_info': {},
            'apps': [],
            'contacts': [],
            'messages': [],
            'call_logs': [],
            'locations': [],
            'files': [],
            'network': []
        }
    
    def get_device_info(self):
        """
        Collect device information
        Model, OS version, IMEI, phone number, etc.
        """
        
        print("[*] Collecting device information...")
        
        # Device properties
        props = subprocess.run(
            ['adb', '-s', self.device_id, 'shell', 'getprop'],
            capture_output=True,
            text=True
        ).stdout
        
        info = {}
        for line in props.split('\n'):
            if ':' in line:
                key, value = line.split(':', 1)
                key = key.strip('[] ')
                value = value.strip('[] ')
                info[key] = value
        
        self.evidence['device_info'] = {
            'manufacturer': info.get('ro.product.manufacturer'),
            'model': info.get('ro.product.model'),
            'android_version': info.get('ro.build.version.release'),
            'build_id': info.get('ro.build.id'),
            'serial': info.get('ro.serialno'),
            'imei': self._get_imei()
        }
        
        return self.evidence['device_info']
    
    def extract_installed_apps(self):
        """
        List all installed applications
        Including system and user apps
        """
        
        print("[*] Extracting installed applications...")
        
        result = subprocess.run(
            ['adb', '-s', self.device_id, 'shell', 'pm', 'list', 'packages', '-f'],
            capture_output=True,
            text=True
        )
        
        apps = []
        for line in result.stdout.split('\n'):
            if line.startswith('package:'):
                # Format: package:/data/app/package.name/base.apk=package.name
                parts = line.replace('package:', '').split('=')
                if len(parts) == 2:
                    apps.append({
                        'path': parts[0],
                        'package': parts[1]
                    })
        
        self.evidence['apps'] = apps
        print(f"[+] Found {len(apps)} installed applications")
        
        return apps
    
    def extract_contacts(self):
        """
        Extract contacts database
        """
        
        print("[*] Extracting contacts...")
        
        # Pull contacts database
        subprocess.run([
            'adb', '-s', self.device_id, 'pull',
            '/data/data/com.android.providers.contacts/databases/contacts2.db',
            './contacts2.db'
        ])
        
        # Parse database
        conn = sqlite3.connect('./contacts2.db')
        cursor = conn.cursor()
        
        # Query contacts
        cursor.execute('''
        SELECT display_name, data1, data2
        FROM raw_contacts
        JOIN data ON raw_contacts._id = data.raw_contact_id
        WHERE mimetype='vnd.android.cursor.item/phone_v2'
        ''')
        
        contacts = []
        for row in cursor.fetchall():
            contacts.append({
                'name': row[0],
                'number': row[1],
                'type': row[2]
            })
        
        conn.close()
        
        self.evidence['contacts'] = contacts
        print(f"[+] Extracted {len(contacts)} contacts")
        
        return contacts
    
    def extract_sms_messages(self):
        """
        Extract SMS/MMS messages
        """
        
        print("[*] Extracting SMS messages...")
        
        # Pull SMS database
        subprocess.run([
            'adb', '-s', self.device_id, 'pull',
            '/data/data/com.android.providers.telephony/databases/mmssms.db',
            './mmssms.db'
        ])
        
        # Parse database
        conn = sqlite3.connect('./mmssms.db')
        cursor = conn.cursor()
        
        cursor.execute('''
        SELECT address, date, body, type
        FROM sms
        ORDER BY date DESC
        ''')
        
        messages = []
        for row in cursor.fetchall():
            messages.append({
                'number': row[0],
                'timestamp': datetime.fromtimestamp(row[1]/1000).isoformat(),
                'body': row[2],
                'type': 'received' if row[3] == 1 else 'sent'
            })
        
        conn.close()
        
        self.evidence['messages'] = messages
        print(f"[+] Extracted {len(messages)} SMS messages")
        
        return messages
    
    def extract_call_logs(self):
        """
        Extract call history
        """
        
        print("[*] Extracting call logs...")
        
        # Pull call log database
        subprocess.run([
            'adb', '-s', self.device_id, 'pull',
            '/data/data/com.android.providers.contacts/databases/calllog.db',
            './calllog.db'
        ])
        
        conn = sqlite3.connect('./calllog.db')
        cursor = conn.cursor()
        
        cursor.execute('''
        SELECT number, date, duration, type
        FROM calls
        ORDER BY date DESC
        ''')
        
        calls = []
        for row in cursor.fetchall():
            call_type = {1: 'incoming', 2: 'outgoing', 3: 'missed'}.get(row[3], 'unknown')
            
            calls.append({
                'number': row[0],
                'timestamp': datetime.fromtimestamp(row[1]/1000).isoformat(),
                'duration': row[2],
                'type': call_type
            })
        
        conn.close()
        
        self.evidence['call_logs'] = calls
        print(f"[+] Extracted {len(calls)} call log entries")
        
        return calls
    
    def extract_location_data(self):
        """
        Extract GPS/location history
        """
        
        print("[*] Extracting location data...")
        
        locations = []
        
        # Check Google Maps location history
        # /data/data/com.google.android.gms/databases/
        
        # Check cached locations
        result = subprocess.run([
            'adb', '-s', self.device_id, 'shell',
            'content', 'query',
            '--uri', 'content://com.google.android.gms.location/locations'
        ], capture_output=True, text=True)
        
        # Parse location data (format varies by Android version)
        
        self.evidence['locations'] = locations
        
        return locations
    
    def extract_browser_history(self):
        """
        Extract web browsing history
        Chrome, Firefox, default browser
        """
        
        print("[*] Extracting browser history...")
        
        browsers = {
            'Chrome': '/data/data/com.android.chrome/app_chrome/Default/History',
            'Firefox': '/data/data/org.mozilla.firefox/databases/browser.db'
        }
        
        all_history = []
        
        for browser, db_path in browsers.items():
            try:
                subprocess.run([
                    'adb', '-s', self.device_id, 'pull',
                    db_path, f'./{browser}_history.db'
                ])
                
                conn = sqlite3.connect(f'./{browser}_history.db')
                cursor = conn.cursor()
                
                if browser == 'Chrome':
                    cursor.execute('SELECT url, title, visit_count, last_visit_time FROM urls')
                else:  # Firefox
                    cursor.execute('SELECT url, title, visits FROM history')
                
                for row in cursor.fetchall():
                    all_history.append({
                        'browser': browser,
                        'url': row[0],
                        'title': row[1],
                        'visits': row[2]
                    })
                
                conn.close()
            
            except Exception as e:
                print(f"[-] Could not extract {browser} history: {e}")
        
        print(f"[+] Extracted {len(all_history)} browser history entries")
        
        return all_history
    
    def extract_whatsapp_messages(self):
        """
        Extract WhatsApp messages
        Requires root or backup
        """
        
        print("[*] Extracting WhatsApp messages...")
        
        try:
            # Pull WhatsApp database
            subprocess.run([
                'adb', '-s', self.device_id, 'pull',
                '/data/data/com.whatsapp/databases/msgstore.db',
                './whatsapp_msgstore.db'
            ])
            
            conn = sqlite3.connect('./whatsapp_msgstore.db')
            cursor = conn.cursor()
            
            cursor.execute('''
            SELECT key_remote_jid, data, timestamp, key_from_me
            FROM messages
            ORDER BY timestamp DESC
            ''')
            
            messages = []
            for row in cursor.fetchall():
                messages.append({
                    'contact': row[0],
                    'message': row[1],
                    'timestamp': datetime.fromtimestamp(row[2]/1000).isoformat(),
                    'outgoing': bool(row[3])
                })
            
            conn.close()
            
            print(f"[+] Extracted {len(messages)} WhatsApp messages")
            
            return messages
        
        except Exception as e:
            print(f"[-] Could not extract WhatsApp data: {e}")
            print("    (Requires root or backup extraction)")
            return []
    
    def extract_app_data(self, package_name):
        """
        Extract application data for specific app
        """
        
        print(f"[*] Extracting data for: {package_name}")
        
        # Create backup
        subprocess.run([
            'adb', '-s', self.device_id, 'backup',
            '-f', f'./{package_name}.ab',
            package_name
        ])
        
        # Convert backup to tar
        # (requires Android backup extractor)
        
        print(f"[+] Backup created: {package_name}.ab")
    
    def take_screenshots(self):
        """
        Capture screenshots of current state
        """
        
        print("[*] Taking screenshot...")
        
        subprocess.run([
            'adb', '-s', self.device_id, 'shell', 'screencap',
            '/sdcard/forensic_screenshot.png'
        ])
        
        subprocess.run([
            'adb', '-s', self.device_id, 'pull',
            '/sdcard/forensic_screenshot.png',
            './screenshot.png'
        ])
        
        print("[+] Screenshot saved")
    
    def logical_acquisition(self, output_dir='./android_acquisition'):
        """
        Perform logical acquisition
        Extract all accessible data
        """
        
        import os
        os.makedirs(output_dir, exist_ok=True)
        
        print("="*60)
        print("Android Logical Acquisition")
        print("="*60)
        
        # Device info
        self.get_device_info()
        
        # Applications
        self.extract_installed_apps()
        
        # Communications
        self.extract_contacts()
        self.extract_sms_messages()
        self.extract_call_logs()
        
        # Location
        self.extract_location_data()
        
        # Browser
        self.extract_browser_history()
        
        # Messaging apps
        self.extract_whatsapp_messages()
        
        # Screenshots
        self.take_screenshots()
        
        # Generate report
        report_path = os.path.join(output_dir, 'forensic_report.json')
        with open(report_path, 'w') as f:
            json.dump(self.evidence, f, indent=2)
        
        print("="*60)
        print(f"Acquisition complete: {output_dir}")
        print(f"Report: {report_path}")
        print("="*60)
        
        return self.evidence
    
    def _get_imei(self):
        """Helper: Get device IMEI"""
        
        result = subprocess.run([
            'adb', '-s', self.device_id, 'shell',
            'service', 'call', 'iphonesubinfo', '1'
        ], capture_output=True, text=True)
        
        # Parse IMEI from output
        # Format varies by device
        
        return "IMEI_EXTRACTION_PENDING"

# Usage
if __name__ == "__main__":
    # List connected devices
    devices = subprocess.run(['adb', 'devices'], capture_output=True, text=True)
    print(devices.stdout)
    
    # Perform acquisition on first device
    forensics = AndroidForensics()
    evidence = forensics.logical_acquisition()
    
    print(f"Extracted {len(evidence['contacts'])} contacts")
    print(f"Extracted {len(evidence['messages'])} SMS messages")
    print(f"Extracted {len(evidence['call_logs'])} call logs")
```

[Continuing with cryptocurrency forensics, IoT forensics, and more advanced topics...]

## 3.11 Cryptocurrency Forensics

### Blockchain Analysis

**Bitcoin Transaction Tracing:**

```python
#!/usr/bin/env python3
"""
Cryptocurrency Forensics - Bitcoin Analysis
Trace transactions and identify wallet owners
"""

import requests
import json
from collections import defaultdict

class BitcoinForensics:
    """
    Bitcoin blockchain analysis for forensic investigations
    """
    
    def __init__(self):
        self.blockchain_api = "https://blockchain.info"
        self.transactions = []
        self.addresses = {}
    
    def analyze_address(self, address):
        """
        Analyze Bitcoin address for transaction history
        """
        
        print(f"[*] Analyzing Bitcoin address: {address}")
        
        # Get address information
        url = f"{self.blockchain_api}/rawaddr/{address}"
        response = requests.get(url)
        
        if response.status_code != 200:
            print(f"[!] Error fetching address data: {response.status_code}")
            return None
        
        data = response.json()
        
        analysis = {
            'address': address,
            'total_received': data['total_received'] / 100000000,  # Convert satoshis to BTC
            'total_sent': data['total_sent'] / 100000000,
            'final_balance': data['final_balance'] / 100000000,
            'n_tx': data['n_tx'],
            'transactions': []
        }
        
        # Analyze each transaction
        for tx in data.get('txs', []):
            tx_analysis = {
                'hash': tx['hash'],
                'time': tx['time'],
                'result': tx.get('result', 0) / 100000000,
                'inputs': [],
                'outputs': []
            }
            
            # Analyze inputs
            for inp in tx.get('inputs', []):
                if 'prev_out' in inp:
                    tx_analysis['inputs'].append({
                        'address': inp['prev_out'].get('addr'),
                        'value': inp['prev_out'].get('value', 0) / 100000000
                    })
            
            # Analyze outputs
            for out in tx.get('out', []):
                tx_analysis['outputs'].append({
                    'address': out.get('addr'),
                    'value': out.get('value', 0) / 100000000
                })
            
            analysis['transactions'].append(tx_analysis)
        
        self.addresses[address] = analysis
        
        print(f"[+] Address analyzed: {analysis['n_tx']} transactions")
        print(f"    Total received: {analysis['total_received']} BTC")
        print(f"    Final balance: {analysis['final_balance']} BTC")
        
        return analysis
    
    def trace_transaction_flow(self, tx_hash, depth=3):
        """
        Trace the flow of funds from a transaction
        Follow inputs and outputs to specified depth
        """
        
        print(f"[*] Tracing transaction: {tx_hash}")
        
        flow = {
            'transaction': tx_hash,
            'depth': depth,
            'flow_map': {}
        }
        
        # Get transaction details
        url = f"{self.blockchain_api}/rawtx/{tx_hash}"
        response = requests.get(url)
        
        if response.status_code != 200:
            return None
        
        tx_data = response.json()
        
        # Build flow map
        self._build_flow_map(tx_data, flow['flow_map'], depth, visited=set())
        
        return flow
    
    def _build_flow_map(self, tx_data, flow_map, remaining_depth, visited):
        """Recursively build transaction flow map"""
        
        if remaining_depth == 0:
            return
        
        tx_hash = tx_data['hash']
        
        if tx_hash in visited:
            return
        
        visited.add(tx_hash)
        
        flow_map[tx_hash] = {
            'time': tx_data.get('time'),
            'inputs': [],
            'outputs': []
        }
        
        # Record inputs
        for inp in tx_data.get('inputs', []):
            if 'prev_out' in inp:
                flow_map[tx_hash]['inputs'].append({
                    'address': inp['prev_out'].get('addr'),
                    'value': inp['prev_out'].get('value', 0) / 100000000
                })
        
        # Record outputs and follow them
        for out in tx_data.get('out', []):
            output_info = {
                'address': out.get('addr'),
                'value': out.get('value', 0) / 100000000
            }
            flow_map[tx_hash]['outputs'].append(output_info)
            
            # If output was spent, follow it
            if out.get('spent'):
                # Get spending transaction
                # (This requires additional API calls)
                pass
    
    def identify_mixing_services(self, address):
        """
        Identify if address is associated with mixing services
        Mixing services obscure transaction trails
        """
        
        # Known mixing service patterns
        mixing_indicators = {
            'multiple_small_transactions': False,
            'rapid_turnover': False,
            'many_outputs': False,
            'suspicious_patterns': []
        }
        
        analysis = self.addresses.get(address)
        if not analysis:
            analysis = self.analyze_address(address)
        
        # Check for mixing patterns
        if analysis['n_tx'] > 100:
            mixing_indicators['multiple_small_transactions'] = True
            mixing_indicators['suspicious_patterns'].append(
                "High transaction volume"
            )
        
        # Check for rapid turnover (funds in and out quickly)
        for tx in analysis['transactions']:
            if abs(tx['result']) < 0.1:  # Small amounts
                mixing_indicators['suspicious_patterns'].append(
                    "Small amount transactions"
                )
        
        return mixing_indicators
    
    def cluster_analysis(self, seed_address):
        """
        Cluster analysis - identify related addresses
        Common ownership heuristics
        """
        
        print(f"[*] Performing cluster analysis from: {seed_address}")
        
        cluster = {
            'seed': seed_address,
            'related_addresses': set(),
            'confidence': {}
        }
        
        analysis = self.addresses.get(seed_address)
        if not analysis:
            analysis = self.analyze_address(seed_address)
        
        # Multi-input heuristic
        # If multiple inputs are used in a transaction,
        # they likely belong to the same entity
        for tx in analysis['transactions']:
            if len(tx['inputs']) > 1:
                for inp in tx['inputs']:
                    if inp['address'] and inp['address'] != seed_address:
                        cluster['related_addresses'].add(inp['address'])
                        cluster['confidence'][inp['address']] = 'high'
        
        # Change address heuristic
        # Identify likely change addresses
        for tx in analysis['transactions']:
            if len(tx['outputs']) == 2:
                # One output to recipient, one change back to sender
                for out in tx['outputs']:
                    if out['address'] not in cluster['related_addresses']:
                        cluster['related_addresses'].add(out['address'])
                        cluster['confidence'][out['address']] = 'medium'
        
        print(f"[+] Identified {len(cluster['related_addresses'])} related addresses")
        
        return cluster
    
    def identify_exchange_deposits(self, address):
        """
        Identify deposits to cryptocurrency exchanges
        Exchanges are common cash-out points
        """
        
        # Known exchange addresses (partial list)
        known_exchanges = {
            'Coinbase': ['1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa'],  # Example
            'Binance': ['...'],
            'Kraken': ['...']
        }
        
        analysis = self.addresses.get(address)
        if not analysis:
            analysis = self.analyze_address(address)
        
        exchange_interactions = []
        
        for tx in analysis['transactions']:
            for out in tx['outputs']:
                for exchange_name, exchange_addrs in known_exchanges.items():
                    if out['address'] in exchange_addrs:
                        exchange_interactions.append({
                            'exchange': exchange_name,
                            'transaction': tx['hash'],
                            'amount': out['value'],
                            'time': tx['time']
                        })
        
        return exchange_interactions
    
    def generate_visual_flow(self, tx_hash, output_file='transaction_flow.dot'):
        """
        Generate visual representation of transaction flow
        Uses Graphviz DOT format
        """
        
        flow = self.trace_transaction_flow(tx_hash, depth=2)
        
        if not flow:
            return None
        
        dot_content = "digraph TransactionFlow {\n"
        dot_content += "  rankdir=LR;\n"
        dot_content += "  node [shape=box];\n\n"
        
        for tx_hash, tx_data in flow['flow_map'].items():
            # Add transaction node
            dot_content += f'  "{tx_hash[:8]}..." [label="TX\\n{tx_hash[:8]}..."];\n'
            
            # Add edges from inputs
            for inp in tx_data['inputs']:
                if inp['address']:
                    dot_content += f'  "{inp["address"][:8]}..." -> "{tx_hash[:8]}..." [label="{inp["value"]:.4f} BTC"];\n'
            
            # Add edges to outputs
            for out in tx_data['outputs']:
                if out['address']:
                    dot_content += f'  "{tx_hash[:8]}..." -> "{out["address"][:8]}..." [label="{out["value"]:.4f} BTC"];\n'
        
        dot_content += "}\n"
        
        with open(output_file, 'w') as f:
            f.write(dot_content)
        
        print(f"[+] Transaction flow diagram saved: {output_file}")
        print(f"    Generate PNG: dot -Tpng {output_file} -o transaction_flow.png")
        
        return dot_content
    
    def generate_forensic_report(self, addresses, output_file='crypto_forensics_report.json'):
        """
        Generate comprehensive cryptocurrency forensics report
        """
        
        report = {
            'investigation_date': datetime.now().isoformat(),
            'addresses_analyzed': len(addresses),
            'findings': []
        }
        
        for addr in addresses:
            if addr not in self.addresses:
                self.analyze_address(addr)
            
            findings = {
                'address': addr,
                'analysis': self.addresses[addr],
                'mixing_indicators': self.identify_mixing_services(addr),
                'cluster': self.cluster_analysis(addr),
                'exchange_interactions': self.identify_exchange_deposits(addr)
            }
            
            report['findings'].append(findings)
        
        with open(output_file, 'w') as f:
            json.dump(report, f, indent=2, default=str)
        
        print(f"[+] Forensic report generated: {output_file}")
        
        return report

# Example Investigation
"""
Investigation Scenario:
Ransomware payment sent to Bitcoin address: 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa

Steps:
1. Analyze the payment address
2. Trace where funds went
3. Identify mixing services used
4. Find potential cash-out points (exchanges)
5. Cluster analysis to find related addresses
6. Report findings to law enforcement
"""

if __name__ == "__main__":
    forensics = BitcoinForensics()
    
    # Ransomware payment address
    suspect_address = "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"
    
    # Analyze address
    analysis = forensics.analyze_address(suspect_address)
    
    # Check for mixing
    mixing = forensics.identify_mixing_services(suspect_address)
    
    # Cluster analysis
    cluster = forensics.cluster_analysis(suspect_address)
    
    # Generate report
    report = forensics.generate_forensic_report([suspect_address])
```

### Monero and Privacy Coins

```markdown
## Privacy Cryptocurrency Forensics

### Challenges with Monero/Zcash:

**Monero:**
- Ring signatures hide sender
- Stealth addresses hide receiver
- RingCT hides amount

**Limited Forensic Approaches:**
1. **Exchange analysis**: Track when converted to/from Bitcoin
2. **Network analysis**: IP addresses when broadcasting transactions
3. **Timing analysis**: Correlation with known events
4. **Chain hopping**: Follow conversions between coins

**Investigative Techniques:**
- Subpoena exchange records
- Analyze on-chain timing patterns
- Monitor known addresses (if any)
- Blockchain analytics companies (Chainalysis, CipherTrace)
```

## Advanced Interview Questions

### Question 6: How would you investigate a Business Email Compromise (BEC) incident?

**Model Answer:**

"Business Email Compromise requires both email forensics and financial investigation. Here's my approach:

**Phase 1: Immediate Response (0-2 hours)**

First, I'd secure the compromised email account:
- Reset password immediately
- Enable MFA if not already active
- Review account permissions and delegations
- Check for forwarding rules or inbox rules
- Preserve mailbox before changes

**Phase 2: Email Analysis (2-8 hours)**

I would examine the email headers and logs:

```python
# Analyze email headers for spoofing
def analyze_email_headers(email_file):
    with open(email_file) as f:
        msg = email.message_from_file(f)
    
    # Check authentication results
    spf_result = msg.get('Received-SPF')
    dkim_result = msg.get('DKIM-Signature')
    dmarc_result = msg.get('ARC-Authentication-Results')
    
    # Extract sending IP
    received_headers = msg.get_all('Received')
    
    # Check for reply-to manipulation
    from_addr = msg.get('From')
    reply_to = msg.get('Reply-To')
    
    if reply_to and reply_to != from_addr:
        print(f"[!] Reply-To manipulation: {reply_to}")
```

Key artifacts to examine:
- **Message headers**: Originating IP, SPF/DKIM/DMARC results
- **Email rules**: Auto-forwarding to external addresses
- **Sent items**: Fraudulent wire transfer requests
- **Deleted items**: Attacker often deletes evidence
- **Login logs**: Unusual access times or locations

**Phase 3: Account Compromise Investigation**

Determine how the account was compromised:

1. **Credential Theft**:
   - Check for phishing emails in user's inbox
   - Review recent password changes
   - Check for MFA bypass attempts
   - Examine sign-in logs for failed attempts

2. **Session Hijacking**:
   - Review active sessions
   - Check for session token theft
   - Analyze concurrent logins from different locations

3. **Email Rules/Forwarding**:
   - Check all inbox rules (especially hidden ones)
   - Review mailbox delegates
   - Check auto-forwarding configuration

**Phase 4: Financial Impact Assessment**

Work with finance team to identify:
- Fraudulent wire transfers sent
- Amounts and destinations
- Timing of transfers vs. fraudulent emails
- Whether funds can be recovered (swift recall)

**Phase 5: Attacker Profiling**

Analyze attacker behavior:
```
Indicators of Compromise:
- Email sent from unusual location (Nigeria, Russia, etc.)
- Sent during off-hours (3 AM local time)
- Slight change in email address (CEO@company.com → CE0@company.com)
- Urgent language: "Need wire transfer immediately"
- Request to bypass normal procedures
- Personal email in CC or Reply-To
```

**Phase 6: Timeline Reconstruction**

Build complete timeline:
```
Jan 15 10:30 AM - Phishing email received
Jan 15 10:45 AM - User clicked link, entered credentials
Jan 15 10:50 AM - Attacker logged in from 185.220.101.45 (Nigeria)
Jan 15 11:00 AM - Attacker created inbox rule to hide responses
Jan 15 11:30 AM - Attacker sent first fraudulent email to Finance
Jan 15 02:00 PM - Finance initiated wire transfer ($250,000)
Jan 15 02:30 PM - User noticed unauthorized sent items
Jan 15 02:45 PM - Incident reported to IT/Security
```

**Phase 7: Evidence Preservation**

Collect and preserve:
- Mailbox export (.PST file)
- Email server logs
- Authentication logs
- Network logs showing login IPs
- Screenshots of fraudulent emails
- Financial transaction records

**Phase 8: Law Enforcement Coordination**

- File report with FBI (IC3.gov)
- Contact bank immediately for wire recall
- Provide timeline and evidence to investigators
- Share IOCs with information sharing groups

**Phase 9: Remediation**

- Force password reset for all accounts
- Implement MFA organization-wide
- Remove malicious inbox rules
- Review email security gateway settings
- Implement DMARC policy (reject)
- Train staff on BEC tactics
- Implement financial controls (callback verification)

**Phase 10: Reporting**

Create report with:
- Executive summary (financial impact, how it happened)
- Technical timeline
- Evidence collected
- Attacker tactics
- Recommendations to prevent recurrence

**Key Forensic Artifacts:**

1. **Office 365 Unified Audit Log**:
```powershell
# Export all mailbox activity
Search-UnifiedAuditLog -StartDate 01/15/2024 -EndDate 01/16/2024 `
    -RecordType ExchangeItem -ResultSize 5000 | Export-Csv audit.csv
```

2. **Message Trace**:
```powershell
# Find all emails sent by compromised account
Get-MessageTrace -SenderAddress ceo@company.com `
    -StartDate 01/15/2024 -EndDate 01/16/2024
```

3. **Sign-In Logs**:
```powershell
# Review sign-in activity
Get-AzureADAuditSignInLogs -Filter "userPrincipalName eq 'ceo@company.com'" `
    | Where {$_.CreatedDateTime -gt '2024-01-15'}
```

**Common BEC Indicators I'd Look For:**

- Email from executive requesting urgent wire transfer
- Request to bypass normal procedures
- Sent when executive is traveling or unavailable
- Reply-to address different from sender
- Slight misspelling in domain (company.co instead of company.com)
- Reference to confidential acquisition or legal matter
- Request for gift cards or wire transfers
- Poor grammar or unusual phrasing

**Prevention Recommendations:**

1. **Technical**:
   - Enforce MFA on all accounts
   - Implement DMARC with reject policy
   - Use advanced threat protection
   - Flag external emails
   - Banner warnings on external emails

2. **Process**:
   - Callback verification for wire transfers
   - Dual authorization for large transfers
   - Out-of-band verification (phone call)
   - Financial transaction limits

3. **Training**:
   - Regular phishing simulations
   - BEC awareness training
   - Executive impersonation scenarios
   - Reporting procedures

**Expected Financial Recovery:**

Depends on speed of response:
- Within 24 hours: 50-70% recovery rate
- After 24 hours: 20-30% recovery rate
- After 72 hours: <10% recovery rate

The key is immediate coordination with banks, FBI, and destination bank to freeze funds before they're moved again."

**Follow-up Handled:**

Q: What if the attacker used the actual CEO's real account?
A: "Then it's true account compromise, not spoofing. Investigation focuses on how credentials were stolen - phishing, malware, credential stuffing. We'd examine all account activity for the reconnaissance phase where attackers read emails to understand business processes before striking. Preserved mailbox would show this surveillance activity."

### Question 7: Walk me through memory forensics on a suspected APT compromise.

**Model Answer:**

"Advanced Persistent Threat memory forensics requires systematic analysis to find sophisticated malware. Here's my methodology:

**Phase 1: Memory Acquisition (0-15 minutes)**

First, capture memory without alerting the attacker:

```bash
# On suspect system, using DumpIt or WinPmem
# DO NOT use Task Manager or shutdown - alerts attacker

# Windows
winpmem_mini_x64.exe memory.raw

# Verify capture
sha256sum memory.raw > memory.raw.sha256
```

Critical considerations:
- Capture before any remediation
- Don't run multiple tools (minimal footprint)
- Preserve for multiple analysis passes
- Document exact time of capture

**Phase 2: Profile Identification (5 minutes)**

```bash
# Volatility 3
volatility3 -f memory.raw windows.info

# Confirms:
# - Windows version
# - Build number  
# - Architecture (x64/x86)
# - Kernel base address
# - DTB (Directory Table Base)
```

**Phase 3: Process Analysis (30 minutes)**

Look for process anomalies:

```bash
# Process list
volatility3 -f memory.raw windows.pslist > pslist.txt

# Process tree (parent-child relationships)
volatility3 -f memory.raw windows.pstree > pstree.txt

# Scan for hidden processes (rootkit detection)
volatility3 -f memory.raw windows.psscan > psscan.txt

# Compare pslist vs psscan
# Processes in psscan but not pslist = hidden by rootkit
```

**APT Process Indicators:**

Looking for:
- Misspelled system processes (svchοst.exe with unicode 'o')
- System processes running from wrong locations
- Unusual parent-child relationships (cmd.exe parent of svchost.exe)
- Processes with no on-disk executable
- Injected threads in legitimate processes

Example findings:
```
[!] svchost.exe running from C:\Users\Public\ (should be C:\Windows\System32\)
[!] lsass.exe has 15 threads, 3 appear injected (different start addresses)
[!] rundll32.exe with no command-line arguments (common for reflective DLL injection)
```

**Phase 4: Network Connections (15 minutes)**

```bash
# Active network connections
volatility3 -f memory.raw windows.netscan > netscan.txt

# Analyze for:
# - Connections to foreign IPs
# - Unusual ports (not 80, 443, 53)
# - Multiple connections to single IP (C2 beacon)
# - Connections from system processes
```

APT C2 patterns:
```
[!] svchost.exe (PID 1234) connected to 185.220.101.45:443
    - Foreign IP (Russia)
    - HTTPS but to IP, not domain
    - Not legitimate Windows Update

[!] explorer.exe (PID 5678) multiple connections to same IP
    - Beaconing pattern every 60 seconds
    - Indicates C2 callback
```

**Phase 5: Code Injection Detection (45 minutes)**

```bash
# Malfind - detects injected code
volatility3 -f memory.raw windows.malfind > malfind.txt

# For each suspicious process, examine:
# - Memory protection (PAGE_EXECUTE_READWRITE suspicious)
# - Disassembly of injected regions
# - Presence of PE headers in memory
```

Analysis of findings:
```python
# Extract and analyze suspicious memory regions
def analyze_injection(pid, start_addr):
    # Dump memory region
    volatility3 -f memory.raw -o extracted/ \
        windows.memmap --pid {pid} --dump
    
    # Look for:
    # - Shellcode patterns
    # - Embedded PE files
    # - Encoded/encrypted regions (high entropy)
    # - Known malware signatures
```

**Phase 6: DLL Analysis (30 minutes)**

```bash
# List loaded DLLs for suspicious processes
volatility3 -f memory.raw windows.dlllist --pid 1234

# Look for:
# - Unsigned DLLs
# - DLLs loaded from unusual locations (%TEMP%, %APPDATA%)
# - Suspicious DLL names
# - DLLs not on disk (reflective loading)
```

**Phase 7: Malware Identification (1 hour)**

Extract suspicious processes for analysis:

```bash
# Dump process executable
volatility3 -f memory.raw -o extracted/ windows.pslist --pid 1234 --dump

# Calculate hash
sha256sum extracted/1234.dmp

# Check VirusTotal
curl --request POST \
  --url 'https://www.virustotal.com/api/v3/files/[hash]' \
  --header 'x-apikey: YOUR_API_KEY'
```

Static analysis of extracted binary:
```bash
# Strings analysis
strings 1234.dmp | grep -iE '(http|https|\.dll|\.exe)'

# Check for packing
python3 << EOF
import pefile
pe = pefile.PE('1234.dmp')
for section in pe.sections:
    entropy = section.get_entropy()
    if entropy > 7.0:
        print(f"High entropy section: {section.Name}")
EOF
```

**Phase 8: Registry Persistence (20 minutes)**

```bash
# Extract registry hives from memory
volatility3 -f memory.raw windows.registry.hivelist

# Check common persistence locations
volatility3 -f memory.raw windows.registry.printkey \
    --key "Software\Microsoft\Windows\CurrentVersion\Run"

# Services
volatility3 -f memory.raw windows.registry.printkey \
    --key "System\CurrentControlSet\Services"
```

**Phase 9: Credential Extraction (15 minutes)**

```bash
# Extract password hashes
volatility3 -f memory.raw windows.hashdump

# Extract LSA secrets
volatility3 -f memory.raw windows.lsadump

# Look for cached credentials
volatility3 -f memory.raw windows.cached
```

**Phase 10: Timeline Reconstruction (30 minutes)**

```bash
# Create timeline of all memory artifacts
volatility3 -f memory.raw windows.timeliner > timeline.csv

# Import into timeline analysis tool
# Correlate with:
# - Disk forensics timeline
# - Network flow data
# - Event logs
```

**Complete APT Analysis:**

Comprehensive findings report:
```markdown
## APT Memory Forensics Findings

### Malware Identified
- **Family**: Cobalt Strike Beacon
- **Hash**: a1b2c3d4...
- **PID**: 1234 (masquerading as svchost.exe)
- **Location**: C:\Users\Public\svchost.exe

### C2 Communication
- **C2 Server**: 185.220.101.45
- **Port**: 443 (HTTPS)
- **Beaconing**: Every 60 seconds
- **Protocol**: Encrypted Cobalt Strike protocol

### Persistence
- **Method**: Service creation
- **Service Name**: "WindowsUpdateService"  
- **Registry**: HKLM\System\CurrentControlSet\Services\WindowsUpdateService
- **Binary**: C:\Windows\System32\wuauserv.dll (fake)

### Lateral Movement Evidence
- **Technique**: Pass-the-Hash
- **Compromised Accounts**: 5 domain accounts
- **Credential Dumping**: Mimikatz evidence in lsass.exe memory

### Data Staging
- **Location**: C:\Windows\Temp\update.cab
- **Size**: 250 MB
- **Content**: Compressed corporate documents
- **Exfiltration Status**: Not yet exfiltrated (upload interrupted)
```

**Indicators of APT vs. Commodity Malware:**

APT Characteristics Found:
- Legitimate-looking process names
- Signed C2 SSL certificate (stolen)
- Low and slow C2 (60 sec beacon vs 1 sec)
- Living-off-the-land tools (PsExec, Mimikatz)
- Manual keyboard activity (not automated)
- Data staging before exfiltration
- Persistence across reboots
- Multi-stage payload deployment

**Recommendations:**

Immediate:
- Isolate compromised system
- Reset all credentials
- Block C2 IP at perimeter
- Hunt for lateral movement

Long-term:
- Deploy EDR across environment
- Implement application whitelisting
- Network segmentation
- Enhanced monitoring

**Reporting:**

Provide to stakeholders:
- Executive summary (business impact)
- Technical details (for IT/Security)
- IOCs for threat hunting
- MITRE ATT&CK mapping
- Recommendations"

### Question 8: How do you handle anti-forensics and evidence destruction techniques?

**Model Answer:**

"Anti-forensics is specifically designed to frustrate investigators. Here's how I detect and counteract common techniques:

**1. Timestamp Manipulation (Timestomping)**

Detection method:
```python
def detect_timestomping(file_path):
    """
    Compare $STANDARD_INFORMATION vs $FILE_NAME timestamps
    $SI can be easily modified, $FN cannot
    """
    
    import pytsk3
    import pyewf
    
    # Open NTFS filesystem
    img = pytsk3.Img_Info(file_path)
    fs = pytsk3.FS_Info(img)
    
    # Get file's MFT entry
    f = fs.open("/path/to/suspicious/file")
    
    # Read $STANDARD_INFORMATION
    si_attr = f.info.meta.attr
    si_created = si_attr.created
    si_modified = si_attr.modified
    
    # Read $FILE_NAME  
    fn_attr = f.info.name
    fn_created = fn_attr.created
    fn_modified = fn_attr.modified
    
    # Compare
    if si_modified != fn_modified:
        return True, "Timestamp manipulation detected"
    
    # Additional checks:
    if si_created > si_modified:
        return True, "Created time after modified time (impossible)"
    
    if si_modified > current_time:
        return True, "Future timestamp detected"
    
    return False, "Timestamps appear legitimate"
```

Indicators:
- Created time after modified time
- Future timestamps
- All files in directory with identical timestamps
- Timestamps don't match file system journal

**2. Data Wiping**

Detection and recovery:
```bash
# Check for wiping tool artifacts
strings disk.dd | grep -i "eraser\|wipe\|shred\|ccleaner"

# Examine deleted file inodes for wiping patterns
# Gutmann method: 35 passes
# DOD 5220.22-M: 7 passes  
# Zero-fill: Single pass of 0x00

# Look for patterns indicating wiping
hexdump -C deleted_file | grep "00 00 00 00"  # Zero fill
hexdump -C deleted_file | grep "ff ff ff ff"  # One fill

# Check $UsnJrnl for deletion + overwrite sequence
# Normal delete: DELETE entry
# Wiped: DELETE + multiple WRITE entries
```

Cannot recover truly wiped data, but can prove wiping occurred (evidence of consciousness of guilt).

**3. Encryption**

Approaches:
```markdown
A) Memory Analysis (if system running)
   - Encryption keys often in RAM
   - Use Volatility to extract keys
   - Access mounted encrypted volumes

B) Key Recovery
   - BitLocker: Check TPM, Active Directory, Microsoft Account
   - FileVault: Check iCloud, recovery key file
   - Corporate: Check key escrow systems

C) Legal Compulsion
   - Court order to provide decryption key
   - Contempt charges for refusal
   - Fifth Amendment considerations (US)

D) Alternative Evidence
   - Cloud backups (often unencrypted)
   - Email on server
   - Network traffic captures
   - Memory dumps
   - Partial data from unencrypted swap/hibernate
```

**4. Steganography**

Detection techniques:
```python
def detect_steganography(image_path):
    """
    Statistical analysis for hidden data in images
    """
    
    from PIL import Image
    import numpy as np
    
    img = Image.open(image_path)
    pixels = np.array(img)
    
    # Check LSB (Least Significant Bit) distribution
    # Random image: LSB should be ~50% 0s and 50% 1s
    # Steganography: skewed distribution
    
    lsb_values = pixels.flatten() & 1
    lsb_ratio = np.sum(lsb_values) / len(lsb_values)
    
    # Expected: ~0.5
    # Suspicious: >0.6 or <0.4
    
    if abs(lsb_ratio - 0.5) > 0.1:
        return True, f"Suspicious LSB ratio: {lsb_ratio}"
    
    # Chi-square test for randomness
    # High chi-square = potential steganography
    
    return False, "No steganography detected"
```

Tools:
- StegDetect: Automated detection
- StegExpose: Machine learning detection
- Steganalysis: Statistical analysis

**5. Log Deletion**

Detection:
```powershell
# Windows Event Log clearing
# Event ID 1102 - Security log was cleared

Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=1102
}

# Check for gaps in event sequence
# Event logs have sequential Event Record IDs
# Gap indicates deletion

$events = Get-WinEvent -LogName Security -MaxEvents 10000
$missing_ids = @()

for ($i = 0; $i -lt $events.Count - 1; $i++) {
    $current_id = $events[$i].RecordId
    $next_id = $events[$i+1].RecordId
    
    if ($current_id - $next_id > 1) {
        $missing_ids += "$next_id to $current_id"
    }
}

if ($missing_ids.Count -gt 0) {
    Write-Host "[!] Missing event IDs detected: $missing_ids"
}
```

Alternative log sources:
- SIEM (centralized logging)
- Syslog forwarding
- Cloud logging (CloudTrail, Azure Monitor)
- Firewall/proxy logs
- Application-specific logs

**6. File System Manipulation**

```bash
# Check for alternate data streams (ADS)
dir /r C:\

# Or using PowerShell
Get-Item C:\file.txt -Stream *

# Access hidden stream
notepad C:\file.txt:hidden.txt

# Extract ADS for analysis
Get-Content C:\file.txt -Stream hidden.txt
```

**7. Virtual Machine/Sandbox Detection**

Malware checks for analysis environment:
```python
# Common VM detection techniques used by malware

def detect_vm_artifacts():
    """
    Check if running in VM
    Malware uses this to avoid analysis
    """
    
    indicators = []
    
    # Check for VM-specific hardware
    # VMware: VMware SCSI Disk
    # VirtualBox: VBOX
    # Hyper-V: Hyper-V
    
    # Check for VM processes
    vm_processes = [
        'vmtoolsd.exe',  # VMware Tools
        'vboxservice.exe',  # VirtualBox
        'vmware.exe'
    ]
    
    # Check registry for VM indicators
    vm_registry_keys = [
        r'SYSTEM\ControlSet001\Services\Disk\Enum',  # Disk info
        r'HARDWARE\Description\System\SystemBiosVersion',  # BIOS
    ]
    
    # If VM detected, malware may:
    # - Refuse to run
    # - Show benign behavior
    # - Self-delete
    
    return indicators

# Countermeasure: Use bare metal analysis or advanced VM evasion
```

**8. Fileless Malware**

Detection in memory:
```bash
# No files on disk = no disk forensics
# Must rely on memory forensics

# Find injected code
volatility -f memory.raw --profile=Win10x64_19041 malfind

# Extract suspicious memory regions
volatility -f memory.raw --profile=Win10x64_19041 \
    memdump -p 1234 -D output/

# Analyze for:
# - PowerShell scripts (in memory)
# - WMI persistence
# - Registry-only persistence
# - Living-off-the-land binaries
```

**9. Secure Deletion**

Cannot recover, but can prove it occurred:
```markdown
Evidence of Secure Deletion:
- Presence of wiping tool
- File system journal shows multiple overwrites
- Unallocated space shows wiping patterns
- Deleted files all have same characteristics

Legal Implications:
- Spoliation of evidence
- Obstruction of justice
- Consciousness of guilt
- Adverse inference instruction to jury
```

**10. Anti-Forensics Forensics**

The presence of anti-forensics is itself evidence:
```markdown
Why would innocent user:
- Install file wiping software?
- Use encryption on personal laptop?
- Clear event logs?
- Use privacy coin (Monero)?
- Access computer at 3 AM before interview?

Document all anti-forensics:
- Screenshots of tools
- Evidence of execution
- Timing (often just before forensic examination)
- Intent to destroy evidence

This becomes part of the case:
- Shows consciousness of guilt
- Supports obstruction charges
- Justifies adverse inference
```

**Best Practices When Encountering Anti-Forensics:**

1. **Document everything**
   - Screenshot wiping tools
   - Photograph encrypted drives
   - Note timestamp anomalies

2. **Look for alternative evidence**
   - Cloud backups
   - Email on server
   - Network logs
   - Mobile devices

3. **Act quickly**
   - Get legal hold notice immediately
   - Preserve before more destruction

4. **Get expert help**
   - Advanced techniques may be needed
   - Consider government resources (FBI lab)

5. **Focus on what you CAN prove**
   - Obstruction of justice
   - That sensitive data existed
   - That it was deliberately destroyed

The key is: you may not recover the data, but you can prove it existed and was intentionally destroyed, which itself is powerful evidence."

[Continuing with more questions and specialized topics...]

## 3.12 Log Analysis & SIEM Deep Dive

### Windows Event Log Forensics

**Critical Event IDs for Investigations:**

```powershell
# Windows Event Log Analysis Script
# Comprehensive analysis of security-relevant events

param(
    [string]$ComputerName = "localhost",
    [int]$DaysBack = 7
)

$StartDate = (Get-Date).AddDays(-$DaysBack)

Write-Host "[*] Analyzing Windows Event Logs for: $ComputerName" -ForegroundColor Green
Write-Host "[*] Date Range: $StartDate to $(Get-Date)" -ForegroundColor Green

# Authentication Events
Write-Host "`n[+] Analyzing Authentication Events..." -ForegroundColor Cyan

# Successful Logons (4624)
$SuccessfulLogons = Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4624
    StartTime=$StartDate
} -ComputerName $ComputerName -ErrorAction SilentlyContinue

Write-Host "    Successful Logons: $($SuccessfulLogons.Count)"

# Break down by logon type
$LogonTypes = @{
    2 = "Interactive (Console)"
    3 = "Network (SMB, RDP)"
    4 = "Batch"
    5 = "Service"
    7 = "Unlock"
    8 = "NetworkCleartext"
    9 = "NewCredentials"
    10 = "RemoteInteractive (RDP)"
    11 = "CachedInteractive"
}

$LogonsByType = $SuccessfulLogons | ForEach-Object {
    $xml = [xml]$_.ToXml()
    $logonType = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'LogonType'} | Select-Object -ExpandProperty '#text'
    [PSCustomObject]@{
        Time = $_.TimeCreated
        User = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'TargetUserName'} | Select-Object -ExpandProperty '#text'
        LogonType = $logonType
        SourceIP = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'IpAddress'} | Select-Object -ExpandProperty '#text'
        WorkstationName = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'WorkstationName'} | Select-Object -ExpandProperty '#text'
    }
} | Group-Object LogonType

foreach ($group in $LogonsByType) {
    $typeName = $LogonTypes[[int]$group.Name]
    Write-Host "    Type $($group.Name) ($typeName): $($group.Count) events"
}

# Failed Logons (4625)
Write-Host "`n[+] Analyzing Failed Logon Attempts..." -ForegroundColor Cyan

$FailedLogons = Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4625
    StartTime=$StartDate
} -ComputerName $ComputerName -ErrorAction SilentlyContinue

Write-Host "    Failed Logons: $($FailedLogons.Count)"

# Find potential brute force attempts
$FailedByUser = $FailedLogons | ForEach-Object {
    $xml = [xml]$_.ToXml()
    [PSCustomObject]@{
        Time = $_.TimeCreated
        User = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'TargetUserName'} | Select-Object -ExpandProperty '#text'
        SourceIP = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'IpAddress'} | Select-Object -ExpandProperty '#text'
        FailureReason = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'SubStatus'} | Select-Object -ExpandProperty '#text'
    }
} | Group-Object User | Where-Object {$_.Count -gt 10} | Sort-Object Count -Descending

if ($FailedByUser) {
    Write-Host "`n[!] Potential Brute Force Attempts Detected:" -ForegroundColor Yellow
    foreach ($user in $FailedByUser | Select-Object -First 10) {
        Write-Host "    User: $($user.Name) - Failed Attempts: $($user.Count)" -ForegroundColor Yellow
    }
}

# Account Lockouts (4740)
Write-Host "`n[+] Analyzing Account Lockouts..." -ForegroundColor Cyan

$Lockouts = Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4740
    StartTime=$StartDate
} -ComputerName $ComputerName -ErrorAction SilentlyContinue

Write-Host "    Account Lockouts: $($Lockouts.Count)"

if ($Lockouts) {
    $Lockouts | ForEach-Object {
        $xml = [xml]$_.ToXml()
        Write-Host "    - $($_.TimeCreated): User $($xml.Event.EventData.Data | Where-Object {$_.Name -eq 'TargetUserName'} | Select-Object -ExpandProperty '#text')"
    }
}

# Privileged Activity
Write-Host "`n[+] Analyzing Privileged Activity..." -ForegroundColor Cyan

# Special Privileges Assigned (4672)
$SpecialPrivileges = Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4672
    StartTime=$StartDate
} -ComputerName $ComputerName -ErrorAction SilentlyContinue

Write-Host "    Special Privileges Assigned: $($SpecialPrivileges.Count)"

# User Account Created (4720)
$UserCreated = Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4720
    StartTime=$StartDate
} -ComputerName $ComputerName -ErrorAction SilentlyContinue

if ($UserCreated) {
    Write-Host "`n[!] New User Accounts Created:" -ForegroundColor Yellow
    $UserCreated | ForEach-Object {
        $xml = [xml]$_.ToXml()
        Write-Host "    - $($_.TimeCreated): $($xml.Event.EventData.Data | Where-Object {$_.Name -eq 'TargetUserName'} | Select-Object -ExpandProperty '#text')"
    }
}

# User Added to Group (4728, 4732, 4756)
$GroupChanges = Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4728,4732,4756
    StartTime=$StartDate
} -ComputerName $ComputerName -ErrorAction SilentlyContinue

if ($GroupChanges) {
    Write-Host "`n[!] Group Membership Changes:" -ForegroundColor Yellow
    $GroupChanges | ForEach-Object {
        $xml = [xml]$_.ToXml()
        $member = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'MemberName'} | Select-Object -ExpandProperty '#text'
        $group = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'TargetUserName'} | Select-Object -ExpandProperty '#text'
        Write-Host "    - $($_.TimeCreated): $member added to $group"
    }
}

# Process Execution
Write-Host "`n[+] Analyzing Process Execution..." -ForegroundColor Cyan

# Process Creation (4688) - Requires audit policy enabled
$ProcessCreation = Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4688
    StartTime=$StartDate
} -ComputerName $ComputerName -ErrorAction SilentlyContinue

Write-Host "    Process Creation Events: $($ProcessCreation.Count)"

# Look for suspicious processes
$SuspiciousKeywords = @('powershell', 'cmd', 'psexec', 'wmic', 'mimikatz', 'procdump', 'psexesvc')

$SuspiciousProcesses = $ProcessCreation | ForEach-Object {
    $xml = [xml]$_.ToXml()
    $processName = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'NewProcessName'} | Select-Object -ExpandProperty '#text'
    
    foreach ($keyword in $SuspiciousKeywords) {
        if ($processName -like "*$keyword*") {
            [PSCustomObject]@{
                Time = $_.TimeCreated
                Process = $processName
                User = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'SubjectUserName'} | Select-Object -ExpandProperty '#text'
                CommandLine = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'CommandLine'} | Select-Object -ExpandProperty '#text'
            }
            break
        }
    }
} | Where-Object {$_ -ne $null}

if ($SuspiciousProcesses) {
    Write-Host "`n[!] Suspicious Processes Detected:" -ForegroundColor Yellow
    $SuspiciousProcesses | Select-Object -First 20 | ForEach-Object {
        Write-Host "    - $($_.Time): $($_.Process) by $($_.User)"
        if ($_.CommandLine) {
            Write-Host "      CMD: $($_.CommandLine)" -ForegroundColor Gray
        }
    }
}

# Scheduled Tasks
Write-Host "`n[+] Analyzing Scheduled Task Activity..." -ForegroundColor Cyan

# Task Scheduler events (4698 - task created)
$TasksCreated = Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4698
    StartTime=$StartDate
} -ComputerName $ComputerName -ErrorAction SilentlyContinue

if ($TasksCreated) {
    Write-Host "`n[!] Scheduled Tasks Created:" -ForegroundColor Yellow
    $TasksCreated | ForEach-Object {
        $xml = [xml]$_.ToXml()
        Write-Host "    - $($_.TimeCreated): $($xml.Event.EventData.Data | Where-Object {$_.Name -eq 'TaskName'} | Select-Object -ExpandProperty '#text')"
    }
}

# Clearing Event Logs (1102, 104)
Write-Host "`n[+] Checking for Log Clearing..." -ForegroundColor Cyan

$LogClearing = Get-WinEvent -FilterHashtable @{
    LogName='Security','System'
    ID=1102,104
    StartTime=$StartDate
} -ComputerName $ComputerName -ErrorAction SilentlyContinue

if ($LogClearing) {
    Write-Host "`n[!!] LOG CLEARING DETECTED - ANTI-FORENSICS ACTIVITY:" -ForegroundColor Red
    $LogClearing | ForEach-Object {
        Write-Host "    - $($_.TimeCreated): Log cleared by $($_.UserId)" -ForegroundColor Red
    }
}

# Kerberos Ticket Operations
Write-Host "`n[+] Analyzing Kerberos Activity..." -ForegroundColor Cyan

# Kerberos TGT Requested (4768)
$TGTRequests = Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4768
    StartTime=$StartDate
} -ComputerName $ComputerName -ErrorAction SilentlyContinue

Write-Host "    Kerberos TGT Requests: $($TGTRequests.Count)"

# Kerberos Service Ticket (4769)
$ServiceTickets = Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4769
    StartTime=$StartDate
} -ComputerName $ComputerName -ErrorAction SilentlyContinue

Write-Host "    Kerberos Service Tickets: $($ServiceTickets.Count)"

# Look for potential Kerberoasting (many service tickets in short time)
$PotentialKerberoasting = $ServiceTickets | Group-Object {$_.TimeCreated.ToString('yyyy-MM-dd HH:mm')} | 
    Where-Object {$_.Count -gt 20}

if ($PotentialKerberoasting) {
    Write-Host "`n[!] Potential Kerberoasting Detected:" -ForegroundColor Yellow
    $PotentialKerberoasting | Select-Object -First 5 | ForEach-Object {
        Write-Host "    - $($_.Name): $($_.Count) service ticket requests"
    }
}

# Generate Summary Report
Write-Host "`n========================================" -ForegroundColor Green
Write-Host "Event Log Analysis Summary" -ForegroundColor Green
Write-Host "========================================" -ForegroundColor Green
Write-Host "Total Successful Logons: $($SuccessfulLogons.Count)"
Write-Host "Total Failed Logons: $($FailedLogons.Count)"
Write-Host "Account Lockouts: $($Lockouts.Count)"
Write-Host "New User Accounts: $($UserCreated.Count)"
Write-Host "Group Changes: $($GroupChanges.Count)"
Write-Host "Process Executions: $($ProcessCreation.Count)"
Write-Host "Suspicious Processes: $($SuspiciousProcesses.Count)"
Write-Host "Scheduled Tasks Created: $($TasksCreated.Count)"
Write-Host "Log Clearing Events: $($LogClearing.Count)"
Write-Host "========================================" -ForegroundColor Green
```

### Linux Log Analysis

```bash
#!/bin/bash
# Linux Log Analysis Script
# Comprehensive analysis of system logs

echo "[*] Linux Log Analysis"
echo "======================"

LOG_DIR="/var/log"
DAYS_BACK=7
OUTPUT_DIR="./log_analysis_$(date +%Y%m%d_%H%M%S)"

mkdir -p "$OUTPUT_DIR"

# Authentication Logs
echo "[+] Analyzing authentication logs..."

# Successful SSH logins
echo "  - Successful SSH logins:"
grep "Accepted" /var/log/auth.log* 2>/dev/null | tail -20 | tee "$OUTPUT_DIR/ssh_successful.txt"

# Failed SSH attempts
echo "  - Failed SSH attempts:"
grep "Failed password" /var/log/auth.log* 2>/dev/null | 
    awk '{print $1, $2, $3, $11, $13}' | 
    sort | uniq -c | sort -rn | head -20 | tee "$OUTPUT_DIR/ssh_failed.txt"

# sudo usage
echo "  - sudo command usage:"
grep "sudo" /var/log/auth.log* 2>/dev/null | 
    grep "COMMAND" | tail -20 | tee "$OUTPUT_DIR/sudo_usage.txt"

# User account changes
echo "  - User account modifications:"
grep -E "useradd|userdel|usermod|passwd" /var/log/auth.log* 2>/dev/null | 
    tail -20 | tee "$OUTPUT_DIR/user_changes.txt"

# System Events
echo "[+] Analyzing system logs..."

# System boots/shutdowns
echo "  - System boots:"
grep -E "boot|shutdown|reboot" /var/log/syslog* 2>/dev/null | 
    tail -20 | tee "$OUTPUT_DIR/boot_shutdown.txt"

# Kernel messages
echo "  - Recent kernel messages:"
dmesg -T | tail -50 | tee "$OUTPUT_DIR/kernel_messages.txt"

# Service failures
echo "  - Service failures:"
grep -i "failed\|error" /var/log/syslog* 2>/dev/null | 
    grep -v "CRON" | tail -30 | tee "$OUTPUT_DIR/service_failures.txt"

# Package Management
echo "[+] Analyzing package management..."

# Recently installed packages
echo "  - Recently installed packages:"
if [ -f /var/log/dpkg.log ]; then
    grep "install " /var/log/dpkg.log* 2>/dev/null | 
        tail -20 | tee "$OUTPUT_DIR/packages_installed.txt"
elif [ -f /var/log/yum.log ]; then
    grep "Installed:" /var/log/yum.log* 2>/dev/null | 
        tail -20 | tee "$OUTPUT_DIR/packages_installed.txt"
fi

# Web Server Logs (if Apache/Nginx installed)
echo "[+] Analyzing web server logs..."

if [ -d /var/log/apache2 ]; then
    echo "  - Apache access patterns:"
    cat /var/log/apache2/access.log* 2>/dev/null | 
        awk '{print $1}' | sort | uniq -c | sort -rn | head -20 | 
        tee "$OUTPUT_DIR/apache_top_ips.txt"
    
    echo "  - Apache errors:"
    tail -50 /var/log/apache2/error.log 2>/dev/null | tee "$OUTPUT_DIR/apache_errors.txt"
fi

if [ -d /var/log/nginx ]; then
    echo "  - Nginx access patterns:"
    cat /var/log/nginx/access.log* 2>/dev/null | 
        awk '{print $1}' | sort | uniq -c | sort -rn | head -20 | 
        tee "$OUTPUT_DIR/nginx_top_ips.txt"
    
    echo "  - Nginx errors:"
    tail -50 /var/log/nginx/error.log 2>/dev/null | tee "$OUTPUT_DIR/nginx_errors.txt"
fi

# Database Logs (if MySQL/PostgreSQL installed)
echo "[+] Analyzing database logs..."

if [ -f /var/log/mysql/error.log ]; then
    echo "  - MySQL errors:"
    tail -50 /var/log/mysql/error.log | tee "$OUTPUT_DIR/mysql_errors.txt"
fi

if [ -d /var/log/postgresql ]; then
    echo "  - PostgreSQL recent activity:"
    tail -50 /var/log/postgresql/postgresql-*.log 2>/dev/null | tee "$OUTPUT_DIR/postgresql_activity.txt"
fi

# Suspicious Activity Detection
echo "[+] Detecting suspicious activity..."

# Look for potential privilege escalation
echo "  - Checking for privilege escalation attempts:"
grep -i "su:\|sudo:" /var/log/auth.log* 2>/dev/null | 
    grep -i "failed\|incorrect" | tail -20 | tee "$OUTPUT_DIR/privesc_attempts.txt"

# Check for unusual cron jobs
echo "  - Unusual cron job execution:"
grep "CRON" /var/log/syslog* 2>/dev/null | 
    grep -v "session opened\|session closed" | tail -30 | tee "$OUTPUT_DIR/cron_unusual.txt"

# Look for port scanning indicators
echo "  - Potential port scanning:"
grep -E "SYN.*DPT" /var/log/kern.log* 2>/dev/null | 
    awk '{print $12}' | sort | uniq -c | sort -rn | head -10 | tee "$OUTPUT_DIR/port_scan_indicators.txt"

# Check for rootkit indicators
echo "  - Checking for rootkit indicators:"
strings /var/log/syslog* 2>/dev/null | grep -iE "(rootkit|backdoor|trojan)" | 
    tee "$OUTPUT_DIR/rootkit_indicators.txt"

# Timeline Generation
echo "[+] Generating event timeline..."

# Combine all logs into timeline
{
    grep "^" /var/log/auth.log* 2>/dev/null | head -100
    grep "^" /var/log/syslog* 2>/dev/null | head -100
} | sort -k1,2M -k3 | tee "$OUTPUT_DIR/event_timeline.txt" > /dev/null

# Summary Statistics
echo ""
echo "========================================="
echo "Log Analysis Summary"
echo "========================================="
echo "Output directory: $OUTPUT_DIR"
echo ""
echo "Key Findings:"
echo "  - Successful SSH logins: $(grep -c "Accepted" /var/log/auth.log* 2>/dev/null || echo 0)"
echo "  - Failed SSH attempts: $(grep -c "Failed password" /var/log/auth.log* 2>/dev/null || echo 0)"
echo "  - sudo commands: $(grep -c "sudo.*COMMAND" /var/log/auth.log* 2>/dev/null || echo 0)"
echo "  - System boots: $(grep -c "boot" /var/log/syslog* 2>/dev/null || echo 0)"
echo "========================================="

# Generate HTML report (optional)
cat > "$OUTPUT_DIR/report.html" << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Linux Log Analysis Report</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        h1 { color: #333; }
        h2 { color: #666; border-bottom: 2px solid #ddd; }
        pre { background: #f4f4f4; padding: 10px; border-radius: 5px; }
        .warning { color: #ff6600; }
        .critical { color: #ff0000; font-weight: bold; }
    </style>
</head>
<body>
    <h1>Linux Log Analysis Report</h1>
    <p>Generated: $(date)</p>
    
    <h2>Executive Summary</h2>
    <ul>
        <li>Analysis Period: Last 7 days</li>
        <li>Logs Analyzed: auth.log, syslog, kernel logs</li>
        <li>Total Events: $(cat $OUTPUT_DIR/*.txt | wc -l)</li>
    </ul>
    
    <h2>Key Findings</h2>
    <p>Detailed findings available in individual text files.</p>
</body>
</html>
EOF

echo "[+] HTML report generated: $OUTPUT_DIR/report.html"
echo "[+] Analysis complete!"
```

### SIEM Correlation and Analysis

**Splunk SPL Queries for Incident Investigation:**

```spl
# Splunk Search Processing Language (SPL) Examples

# 1. Failed Login Investigation
index=windows EventCode=4625 
| stats count by src_ip, user, ComputerName 
| where count > 10 
| sort -count
| table src_ip, user, ComputerName, count

# 2. Lateral Movement Detection
# Multiple authentications from single source to multiple destinations
index=windows EventCode=4624 Logon_Type=3 
| stats dc(dest) as unique_targets by src_ip, user 
| where unique_targets > 5
| sort -unique_targets
| table src_ip, user, unique_targets

# 3. Privilege Escalation
# User added to administrators group
index=windows EventCode=4732 TargetUserName="Administrators"
| table _time, SubjectUserName, MemberName, ComputerName

# 4. Data Exfiltration Detection
# Large outbound data transfers
index=firewall action=allowed direction=outbound 
| stats sum(bytes_out) as total_bytes by src_ip, dest_ip 
| where total_bytes > 100000000 
| eval total_MB=round(total_bytes/1024/1024,2)
| sort -total_MB
| table src_ip, dest_ip, total_MB

# 5. Malware C2 Beaconing
# Regular intervals indicate automated callbacks
index=proxy 
| transaction src_ip dest_ip maxspan=1h 
| where eventcount > 10 
| stats stdev(duration) as duration_stdev, avg(duration) as duration_avg by src_ip, dest_ip 
| where duration_stdev < 5
| table src_ip, dest_ip, duration_avg, duration_stdev

# 6. PowerShell Execution Analysis
index=windows EventCode=4104 
| rex field=ScriptBlockText "(?<command>Invoke-\w+)"
| stats count by command, ComputerName, User
| sort -count
| table command, ComputerName, User, count

# 7. Account Creation Timeline
index=windows (EventCode=4720 OR EventCode=4732)
| transaction user maxspan=1h
| table _time, EventCode, user, SubjectUserName

# 8. File Integrity Monitoring
# Detect changes to critical files
index=fim file_path IN ("/etc/passwd", "/etc/shadow", "C:\\Windows\\System32\\config\\SAM")
| table _time, file_path, action, user, hash_before, hash_after

# 9. Network Anomaly Detection
index=netflow 
| stats avg(bytes) as avg_bytes, stdev(bytes) as stdev_bytes by src_ip 
| eval threshold=avg_bytes+(3*stdev_bytes)
| join src_ip [search index=netflow | stats sum(bytes) as current_bytes by src_ip]
| where current_bytes > threshold
| table src_ip, avg_bytes, current_bytes, threshold

# 10. Compromised Credentials
# Password spray attack detection
index=windows EventCode=4625
| bucket _time span=1m
| stats dc(TargetUserName) as unique_users, count by src_ip, _time
| where unique_users > 10 AND count > 20
| table _time, src_ip, unique_users, count
```

**ELK Stack Queries (Kibana):**

```json
// Elasticsearch Query DSL Examples

// 1. Failed Authentication Spike
{
  "query": {
    "bool": {
      "must": [
        { "match": { "event.code": "4625" }},
        { "range": { "@timestamp": { "gte": "now-1h" }}}
      ]
    }
  },
  "aggs": {
    "failed_by_ip": {
      "terms": {
        "field": "source.ip",
        "size": 20
      }
    }
  }
}

// 2. Lateral Movement
{
  "query": {
    "bool": {
      "must": [
        { "match": { "event.code": "4624" }},
        { "match": { "winlog.event_data.LogonType": "3" }}
      ]
    }
  },
  "aggs": {
    "by_source": {
      "terms": {
        "field": "source.ip"
      },
      "aggs": {
        "unique_destinations": {
          "cardinality": {
            "field": "destination.ip"
          }
        }
      }
    }
  }
}

// 3. Malware Execution
{
  "query": {
    "bool": {
      "should": [
        { "wildcard": { "process.name": "*mimikatz*" }},
        { "wildcard": { "process.name": "*procdump*" }},
        { "wildcard": { "process.command_line": "*invoke-mimikatz*" }}
      ],
      "minimum_should_match": 1
    }
  }
}

// 4. Data Exfiltration
{
  "query": {
    "bool": {
      "must": [
        { "match": { "network.direction": "outbound" }},
        { "range": { "network.bytes": { "gte": 100000000 }}}
      ]
    }
  },
  "aggs": {
    "by_destination": {
      "terms": {
        "field": "destination.ip",
        "order": { "total_bytes": "desc" }
      },
      "aggs": {
        "total_bytes": {
          "sum": {
            "field": "network.bytes"
          }
        }
      }
    }
  }
}
```

[Continuing with more forensics topics and interview questions...]

## Additional Interview Questions & Answers

### Question 9: Explain your approach to investigating an insider threat case.

**Model Answer:**

"Insider threat investigations are particularly sensitive because they involve employees and require balancing investigation with privacy and legal considerations. Here's my systematic approach:

**Phase 1: Initial Assessment & Authorization (Day 1)**

Before any technical work, I ensure proper authorization:
- Written approval from legal department
- HR involvement and guidance
- Clear scope definition
- Privacy law compliance (especially EU/GDPR)
- Union notification if applicable

Document the allegations:
- What suspicious activity was reported?
- Who reported it and when?
- What data/systems are at risk?
- Is the employee still active?
- Is there imminent risk?

**Phase 2: Covert Data Collection (Days 1-3)**

The key is collecting evidence without alerting the suspect:

```powershell
# Covert evidence collection checklist

# 1. Email Analysis
# DON'T access their inbox directly (alerts them)
# DO work with email admin to export mailbox

# Export user's mailbox (O365 example)
New-ComplianceSearch -Name "InvestigationCase001" `
    -ExchangeLocation user@company.com `
    -ContentMatchQuery "sent>=$(Get-Date).AddDays(-90)"

Start-ComplianceSearch -Identity "InvestigationCase001"
New-ComplianceSearchAction -SearchName "InvestigationCase001" -Export

# 2. File Access Logs
# Check what files they accessed recently
# SharePoint/OneDrive activity
Search-UnifiedAuditLog -UserIds user@company.com `
    -StartDate (Get-Date).AddDays(-90) `
    -EndDate (Get-Date) `
    -RecordType SharePointFileOperation

# 3. Network Activity
# Work with network team to collect:
# - Proxy logs (what websites visited)
# - Firewall logs (what IPs contacted)
# - VPN logs (when/where connected from)
# - Email gateway logs (attachments sent externally)

# 4. Endpoint Forensics
# If EDR deployed, pull:
# - Process execution history
# - File operations
# - USB device usage
# - Print jobs
# - Screen recordings (if policy allows)
```

**Artifacts to Collect:**

1. **Email Evidence**:
   - Sent items (especially to personal email)
   - Deleted items
   - Auto-forwarding rules
   - Email to competitors
   - Large attachments sent externally

2. **File Activity**:
   - Files accessed from sensitive directories
   - Files copied to USB/external drives
   - Files uploaded to cloud storage
   - Files compressed into archives
   - Files renamed/moved

3. **Network Activity**:
   - Websites visited (cloud storage, job boards)
   - Large data uploads
   - Connections to competitor IPs
   - Personal email access during work hours
   - VPN usage from unusual locations

4. **Endpoint Activity**:
   - USB device connections
   - Print job history
   - Screenshot/screen recording (if enabled)
   - Application usage
   - Clipboard history

**Phase 3: Timeline Analysis (Days 3-5)**

Build comprehensive timeline of suspect's activities:

```python
# Timeline Analysis Script
import pandas as pd
from datetime import datetime

class InsiderThreatTimeline:
    def __init__(self):
        self.events = []
    
    def add_email_event(self, timestamp, action, recipient, subject, attachments):
        self.events.append({
            'timestamp': timestamp,
            'category': 'Email',
            'action': action,
            'detail': f'To: {recipient}, Subject: {subject}, Attachments: {attachments}'
        })
    
    def add_file_event(self, timestamp, action, file_path, destination):
        self.events.append({
            'timestamp': timestamp,
            'category': 'File',
            'action': action,
            'detail': f'File: {file_path}, Destination: {destination}'
        })
    
    def add_usb_event(self, timestamp, device_name, files_copied):
        self.events.append({
            'timestamp': timestamp,
            'category': 'USB',
            'action': 'Device Connected',
            'detail': f'Device: {device_name}, Files: {files_copied}'
        })
    
    def add_web_event(self, timestamp, url, action):
        self.events.append({
            'timestamp': timestamp,
            'category': 'Web',
            'action': action,
            'detail': f'URL: {url}'
        })
    
    def generate_timeline(self):
        df = pd.DataFrame(self.events)
        df = df.sort_values('timestamp')
        return df
    
    def detect_patterns(self):
        """Identify suspicious patterns"""
        
        df = self.generate_timeline()
        
        patterns = {
            'data_exfiltration': [],
            'competitor_contact': [],
            'unusual_hours': [],
            'personal_email_usage': []
        }
        
        # Pattern 1: Multiple file copies followed by USB or email
        # Pattern 2: Access to competitor websites
        # Pattern 3: Activity outside normal hours
        # Pattern 4: Large emails to personal accounts
        
        return patterns

# Example usage:
timeline = InsiderThreatTimeline()

# Employee accessed sensitive documents
timeline.add_file_event(
    '2024-01-15 09:00:00',
    'Accessed',
    'Q4_Financial_Results.xlsx',
    'Local Disk'
)

# Copied to USB
timeline.add_usb_event(
    '2024-01-15 09:05:00',
    'Kingston DataTraveler',
    'Q4_Financial_Results.xlsx'
)

# Sent to personal email
timeline.add_email_event(
    '2024-01-15 09:10:00',
    'Sent',
    'personal@gmail.com',
    'Backup files',
    ['Q4_Financial_Results.xlsx']
)

# Visited competitor website
timeline.add_web_event(
    '2024-01-15 14:30:00',
    'https://competitor-careers.com/apply',
    'Visited'
)

df = timeline.generate_timeline()
print(df)
```

**Key Patterns to Look For:**

1. **Pre-Departure Indicators**:
   - Increased file access before resignation
   - Downloads of customer lists
   - Access to strategic documents
   - Copying of source code
   - LinkedIn profile updates
   - Resume updates on job sites
   - Contact with competitors

2. **Data Staging**:
   - Creating compressed archives
   - Copying files to personal folders
   - Moving files to accessible locations
   - Organizing files in systematic way

3. **Exfiltration Methods**:
   - Email to personal accounts
   - USB device usage
   - Cloud storage uploads
   - Printing documents
   - Mobile device screenshots
   - Photos of screens

**Phase 4: Evidence Preservation (Ongoing)**

Maintain strict forensic procedures:

```bash
# Forensic imaging of employee workstation
# Do this after hours or when employee is on leave

# 1. Create forensic image
dcfldd if=/dev/sda of=/mnt/evidence/employee_laptop.dd \
       hash=md5 hash=sha256 \
       hashlog=/mnt/evidence/employee_laptop.hash

# 2. Verify image
md5sum /mnt/evidence/employee_laptop.dd

# 3. Document chain of custody
# Who: Forensic Examiner John Doe
# What: Dell Latitude 7420, Serial: ABC123
# When: 2024-01-20 22:00 UTC
# Where: Building A, Cubicle 42
# Why: Insider threat investigation CASE-2024-001
# How: Write-blocked forensic imaging

# 4. Preserve in secure location
# Encrypted storage, access-controlled
# Multiple copies (different locations)
```

**Phase 5: Analysis and Correlation (Days 5-10)**

Analyze collected evidence:

1. **Email Analysis**:
   - Search for keywords: "confidential", "competitor", "opportunity"
   - Look for forwarding to personal email
   - Check deleted items
   - Identify sensitive attachments

2. **File Forensics**:
   - Identify accessed sensitive files
   - Check USB device history in registry
   - Analyze $USNJrnl for file operations
   - Check cloud sync folders

3. **Browser Forensics**:
   - History of job sites
   - Competitor websites
   - Cloud storage access
   - Personal email access

**Phase 6: Interview Preparation (Days 10-12)**

Prepare for HR interview:

Document evidence:
- Timeline of activities
- Screenshots of evidence
- List of files accessed/copied
- Network connections made
- Policy violations identified

Red flags to present:
```markdown
Evidence Summary for HR:

1. Data Access
   - Accessed 247 customer records (Jan 10-15)
   - Downloaded entire customer database (Jan 14)
   - Accessed engineering designs (Jan 15)

2. Data Exfiltration
   - Copied 15GB to USB device (Jan 15, 4:30 PM)
   - Emailed 45 files to personal account (Jan 10-15)
   - Uploaded files to Dropbox (Jan 14)

3. External Contact
   - Applied to competitor on Jan 12
   - LinkedIn messages with competitor employees
   - Website visits to 3 competitors

4. Policy Violations
   - USB policy: Violated (unauthorized device)
   - Email policy: Violated (sensitive data to personal email)
   - Data handling: Violated (unauthorized copying)

Timeline of Intent:
Jan 10: Applied to competitor
Jan 12: Increased access to sensitive files
Jan 14: Downloaded customer database
Jan 15: Copied to USB and personal email
Jan 16: Submitted resignation
```

**Phase 7: Legal Coordination**

Work with legal team:

Legal considerations:
- Is evidence admissible?
- Were privacy laws followed?
- Can we prove intent?
- Criminal vs. civil action?
- Cease and desist letter?
- Injunction to prevent disclosure?

Evidence requirements:
- Forensically sound collection
- Documented chain of custody
- No illegal surveillance
- Compliance with employment laws
- Proper authorization obtained

**Phase 8: Remediation (Post-Investigation)**

After investigation concludes:

1. **Immediate Actions**:
   - Disable all accounts
   - Revoke VPN/remote access
   - Change passwords for shared accounts
   - Remote wipe company mobile device
   - Collect company laptop

2. **Data Protection**:
   - Notify customers (if PII exposed)
   - Monitor for data appearing on dark web
   - Send cease and desist to employee
   - Contact new employer if appropriate

3. **Prevention**:
   - Implement DLP (Data Loss Prevention)
   - Enhance USB device controls
   - Monitor high-risk employees (post-resignation)
   - Improve access controls
   - Regular access reviews

**Phase 9: Reporting**

Two reports required:

1. **Technical Report** (for legal/security):
   - Complete timeline
   - All evidence with screenshots
   - Methods used
   - Chain of custody
   - Expert analysis

2. **Executive Summary** (for management):
   - What happened
   - What data was taken
   - Business impact
   - Recommended actions
   - Prevention measures

**Key Challenges and Solutions:**

**Challenge 1: Privacy Laws**
- Solution: Get legal approval first, document everything

**Challenge 2: Alerting the Suspect**
- Solution: Covert collection, after-hours imaging

**Challenge 3: Encrypted Devices**
- Solution: Capture while device is on/unlocked, legal compulsion

**Challenge 4: Cloud Data**
- Solution: Work with SaaS providers, legal data requests

**Challenge 5: Proof of Intent**
- Solution: Show pattern of behavior, timing with job search

**Red Flags I Watch For:**

Early indicators:
- Performance reviews declining
- Passed over for promotion
- Conflict with management
- Financial difficulties
- Contact with competitors
- LinkedIn activity increase

Immediate indicators:
- Resignation submitted
- Large file access spike
- USB usage increase
- Off-hours activity
- VPN from unusual locations
- Deleting browser history

**Expert Tips:**

1. **Speed vs. Stealth**: Balance thorough investigation with timely action
2. **Legal First**: Always involve legal before technical work
3. **Document Everything**: Assume you'll testify in court
4. **Multiple Evidence Sources**: Corroborate across email, files, network
5. **Context Matters**: Distinguish between legitimate and malicious activity
6. **HR Partnership**: Work closely with HR throughout

**Expected Outcomes:**

Based on evidence strength:
- **Strong evidence**: Termination, legal action, criminal charges
- **Moderate evidence**: Termination, cease and desist
- **Weak evidence**: Warning, enhanced monitoring
- **Inconclusive**: Close investigation, document findings

**Key Metrics:**

Success measurement:
- Data recovered: 100% of stolen files identified
- Time to detection: 72 hours from initial activity
- Evidence quality: Court-admissible
- Business impact: Prevented disclosure to competitor
- Prevention: DLP implemented preventing recurrence

The key to insider threat investigations is balancing thorough investigation with employee privacy, maintaining legal defensibility, and acting quickly enough to prevent or minimize data loss."

### Question 10: How do you conduct forensics on a virtual machine or cloud instance?

**Model Answer:**

"Cloud and VM forensics present unique challenges due to ephemeral resources, multi-tenancy, and limited physical access. Here's my comprehensive approach:

**Challenge 1: Evidence Volatility**

Unlike physical machines that stay intact when powered off, cloud instances can be:
- Auto-scaled down (destroyed automatically)
- Snapshots deleted
- Logs rotated
- IP addresses reassigned

**My immediate response**:

```python
#!/usr/bin/env python3
"""
AWS EC2 Incident Response - Quick Actions
Execute IMMEDIATELY when compromise detected
"""

import boto3
from datetime import datetime

def emergency_preserve_instance(instance_id, region='us-east-1'):
    """
    Immediate preservation actions for compromised instance
    """
    
    ec2 = boto3.client('ec2', region_name=region)
    
    print(f"[*] Emergency preservation for {instance_id}")
    
    # 1. Tag instance immediately
    ec2.create_tags(
        Resources=[instance_id],
        Tags=[
            {'Key': 'IncidentResponse', 'Value': 'True'},
            {'Key': 'PreservationTime', 'Value': datetime.now().isoformat()},
            {'Key': 'CaseNumber', 'Value': 'CASE-2024-001'},
            {'Key': 'Status', 'Value': 'DO-NOT-TERMINATE'}
        ]
    )
    print("[+] Instance tagged for preservation")
    
    # 2. Disable termination protection
    ec2.modify_instance_attribute(
        InstanceId=instance_id,
        DisableApiTermination={'Value': True}
    )
    print("[+] Termination protection enabled")
    
    # 3. Snapshot all volumes IMMEDIATELY
    # Get all volumes
    instance = ec2.describe_instances(InstanceIds=[instance_id])
    volumes = instance['Reservations'][0]['Instances'][0]['BlockDeviceMappings']
    
    snapshots = []
    for volume in volumes:
        volume_id = volume['Ebs']['VolumeId']
        
        snapshot = ec2.create_snapshot(
            VolumeId=volume_id,
            Description=f'Emergency IR snapshot - {instance_id} - {datetime.now()}',
            TagSpecifications=[{
                'ResourceType': 'snapshot',
                'Tags': [
                    {'Key': 'IncidentResponse', 'Value': 'True'},
                    {'Key': 'SourceInstance', 'Value': instance_id},
                    {'Key': 'CaseNumber', 'Value': 'CASE-2024-001'},
                    {'Key': 'PreservationTime', 'Value': datetime.now().isoformat()}
                ]
            }]
        )
        
        snapshots.append(snapshot['SnapshotId'])
        print(f"[+] Snapshot created: {snapshot['SnapshotId']}")
    
    # 4. Capture instance metadata
    metadata = {
        'instance_id': instance_id,
        'instance_type': instance['Reservations'][0]['Instances'][0]['InstanceType'],
        'ami_id': instance['Reservations'][0]['Instances'][0]['ImageId'],
        'launch_time': instance['Reservations'][0]['Instances'][0]['LaunchTime'].isoformat(),
        'private_ip': instance['Reservations'][0]['Instances'][0].get('PrivateIpAddress'),
        'public_ip': instance['Reservations'][0]['Instances'][0].get('PublicIpAddress'),
        'security_groups': [sg['GroupId'] for sg in instance['Reservations'][0]['Instances'][0]['SecurityGroups']],
        'snapshots': snapshots,
        'preservation_time': datetime.now().isoformat()
    }
    
    # 5. Export metadata
    import json
    with open(f'instance_{instance_id}_metadata.json', 'w') as f:
        json.dump(metadata, f, indent=2)
    
    print("[+] Instance metadata preserved")
    print(f"[+] Created {len(snapshots)} snapshots")
    
    return metadata

# Execute immediately on detection
emergency_preserve_instance('i-1234567890abcdef0')
```

**Phase 1: Triage (0-30 minutes)**

Determine investigation approach:

```markdown
## VM/Cloud Forensics Decision Tree

### Is instance still running?
YES → Capture memory first, then disk
NO → Skip to disk forensics

### Can we afford downtime?
YES → Take instance offline for forensic imaging
NO → Live forensics, snapshot while running

### Is it cloud-managed (AWS/Azure/GCP)?
YES → Use cloud-native tools (snapshots, APIs)
NO → Traditional VM forensics (VMware, Hyper-V)

### Do we have admin access?
YES → Full forensic acquisition
NO → Work with cloud provider, legal subpoena
```

**Phase 2: Memory Capture (30-60 minutes)**

For running instances, memory first:

```bash
#!/bin/bash
# Cloud Instance Memory Capture

INSTANCE_IP="10.0.1.50"
SSH_KEY="/path/to/forensics-key.pem"

echo "[*] Capturing memory from running instance"

# 1. Upload memory acquisition tool
scp -i $SSH_KEY avml ubuntu@$INSTANCE_IP:/tmp/

# 2. Execute memory capture
ssh -i $SSH_KEY ubuntu@$INSTANCE_IP "sudo /tmp/avml /tmp/memory.lime"

# 3. Calculate hash on remote system
ssh -i $SSH_KEY ubuntu@$INSTANCE_IP "sudo sha256sum /tmp/memory.lime"

# 4. Download memory dump
scp -i $SSH_KEY ubuntu@$INSTANCE_IP:/tmp/memory.lime ./evidence/

# 5. Verify locally
sha256sum ./evidence/memory.lime

# 6. Clean up remote system (optional, depends on case)
# ssh -i $SSH_KEY ubuntu@$INSTANCE_IP "sudo shred -vfz /tmp/memory.lime /tmp/avml"

echo "[+] Memory capture complete"
```

**Phase 3: Disk Acquisition (1-3 hours)**

Different approaches based on platform:

**AWS EC2:**
```python
def acquire_aws_disk(instance_id, region='us-east-1'):
    """
    Forensic disk acquisition from AWS EC2
    """
    ec2 = boto3.client('ec2', region_name=region)
    
    # Method 1: Snapshot-based (preferred - no downtime)
    print("[*] Method 1: Snapshot-based acquisition")
    
    # Get root volume
    instance = ec2.describe_instances(InstanceIds=[instance_id])
    root_volume = instance['Reservations'][0]['Instances'][0]['BlockDeviceMappings'][0]['Ebs']['VolumeId']
    
    # Create snapshot
    snapshot = ec2.create_snapshot(
        VolumeId=root_volume,
        Description=f'Forensic acquisition - {instance_id}'
    )
    snapshot_id = snapshot['SnapshotId']
    
    # Wait for snapshot completion
    waiter = ec2.get_waiter('snapshot_completed')
    print(f"[*] Waiting for snapshot {snapshot_id} to complete...")
    waiter.wait(SnapshotIds=[snapshot_id])
    
    # Create volume from snapshot in forensics account/VPC
    volume = ec2.create_volume(
        SnapshotId=snapshot_id,
        AvailabilityZone=f'{region}a',
        VolumeType='gp3',
        TagSpecifications=[{
            'ResourceType': 'volume',
            'Tags': [
                {'Key': 'Purpose', 'Value': 'Forensic Evidence'},
                {'Key': 'SourceInstance', 'Value': instance_id}
            ]
        }]
    )
    
    volume_id = volume['VolumeId']
    
    # Attach to forensic workstation
    # (Launch forensic workstation first if not exists)
    forensic_instance = launch_forensic_workstation(region)
    
    ec2.attach_volume(
        VolumeId=volume_id,
        InstanceId=forensic_instance,
        Device='/dev/sdf'
    )
    
    print(f"[+] Evidence volume {volume_id} attached to forensic workstation")
    print(f"[+] SSH to {forensic_instance} and mount read-only:")
    print(f"    sudo mount -o ro,noload /dev/sdf1 /mnt/evidence")
    
    return {
        'snapshot_id': snapshot_id,
        'volume_id': volume_id,
        'forensic_instance': forensic_instance
    }

def launch_forensic_workstation(region):
    """
    Launch SIFT workstation for analysis
    """
    ec2 = boto3.client('ec2', region_name=region)
    
    # Use SIFT AMI (or custom forensics AMI)
    sift_ami = 'ami-xxxxxxxxx'  # Replace with actual SIFT AMI
    
    instance = ec2.run_instances(
        ImageId=sift_ami,
        InstanceType='m5.2xlarge',
        MinCount=1,
        MaxCount=1,
        KeyName='forensics-key',
        SecurityGroupIds=['sg-forensics'],
        SubnetId='subnet-forensics',
        TagSpecifications=[{
            'ResourceType': 'instance',
            'Tags': [
                {'Key': 'Purpose', 'Value': 'Forensic Analysis'},
                {'Key': 'AutoShutdown', 'Value': 'False'}
            ]
        }]
    )
    
    return instance['Instances'][0]['InstanceId']
```

**Azure:**
```bash
# Azure disk forensics

# 1. Create snapshot
az snapshot create \
    --resource-group production-rg \
    --name forensic-snapshot-$(date +%Y%m%d) \
    --source /subscriptions/.../virtualMachines/vm-name/disks/os-disk

# 2. Create disk from snapshot
az disk create \
    --resource-group forensics-rg \
    --name forensic-disk-$(date +%Y%m%d) \
    --source forensic-snapshot-$(date +%Y%m%d)

# 3. Attach to forensic VM
az vm disk attach \
    --resource-group forensics-rg \
    --vm-name forensic-workstation \
    --name forensic-disk-$(date +%Y%m%d)

# 4. Mount read-only on forensic VM
# SSH to forensic VM
sudo mount -o ro /dev/sdc1 /mnt/evidence
```

**VMware:**
```bash
# VMware VM forensics

# 1. Suspend VM (preserves memory state)
vim-cmd vmsvc/power.suspend <vmid>

# 2. Copy VMDK files
cp /vmfs/volumes/datastore1/vm-name/vm-name.vmdk /mnt/evidence/
cp /vmfs/volumes/datastore1/vm-name/vm-name-flat.vmdk /mnt/evidence/

# 3. Copy VMSS (suspended state - contains memory)
cp /vmfs/volumes/datastore1/vm-name/vm-name.vmss /mnt/evidence/

# 4. Calculate hashes
sha256sum /mnt/evidence/*

# 5. Convert VMDK to raw for analysis (optional)
qemu-img convert -f vmdk -O raw vm-name.vmdk vm-name.dd
```

**Phase 4: Log Collection (ongoing)**

Cloud platforms generate extensive logs:

```python
def collect_aws_logs(instance_id, days_back=30):
    """
    Collect all relevant AWS logs for instance
    """
    
    cloudtrail = boto3.client('cloudtrail')
    cloudwatch = boto3.client('logs')
    
    logs = {
        'cloudtrail': [],
        'vpc_flow': [],
        'system_logs': []
    }
    
    # 1. CloudTrail (API calls)
    print("[*] Collecting CloudTrail logs...")
    
    paginator = cloudtrail.get_paginator('lookup_events')
    for page in paginator.paginate(
        LookupAttributes=[{
            'AttributeKey': 'ResourceName',
            'AttributeValue': instance_id
        }],
        StartTime=datetime.now() - timedelta(days=days_back),
        EndTime=datetime.now()
    ):
        logs['cloudtrail'].extend(page['Events'])
    
    print(f"[+] Collected {len(logs['cloudtrail'])} CloudTrail events")
    
    # 2. VPC Flow Logs
    print("[*] Collecting VPC Flow Logs...")
    # Query CloudWatch Logs for VPC Flow Logs
    # (requires VPC Flow Logs to be enabled)
    
    # 3. System Logs (if CloudWatch agent installed)
    print("[*] Collecting system logs from CloudWatch...")
    
    return logs
```

**Phase 5: Analysis Differences**

Cloud forensics requires different analysis approach:

```markdown
## Cloud-Specific Artifacts

### AWS-Specific:
- **CloudTrail**: API call history (who did what, when)
- **VPC Flow Logs**: Network traffic metadata
- **CloudWatch Logs**: Application and system logs
- **Config**: Configuration change history
- **GuardDuty**: Threat detection findings
- **IAM**: Access keys, roles, policies
- **S3 Access Logs**: Data access patterns
- **ELB Logs**: Load balancer traffic

### Azure-Specific:
- **Activity Log**: Management operations
- **NSG Flow Logs**: Network security group traffic
- **Diagnostic Logs**: Resource-specific logs
- **Security Center**: Security findings
- **Sentinel**: SIEM data if enabled

### GCP-Specific:
- **Cloud Audit Logs**: Admin, data access, system events
- **VPC Flow Logs**: Network traffic
- **Cloud Logging**: All logs aggregation
- **Security Command Center**: Security findings
```

**Phase 6: Challenges and Solutions**

**Challenge 1: Encryption**
```markdown
Problem: EBS volumes encrypted at rest
Solution: 
- Snapshot preserves encryption
- Analyze on instance with same KMS key access
- Decrypt using customer master key
- Memory dump may contain decrypted data
```

**Challenge 2: Ephemeral Storage**
```markdown
Problem: Instance store (ephemeral) lost on shutdown
Solution:
- NEVER stop instance with instance store
- Live forensics only
- Snapshot during operation
- Accept data loss if must stop
```

**Challenge 3: Auto-Scaling**
```markdown
Problem: Instance auto-terminated by scaling policy
Solution:
- Immediately disable auto-scaling
- Set instance to "protected from scale-in"
- Tag instance to exclude from termination
- Detach from auto-scaling group
```

**Challenge 4: Shared Tenancy**
```markdown
Problem: Can't access hypervisor or other VMs
Solution:
- Use cloud provider APIs exclusively
- Request provider assistance for deep forensics
- Legal subpoena if provider cooperation needed
- Accept limitations vs. physical control
```

**Phase 7: Reporting Differences**

Cloud forensics reports include unique sections:

```markdown
## Cloud Forensics Report Additions

### Infrastructure Details:
- Cloud Provider: AWS
- Region: us-east-1
- Account ID: 123456789012
- Instance ID: i-1234567890abcdef0
- VPC ID: vpc-12345678
- Subnet: subnet-12345678
- Security Groups: sg-12345678

### Access Control:
- IAM Role: instance-role
- IAM Policies: ListPolicies
- Security Group Rules: Inbound/Outbound
- NACL Rules: Network ACL
- Instance Profile: ARN

### Forensic Actions Taken:
- Snapshots Created: snap-12345678, snap-23456789
- Termination Protection: Enabled
- Isolation Applied: SG sg-isolation
- Logs Collected: CloudTrail, VPC Flow, CloudWatch

### Cloud-Specific Findings:
- Unauthorized API calls detected in CloudTrail
- GuardDuty alert: Cryptocurrency mining
- Config non-compliance: Security group too permissive
- Unusual data transfer to external IP
```

**Best Practices for Cloud Forensics:**

1. **Speed is Critical**:
   - Snapshot immediately
   - Instances can be terminated any time
   - Logs rotate quickly

2. **Use Native Tools**:
   - Cloud provider APIs
   - Native snapshot features
   - Managed logging services

3. **Preserve Metadata**:
   - Tags, security groups, IAM roles
   - Network configuration
   - CloudFormation templates

4. **Understand Shared Responsibility**:
   - Provider secures infrastructure
   - Customer secures instances and data
   - Know what you control vs. what provider controls

5. **Plan Ahead**:
   - Enable logging BEFORE incident
   - Practice snapshot procedures
   - Establish forensics accounts/VPCs
   - Document runbooks

The key to cloud forensics is acting quickly with native tools, understanding the shared responsibility model, and accepting that you won't have the same level of physical control as traditional forensics."

[Continuing with more content...]

---

**Series navigation:** [🛡️ Landing page](/posts/cybersecurity_mastery_landing_page/) · [📑 Table of contents](/posts/cybersecurity_mastery_toc/)

- Previous: [← Cybersecurity Mastery - Part 2: Offensive Security & Ethical Hacking](/posts/cybersecurity_mastery_part2_offensive_security_ethical_hacking/)
- Next: [Cybersecurity Mastery - Part 4: Advanced Topics & Emerging Technologies →](/posts/cybersecurity_mastery_part4_advanced_topics_emerging_tech/)
