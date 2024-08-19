# Dictionary Attack (`-a 0`)
1. In dictionary attacks, we generate hashes from words in the wordlist provided and compare it with the hash given
2. To run a dictionary attack,
```sh
hashcat -a 0 -m <hash type> <hash file> <wordlist>
```
- `-a 0`: Attack mode 0. It means dictionary attacks
- `-m`: Specify hash type

# Combination Attack (`-a 1`)
1. The combination attack modes take in two wordlists as input and creates combinations from them.
	1. This is because it is common for users to join two or more words like `fsktm1234`
2. To figure out what combination of words is used,
```sh
hashcat -a 1 --stdout file1 file2
```
- `-a 1`: Enable attack mode 1 - Combination attack
- `--stdout`: Print the combination of words produced on to the terminal
Output:
```
$ cat wordlist1                            
jack
superman
jane
joe
 
$ cat wordlist2
hanma
dooper
smith
harris

$ hashcat -a 1 --stdout file1 file2
jackhanma
jackdooper
jacksmith
jackharris
supermanhanma
supermandooper
supermansmith
supermanharris
janehanma
janedooper
janesmith
janeharris
joehanma
joedooper
joesmith
joeharris
```
3. To carry out combination attack,
```sh
hashcat -a 1 -m <hashtype> <hash> <wordlist1> <wordlist2>
```

# Mask Attack (`-a 3)
1. Mask attacks are used to generate words that match a specific pattern.
	1. Particularly useful when the password length is known
2. Placeholder characters are used to generate patterns.

| **Placeholder** | **Meaning**                                             |
| --------------- | ------------------------------------------------------- |
| ?l              | lower-case ASCII letters (a-z)                          |
| ?u              | upper-case ASCII letters (A-Z)                          |
| ?d              | digits (0-9)                                            |
| ?h              | lowercase hex (0123456789abcdef)                        |
| ?H              | uppercase hex (0123456789ABCDEF)                        |
| ?s              | special characters («space»!"#$%&'()*+,-./:;<=>?@[]^_`{ |
| ?a              | ?l?u?d?s                                                |
| ?b              | 0x00 - 0xff                                             |
| `[0-1]`         | Only characters 1 or 2                                  |
3. To define a custom character set, use `-1` to `-4` option
```sh
hashcat -a 3 -m <hashtype> <hash> -1 ?l?d '?1?1?1?1?s123'
```
- `-a 3`: Attack mode 3. Carry out mask attack
- `-1 ?l?d`: Create a custom character sets containing alphanumeric characters only `[0-9][a-f]`
- `?1` in this case is equivalent to one alphanumeric character
- Example outputs include `1d2x?123` and `35g1!123` 
3. We can use `--increment` to increase the mask length.
	1. It will start with the mask "character" = 1, then slowly increase the length of mask

# Hybrid mode (`-a 6` or `-a 7`)
1. Hybrid mode can be used to combine wordlists and masks.
2. Attack mode 6 is used to <u>append</u> the mask to the wordlist
	1. Used for wordlist + mask (Mask behind the wordlist)
	2. To use attack mode 6,
```sh
hashcat -a 6 -m 0 <hash> <wordlist> '?d?d?d?d'
```
- `-a 6`: Specify hybrid attack mode
- `-m 0`: Crack MD5 hash
- This command will append 4 decimal letters behind the wordlist. Examples include `football2019`
3. Attack mode 7 can be used to <u>prepend</u> characters to words using a given mask.
	1. Used for mask + wordlist
	2. To use attack mode 7,
```sh
hashcat -a 7 -m 0 <hash> <wordlist> '?d?d?d?d'
```
- `-a 6`: Specify hybrid attack mode
- `-m 0`: Crack MD5 hash
- This command will append 4 decimal letters in front the wordlist. Examples include `2019football`