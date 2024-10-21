# Lab #1,22110039, Nguyen Quoc Hung, INSE330380E_01FIE
# Task 1: Software buffer overflow attack
 **Question 1**: 
- Compile asm program and C program to executable code. 
- Conduct the attack so that when C program is executed, the /etc/passwd file is copied to /tmp/pwfile. You are free to choose Code Injection or Environment Variable approach to do. 
- Write step-by-step explanation and clearly comment on instructions and screenshots that you have made to successfully accomplished the attack.
**Answer 1**: Must conform to below structure:

**Answer 1**:
## 1. Compile the Shellcode (Assembly) Progra
First, we need to compile the assembly program containing the shellcode that copies `/etc/passwd` to `/tmp/pwfile`.

```sh
nasm -f elf32 shellcode.asm -o shellcode.o
ld -m elf_i386 -o shellcode shellcode.o
```
<img width="500" alt="Screenshot" src="https://github.com/user-attachments/assets/7d3089e1-e178-4604-ae73-258361af71df"><br>


## 2.  Compile the Vulnerable C Program

We then compile the vulnerable C program without stack protection and with executable stack, so it can be exploited using the shellcode.
```sh
gcc -fno-stack-protector -z execstack -o vuln1 vuln1.c
``` 
![alt text](image-1.png)<br>
Running the program without any arguments will not do anything because it relies on the user input for the buffer overflow. We will handle this in the next steps.

-`fno-stack-protector`: Disables stack protection, making buffer overflow easier.
-`z execstack`: Allows code execution from the stack, which is necessary for running the injected shellcode.

## 3. Prepare Shellcode for Injection

To inject the shellcode bytes into the C program, we must first extract them from the compiled assembly program. To achieve this, the opcode is extracted using `objdump`.

The shellcode in the form \x... that can be injected into the susceptible C program will be provided by this command.
This will produce shellcode in the format:

<img width="500" alt="Screenshot" src="https://github.com/user-attachments/assets/9f51333a-d143-472a-baa8-e16134147516"><br>

You now have the raw shellcode that can be injected into the C program.

## 4. Exploit the Vulnerable C Program
In order to take advantage of the buffer overflow and insert the shellcode into the execution path, we will now run the susceptible C program with an environment variable that contains the shellcode.

Export the shellcode into an environment variable:

```sh
export SHELLCODE=$(python -c 'print "A"*32 + "\x90"*100 + "\x55\x55\x55\x55"' > payload)
```
The `\x90` represents NOPs (No Operation), which create a NOP sled to increase the likelihood that the CPU will jump into our shellcode during the overflow.
```sh
./vulnerable $(python -c 'print("A"*20 + "\x12\x34\x56\x78")')
```
In this example, `"A"*20` represents padding, and `"\x12\x34\x56\x78"` would be replaced by the address pointing to the NOP sled.<br>
When the buffer overflow occurs, the program's return address will be overwritten with the address pointing to our shellcode. As a result, the injected shellcode will execute, copying `/etc/passwd` to `/tmp/pwfile`.

If successful, there will be no visible output on the terminal. You can verify the success of the exploit by checking the contents of /`tmp/pwfile`.
## 5. Verify the Attack:
After running the program, the `/etc/passwd` file should be copied to `/tmp/pwfile`. We can verify it using the following command:
```sh
cat /tmp/pwfile
```
After running the vulnerable program, check if the shellcode was executed successfully.
<img width="500" alt="Screenshot" src=https://github.com/user-attachments/assets/486507f2-fe1f-49b4-812b-248cb4d6c1e0><br>

The contents of /`etc/passwd` will be displayed, confirming that the exploit worked and the file was copied successfully.
<img width="500" alt="Screenshot" src=https://github.com/user-attachments/assets/24200c0e-cd65-4a9c-92e8-64abc1db05ca><br>


**Conclusion**: By executing these steps, we exploited a buffer overflow vulnerability in the C program and successfully injected shellcode that copied the `/etc/passwd` file to `/tmp/pwfile`.
