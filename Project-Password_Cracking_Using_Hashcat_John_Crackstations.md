**Cryptography**

Cryptography is used to protect confidentiality, integrity, and authenticity.


**Consider the following scenarios where you would use cryptography:**

When you log in to Google.com, your credentials are encrypted and sent to the server so that no one can retrieve them by snooping on your connection.

When you connect over SSH, your SSH client and the server establish an encrypted tunnel so no one can eavesdrop on your session.

When you conduct online banking, your browser checks the remote server’s certificate to confirm that you are communicating with your bank’s server and not an attacker’s.

When you download a file, how do you check if it was downloaded correctly? Cryptography provides a solution through hash functions to confirm that your file is identical to the original one.


**Cryptography Working**

**Plaintext** is the original, readable message or data before it’s encrypted. It can be a document, an image, a multimedia file, or any other binary data.

**Ciphertext** is the scrambled, unreadable version of the message after encryption. Ideally, we cannot get any information about the original plaintext except its approximate size.

**Cipher** is an algorithm or method to convert plaintext into ciphertext and back again. A cipher is usually developed by a mathematician.

**Key** is a string of bits the cipher uses to encrypt or decrypt data. In general, the used cipher is public knowledge; however, the key must remain secret unless it is the public key in asymmetric encryption. We will visit asymmetric encryption in a later task.

**Encryption** is the process of converting plaintext into ciphertext using a cipher and a key. Unlike the key, the choice of the cipher is disclosed.

**Decryption** is the reverse process of encryption, converting ciphertext back into plaintext using a cipher and a key. Although the cipher would be public knowledge, recovering the plaintext without knowledge of the key should be impossible (infeasible).


**Types of Encryption**

1. **Symmetric encryption,** also known as symmetric cryptography, uses the same key to encrypt and decrypt the data, as shown in the figure below. Keeping the key secret is a must; it is also called private key cryptography.

2. **Asymmetric Encryption,** which uses the same key for encryption and decryption, asymmetric encryption uses a pair of keys, one to encrypt and the other to decrypt. To protect confidentiality, asymmetric encryption or asymmetric cryptography encrypts the data using the public key; hence, it is also called public key cryptography.

3. **Hash Function**

**What is a Hash Function?**


Hash functions are different from encryption. There is no key, and it’s meant to be impossible (or computationally impractical) to go from the output back to the input.

**Why is Hashing Important?**

Hashing plays a vital role in our daily use of the Internet. Like other cryptographic functions, hashing remains hidden from the user. Hashing helps protect data’s integrity and ensure password confidentiality.


Storing Passwords in Plaintext

Quite a few data breaches have leaked plaintext passwords. You’re probably familiar with the “rockyou.txt” password list on Kali Linux, among many other offensive security distributions. This password list came from RockYou, a company that developed social media applications and widgets. They stored their passwords in plaintext, and the company had a data breach. The text file contains over 14 million passwords. You can find rockyou.txt in the /usr/share/wordlists directory.


**Using an Insecure Hash Function**

LinkedIn also suffered a data breach in 2012. LinkedIn used an insecure hashing algorithm, the SHA-1, to store user passwords. Furthermore, no password salting was used. Password salting refers to adding a salt, i.e., a random value, to the password before it is hashed.


**Using Hashing for Secure Password Storage**

Using Hashing to Store Passwords

This is where hashing comes in. What if, instead of storing the password, you just stored its hash value using a secure hashing function? This process means you never have to store the user’s password, and if your database is leaked, an attacker will have to crack each password to find out what the password was.


A **Rainbow Table** is a lookup table of hashes to plaintexts, so you can quickly find out what password a user had just from the hash. A rainbow table trades the time to crack a hash for hard disk space, but it takes time to create. Here’s a quick example to get an idea of what a rainbow table looks like.


**Hash	                                Password**

02c75fb22c75b23dc963c7eb91a062cc	zxcvbnm

b0baee9d279d34fa1dfd71aadb908c3f	11111

c44a471bd78cc6c2fea32b9fe028d30a	asdfghjkl


Websites like **CrackStation.com** and **Hashes.com** internally use massive rainbow tables to provide fast password cracking for hashes without salts. Doing a lookup in a sorted list of hashes is quicker than trying to crack the hash.

