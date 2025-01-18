
# Nmap Cheatsheet for Penetration Testing

## Basic Commands

- `nmap <target>`  
  Simple scan to discover open ports on the target.

- `nmap -sP <target>`  
  Ping scan to check if the target is alive.

- `nmap -v <target>`  
  Verbose output for scanning details.

- `nmap -A <target>`  
  Aggressive scan to detect OS, version, script scanning, and traceroute.

- `nmap -T4 <target>`  
  Set timing template to 4 (faster scan).

- `nmap -C <target>`  
This option in Nmap enables the use of "Canonical" scripts.  

## Port Scanning

- `nmap -p <ports> <target>`  
  Scan specific ports (e.g., `nmap -p 80,443 <target>`).

- `nmap -p- <target>`  
  Scan all 65535 ports.

- `nmap -p 1-1000 <target>`  
  Scan ports from 1 to 1000.

- `nmap -p U:53,T:21-80 <target>`  
  Scan specific UDP and TCP ports (e.g., UDP port 53 and TCP ports 21-80).

## OS Detection and Service Version

- `nmap -O <target>`  
  Detect the operating system of the target.

- `nmap -sV <target>`  
  Service version detection to find the version of services running on open ports.

- `nmap -A <target>`  
  Aggressive scan that includes OS detection, version detection, script scanning, and traceroute.

## Scan Techniques

- `nmap -sS <target>`  
  TCP SYN scan (stealth scan, works by sending SYN packets).

- `nmap -sT <target>`  
  TCP connect scan (completes the handshake).

- `nmap -sU <target>`  
  UDP scan.

- `nmap -sN <target>`  
  TCP Null scan (sends a packet with no flags set).

- `nmap -sF <target>`  
  TCP FIN scan (finishes the connection without fully opening it).

- `nmap -sX <target>`  
  TCP Xmas scan (sets FIN, PSH, and URG flags).

## Firewall Evasion

- `nmap -f <target>`  
  Fragment packets to avoid detection by firewalls.

- `nmap -D RND:10 <target>`  
  Decoy scan using 10 random IP addresses to obfuscate the source.

- `nmap --source-port <port> <target>`  
  Set a custom source port for the scan.

## Service and Version Detection

- `nmap -sV -p <port> <target>`  
  Detect version of a specific service.

- `nmap --script <script> <target>`  
  Use NSE scripts for detailed service scanning.

## Output Options

- `nmap -oN <output-file> <target>`  
  Output to a normal file.

- `nmap -oX <output-file> <target>`  
  Output to XML format.

- `nmap -oG <output-file> <target>`  
  Output to grepable format.

- `nmap -oA <basename> <target>`  
  Output in all formats (normal, XML, and grepable) with a base filename.

## Example Scans

- Scan a target for all open ports:  
  `nmap -p- 192.168.1.1`

- Scan a range of ports on a target:  
  `nmap -p 22,80,443 192.168.1.1`

- Perform a SYN scan and OS detection on a target:  
  `nmap -sS -O 192.168.1.1`

- Perform an aggressive scan with service version detection and script scanning:  
  `nmap -A 192.168.1.1`

- Scan for vulnerabilities using NSE scripts:  
  `nmap --script=vuln 192.168.1.1`

## Nmap Scripting Engine (NSE)

- `nmap --script=<script-name> <target>`  
  Run a specific script (e.g., `nmap --script=http-vuln-cve2006-3392 <target>`).

- `nmap --script <category> <target>`  
  Run a category of scripts (e.g., `nmap --script=http <target>` for HTTP related scripts).

## Timing and Performance

- `nmap -T0 <target>`  
  Slow scan (useful for stealth).

- `nmap -T5 <target>`  
  Very fast scan (use with caution).

## DNS and IP Geolocation

- `nmap --dns-servers <dns> <target>`  
  Use specific DNS servers for name resolution.

- `nmap --source-ip <ip> <target>`  
  Use a specific IP address for the scan.

- `nmap --max-retries <count> <target>`  
  Limit the number of retries for a given scan.

