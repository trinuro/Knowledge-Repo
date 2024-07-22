1. One of the reasons people have a hard time learning hashcat is because they do not know how to identify the hash mode.
2. In this writeup, I will write about a few ways to detect types of hash given a hash. I have divided the writeup into two cases:
	1. Given known hash, get the mode
	2. Given unknown hash, get the mode 

# Given known hash, get mode
For example, you know that the hash is a NTLM hash. However, what hash mode should we use to crack the hash?
## Hashcat Example Hashes
1. Hashcat has a list of example hashes that maps hashes with their types and hash mode.
2. It can be accessed in the [Hashcat website](https://hashcat.net/wiki/doku.php?id=example_hashes) 
3. However, if the hashcat website is blocked by your ISP or university administrator or you just like to use CLI (both true in my case :) ), you can actually get the same content using the `--example-hashes` option
```sh
hashcat --example-hashes
```
Output:
```
hashcat (v6.2.6) starting in autodetect mode

Hash Info:
==========

Hash mode #0
  Name................: MD5
  Category............: Raw Hash
  Slow.Hash...........: No
  Password.Len.Min....: 0
  Password.Len.Max....: 256
  Kernel.Type(s)......: pure, optimized
  Example.Hash.Format.: plain
  Example.Hash........: 8743b52063cd84097a65d1633f5c74f5
  Example.Pass........: hashcat
  Benchmark.Mask......: ?b?b?b?b?b?b?b
  Autodetect.Enabled..: Yes
  Self.Test.Enabled...: Yes
  Potfile.Enabled.....: Yes
  Custom.Plugin.......: No
  Plaintext.Encoding..: ASCII, HEX

Hash mode #10
  Name................: md5($pass.$salt)
  Category............: Raw Hash salted and/or iterated
  Slow.Hash...........: No
  Password.Len.Min....: 0
  Password.Len.Max....: 256
  Salt.Type...........: Generic
  Salt.Len.Min........: 0
  Salt.Len.Max........: 256
  Kernel.Type(s)......: pure, optimized
  Example.Hash.Format.: plain
  Example.Hash........: 3d83c8e717ff0e7ecfe187f088d69954:343141
  Example.Pass........: hashcat
  Benchmark.Mask......: ?b?b?b?b?b?b?b
  Autodetect.Enabled..: Yes
  Self.Test.Enabled...: Yes
  Potfile.Enabled.....: Yes
  Custom.Plugin.......: No
  Plaintext.Encoding..: ASCII, HEX

...
```
4. I like to pipe it to `less` and search for it in a `vim`-like way
```sh
hashcat --example-hashes | less
```
- To search for a pattern, type `/<pattern>` to find the occurrence of the pattern
5. Alternatively, `grep` is also good.
```sh
hashcat --example-hashes | grep -i sha1
```
- `-i`: Ignore case (Recommended)
# Given unknown hash, get mode

## `hashid`
1. `hashid` is a python module that can automagically detect the possible types of hash for us.
2. To search for a pattern,
```sh
hashid '0c67ac18f50c5e6b9398bfe1dc3e156163ba10ef' -m               
```
- `-m`: Show hashcat mode 
Output:
```
Analyzing '0c67ac18f50c5e6b9398bfe1dc3e156163ba10ef'
[+] SHA-1 [Hashcat Mode: 100]
[+] Double SHA-1 [Hashcat Mode: 4500]
[+] RIPEMD-160 [Hashcat Mode: 6000]
[+] Haval-160 
[+] Tiger-160 
[+] HAS-160 
[+] LinkedIn [Hashcat Mode: 190]
[+] Skein-256(160) 
[+] Skein-512(160) 
```
- As you can see, `hashid` is not able to detect the type of hash to a 100 percent confidence
- I recommend that if this happens, you should try each hash mode, starting with the easiest hash to crack. In this case, it is `SHA-1`.
3. Examples of hashes that are easy to crack `NTLM`, `MD5` and `SHA-1`
4. Note that `hashid` github repo has not been updated for a long time. However, it should be good enough to detect simple hashes 
## `hashcat --identify`
1. `hashcat` has an option to identify hashes too. I use this as my second resort.
2. To use hashcat to identify hashes,
```sh
hashcat --identify '$pkzip$1*2*2*0*26*1a*d0ced23b*0*43*0*26*7ef8*b154046595e5f738ad20bd1cda08958a8814bd6c6153218183c0496d728da36461c0c7b77e1c*$/pkzip$'
```
Output:
```
The following 2 hash-modes match the structure of your input hash:

#     | Name                         | Category
======+==============================+===============
17225 | PKZIP (Mixed Multi-File)     | Archive
17210 | PKZIP (Uncompressed)         | Archive
```
- Try the hash modes one-by-one
