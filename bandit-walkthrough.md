# OverTheWire: Bandit Walkthrough
> Speedrun completed in ~6 hours · Levels 0–33

---

## Bandit 0

**Concept:** Basic SSH login and reading a file.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
cat readme
```

**Password:** `ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If`

---

## Bandit 1

**Concept:** Reading a file named `-` (dashed filename). Shell interprets `-` as stdin, so you need an explicit path.

```bash
cat ./"-"
```

**Password:** `263JGJPfgU6LtdEvgfWU1XP5yac29mFx`

---

## Bandit 2

**Concept:** Reading a file with spaces in its name. Wrap the name in quotes or escape the spaces.

```bash
cat ./"---spaces in the filename---"
```

**Password:** `MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx`

---

## Bandit 3

**Concept:** Finding a hidden file (prefixed with `.`) inside a directory.

```bash
cat ./*
# or explicitly:
cat ./"...Hiding from you"
```

**Password:** `2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ`

---

## Bandit 4

**Concept:** Among multiple files, find the one with human-readable (ASCII) content using `file`.

```bash
file ./* | grep "ASCII"
cat ./-file07   # whichever file is ASCII
```

> `file` inspects actual file content type, not just the extension.

**Password:** `4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw`

---

## Bandit 5

**Concept:** Find a file that is human-readable, not executable, and exactly 1033 bytes.

```bash
du -ab | grep 1033
```

**Password:** `HWasnPhtq9AVKe0dmk45nxy20cvUa6EG`

---

## Bandit 6

**Concept:** Find a file anywhere on the server owned by user `bandit7`, group `bandit6`, and 33 bytes in size. Suppress permission errors with `2>/dev/null`.

```bash
cd /
find . -type f -size 33c -user bandit7 -group bandit6 2>/dev/null
```

**Password:** `morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj`

---

## Bandit 7

**Concept:** Find the word `millionth` in a large text file using `grep`.

```bash
cat data.txt | grep -w "millionth"
```

**Password:** `dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc`

---

## Bandit 8

**Concept:** Find the one line that appears only once in a file. Sort first so `uniq` can detect duplicates.

```bash
sort data.txt | uniq -c | grep -w 1
```

**Password:** `4CKMh1JI91bUIZZPXDqGanal4xvAg0JM`

---

## Bandit 9

**Concept:** Extract human-readable strings from a binary file, then filter for lines starting with `===`.

```bash
strings data.txt | grep "==="
```

**Password:** `FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey`

---

## Bandit 10

**Concept:** Decode a Base64-encoded file.

```bash
base64 -d data.txt
```

**Password:** `dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr`

---

## Bandit 11

**Concept:** ROT13 cipher — each letter is rotated 13 places. A→N, N→A, etc. `tr` handles character mapping.

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

> ROT13 maps A–M → N–Z and N–Z → A–M (same for lowercase). Applying it twice gives back the original.

**Password:** `7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4`

---

## Bandit 12

**Concept:** A hexdump of a file that has been compressed multiple times (gzip, bzip2, tar in various orders). Reverse the hexdump, then keep checking `file` and decompressing accordingly.

```bash
mkdir /tmp/work && cd /tmp/work
cp ~/data.txt .

xxd -r data.txt > output.bin     # hexdump → binary

file output.bin                   # gzip
mv output.bin output.gz
gzip -d output.gz

file output                       # bzip2
bzip2 -d output

file output.out                   # gzip
mv output.out data4.gz
gzip -d data4.gz

file data4                        # tar
mv data4 data4.tar
tar -xvf data4.tar

file data5.bin                    # tar
tar -xvf data5.bin

file data6.bin                    # bzip2
bzip2 -d data6.bin

file data6.bin.out                # tar
tar -xvf data6.bin.out

file data8.bin                    # gzip
mv data8.bin data9.gz
gzip -d data9.gz

file data9                        # ASCII text — done
cat data9
```

**Password:** `FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn`

---

## Bandit 13

**Concept:** No direct password — you get an SSH private key to log in as `bandit14`.

```bash
# On bandit13:
cat sshkey.private   # copy the key contents

# On local machine:
nano sshkey.private  # paste the key
chmod 600 sshkey.private
ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
```

---

## Bandit 14

**Concept:** Submit the current level's password to localhost port 30000 to receive the next one.

```bash
# Get bandit14's password first:
cat /etc/bandit_pass/bandit14

# Submit it:
nc localhost 30000
# paste: MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS
```

**Password:** `8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo`

---

## Bandit 15

**Concept:** Same as level 14 but the port uses SSL/TLS. Use `openssl s_client` instead of `nc`.

```bash
openssl s_client -connect localhost:30001
# paste: 8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo
```

**Password:** `kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx`

---

## Bandit 16

**Concept:** Scan ports 31000–32000, find the one speaking SSL, submit the password, and receive an SSH private key for bandit17.

```bash
# Scan for open ports:
for port in {31000..32000}; do nc -z localhost $port; done

# Submit to the SSL port (31790):
echo "kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx" | openssl s_client -connect localhost:31790 -quiet

# Save the returned private key, chmod 600, then:
ssh -i key17 bandit17@bandit.labs.overthewire.org -p 2220
cat /etc/bandit_pass/bandit17
```

**Password:** `EReVavePLFHtFlFsjn3hyzMlvSuSAcRD`

---

## Bandit 17

**Concept:** Two password files exist — `passwords.old` and `passwords.new`. The changed line in `.new` is the password.

```bash
diff passwords.new passwords.old | grep ">"
```

**Password:** `x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO`

---

## Bandit 18

**Concept:** `.bashrc` has been modified to log you out immediately on login. Bypass by running a command directly over SSH without starting an interactive shell.

```bash
echo "cat readme" | ssh bandit18@bandit.labs.overthewire.org -p 2220
```

**Password:** `cGWpMaKXVwDUNgPAVJbWYuGHVn9zl3j8`

---

## Bandit 19

**Concept:** Use a setuid binary (`bandit20-do`) that runs commands as `bandit20`.

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

**Password:** `0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO`

---

## Bandit 20

**Concept:** A setuid binary connects back to a port you're listening on, verifies the current password, and sends back the next one. Requires two terminals — use `tmux`.

```bash
# Pane 1 — start listener:
tmux
nc -l -p 1234
# Ctrl+B, D  (detach)

# Pane 2 — run the binary:
tmux new-window
./suconnect 1234
# Ctrl+B, D

# Re-attach to pane 1 and paste bandit20 password:
tmux attach -t 0
# paste: 0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO
```

**Password:** `EeoULMCra2q0dSkYj561DX7s1CpBuOBt`

---

## Bandit 21

**Concept:** A cron job runs as `bandit22` and writes its password to a temp file. Read the cron script to find the file path.

```bash
cat /etc/cron.d/* 2>/dev/null | grep "bandit22"
cat /usr/bin/cronjob_bandit22.sh
cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

**Password:** `tRae0UfB9v0UzbCdn9cY0gQnds9GF58Q`

---

## Bandit 22

**Concept:** The cron script generates a filename using an md5 hash of `"I am user bandit23"`. Reproduce the hash to find the file.

```bash
cat /etc/cron.d/* 2>/dev/null | grep bandit23
cat /usr/bin/cronjob_bandit23.sh

# Reproduce the filename:
echo "I am user bandit23" | md5sum | cut -d ' ' -f 1
# → 8ca319486bfbbc3663ea0fbe81326349

cat /tmp/8ca319486bfbbc3663ea0fbe81326349
```

**Password:** `0Zf11ioIjMVN551jX3CmStKLYqjk54Ga`

---

## Bandit 23

**Concept:** The cron job for `bandit24` executes and deletes every script placed in `/var/spool/bandit24/foo/`. Drop a script there that writes bandit24's password to `/tmp`, wait ~1 minute.

```bash
cat /etc/cron.d/* 2>/dev/null | grep bandit24
cat /usr/bin/cronjob_bandit24.sh

# Write the payload script:
echo "cat /etc/bandit_pass/bandit24 > /tmp/dekhoyepass.txt" > /var/spool/bandit24/foo/hello.sh
chmod +x /var/spool/bandit24/foo/hello.sh

# Wait ~60 seconds, then:
cat /tmp/dekhoyepass.txt
```

**Password:** `gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8`

---

## Bandit 24

**Concept:** Brute-force a 4-digit PIN (0000–9999) combined with the password, submitted to port 30002.

```bash
for i in {0..9999}; do echo "gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8 $i"; done | nc localhost 30002
```

**Password:** `iCi86ttT4KSNe1armKiwbQNmB3YJP3q4`

---

## Bandit 25

**Concept:** An SSH key for `bandit26` is provided. The trick is that bandit26's shell is not `/bin/bash`.

```bash
# Use the provided SSH key to login to bandit26
ssh -i bandit26.sshkey bandit26@bandit.labs.overthewire.org -p 2220
```

**Password for bandit26:** `s0773xxkk0MXfdqOfPRVr9L3jJBUOgCZ`

---

## Bandit 26

**Concept:** bandit26's shell is `more` (a pager). Make your terminal window very small so `more` doesn't auto-exit, then escape into vim → spawn a real shell.

```bash
# 1. Resize terminal to ~5 lines tall
# 2. SSH in — more will pause instead of exiting
ssh -i bandit26.sshkey bandit26@bandit.labs.overthewire.org -p 2220

# Inside 'more':
v                              # open vim

# Inside vim:
:e /etc/bandit_pass/bandit26   # confirm bandit26 password
:set shell=/bin/bash
:shell                         # spawn bash as bandit26

# Now run the setuid binary for bandit27:
./bandit27-do cat /etc/bandit_pass/bandit27
```

**Password bandit26:** `s0773xxkk0MXfdqOfPRVr9L3jJBUOgCZ`  
**Password bandit27:** `upsNCc7vzaRDx6oZC6GiR6ERwe1MowGB`

---

## Bandit 27

**Concept:** Clone a git repo over SSH and read the README.

```bash
# Run from local machine or /tmp:
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
cd repo
cat README
```

**Password:** `Yz9IpL0sBcCeuG7m9uQFt8ZNpS4HZRcN`

---

## Bandit 28

**Concept:** The password was committed to a git repo but then redacted. Check the git log for an earlier commit.

```bash
git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
cd repo
git log --oneline
git show <commit-hash-before-redaction>
# or use lazygit → press 4 to browse commit history
```

**Password:** `4pT1t5DENaYuqnqvadYs1oE4QLCdjmJ7`

---

## Bandit 29

**Concept:** The password isn't on the `master` branch — it's on the `dev` branch.

```bash
git clone -b dev --single-branch ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo
cd repo
cat README.md
```

**Password:** `qp30ex3VLz5MDG1n91YowTv4Q8l7CDZL`

---

## Bandit 30

**Concept:** Nothing interesting in commits or branches. The password is stored as a git **tag**.

```bash
git clone ssh://bandit30-git@bandit.labs.overthewire.org:2220/home/bandit30-git/repo
cd repo
git tag
git show secret
```

**Password:** `fb5S2xb7bRyFmAvQYQGEqsbhVyJqhnDy`

---

## Bandit 31

**Concept:** Push a specific file to the remote repo. `.gitignore` blocks `*.txt`, so remove/empty it first.

```bash
git clone ssh://bandit31-git@bandit.labs.overthewire.org:2220/home/bandit31-git/repo
cd repo
echo 'May I come in?' > key.txt
rm .gitignore && touch .gitignore   # clear gitignore
git add .
git commit -m "add: added a key.txt"
git push
# Server responds with the password
```

**Password:** `3O9RfhqyAlVBEZpVb6LYStshZoqoSx5K`

---

## Bandit 32

**Concept:** The shell converts all input to uppercase (UPPERCASE SHELL). `$0` is a special variable that expands to the shell name itself, effectively invoking `/bin/sh` and escaping the restriction.

```bash
$0
cat /etc/bandit_pass/bandit33
```

**Password:** `tQdtbs5D5i2vJwkO8mEyYEyTL8izoeJ0`

---

## Bandit 33

**Concept:** Final level — no more challenges (as of this writing). Congratulations!

```
cat README.txt
# "At this moment, level 34 does not exist yet."
```

---

*Completed at ~4:50 AM. Based on personal notes — commands may vary slightly by session.*
