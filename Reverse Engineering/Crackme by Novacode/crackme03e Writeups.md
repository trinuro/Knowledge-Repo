1. This is from https://github.com/noracodes/crackmes. Special thanks to `Noracode`!
2. To create the executable,
```sh
make crackeme03e
```
# Triage
1. Let's see what is the output of file command.
```
file crackme03e.64
```
Output:
```
crackme03e.64: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6772f181c36e1b3c5ec0a448693094c5c681a01a, for GNU/Linux 3.2.0, not stripped
```
2. Let's see the output of checksec
```sh
checksec --file=crackme03e.64 --output=json | jq
```
Output:
```json
{
  "crackme03e.64": {
    "relro": "partial",
    "canary": "no",
    "nx": "yes",
    "pie": "yes",
    "rpath": "no",
    "runpath": "no",
    "symbols": "yes",
    "fortify_source": "no",
    "fortified": "0",
    "fortify-able": "1"
  }
}
```
3. Let's see what library functions were called.
```sh
ltrace ./crackme03e.64 abc          
```
Output:
```
strlen("abc")                                                                                                 = 3
printf("No, %s is not correct.\n", "abc"No, abc is not correct.
)                                                                     = 24
+++ exited (status 1) +++

```
- Interesting, it tests the length of the string first
4. Let's see whether there is any interesting system call.
```sh
strace ./crackme03e.64 abc
```
- Nothing really interesting.
## Static analysis
1. Now, let's read the assembly.
```sh
objdump -d crackme03e.64 -Mintel | less
```
### Main
```
000000000000118d <main>:
    118d:       53                      push   rbx
    118e:       48 83 ec 20             sub    rsp,0x20
    1192:       83 ff 02                cmp    edi,0x2
    1195:       75 6f                   jne    1206 <main+0x79>
    1197:       48 b8 70 61 73 73 77    movabs rax,0x64726f7773736170
    119e:       6f 72 64 
    11a1:       48 89 44 24 17          mov    QWORD PTR [rsp+0x17],rax
    11a6:       c6 44 24 1f 00          mov    BYTE PTR [rsp+0x1f],0x0
    11ab:       48 b8 03 05 02 04 01    movabs rax,0x103000104020503
    11b2:       00 03 01 
    11b5:       48 89 44 24 0e          mov    QWORD PTR [rsp+0xe],rax
    11ba:       c6 44 24 16 00          mov    BYTE PTR [rsp+0x16],0x0
    11bf:       48 8b 5e 08             mov    rbx,QWORD PTR [rsi+0x8]
    11c3:       48 89 df                mov    rdi,rbx
    11c6:       e8 75 fe ff ff          call   1040 <strlen@plt>
    11cb:       48 83 f8 08             cmp    rax,0x8
    11cf:       75 16                   jne    11e7 <main+0x5a>
    11d1:       48 8d 54 24 0e          lea    rdx,[rsp+0xe]
    11d6:       48 8d 74 24 17          lea    rsi,[rsp+0x17]
    11db:       48 89 df                mov    rdi,rbx
    11de:       e8 76 ff ff ff          call   1159 <check_pw>
    11e3:       85 c0                   test   eax,eax
    11e5:       75 32                   jne    1219 <main+0x8c>
    11e7:       48 89 de                mov    rsi,rbx
    11ea:       48 8d 3d 43 0e 00 00    lea    rdi,[rip+0xe43]        # 2034 <_IO_stdin_used+0x34>
    11f1:       b8 00 00 00 00          mov    eax,0x0
    11f6:       e8 55 fe ff ff          call   1050 <printf@plt>
    11fb:       b8 01 00 00 00          mov    eax,0x1
    1200:       48 83 c4 20             add    rsp,0x20
    1204:       5b                      pop    rbx
    1205:       c3                      ret
    1206:       48 8d 3d f7 0d 00 00    lea    rdi,[rip+0xdf7]        # 2004 <_IO_stdin_used+0x4>
    120d:       e8 1e fe ff ff          call   1030 <puts@plt>
    1212:       b8 ff ff ff ff          mov    eax,0xffffffff
    1217:       eb e7                   jmp    1200 <main+0x73>
    1219:       48 89 de                mov    rsi,rbx
    121c:       48 8d 3d fc 0d 00 00    lea    rdi,[rip+0xdfc]        # 201f <_IO_stdin_used+0x1f>
    1223:       b8 00 00 00 00          mov    eax,0x0
    1228:       e8 23 fe ff ff          call   1050 <printf@plt>
    122d:       b8 00 00 00 00          mov    eax,0x0
    1232:       eb cc                   jmp    1200 <main+0x73>
```
1. Let's start from the beginning.
```
    118d:       53                      push   rbx ; save the current value of rbx
    118e:       48 83 ec 20             sub    rsp,0x20 ; increase the size of the stack by 16*2=32 bytes
    1192:       83 ff 02                cmp    edi,0x2 ; check if first argument is 2
    1195:       75 6f                   jne    1206 <main+0x79> ; jump here if first argument is not 2
```
- The first argument of a main function is always `argc`. 
- In main, `argv[0]` is always the name of the program. Thus, the value of `argc`>=1. In this case, it is checking if one argument is passed into the command line
2. Let's see what happens if one argument is not passed into the program in the command line.
```
    1206:       48 8d 3d f7 0d 00 00    lea    rdi,[rip+0xdf7]        # 2004 <_IO_stdin_used+0x4> ; load a pointer to rdi as the first argument of a function
    120d:       e8 1e fe ff ff          call   1030 <puts@plt> ; call puts(rdi)
    1212:       b8 ff ff ff ff          mov    eax,0xffffffff ; eax = -1
    1217:       eb e7                   jmp    1200 <main+0x73> ; jump to 1200
```
- Let's see what 0x2004 contains.
```sh
xxd -s 0x2004 -l 0x20 crackme03e.64
```
Output:
```
00002004: 4e65 6564 2065 7861 6374 6c79 206f 6e65  Need exactly one
00002014: 2061 7267 756d 656e 742e 0059 6573 2c20   argument..Yes
```
- This is definitely not good
3. Let's continue at line 1200
```
    1200:       48 83 c4 20             add    rsp,0x20 ; decrease the size of the stack by 32 bytes
    1204:       5b                      pop    rbx ; restore value of rbx
	1205:       c3                      ret ; return -1 (eax is -1)
```
4. Let's continue at line 1197
```
    1197:       48 b8 70 61 73 73 77    movabs rax,0x64726f7773736170
    119e:       6f 72 64 ; basically, store 8 bytes in rax
    11a1:       48 89 44 24 17          mov    QWORD PTR [rsp+0x17],rax  ; store this 8 bytes into rsp+23
    11a6:       c6 44 24 1f 00          mov    BYTE PTR [rsp+0x1f],0x0 ; put 00 byte into rsp+31
    11ab:       48 b8 03 05 02 04 01    movabs rax,0x103000104020503
    11b2:       00 03 01 ; store 8 bytes in rax
    11b5:       48 89 44 24 0e          mov    QWORD PTR [rsp+0xe],rax ; store 8 bytes in rsp+14
    11ba:       c6 44 24 16 00          mov    BYTE PTR [rsp+0x16],0x0 ; insert null byte in rsp+22
    11bf:       48 8b 5e 08             mov    rbx,QWORD PTR [rsi+0x8] ; rbx = *(rsi+8)/ rbx = argv[1]
    11c3:       48 89 df                mov    rdi,rbx ; first argument is rbx
    11c6:       e8 75 fe ff ff          call   1040 <strlen@plt> ; strlen(argv[1])
```
- Apparently, `movabs` is the version of `mov` to move 64 bits/ 8 bytes.
- I see, it is inserting a null-terminated string containing 8 characters into rsp+23 and rsp
- `rsi` stores the second argument of any function, in this case, main. `rsi` will contain a pointer to `argv`
- Let's decode the string stored in `rsp+23`
```python
>>> bytearray.fromhex('64726f7773736170').decode()[::-1]
'password'
```
- `[::-1]` is used to reverse the string because it is little-endian
- Let's decode the string stored in `rsp+14`
```python
bytearray.fromhex('0103000104020503').decode()[::-1]
'\x03\x05\x02\x04\x01\x00\x03\x01'
```
- Ok, interesting `rsp` contains bytes that are not ASCII
5. Let's continue:
```
    11cb:       48 83 f8 08             cmp    rax,0x8 ; check if rax = 8
    11cf:       75 16                   jne    11e7 <main+0x5a> ; if it is not 8, jump here
```
- `rax` register will contain the return value of any function
6. Let's jump to 11e7.
```
    11e7:       48 89 de                mov    rsi,rbx
    11ea:       48 8d 3d 43 0e 00 00    lea    rdi,[rip+0xe43]        # 2034 <_IO_stdin_used+0x34>
    11f1:       b8 00 00 00 00          mov    eax,0x0 ; eax =0. Tells variadic function that there is no additional parameter
    11f6:       e8 55 fe ff ff          call   1050 <printf@plt> ; printf("No, %s is not correct", argv[1])
    11fb:       b8 01 00 00 00          mov    eax,0x1 ; eax=1
    1200:       48 83 c4 20             add    rsp,0x20 ; reduce size of rsp
    1204:       5b                      pop    rbx
    1205:       c3                      ret ; return 1 because eax = 1
```
- Let's see the string at 2034
```
 xxd -s 0x2034 -l 0x20 crackme03e.64
00002034: 4e6f 2c20 2573 2069 7320 6e6f 7420 636f  No, %s is not co
00002044: 7272 6563 742e 0a00 011b 033b 3000 0000  rrect......;0...
```
- This is a dead end
7. Let's continue at 11d1
```
    11d1:       48 8d 54 24 0e          lea    rdx,[rsp+0xe] ; third argument = "\x03\x05\x02\x04\x01\x00\x03\x01"
    11d6:       48 8d 74 24 17          lea    rsi,[rsp+0x17] ; second argument = 'password'
    11db:       48 89 df                mov    rdi,rbx ; first argument = argv[1]
    11de:       e8 76 ff ff ff          call   1159 <check_pw> ;
    check_pw(argv[1], "password", "\x03\x05\x02\x04\x01\x00\x03\x01")
    11e3:       85 c0                   test   eax,eax ; check if the returned value is 0
    11e5:       75 32                   jne    1219 <main+0x8c> ; jump here if not 0
```
8. Let's see what happens when return value is not 0.
```
    1219:       48 89 de                mov    rsi,rbx ; second argument is argv[1]
    121c:       48 8d 3d fc 0d 00 00    lea    rdi,[rip+0xdfc]        # 201f <_IO_stdin_used+0x1f>
    1223:       b8 00 00 00 00          mov    eax,0x0 ; eax = 0
    1228:       e8 23 fe ff ff          call   1050 <printf@plt> ; printf("Yes, %s is correct!", argv[1])
    122d:       b8 00 00 00 00          mov    eax,0x0 ; eax = 0
    1232:       eb cc                   jmp    1200 <main+0x73>
```
- Let's see what string is at location 201f
```sh
xxd -s 0x201f -l 0x20 crackme03e.64
0000201f: 5965 732c 2025 7320 6973 2063 6f72 7265  Yes, %s is corre
0000202f: 6374 210a 004e 6f2c 2025 7320 6973 206e  ct!..No, %s is n
```
- Location 1200 will just return 0 (Successful execution)
- This is where we want to reach.
9. So, basically we need to supply a string that is 8 characters long and then satisfy the condition `check_pw`
### check_pw
```
0000000000001159 <check_pw>:
    1159:       b8 00 00 00 00          mov    eax,0x0 ; eax = 0
    115e:       0f b6 0c 02             movzx  ecx,BYTE PTR [rdx+rax*1] ; rdx is the third argument. Equivalent to cl = (char) third_argument[rax] 
    1162:       02 0c 06                add    cl,BYTE PTR [rsi+rax*1]
    ; rsi is the second argument. Equivalent to cl += (char) second_argument[rax]
    1165:       38 0c 07                cmp    BYTE PTR [rdi+rax*1],cl ; rdi is the first argument. Compare the rax-th byte of first_argument
    1168:       75 17                   jne    1181 <check_pw+0x28>
```
- I can sense a loop here
2. Let's continue at 1181
```
    1181:       b8 00 00 00 00          mov    eax,0x0
    1186:       c3                      ret
```
- Basically return 0, which is not what I want
3. Let's continue at 116a
```
    116a:       80 7c 06 01 00          cmp    BYTE PTR [rsi+rax*1+0x1],0x0 ; basically checks if the next byte in the second argument is null
    116f:       74 16                   je     1187 <check_pw+0x2e>
```
4. Let's jump to 1187
```
    1187:       b8 01 00 00 00          mov    eax,0x1
    118c:       c3                      ret
```
- Basically returns 1. 
- This is the winning condition
5. Let's continue after 116f
```
    1171:       48 83 c0 01             add    rax,0x1 ; increase rax by 1
    1175:       80 3c 07 00             cmp    BYTE PTR [rdi+rax*1],0x0 ; Checks if the current character in the string is a null byte
    1179:       75 e3                   jne    115e <check_pw+0x5> ; loop if not a null byte
    117b:       b8 01 00 00 00          mov    eax,0x1
    1180:       c3                      ret ; return 1 if the end of first argument is reached.
```
6. In conclusion, this will be the pseudo-code of the challenge
```c
int check_pw(char * first_argument, char[] second_argument, char[] third_argument){
	for(int i = 0; second_argument[i] && first_argument[i]; i++){
		char sum = second_argument[i] + third_argument[i];
		if(sum!=first_argument[i]){
			return 0;
		}
	}
	return 1;
}
```
# Solution
1. First, we need to create a function that splits a string.
```python
>>> def split(s:str, k:int) -> list:
...     return [s[i:i+k] for i in range(0, len(s), k)]
... 
```
2. Then, we split the byte array and invert it because it is little endian
```python
>>> split('64726f7773736170', 2)[::-1]
['70', '61', '73', '73', '77', '6f', '72', '64']
>>> split('0103000104020503', 2)[::-1]
['03', '05', '02', '04', '01', '00', '03', '01']
>>> addition = split('0103000104020503', 2)[::-1]
>>> originalString = split('64726f7773736170', 2)[::-1]
```
3. Now, we add the first byte of originalString list with that of addition list
```python
>>> for i in range(len(originalString)):
...     print(bytearray.fromhex(hex(int(originalString[i], 16)+int(addition[i],16))[2:]).decode(), end='')
...
sfuwxoue
```
- It is also 8 characters long
4. Let's try this payload.
```sh
./crackme03e.64 'sfuwxoue'     
Yes, sfuwxoue is correct!
```