Crack the hash “5b31f93c09ad1d065c0491b764d04933” using an online tool.


**To Securely Storing Passwords**

You can find many good guides online that promote best security practices when storing passwords. Please check if there are any standards you need to follow when storing passwords before adopting one.

Crack the hash “5b31f93c09ad1d065c0491b764d04933” using an online tool.


**Recognising Password Hashes**

**Linux Passwords**

On Linux, password hashes are stored in /etc/shadow, which is normally only readable by root. They used to be stored in /etc/passwd, which was readable by everyone.

**MS Windows Passwords**

MS Windows passwords are hashed using NTLM, a variant of MD4. They’re visually identical to MD4 and MD5 hashes, so it’s very important to use context to determine the hash type.

On MS Windows, password hashes are stored in the SAM (Security Accounts Manager). MS Windows tries to prevent normal users from dumping them, but tools like mimikatz exist to circumvent MS Windows security. Notably, the hashes found there are split into NT hashes and LM hashes.


**Password Cracking**

We’ve already mentioned rainbow tables as a method to crack hashes that don’t use a salt, but what if there’s a salt involved?

You can’t “decrypt” password hashes. They’re not encrypted. You have to crack the hashes by hashing many different inputs (such as rockyou.txt as it covers many possible passwords), potentially adding the salt if there is one and comparing it to the target hash. Once it matches, you know what the password was. Tools like Hashcat and John the Ripper are commonly used for these purposes.


**Time to Crack Some Hashes (USE TASK1,TASK2,TASK3)**


Hashcat uses the following basic syntax: hashcat -m <hash_type> -a <attack_mode> hashfile wordlist

Hash.txt files will be saved inside /Download/Hashing-Basics/

**For example**

hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt


**-m** <hash_type> specifies the hash-type in numeric format. For example, -m 1000 is for NTLM.

**-a** <attack_mode> specifies the attack-mode. For example, -a 0 is for straight, i.e., trying one password from the wordlist after the other.


**hashfile** is the file containing the hash you want to crack.

**wordlist** is the security word list you want to use in your attack.

**For example,** hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt will treat the hash as Bcrypt and try the passwords in the rockyou.txt file.

1. Use hashcat to crack the hash, $2a$06$7yoU3Ng8dHTXphAg913cyO6Bjs3K5lBnwq5FJyA6d01pMSrddr1ZG


2. Use hashcat to crack the SHA2-256 hash, 9eb7ee7f551d2f0ac684981bd1f1e2fa4a37590199636753efe614d4db30e8e1 saved in saved in ~/Hashing-Basics/Task-6/hash2.txt.


3.Crack the hash, b6b0d451bbf6fed658659a9e7e5598fe, saved in ~/Hashing-Basics/Task-6/hash4.txt


**MORE PRACTICE**

1. What is the SHA256 hash of the passport.jpg file in ~/Hashing-Basics/Task-2?

2. What is the output size in bytes of the MD5 hash function?

3. Crack the hash “5b31f93c09ad1d065c0491b764d04933” using an online tool.

4. Use hashcat to crack the hash, $2a$06$7yoU3Ng8dHTXphAg913cyO6Bjs3K5lBnwq5FJyA6d01pMSrddr1ZG


**Hashing for Integrity Checking**

Hashing = password storage and data integrity. We have extensively discussed how hashing secures passwords in authentication systems. Here we discuss how we can use hash functions to check the integrity of files.

**Integrity Checking**

Hashing can be used to check that files haven’t been changed. If you put the same data in, you always get the same data out. Even if a single bit changes, the hash will change significantly.

This means you can use it to check that files haven’t been modified or to ensure that the file you downloaded is identical to the file on the web server.

You can also use hashing to find duplicate files; if two documents have the same hash, they are the same document. This is very convenient for finding and deleting duplicate files.


**Checking Integrity (USE TASK-7)**

What is SHA256 hash of libgcrypt-1.11.0.tar.bz2 found in ~/Hashing-Basics/Task-7?


**Cracking Basic Hashes (Use Task04)**

A hash is a way of taking a piece of data of any length and representing it in another fixed-length form. This process masks the original value of the data. The hash value is obtained by running the original data through a hashing algorithm. Many popular hashing algorithms exist, such as MD4, MD5, SHA1 and NTLM.

