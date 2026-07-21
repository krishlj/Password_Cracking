# Password Hash Auditing Using Hashcat, John the Ripper, and CrackStation

## Introduction

Passwords should never be stored as readable plaintext. Secure systems store a one-way representation of each password called a hash. During authentication, the system hashes the entered password and compares the result with the stored value.

This project demonstrates how weak passwords can be recovered from authorized sample hashes using Hashcat, John the Ripper, and CrackStation. The goal is to understand password-storage risks and help defensive security professionals evaluate password strength, investigate exposed credentials, and recommend stronger controls.

> **Legal and ethical notice:** Use these techniques only on hashes, files, and systems that you own or have explicit permission to test. Never upload real organizational or personal password hashes to a public website.

## Project Overview

In this project, you will:

- Understand encryption, hashing, and password storage.
- Identify common password-hash formats.
- Create safe sample hashes for testing.
- Perform authorized offline dictionary attacks with Hashcat.
- Audit sample hashes with John the Ripper.
- Understand how online hash lookup services work.
- Verify file integrity using SHA-256.
- Test password-protected lab archives and private keys.
- Document defensive findings and recommendations.
- Complete the included Task04, Task05, Task06, Task07, Task09, Task10, and Task11 exercises.

This is an **offline password-auditing project**. Online login brute forcing with tools such as Hydra is outside the scope of this guide.

## Prerequisites

Before starting the project, ensure the following:

- Kali Linux, Ubuntu, or another authorized Linux lab system is available.
- Hashcat and John the Ripper are installed.
- You have a small test wordlist.
- You understand basic Linux terminal commands.
- You have permission to test every hash and file used in the project.
- No real production credentials are included in the lab.

## Included Lab Materials

The `Password_Cracking.zip` archive contains this guide and the following authorized practice files:

```text
Password_Cracking/
├── Project-Password_Cracking_Using_Hashcat_John_Crackstations.md
└── John-the-Ripper-Task_Materials/
    ├── Task04/
    │   ├── hash-id.py
    │   ├── hash1.txt
    │   ├── hash2.txt
    │   ├── hash3.txt
    │   └── hash4.txt
    ├── Task05/
    │   └── ntlm.txt
    ├── Task06/
    │   ├── etc_hashes.txt
    │   ├── local_passwd
    │   └── local_shadow
    ├── Task07/
    │   └── hash07.txt
    ├── Task09/
    │   └── secure.zip
    ├── Task10/
    │   └── secure.rar
    └── Task11/
        └── id_rsa
```

The files contain training data only. The README does not reveal the recovered passwords or flags, allowing learners to complete and document the exercises themselves.

## Important Cryptography Concepts

### 1. Plaintext and Ciphertext

- **Plaintext** is the original readable data.
- **Ciphertext** is the unreadable output created by encryption.
- **Encryption** converts plaintext into ciphertext using an algorithm and a key.
- **Decryption** uses the correct key to convert ciphertext back into plaintext.

Encryption is reversible when the correct key is available.

### 2. Symmetric and Asymmetric Encryption

- **Symmetric encryption** uses the same secret key for encryption and decryption.
- **Asymmetric encryption** uses a public and private key pair. Data encrypted with one key is processed using the corresponding key in the pair.

### 3. Hash Functions

A hash function converts data of any size into a fixed-length value. Hashing is designed to be one-way and does not use a decryption key.

Hash functions are commonly used for:

- Password verification
- File-integrity checking
- Digital signatures
- Duplicate-file identification

Password cracking does not decrypt a hash. A cracking tool hashes possible passwords and compares each result with the target hash. A match reveals the original candidate used to create that hash.

### 4. Salting and Password Storage

A salt is a random value added to a password before it is hashed. Salting prevents identical passwords from producing identical stored hashes and reduces the effectiveness of precomputed rainbow tables.

