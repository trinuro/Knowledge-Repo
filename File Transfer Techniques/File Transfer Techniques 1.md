# Introduction
1. File transfer is the transferring of files in between two machines.
2. File transfer is important when you want to transfer certain files to the computer (Shell scripts, other malwares) or you want to download certain files to our computer (For example, credential files, the NTDS.dit file in Active Directory or a shadow copy of the LSASS.exe)
3. In this post, I will assume that our attacker host is a Linux machine while the other machine is a Windows machine.

# Base 64
1. This technique converts all strings into a hexadecimal representation and takes advantage of the clipboard.
2. This is very useful if we need a quick and dirty way to transfer programs between machines. We do not need to set up any port/service to listen for incoming connections or send data out of.
3. Another benefit is this technique can usually bypass firewalls.

# Linux -> Windows 
1. To base 64 encode a string,
```sh
base64 "<string>" -w 0
```
- `-w 0`: Disable line wrapping
5. To base64 encode a file,
```sh
base64 shell.txt -w 0
```
6. Note, you can pipe the output into a file if you desire to do so.
```sh
base64 "string" -w 0 | tee base64encoded.txt
```
- `tee`: Prints the output onto the terminal and store it in a file at the same time
7. After that, we can copy the string onto our clip board (`CTRL-SHIFT-C`) and paste the string in the target host. 
8. On the Windows machine, to convert a base-64 string back to a normal string,
```powershell
[IO.File]::WriteAllBytes("C:\shell.txt", [Convert]::FromBase64String("<Base-64 String>"))
```
- `[Convert]::FromBase64String`: Command to convert base 64 string to normal string
- `[IO.File]::WriteAllBytes`: Command to write to a file
- This command will first decode the base-64 string. Then, it will write the output to a file.

# Windows -> Linux
1. To base-64 encode a file on Windows machine.
```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("password.txt"))
```
- `[IO.File]::ReadAllBytes`: Read a file 
- `[Convert]::ToBase64String`: Base 64 encode a string
- This command will read a file and print out the base-64 encoded version of the file
2. We can copy (`CTRL-C`) the string and paste it into our linux host
```sh
base64 -d "<encoded-string>" | tee shell.txt
```
- `base64 -d`: Decode the base64 string
- `tee shell.txt`: Prints the output to terminal and copies it into the `shell.txt`
