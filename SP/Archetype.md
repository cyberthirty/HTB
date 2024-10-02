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






# Exploring SMB Shares with smbclient

In this post, we'll explore the use of `smbclient` to connect to SMB shares and the resulting interactions with a Windows system.

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