Fast, older algorithms such as MD5, SHA-1, and unsalted NTLM are not suitable for modern password storage. Secure applications should use a dedicated password-hashing algorithm such as Argon2id, bcrypt, scrypt, or PBKDF2 with a unique salt and an appropriate work factor.

## Tools Used in This Project

### Hashcat

Hashcat is an offline password-recovery and auditing tool. It supports multiple hash formats and attack modes.

Basic syntax:

```bash
hashcat -m <hash_mode> -a <attack_mode> <hash_file> <wordlist>
```

Important options:

- `-m` selects the hash mode.
- `-a 0` selects a straight dictionary attack.
- `<hash_file>` contains the authorized test hash.
- `<wordlist>` contains possible password candidates.

Common Hashcat modes:

| Hash type | Hashcat mode |
| --- | ---: |
| MD5 | `0` |
| SHA-1 | `100` |
| SHA-256 | `1400` |
| NTLM | `1000` |
| bcrypt | `3200` |

Always verify the hash type before selecting a mode.

### John the Ripper

John the Ripper is an offline password-security auditing tool. It can process common hashes and convert supported archives or keys into formats that John can test.

Basic syntax:

```bash
john --format=<format> --wordlist=<wordlist> <hash_file>
```

### CrackStation

CrackStation is an online hash-lookup service. It compares a submitted hash against precomputed values and may recover weak, unsalted passwords quickly.

Use it only for intentionally created lab hashes. Do not submit real password hashes because the data leaves your environment.

## Steps to Prepare the Password-Auditing Lab

### 1. Install the Required Tools

On Kali Linux, the tools may already be installed. Verify them with:

```bash
hashcat --version
john --version
```

If a tool is missing, install it from the authorized system repository:

```bash
sudo apt update
sudo apt install hashcat john -y
```

### 2. Extract the Included Project Files

Move `Password_Cracking.zip` to your authorized Linux lab and extract it:

```bash
mkdir -p ~/password-auditing-lab
unzip Password_Cracking.zip -d ~/password-auditing-lab
cd ~/password-auditing-lab
```

Verify the included materials:

```bash
find John-the-Ripper-Task_Materials -maxdepth 2 -type f | sort
```

All commands in this guide assume that your terminal is open in `~/password-auditing-lab`.

### 3. Prepare the Practice Wordlist

Check whether the common Kali Linux practice wordlist is available:

```bash
ls -lh /usr/share/wordlists/rockyou.txt
```

If only the compressed version exists, extract it:

```bash
sudo gzip -dk /usr/share/wordlists/rockyou.txt.gz
```

Set a variable to make the remaining commands shorter:

```bash
WORDLIST=/usr/share/wordlists/rockyou.txt
```

You may use a smaller authorized wordlist if system resources are limited.

### 4. Confirm the Working Directory

Run:

```bash
pwd
ls -la John-the-Ripper-Task_Materials
```

The expected working directory is:

```text
~/password-auditing-lab
```

## Task04: Identify and Audit Basic Hashes

Task04 contains four hashes that use different algorithms. Identify the likely type before running a password-auditing tool.

### 1. Review the Hash Files

```bash
for file in John-the-Ripper-Task_Materials/Task04/hash*.txt; do
  echo "--- $file ---"
  cat "$file"
done
```

### 2. Run the Included Hash Identifier

The included `hash-id.py` script can suggest possible hash types:

```bash
python3 John-the-Ripper-Task_Materials/Task04/hash-id.py
```

Copy and paste one Task04 hash when prompted. Repeat this process for all four hashes.

Hash identification is based on structure and length, so verify the result using the file context and the tool documentation.

### 3. Audit `hash1.txt`

`hash1.txt` contains a 32-character hexadecimal value. For this exercise, test it as raw MD5:

```bash
hashcat -m 0 -a 0 John-the-Ripper-Task_Materials/Task04/hash1.txt "$WORDLIST"
```

Display the result:

```bash
hashcat -m 0 John-the-Ripper-Task_Materials/Task04/hash1.txt --show
```

John alternative:

```bash
john --format=raw-md5 --wordlist="$WORDLIST" John-the-Ripper-Task_Materials/Task04/hash1.txt
john --format=raw-md5 --show John-the-Ripper-Task_Materials/Task04/hash1.txt
```

### 4. Audit `hash2.txt`

For this exercise, test `hash2.txt` as raw SHA-1:

```bash
hashcat -m 100 -a 0 John-the-Ripper-Task_Materials/Task04/hash2.txt "$WORDLIST"
hashcat -m 100 John-the-Ripper-Task_Materials/Task04/hash2.txt --show
```

John alternative:

```bash
john --format=raw-sha1 --wordlist="$WORDLIST" John-the-Ripper-Task_Materials/Task04/hash2.txt
john --format=raw-sha1 --show John-the-Ripper-Task_Materials/Task04/hash2.txt
```

### 5. Audit `hash3.txt`

For this exercise, test `hash3.txt` as raw SHA-256:

```bash
hashcat -m 1400 -a 0 John-the-Ripper-Task_Materials/Task04/hash3.txt "$WORDLIST"
hashcat -m 1400 John-the-Ripper-Task_Materials/Task04/hash3.txt --show
```

John alternative:

```bash
john --format=Raw-SHA256 --wordlist="$WORDLIST" John-the-Ripper-Task_Materials/Task04/hash3.txt
john --format=Raw-SHA256 --show John-the-Ripper-Task_Materials/Task04/hash3.txt
```

### 6. Audit `hash4.txt`

For this exercise, test `hash4.txt` as raw SHA-512:

```bash
hashcat -m 1700 -a 0 John-the-Ripper-Task_Materials/Task04/hash4.txt "$WORDLIST"
hashcat -m 1700 John-the-Ripper-Task_Materials/Task04/hash4.txt --show
```

John alternative:

```bash
john --format=Raw-SHA512 --wordlist="$WORDLIST" John-the-Ripper-Task_Materials/Task04/hash4.txt
john --format=Raw-SHA512 --show John-the-Ripper-Task_Materials/Task04/hash4.txt
```

Record the identified algorithm, recovered result, and tool used for every Task04 file.

## Task05: Audit the Included NTLM Hash

Task05 contains an authorized Windows NTLM training hash.

### 1. Review the File

```bash
cat John-the-Ripper-Task_Materials/Task05/ntlm.txt
```

### 2. Audit with Hashcat

Use Hashcat mode `1000` for NTLM:

```bash
hashcat -m 1000 -a 0 John-the-Ripper-Task_Materials/Task05/ntlm.txt "$WORDLIST"
hashcat -m 1000 John-the-Ripper-Task_Materials/Task05/ntlm.txt --show
```

### 3. Audit with John

```bash
john --format=nt --wordlist="$WORDLIST" John-the-Ripper-Task_Materials/Task05/ntlm.txt
john --format=nt --show John-the-Ripper-Task_Materials/Task05/ntlm.txt
```

NTLM is a fast, unsalted password hash and should not be treated as secure password storage.

## Task06: Audit the Included Linux Password Hash

Task06 contains lab copies of `passwd` and `shadow` entries. Do not replace these files with data from a real system.

### 1. Review the Training Files

```bash
cat John-the-Ripper-Task_Materials/Task06/local_passwd
cat John-the-Ripper-Task_Materials/Task06/local_shadow
```

The `$6$` prefix in the shadow entry indicates SHA-512 crypt.

### 2. Combine the Files with `unshadow`

```bash
unshadow \
  John-the-Ripper-Task_Materials/Task06/local_passwd \
  John-the-Ripper-Task_Materials/Task06/local_shadow \
  > John-the-Ripper-Task_Materials/Task06/combined.txt
```

### 3. Audit the Combined Entry

```bash
john --format=sha512crypt --wordlist="$WORDLIST" \
  John-the-Ripper-Task_Materials/Task06/combined.txt
```

Display the result:

