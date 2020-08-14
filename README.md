# Cracking Passwords with Cheap Hardware at Defcon

There were roughly 30 Street teams that participated in Korelogic's [2020 Crack Me If You Can](https://contest-2020.korelogic.com/) password cracking contest at Defcon this year. [I took 4th place](https://contest-2020.korelogic.com/graphs-street.html).

Some of the teams had two or three team members (some had more) and used multiple high end GPUs. It's always fun to compete with these teams using cheap hardware and homemade software. I normally place 3rd, 4th or 5th. I won in [2013](https://contest-2013.korelogic.com/teams.html). If I do it again next year, I may try to find a team to join. I think it's best to work with a few team members so the work can be divided up a bit.

## Hardware I used

I used my home desktop to crack the hashes. It's just over a year old. I built it from parts in July 2019. I used the GPU to do most of the cracking. It's under powered, compared to high end units, but worked OK for most hashes. My CPU is a first generation AMD Ryzen Threadripper (which isn't too shabby but pales in comparison to the third gen Threadripper). I only used the CPU to crack the keychain hashes (see explanation below). In total, it cost about $1,900 US dollars to build this computer.

```bash
$ lspci | grep -i nvidia
08:00.0 VGA compatible controller: NVIDIA Corporation GP104 [GeForce GTX 1060 6GB]
```
```bash
$ grep "model name" /proc/cpuinfo | sort -u
model name : AMD Ryzen Threadripper 1950X 16-Core Processor
```

## Software I used

 * Word Machine (wm)
 * [John the Ripper (John)](https://www.openwall.com/john/)
 * [Hashcat](https://hashcat.net/hashcat/)

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

I used word machine (my homemade software written in Go) to generate common password patterns (appends, prepends, inserts, etc.). The generated patterns were piped to Hashcat or John via stdin for cracking. Once I found a pattern for a certain user or type of hash, then I configured wm to focus on that particular pattern for awhile. Below are a few examples.

The sha512crypt hashes I cracked were largely made up of two words ('fuck' and 'shit') with common numeric appends and prepends. Once wm discovered that, I began focusing on just those two words and that specific pattern. I think I cracked about 1/3 of the sha512crypts on the first day of the contest. Because of this, I took the lead and held it for awhile.

sha512crypt is a hard, slow hash to attack and many teams didn't bother trying. While simple cryptographic hashes, such as sha1, are much faster to attack, they were not worth many points. A raw-sha1 crack earned only 1 point. A sha512crypt crack earned 800,000 points. I cracked 220 of them.

Word Machine discovered 'Dallas214' in the sha256crypt hashes on the last day of the contest. Having more than a dozen hash types to attack and being a one man team (with only one inexpensive GPU) makes it difficult to discover and then spend time focusing on all the patterns in the various hash types. I wish I had discovered this pattern sooner. I cracked 88% of them, but only 53% were accepted due to some of the passwords being expired/changed on the second day of the contest (no credit for cracking them). Anyway, these were mostly cities with area codes appended.

The keychain hashes were capitalized passphrases such as 'Abidewithme', 'Whenpigsfly', and 'Sonofagun'. word machine quickly discovered that pattern and I was able to focus on it early on. I got a lot of points for these cracks, however, my version of Hashcat did not recognize the keychain hashes, so I had to pipe to John (use the CPU) to crack these. That made this attack even slower. 

For some reason, Hashcat did not recognize all the the PBKDF2-HMAC-SHA256 hashes. There were 1,000 but Hashcat only saw 528. These hashes were really too difficult for John and my CPU to handle. I did uncover the pattern (after the contest had ended) and cracked most of them. **Update:** The hash/digest portion of 472 of the PBKDF2-HMAC-SHA256 hashes contained periods (which is an invalid base64 character). The salts were all valid. I wrote a small Go program to figure this out and correct it. I'm not sure if this was intentional or an accident. Either way, kudos to the teams who figured this out and cracked these during the contest. My failure to crack these (during the contest) is the main reason I was 4th instead of 3rd.

## Conclusion

The contest was fun. It always is. It's a great way to learn more about hacking, cracking and encryption. I encourage everyone who likes to write code and fiddle with computers to try it next year. A big thanks to Hank and the Korelogic team for putting on the contest this year despite the pandemic.

## The Hashes and My Pot Files

Some of these hashes were cracked during the contest while others were cracked after it ended. I like to crack some of each type so that I understand the patterns that were used to create the passwords. I've lost a few cracks, too, since the contest ended. Also, the hash file contains the fixed PBKDF2-HMAC-SHA256 hashes and the originals (oracle11 too). 

  * [kl.jtr.pot.txt](kl.jtr.pot.txt)
  * [kl.hc.pot.txt](kl.hc.pot.txt)
  * [hashes.txt](hashes.txt)