Where John Comes in

Even though the algorithm is not feasibly reversible, that doesn’t mean cracking the hashes is impossible. If you have the hashed version of a password, for example, and you know the hashing algorithm, you can use that hashing algorithm to hash a large number of words, called a dictionary. You can then compare these hashes to the one you’re trying to crack to see if they match. If they do, you know what word corresponds to that hash- you’ve cracked it!


1. What type of hash is hash1.txt?

install hash-identifier in local machine/use hashs.com website/hashcat

Using the hash-identifier, we are able to guess the hash type.

$ cat hash1.txt | hash-identifier


2. What is the cracked value of hash1.txt?

hashcat -m 900 hash1.txt

john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash1.txt

3. What type of hash is hash2.txt?

4. What is the cracked value of hash2.txt?

5. What type of hash is hash3.txt?

6. What is the cracked value of hash3.txt?

7. What type of hash is hash4.txt?

8. What is the cracked value of hash4.txt?


**Cracking Windows Authentication Hashes (Use TASK05)**


In Windows, SAM (Security Account Manager) is used to store user account information, including usernames and hashed passwords. You can acquire NTHash/NTLM hashes by dumping the SAM database on a Windows machine, using a tool like Mimikatz, or using the Active Directory database: NTDS.dit

What is the cracked value of this password?

**Cracking Hashes from /etc/shadow**


John can be very particular about the formats it needs data in to be able to work with it; for this reason, to crack /etc/shadow passwords, you must combine it with the /etc/passwd file for John to understand the data it’s being given. To do this, we use a tool built into the John suite of tools called unshadow. The basic syntax of unshadow is as follows:

unshadow local_passwd local_shadow > youname.txt


**Example Command**

john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt yourname.txt

What is the root password?


**Cracking Password Protected Zip Files (Use TASK09)**

We can use John to crack the password on password-protected Zip files. Again, we’ll use a separate part of the John suite of tools to convert the Zip file into a format that John will understand.


Similarly to the unshadow tool we used previously, we will use the zip2john tool to convert the Zip file into a hash format that John can understand and hopefully crack. The primary usage is like this:

**Example Usage**


zip2john zipfile.zip > yourname_hash.txt


Cracking

john --wordlist=/usr/share/wordlists/rockyou.txt yourname_hash.txt

1. What is the password for the secure.zip file?

2. What is the contents of the flag inside the zip file?


**Cracking Password-Protected RAR Archives**

Almost identical to the zip2john tool, we will use the rar2john tool to convert the RAR file into a hash format that John can understand. The basic syntax is as follows:

**Example Usage**

/usr/sbin/rar2john rarfile.rar > yourname_hash.txt


Cracking

john --wordlist=/usr/share/wordlists/rockyou.txt yourname_hash.txt

1. What is the password for the secure.rar file?

2. What are the contents of the flag inside the zip file?


**Cracking SSH Key Passwords (Use TASK11)**


Using John to crack the SSH private key password of id_rsa files.


Unless configured otherwise, you authenticate your SSH login using a password. However, you can configure key-based authentication, which lets you use your private key, id_rsa, as an authentication key to log in to a remote machine over SSH. However, doing so will often require a password to access the private key; here, we will be using John to crack this password to allow authentication over SSH using the key.


**Example Usage**

usr/share/john/ssh2john.py id_rsa > yourname_rsa_hash.txt


Cracking

john --wordlist=/usr/share/wordlists/rockyou.txt yourname_rsa_hash.txt



**Password Cracking Hydra**

What is Hydra?

Hydra is a brute force online password cracking program, a quick system login password “hacking” tool.

Hydra can run through a list and “brute force” some authentication services. Imagine trying to manually guess someone’s password on a particular service (SSH, Web Application Form, FTP or SNMP) - we can use Hydra to run through a password list and speed this process up for us, determining the correct password.


**Brute force**

A brute force attack is a hacking method that uses trial and error to crack passwords, login credentials, and encryption keys. It is a simple yet reliable tactic for gaining unauthorized access to individual accounts and organizations’ systems and networks. The hacker tries multiple usernames and passwords, often using a computer to test a wide range of combinations, until they find the correct login information.


Happy Cracking!