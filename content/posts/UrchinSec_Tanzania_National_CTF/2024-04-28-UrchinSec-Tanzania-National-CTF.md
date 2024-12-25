---
layout: post
title: UrchinSec Tanzania National CTF MMXXIV 
date: 2024-04-28
categories:
  - Ctf
tags:
  - Linux
  - ReverseEngineering
  - Patching
  - Crypto
keywords:
  - ""
  - ""
description: ""
showFullContent: false
images: []
---




This blogpost is a walkthrough of the UrchinSec Tanzania National CTF challenges. The UrchinSec Tanzania National CTF was an interesting  ctf organized by the <a href="https://twitter.com/urchinsec_" target="_blank" rel="noopener">[urchinsec](https://twitter.com/urchinsec_)</a> team . From the ctf, I managed to solve several challenges one of which was a 500 point reverse engineering challenge. 

<!--more-->
{{< image src="/images/UrchinSec_Tanzania_National_CTF/banner.png" alt=" " position="center" style="border-radius: 8px;" >}}

# Catalog
* [Hatari](#hatari)
* [Attachment](#attachment)
* [WormHole](#wormhole)

## Hatari
{{< image src="/images/UrchinSec_Tanzania_National_CTF/hatari_desc.png" alt=" " position="center" style="border-radius: 8px;" >}}

This challenge is a Hard rated reverse engineering challenge. Before even going any further, my approach for solving this challenge was not the intended one, but regardless, the end justifies the means. In this challenge , we are provided with two files, the `hatari` elf binary and an `enc.bin` file containing the encrypted flag , which is encrypted using the binary.


Checking the contents of the `enc.bin` file , we find that it contains the encrypted flag 
```powershell
pqy~~sqxyu}pqp{pR{yQi{i|xzq{`~~q{na{sRp{aQypqhaaht
```
We can run the binary to see how the encryption works . Using the binary to encrypt the string `test` gives us the string `pxqp`. Also encrypting the string twice using the binary gives the same result, which is a good thing for us.

![](/images/UrchinSec_Tanzania_National_CTF/hatari_2.png)

From the ctf instructions , we know that the flag format is `urchinsec{xxxxxx}`. We could try to encrypt a dummy flag and see whether the encrypted text we get, matches with the encrypted flag in the `enc.bin` file

![](/images/UrchinSec_Tanzania_National_CTF/hatari_1.png)

A quick oneliner to achieve the same is as follows
![](/images/UrchinSec_Tanzania_National_CTF/hatari_3.png)

Nice, the encrypted text we get, matches the string from the encrypted flag. Also note that same characters encrypt to the same value i.e  letter `x` in the dummy flag, gets encrypted to `v` multiple times. Using this information we can try to determine the rest of the flag. Since we now know that the string  `urchinsec{`  gets encrypted to `pqy~~sqxyu`, we can try to  figure out what the source characters for the encrypted flag are. But how can we do this? Let's have a recap of what we know so far
```python
pqy~~sqxyu}pqp{pR{yQi{i|xzq{`~~q{na{sRp{aQypqhaaht  # full encrypted flag 

pqy~~sqxyu == urchinsec{
pqy~~sqxyu} == urchinsec{? # where ? represents an unknown character 
```
To get the character represented with the `?`, we could try to bruteforce the `?` character in `urchinsec{?` and then compare result with `pqy~~sqxyu}`. If we get a character that matches, we can repeat this process until we get the full flag. Below is a python function I created to bruteforce the character
```python
#! /usr/bin/python3
import os
import string 
import subprocess

full_flag="pqy~~sqxyu}pqp{pR{yQi{i|xzq{`~~q{na{sRp{aQypqhaaht"
charset="""ABCDEFGHIJKLMNOPQRSTUVWXYZ0213456789abcdefghijklmnopqrstuvwxyz!"#$%&'()*+,-./:;<=>?@[\]_{}~"""

def runner(command): # This function justs runs a command on the terminal and stores the output in a variable
	try:
		result=subprocess.Popen(command,stdout=subprocess.PIPE,stderr=subprocess.PIPE,shell=True)
		output=result.stdout.read().decode("latin-1") + result.stderr.read().decode("latin-1")
	except Exception as m33:
		output="Error"
	return output



def fuzz(known_part):
	for char in charset:
		value=f"{known_part}{char}"
		length=len(value)
		command=f"""rm enc.bin;echo '{value}' | /tmp/hatari;cat enc.bin"""
		flag_snippet=test_flag[:length]
		x=(runner(command).split('\n')[-1])
		if ((flag_snippet)) == x:
			print(f"Found matching character {char} i.e Flag snippet '{flag_snippet}' matches command result '{x}'. in {value}")

test_flag= "pqy~~sqxyu}"
known_part="urchinsec{" # we want to know what character will be encrypted to } in test_flag
fuzz(known_part)
```

After running the script, we notice that we get two characters(`j` and `k`) which when encrypted result to the `}` character. This means that every character in the encrypted flag has two possible characters.
![](/images/UrchinSec_Tanzania_National_CTF/hatari_fuzz.png)

Since our script gives us two possible outcomes, we have to manually repeat this process and try to figure out the most appropriate character. If we try to automate the process, picking the first character every time, we get a flag that does not make sense
![](/images/UrchinSec_Tanzania_National_CTF/hatari_brute.png)

We have to manually repeat this process as we try to predict the correct character in the flag, after three tries, we get the first word of the flag is `just`, We can repeat until we get the full flag
![](/images/UrchinSec_Tanzania_National_CTF/manual_iteration.png)
After manual iteration, we get the correct flag `urchinsec{just_t0_b3C_Clear_This_IS_n0t_S3curERRE}`



## Attachment
{{< image src="/images/UrchinSec_Tanzania_National_CTF/attachment_1.png" alt=" " position="center" style="border-radius: 8px;" >}}

For this challenge, we are provided with a zip with three files:
	1. Attachment.exe
	2. Attachment.pdb
	3. Process.bin 

Opening up the `Attachment.exe` binary in Ghidra , we find a function called `Winner` which is responsible for decrypting the `process.bin` file using a hardcoded key `pr0gre3ss`. It does this by calling another function `GrabWin` with the filename(`process.bin`) and the encryption key `pr0gre3ss` as part of the parameters. 
![](/images/UrchinSec_Tanzania_National_CTF/attachment_2.png)

* The GrabWin function the performs the decryption and then prints the flag. To easily understand how the encryption is done, i re-implemented it in python
![](/images/UrchinSec_Tanzania_National_CTF/attachment_3.png)

Below is the cleaner python code to do the decryption

```python
import os

def GrabWin(return_storage_ptr, process_bin_file, encryption_key):
    # Define variables
    decrypted_data = []
    key_index = 0
    buffer_size = 420
    buffer = [0] * buffer_size
    key_length = len(encryption_key)
    
    # Open binary file
    with open(process_bin_file, 'rb') as f:
        data = f.read()
        
        # Iterate through bytes in the binary file
        for byte in data:
            # Decrypt byte using XOR operation with key
            decrypted_byte = byte ^ ord(encryption_key[key_index])
            # Store decrypted byte
            buffer[0] = decrypted_byte
            decrypted_data.append(chr(buffer[0]))
            # Update key index with wrap-around
            key_index = (key_index + 1) % key_length

    # Extend the storage with decrypted data
    return_storage_ptr.extend(decrypted_data)
    return return_storage_ptr

# Example usage:
process_bin_file = "process.bin"
encryption_key = "pr0gre3ss"
result = []
GrabWin(result, process_bin_file, encryption_key)
print("Result:", ''.join(result))

```


Running the script, we get the first part of the flag
![](/images/UrchinSec_Tanzania_National_CTF/attachment_4.png)
The remaining part of the flag should be the address of the `Winner` function in hex which is `0x140018a80` as follows
![](/images/UrchinSec_Tanzania_National_CTF/attachment_5.png)
Merging the two , we get the final flag `urchinsec{pr0grt3ss_w1th_4ttached_Details_0x140018a80}`



## WormHole


{{< image src="/images/UrchinSec_Tanzania_National_CTF/wormhome_desc.png" alt=" " position="center" style="border-radius: 8px;" >}}


For this challenge , we are provided with an elf binary, and credentials to a machine we can ssh into. Logging in to the machine, we find the `wormhole` binary file which has  the suid bit (`rwsr-sr-x`)  set by the `root` user. This means that when we execute the binary, it will be executed in the context of the root user.
![](/images/UrchinSec_Tanzania_National_CTF/wormhole_1.png)

When we try to run the binary, we get the text `Do the worm hole` and then it hungs/does nothing else. 
![](/images/UrchinSec_Tanzania_National_CTF/wormhole_2.png)

From here, we can open the binary in Ghidra to see what the binary is doing in the background. Below is the main function of the binary 
![](/images/UrchinSec_Tanzania_National_CTF/wormhole_3.png)

We can clean it up to make it more understandable
```c
int main(void)

{
  int flag_file_descriptor;
  int fstat_result;
  char *check_ld_preload;
  time_t current_time;
  undefined8 return_value;
  ssize_t sVar1;
  long in_FS_OFFSET;
  stat stat_buffer;

  check_ld_preload = getenv("LD_PRELOAD");
/* Check if the LD preload variable is set. If it is set, terminate*/
  if (check_ld_preload != (char *)0x0) {
    fwrite("Sorry, you can\'t use ld preload with this program.\n",1,0x33,stderr);
                    /* WARNING: Subroutine does not return */
    exit(1);
  }

  signal(14,wormhole_handler);
  current_time = time((time_t *)0x0);
  srand((uint)current_time);
  puts("Do the worm hole...");
  sleep(10066329);
  puts("Passing through the wormhole... ");
  flag_file_descriptor = open("/flag",0);
  if (flag_file_descriptor == -1) {
    perror("open");
    return_value = 1;
  }
  else {
	/* 
	The fstat function retrieves information about an open file descriptor in
	Unix-like operating systems. It is used to obtain details such as the file
	type, size, permissions, ownership, and timestamps associated with the
	file.It takes a file descriptor and A pointer to a struct stat where the
	information about the file will be stored. 
	*/

    fstat_result = fstat(flag_file_descriptor,&stat_buffer);
    if (fstat_result == -1) {
      perror("fstat");
      return_value = 1;
    }
    else {
      fstat_result = open("/dev/tty",1);
    /* 
    The sendfile function is a system call used in Unix-like operating systems
    for efficiently transferring data between file descriptors. It is commonly
    used for high-performance file copying or sending file contents over a
    network socket without needing to copy data to userspace. Syntax is ssize_t
    sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
    */
      sVar1 = sendfile(fstat_result,flag_file_descriptor,(off_t *)0x0,stat_buffer.st_size);
      if (sVar1 == -1) {
        perror("sendfile");
        close(flag_file_descriptor);
        return_value = 1;
      }
      else {
        close(flag_file_descriptor);
        return_value = 0;
      }
    }
  }
  return return_value;
}
void wormhole_handler(void)

{
  return;
}

```


From the above code snippet, we can derive the following
* The program checks whether we have set a `LD_PRELOAD` environmental variable, if it finds we have set it , it terminates. This is to prevent us from conducting library hooking for the sleep function
* This next line sets up a signal handler for the signal number `0xe`(SIGPWR) which is equal to `14` using the signal function. It will call the function `wormhole_handler` when this signal is received.
* The program then sets the time and the prints the `Do the worm hole` string
* The program then Sleeps for `10066329` seconds which is equivalent to `116.5084375` days. This makes it impractical to wait for the sleep function to complete
* After the sleep function, the program then fetches some information about the flag file and reads the flag. It is important to note that we cannot directly run `cat /flag` since the file is only readable by the `root` user  


When I was solving the challenge, i had the idea of patching the binary and replace the `sleep` function with something else maybe like `puts` so that instead of running `sleep(0x999999)` the program runs `puts(0x999999)` which would eliminate the waiting part. But soon you will realize why my idea was flawed.

To patch the binary i decided to replace the `sleep` function with `nop instructions(0x90)` so that when execution reaches the nop instruction, it will do nothing and proceed to the next instruction hence bypassing the sleep function. To do this , we first need to run `objdump` disassembler  and  find the sleep instruction in the main function.

![](/images/UrchinSec_Tanzania_National_CTF/wormhole_4.png)

Once, we find it, we need to replace it and save the result in a new binary `wormhole_patched` as shown below.
```sh
echo 'flag{Y0u_b347_7h3_w0rmh0l3}' > /flag
xxd -p /tmp/wormhole | tr -d '\n' | sed s/e832feffff/9090909090/g | xxd -r -p > /tmp/wormhole_patched; chmod +x /tmp/wormhole_patched
```

![](/images/UrchinSec_Tanzania_National_CTF/wormhole_5.png)

From the above, we see that we bypassed the `sleep` function and read the flag. But trying this on the provided server did not work. This is because when we create a new binary, it will lose the suid properties and hence cannot read the flag since it is owned by the root user.

To solve the challenge, one was required to terminate the running program with a signal `14 or 0xe` which would cause the `wormhole_handler` to be invoked, hence bypassing the sleep function and printing the flag. To achieve that , we can run the binary, then press `ctrl+z` to background it,  find its pid, then kill it using `kill -14 pidhere`. Once that is done, we can return to the exited program using `fg` (foreground the back grounded process) and we get the flag
![](/images/UrchinSec_Tanzania_National_CTF/wormhole_6.png)



## References
* https://docs.oracle.com/cd/E86824_01/html/E54765/fstat-2.html



[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https://emiliohulbert.github.io/posts/urchinsec_tanzania_national_ctf/2024-04-28-urchinsec-tanzania-national-ctf/&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=views&edge_flat=false)](https://emiliohulbert.github.io)