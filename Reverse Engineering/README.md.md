# A week of Reverse Engineering
In order to participate in MCC 2024, we were given a binary to reverse engineer. Since I did not have any experience with Reverse engineering and binary exploitation, I decided "Hey, why don't I try to learn reverse engineering and solve this challenge in one week?". Oh boy, I overestimated myself, but, at least we learn some things. This README will contain a summary of interesting information I learnt.

Special thanks to [Nora Codes](https://nora.codes/tutorial/an-intro-to-x86_64-reverse-engineering/).

This writeup will be updated as I continue to learn.
# Tools
1. We can use `objdump` to disassemble the binary.
```sh
objdump -M intel -d <crackme>
```
- `-M intel`: Tell `objdump` to use intel syntax
- `-d`: Disassemble mode
2. We can use `xxd` to read specific addresses of the binary.
```sh
xxd -s <offset> -l <bytes> <crackme>
xxd -s 0x2004 -l 0x20 <crackme>
```
- `-s`: Specify the offset
- `-l`: Specify the length of bytes to show after the offset
Output:
```
00002004: 4163 6365 7373 2067 7261 6e74 6564 2100  Access granted!.
00002014: 4163 6365 7373 2064 656e 6965 642e 0045  Access denied..E
```
3. `ltrace` can be used for dynamic analysis to determine C library calls
```sh
ltrace <command>
ltrace ./crackme05.64 password1
```
Output:
```
strnlen(0x7ffffe4ae1f0, 1000, 0x7ffffe4ac7d0, 0x55cb69c55dd8)                                                 = 9
printf("No, %s is not correct.\n", "password1"No, password1 is not correct.
)                                                               = 30
exit(1 <no return ...>
+++ exited (status 1) +++

```
4. `strace` can be used for dynamic analysis to record system calls
```sh
strace <command>
```
Output:
```
execve("./crackme05.64", ["./crackme05.64", "password1"], 0x7ffd3e1414f8 /* 59 vars */) = 0
brk(NULL)                               = 0x55e270dad000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7faa9c3a5000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
<snip>
```
5. Well, of course, you can use ghidra or radare (However, I prefer ghidra)
# Assembly 
1. When a function called, 
	1. the first argument is stored in the `rdi` register
	2. the second argument is stored in the `rsi` register
	3. the third argument is stored in the `rdx` register
	4. For subsequent arguments, refer this [wiki](https://en.wikipedia.org/wiki/X86_calling_conventions#System_V_AMD64_ABI) 
	5. `eax` register may be used before calling a function. It is used to denote the number of extra arguments when calling a variadic function (eg. `printf`).
	6. Example:
```
126d:       48 8b 7b 08             mov    rdi,QWORD PTR [rbx+0x8] 
1271:       ba 09 00 00 00          mov    edx,0x9 
1276:       48 8d 35 df 0d 00 00    lea    rsi,[rip+0xddf]        # 205c <_IO_stdin_used+0x5c> 
127d:       e8 ce fd ff ff          call   1050 <strncmp@plt>
```
- It is equivalent to `strncmp(QWORD PTR [rbx+0x8], [rip+0xddf], 0x9)`
2. The return value of a function is stored in the `rax` register. Thus, if we see this,
```
mov eax,0x0
ret
```
- It means return 0 (No error)
3. Size of data types:

| Type  | Size (bytes) |
| ----- | ------------ |
| char  | 1            |
| int   | 4            |
| word  | 2            |
| dword | 4            |
| qword | 8            |
4. In C, the main function will have at least one argument, `argv[0]` which is equal to the name of the executable.
5. In the main function, initially,
	1. `rdi`, which contains the first argument of a function call will contain `argc` (Argument counter)
	2. `rsi`, which contains the second argument of a function call will contain `argv`. `argv` is an array of variables
6. In C, array variables are pointers!! 
7. Somehow when we index an array, the compiler (I think)  will understand that we are shifting the pointer by 1 element and not 1 byte. Example, `argv[2]` is equal to `argv+2*sizeof(typeof(argv))` (<=pseudocode)
8. `rXX` is for 64-bit registers. `eXX` can refer to 32-bit registers or lower 32-bits of 64-bit registers. There is also `Xl`, `Xh` and so on.
## Hard-to-understand assembly functions:
1. `cdq` is used to zero extend contents of `eax` to `edx:eax`
```
mov rax, 3
mov rdx, 1
cdq
```
- Now, `rax` will contain `00000000 00000000 00000000 00000011` and `rdx` will be all zeroes. 
2. `idiv <register>`: The contents of `eax` will be divided by contents of register specified. The quotient is placed inside `eax` while the remainder is placed inside `edx`
```
cdq ; Let's say edx:eax is 5
idiv   r8d ; Let's say r8d is 2
```
- After this operation, `eax` is 2 and `edx` is 1.
- Usually used together with `cdq` 