# Introduction
1. Hashes are one-way functions that converts a string into another string, called hash. A good hash will not allow people to convert the hash back into the string (easily) and takes a long time to generate from the input string
2. Hashcat is a program that is used to crack hashes, that is we generate hashes from best guesses and compare it with the target hashes.

# General Syntax
```sh
hashcat -a <attack-mode> -m <hashing-mode> <hash> <dictionary>/<mask>
```

1. `-a` or `--attack-mode`
	1. Tells `hashcat` how to crack passwords 
	2. For example, using a dictionary of words or brute-force
		1. `0` is used for dictionary attack
		2. `1` is used for Combination mode
		3. `3` is used for Brute-force
		4. `6` is used for Hybrid Wordlist + Mask
		5. `7` is used for Hybrid Mask + Wordlist
	3. Refer [Hashcat 2 - Detect hash type and mode](./Hashcat%202%20-%20Detect%20hash%20type%20and%20mode.md)
2. `-m` or `--hash-mode`
	1. Tells hashcat what type the hash is. For example, MD5, SHA1, etc
	2. Full list of hash types can be found by using `--example-hashes`
	3. For example,`-m 1000` for Windows NTLM Hashes
3. `[filename|hash]`
	1. Specify a file containing hash(es) you intend to crack.
4. `[dictionary|mask|directory]`
	1. Specifies a directory (wordlist), mask, or directory to be used