1. This is from https://github.com/noracodes/crackmes. Special thanks to Noracode!
2. To create the executable,
```sh
make crackeme01
```
# Triage
1. Let's gather information of the file type.
```sh
file crackme01.64
```
Output:
```
crackme01.64: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=d360d5e1fca9241ecd09e324eb93b78200496294, for GNU/Linux 3.2.0, not stripped
```
- It is a 64 bit binary
- Symbols still intact 
- It is dynamically linked, which means it will call a function in the shared library.

2. Let's look at the output of checksec.
```sh
checksec --file=crackme01.64 --output=json | jq
```
Output:
```json
{
  "crackme01.64": {
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
- Interesting that canary is not enabled.
3. `strings` show that `strncmp` is called
```
...
__cxa_finalize
__libc_start_main
strncmp
puts
printf
libc.so.6
...
Need exactly one argument.
password1
No, %s is not correct.
Yes, %s is correct!
...
```
- `password1` seems extremely suspicious
# Dynamic analysis
1. When I run it, 
```sh
./crackme01.64
Need exactly one argument.

./crackme01.64 123
No, 123 is not correct.
```
- This programs need 1 argument.
2. Let's detect what library functions were called during execution.
```sh
ltrace ./crackme01.64 123
```
Output:
```
strncmp("123", "password1", 9)                                                                                = -63
printf("No, %s is not correct.\n", "123"No, 123 is not correct.
)                                                                     = 24
+++ exited (status 1) +++
```
- Hohoho ok, `strncmp` is called! It is obvious that is comparing the input to `password1`
3. To solve the challange,
```sh
./crackme01.64 password1   
```
Output:
```
Yes, password1 is correct!
```