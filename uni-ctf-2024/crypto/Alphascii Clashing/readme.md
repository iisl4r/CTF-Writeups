
![](/CTF-Writeups/assets/images/banner.png)
# Alphascii Clashing Write-up

## Challenge Overview

In this challenge, we are given a login application that stores usernames and passwords in a dictionary, with the usernames hashed using the MD5 algorithm. The server performs the following checks during login:

1. It hashes the entered username and compares it against the stored hash.
2. If both the username hash and password match, the user is logged in.
3. If the username hash matches, but the username itself doesn't match the stored username, the system prints a message that exposes the flag and shuts down.

The task is to exploit the MD5 hash collision vulnerability to bypass the username comparison and print the flag.

## Key Insights

The challenge uses **MD5**, which is known to be a cryptographically weak hash function susceptible to **hash collisions**. A hash collision occurs when two different inputs produce the same hash output. In this case, the goal is to register a user with one string, but then log in using another string that has the same MD5 hash but is different from the original username. This will cause the `usr == db_user` condition to fail, triggering the `else` block, which prints the flag.

## The Code Logic

The provided Python code works as follows:

```python
from hashlib import md5
import json

'''
Data format:
{
    username: [md5(username).hexdigest(), password],
    .
    .
    .
}
'''

users = {
    'HTBUser132' : [md5(b'HTBUser132').hexdigest(), 'secure123!'],
    'JohnMarcus' : [md5(b'JohnMarcus').hexdigest(), '0123456789']
}

def get_option():
    return input('''Welcome to my login application scaredy cat! I am using MD5 to save the passwords in the database.
    I am more than certain that this is secure.
    You can't prove me wrong!

    [1] Login
    [2] Register
    [3] Exit

    Option (json format) :: ''')

def main():
    while True:
        option = json.loads(get_option())

        if 'option' not in option:
            print('[-] please, enter a valid option!')
            continue

        option = option['option']
        if option == 'login':
            creds = json.loads(input('enter credentials (json format) :: '))

            usr, pwd = creds['username'], creds['password']
            usr_hash = md5(usr.encode()).hexdigest()
            for db_user, v in users.items():
                if [usr_hash, pwd] == v:
                    if usr == db_user:
                        print(f'[+] welcome, {usr} 🤖!')
                    else:
                        print(f"[+] what?! this was unexpected. shutting down the system :: {open('flag.txt').read()} 👽")
                        exit()
                    break
            else:
                print('[-] invalid username and/or password!')

        elif option == 'register':
            creds = json.loads(input('enter credentials (json format) :: '))

            usr, pwd = creds['username'], creds['password']
            if usr.isalnum() and pwd.isalnum():
                usr_hash = md5(usr.encode()).hexdigest()
                if usr not in users.keys():
                    users[usr] = [md5(usr.encode()).hexdigest(), pwd]
                else:
                    print('[-] this user already exists!')
            else:
                print('[-] your credentials must contain only ASCII letters and digits.')

        elif option == 'exit':
            print('byeee.')
            break

if __name__ == '__main__':
    main()
```

## The Exploit

In the code, the system checks whether the hash of the provided username (`usr_hash`) matches the stored hash. If they match, the system then checks if the `usr` (username) matches the `db_user`. If the usernames don't match but the hash matches, the system prints the flag.

- **Register with Username 1**: Register a new username (let's call it `username_1`) with a simple password (`htb`).
- **Login with Username 2**: Attempt to log in using another string (`username_2`) that produces the same MD5 hash as `username_1`. This causes the `usr == db_user` check to fail, triggering the flag print.

## Solution Code

```python
import json
from pwn import *

# Two different usernames that cause an MD5 collision
username1 = "TEXTCOLLBYfGiJUETHQ4hAcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak"
username2 = "TEXTCOLLBYfGiJUETHQ4hEcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak"

# Connect to the remote server
server = remote("83.136.250.185", 43865)

# Step 1: Register the first user (username1)
print(server.sendlineafter(b":: ", json.dumps({"option": "register"})).decode())
print(server.sendlineafter(b":: ", json.dumps({"username": username1, "password": "htb"})).decode())

# Step 2: Login as the second user (username2), which has the same MD5 hash as username1
print(server.sendlineafter(b":: ", json.dumps({"option": "login"})).decode())
print(server.sendlineafter(b":: ", json.dumps({"username": username2, "password": "htb"})).decode())

# Step 3: Receive and print the output (should contain the flag if successful)
print(server.recvall().decode())
```

**Flag:** `HTB{finding_alphanumeric_md5_collisions_for_fun_https://x.com/realhashbreaker/status/1770161965006008570_73cd928afdb3968a7efdc6954fc95bca}`


## Explanation of the Code

- **Register Step**:
   - First, we send a `"register"` option and register `username_1` with the password `htb`.
- **Login Step**:
   - After registration, we send a `"login"` option and attempt to log in using `username_2`, which has the same MD5 hash as `username_1`.
   - The system checks the hashes first, and since they match, it proceeds to compare the usernames. Since the usernames differ, it triggers the `else` condition, printing the flag and shutting down the system.

## Conclusion

By exploiting MD5 hash collisions, we are able to bypass the username verification step and retrieve the flag. This vulnerability highlights the weaknesses in using cryptographically broken hash functions like MD5, especially for critical security operations like user authentication.