```bash
john --format=sha512crypt --show \
  John-the-Ripper-Task_Materials/Task06/combined.txt
```

The `etc_hashes.txt` file provides scenario context and a copy of the same training entry.

## Task07: Identify and Audit `hash07.txt`

### 1. Review and Identify the Hash

```bash
cat John-the-Ripper-Task_Materials/Task07/hash07.txt
```

Use the Task04 identification script and confirm the likely algorithm. For this exercise, test the value as raw MD5.

### 2. Audit the Hash

```bash
hashcat -m 0 -a 0 John-the-Ripper-Task_Materials/Task07/hash07.txt "$WORDLIST"
hashcat -m 0 John-the-Ripper-Task_Materials/Task07/hash07.txt --show
```

John alternative:

```bash
john --format=raw-md5 --wordlist="$WORDLIST" John-the-Ripper-Task_Materials/Task07/hash07.txt
john --format=raw-md5 --show John-the-Ripper-Task_Materials/Task07/hash07.txt
```

## Task09: Audit the Included ZIP Archive

### 1. Convert the ZIP File

```bash
zip2john John-the-Ripper-Task_Materials/Task09/secure.zip \
  > John-the-Ripper-Task_Materials/Task09/zip-hash.txt
```

### 2. Audit the Extracted Hash

```bash
john --wordlist="$WORDLIST" John-the-Ripper-Task_Materials/Task09/zip-hash.txt
john --show John-the-Ripper-Task_Materials/Task09/zip-hash.txt
```

### 3. Open the Archive

Use the recovered lab password only to open the included training archive:

```bash
unzip John-the-Ripper-Task_Materials/Task09/secure.zip \
  -d John-the-Ripper-Task_Materials/Task09/extracted
```

Record the password and the contents of the included exercise file in your lab notes.

## Task10: Audit the Included RAR Archive

### 1. Locate `rar2john`

```bash
command -v rar2john || find /usr -name rar2john 2>/dev/null
```

### 2. Convert the RAR File

Use the path returned by the previous command. A common Kali path is:

```bash
/usr/share/john/rar2john John-the-Ripper-Task_Materials/Task10/secure.rar \
  > John-the-Ripper-Task_Materials/Task10/rar-hash.txt
```

### 3. Audit the Extracted Hash

```bash
john --wordlist="$WORDLIST" John-the-Ripper-Task_Materials/Task10/rar-hash.txt
john --show John-the-Ripper-Task_Materials/Task10/rar-hash.txt
```

Use an installed RAR extraction tool to open the authorized archive and record the exercise result.

## Task11: Audit the Included SSH Private-Key Passphrase

### 1. Protect the Key Permissions

```bash
chmod 600 John-the-Ripper-Task_Materials/Task11/id_rsa
```

### 2. Locate and Run `ssh2john.py`

```bash
python3 /usr/share/john/ssh2john.py \
  John-the-Ripper-Task_Materials/Task11/id_rsa \
  > John-the-Ripper-Task_Materials/Task11/ssh-key-hash.txt
```

### 3. Audit the Extracted Hash

```bash
john --wordlist="$WORDLIST" John-the-Ripper-Task_Materials/Task11/ssh-key-hash.txt
john --show John-the-Ripper-Task_Materials/Task11/ssh-key-hash.txt
```

Do not use the included key to access any system. It is provided only for the offline passphrase-recovery exercise.

## Reviewing Hashcat Status

While Hashcat is running, press `s` to view the status. Important values include:

- **Status:** Current state of the job
- **Hash.Mode:** Selected hash algorithm
- **Speed:** Candidate checks per second
- **Recovered:** Number of recovered hashes
- **Progress:** Candidates already tested

### Understand the Result

A recovered password demonstrates that the password candidate existed in the selected wordlist. It does not mean that the hash algorithm itself was decrypted.

If Hashcat reports `Exhausted`, it tested all candidates without finding a match. Confirm the hash mode, file format, and wordlist before trying a different authorized test strategy.

