# Cracking Passwords with Cheap Hardware at Defcon

There were roughly 30 'Street' teams that participated in Korelogic's [2020 Crack Me If You Can password cracking contest at Defcon](https://contest-2020.korelogic.com/) this year. I took 4th place. 

Some of the teams had two or three team members (some had more) and used multiple high end GPUs. It's always fun to beat these teams using cheap hardware and homemade software. 

## Hardware I used

I used my home desktop to crack the hashes. I built it from parts in August 2019 (new egg). My GPU is rather under powered, but worked OK for most hashes. My CPU is a first generation AMD Ryzen Threadripper (which isn't too shabby but pales in comparison to the third gen Threadripper). In total, I paid about $1,900 US dollars for the parts.

```bash
$ lspci | grep -i nvidia
08:00.0 VGA compatible controller: NVIDIA Corporation GP104 [GeForce GTX 1060 6GB] (rev a1)
```

```bash
$ grep "model name" /proc/cpuinfo | sort -u
model name : AMD Ryzen Threadripper 1950X 16-Core Processor
```

## Software I used

 * Word Machine (wm)
 * John the Ripper (John)
 * Hashcat

## Hash Types

 * type=100  # sha1
 * type=111  # openssha (salted-sha1)
 * type=112  # oracle11
 * type=132  # mssql05
 * type=400  # phpass
 * type=1400 # raw-sha256
 * type=1700 # raw-sha512
 * type=1731 # mssql12
 * type=1800 # sha512crypt
 * type=3711 # mediawiki
 * type=7400 # sha256crypt
 * type=10900 # PBKDF2-HMAC-SHA256
 * type=23100 # Apple keychain

## My Approach

I used wm (my homemade software) to generate common password patterns (appends, prepends, inserts, etc.). The generated patterns were piped to Hashcat or John via stdin for cracking. Once I found a pattern for a certain user or type of hash, then I configured wm to focus on that particular pattern for awhile.

For example, the sha512crypt hashes were largely made up of two words ('fuck' and 'shit') with common numeric appends and prepends. Once wm discovered that, I began focusing on just those two words and that specific pattern. I think I cracked about 33% of the sha512crypts on the first day. Due to this, I took the lead and held it for awhile. sha512crypt is a hard, slow hash to attack and many teams didn't bother trying. While simple cryptographic hashes, such as sha1, are much faster to attack, they aren't worth hardly any points. A raw-sha1 crack earned 1 point. A sha512crypt crack earned 800,000 points. I cracked 220 of them.

As another example, wm discovered 'Dallas214' in the sha256crypt hashes on the last day of the contest. Having more than a dozen hash types to attack and being a one man team (with only one under powered GPU) makes it difficult to discover and then spend time focusing on all the patterns in the various hash types. I wish I had discovered this pattern sooner. I cracked 88% of them, but only 53% were accepted due to some of the passwords being expired/changed on the second day of the contest (no credit for cracking them). Anyway, these were mostly cities with area codes appended.

The keychain hashes were capitalized passphrases such as 'Abidewithme', 'Whenpigsfly', and 'Sonofagun'. wm quickly discovered that pattern and focused on it early on. I got some good points for these cracks, however, my version of Hashcat did not recognize the keychain hashes, so I had to pipe to John (use the CPU which is even slower) to crack these. 

For some reason, Hashcat did not recognize all the the PBKDF2-HMAC-SHA256 hashes. There were 1,000 but Hashcat only saw 528. These hashes were really too difficult for John and my CPU to handle. I did uncover the pattern (after the contest had ended) and cracked most of them. 

Update: The hash/digest portion of 472 of these hashes contained periods (which is an invalid base64 character). The salts were all OK and valid. I wrote a small Go program to figure this out and correct it. I'm not sure if this was intentional or an accident. Either way, kudos to the teams who figured this out and cracked these during the contest. My failure to crack these during the contest cost me 3rd place.

## Conclusion

The contest was fun. It always is. It's a great way to learn more about hacking, cracking and encryption. I encourage everyone who likes to write code and fiddle with computers to try it next year. A big thanks to Hank and the Korelogic team for putting it on this year despite the pandemic.