## Additional John the Ripper Practice

### Audit an Authorized MD5 Sample Hash

Run:

```bash
john --format=raw-md5 --wordlist="$WORDLIST" John-the-Ripper-Task_Materials/Task04/hash1.txt
```

Display the recovered result:

```bash
john --format=raw-md5 --show John-the-Ripper-Task_Materials/Task04/hash1.txt
```

### Audit the Included SHA-256 Hash

Check whether your John installation supports the required format:

```bash
john --list=formats | grep -i sha256
```

If `Raw-SHA256` is available, run:

```bash
john --format=Raw-SHA256 --wordlist="$WORDLIST" John-the-Ripper-Task_Materials/Task04/hash3.txt
```

Display the result:

```bash
john --format=Raw-SHA256 --show John-the-Ripper-Task_Materials/Task04/hash3.txt
```

Format names can vary between John versions. Use the exact format shown by `john --list=formats`.

### 3. Review John’s Status and Results

While John is running, press any key to display its current status.

To list all passwords recovered during the lab, run:

```bash
john --show John-the-Ripper-Task_Materials/Task04/hash1.txt
```

John stores recovered results in its pot file. A later run may finish immediately because the result was already recorded.

## Steps to Identify an Unknown Hash Type

### 1. Examine the Hash Structure

Review the hash length and prefix:

```bash
cat unknown-hash.txt
```

Common visual indicators include:

- 32 hexadecimal characters: possibly MD5, NTLM, or another 128-bit hash
- 40 hexadecimal characters: possibly SHA-1
- 64 hexadecimal characters: possibly SHA-256
- `$2a$`, `$2b$`, or `$2y$`: bcrypt
- `$6$`: commonly SHA-512 crypt

Length alone cannot reliably identify a hash. Context is important.

### 2. Use a Hash-Identification Tool

If `hashid` is installed, run:

```bash
hashid unknown-hash.txt
```

Alternatively, use:

```bash
hash-identifier
```

Paste only an authorized lab hash when prompted.

### 3. Confirm the Likely Type

Compare the result with:

- The system or application that generated the hash
- The hash prefix and length
- The expected password-storage method
- Hashcat’s example-hash documentation

Hash-identification tools return possible types, not guaranteed answers.

## Steps to Use CrackStation Safely

### 1. Select a Lab Hash

Use only an authorized hash included with this project, such as `John-the-Ripper-Task_Materials/Task04/hash1.txt`.

### 2. Submit the Hash

- Open [CrackStation](https://crackstation.net/).
- Paste the authorized sample hash.
- Complete the verification challenge.
- Select **Crack Hashes**.

### 3. Review the Result

If the password is found, the hash likely matched a value in the service’s precomputed database.

If it is not found, the password may be absent from the database, salted, or protected by a slower password-hashing algorithm.

### 4. Understand the Security Limitation

Online lookup services should not be used with real password hashes. Submitting a hash shares sensitive authentication data with an external service. Organizational password audits should be performed offline using approved systems and processes.

## Hashing for File-Integrity Checking

### 1. Calculate a SHA-256 File Hash

Run:

```bash
sha256sum sample-file.txt
```

Save the result:

```bash
sha256sum sample-file.txt > sample-file.sha256
```

### 2. Verify the File Later

Run:

```bash
sha256sum -c sample-file.sha256
```

Expected result:

```text
sample-file.txt: OK
```

If the file changes, verification should fail. This demonstrates how hashing can detect accidental or unauthorized modification.

## Auditing Password-Protected Lab Files with John

Use only archives and private keys created for your own lab.

### 1. Password-Protected ZIP Archive

Convert the authorized ZIP file into a format John understands:

```bash
zip2john secure-lab.zip > zip-hash.txt
```

Audit it with the lab wordlist:

```bash
john --wordlist="$WORDLIST" zip-hash.txt
```

Display the result:

```bash
john --show zip-hash.txt
```

### 2. Password-Protected RAR Archive

Convert the authorized RAR file:

```bash
rar2john secure-lab.rar > rar-hash.txt
```

Audit it:

```bash
john --wordlist="$WORDLIST" rar-hash.txt
```

The location of `rar2john` may differ depending on the John package.

### 3. Password-Protected SSH Private Key

Convert a lab SSH private key into a format John understands:

```bash
ssh2john.py lab_id_rsa > ssh-key-hash.txt
```

Audit it:

```bash
john --wordlist="$WORDLIST" ssh-key-hash.txt
```

Some systems install the conversion script under `/usr/share/john/ssh2john.py`.

## Auditing Linux Password Hashes in an Authorized Lab

Linux account information is stored in `/etc/passwd`, while password hashes are normally stored in `/etc/shadow`. The shadow file is restricted because it contains sensitive authentication data.

Use exported copies from a disposable lab system only. Do not access or copy password databases from systems without authorization.

Combine the authorized lab files:

```bash
unshadow lab-passwd lab-shadow > linux-combined.txt
```

Audit the combined file:

```bash
john --wordlist="$WORDLIST" linux-combined.txt
```

Display any recovered results:

```bash
john --show linux-combined.txt
```

The correct John format depends on the hash prefix in the shadow entry. John can often detect this automatically.

## Troubleshooting

### Hashcat Reports a Token-Length Exception

Check that:

- The hash file contains only the expected hash value.
- There are no labels, quotes, or extra spaces.
- The selected `-m` mode matches the hash type.
- The hash was not broken across multiple lines.

### Hashcat Reports No Devices Found

Run:

```bash
hashcat -I
```

Confirm that Hashcat detects a usable CPU, GPU, or compute runtime. Virtual machines may have limited acceleration, but small lab wordlists can often be tested with CPU support when properly installed.

### The Job Finishes with `Exhausted`

This means every wordlist candidate was tested without a match. Verify:

- The correct hash mode was selected.
- The intended password exists in the lab wordlist.
- The hash was created from the expected password.
- Capitalization and special characters are correct.

### John Shows No Password Hashes Loaded

Check that:

- The correct `--format` value is used.
- The input file uses a format supported by John.
- The conversion tool completed successfully for an archive or key.
- Your John installation includes the required format.

### A Recovered Password Appears Immediately

Hashcat and John save earlier results. Use `--show` to confirm the stored result. For a completely new lab test, create a new sample password and hash instead of using real credentials.

## Defensive Security Findings

After completing the project, document the following:

- Hash type tested
- Tool and mode used
- Wordlist size
- Whether the password was recovered
- Approximate time required
- Password weakness observed
- Recommended security control

Recommended controls include:

- Use long, unique passwords or passphrases.
- Enable multifactor authentication.
- Store passwords using Argon2id, bcrypt, scrypt, or PBKDF2.
- Use a unique salt for every password.
- Configure an appropriate cost or work factor.
- Block known-compromised passwords.
- Protect password databases with strict access controls.
- Monitor authentication systems for unusual access or export activity.
- Use an approved password manager.

## Expected Project Results

After completing this project, you should be able to:

- Explain the difference between encryption and hashing.
- Identify common hash formats using structure and context.
- Create safe sample hashes for a controlled lab.
- Audit authorized hashes using Hashcat and John the Ripper.
- Explain why weak passwords are vulnerable to dictionary attacks.
- Describe the privacy risk of public hash-lookup services.
- Verify file integrity using SHA-256.
- Recommend stronger password-storage and authentication controls.

## Conclusion

Password-hash auditing helps defensive security professionals understand how quickly weak passwords may be recovered after a credential database is exposed. The lab demonstrates that password security depends on both user behaviour and secure application design.

Organizations can reduce risk by using modern password-hashing algorithms, unique salts, appropriate work factors, multifactor authentication, compromised-password screening, and strong monitoring around credential stores.

Practice only in a controlled and authorized environment.
