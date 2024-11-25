# Lab #2, 22110007, Nguyen Nhat An, INSE330380E_01FIE
**Question 1**: 
Conduct transfering a single plaintext file between 2 computers, 
Using openssl to implementing measures manually to ensure file integerity and authenticity at sending side, 
then veryfing at receiving side. 

**Answer 1**:
## 1. Set up VMs for ALL the tasks
This lab, I use vm name `seed` (sender) and `nhatan` (receiver)
![image](https://github.com/user-attachments/assets/abfbeecc-a5c4-4a40-9bca-b8465cc140c3)

![image](https://github.com/user-attachments/assets/d20bc811-e4be-45b0-bc1a-f6c61c8aee70)

## 2. Create sendind file at seed
* Create sending file
  ```
  echo "This is sending file from seed" > plaintext.txt
  ```
* Check if it exists
  ```
  cat plaintext.txt
  ```
  ![image](https://github.com/user-attachments/assets/468c41a3-e0ef-456a-a335-409e8dfcd300)

## 3. Set up ssh
Without SSH, transferring the encrypted file securely would be more difficult, as alternatives like FTP are not encrypted by default. scp uses SSH to securely copy the file from the sender to the receiver, which is essential for the task. So we need install it
* Installation each person
  ```
  sudo apt update
  sudo apt install openssh-server
  ```
* Start ssh 
  ```
  sudo systemctl start ssh
  ```
* Check status
  ```
  sudo systemctl status ssh
  ```
  ![image](https://github.com/user-attachments/assets/9ad7a2a5-2aac-4c71-9bee-f8948fe67722)

  ![image](https://github.com/user-attachments/assets/42698a1d-bc9e-47db-9b3d-4552bc1f4901)


## 4. Hash file and encrypt
To ensure the security, I will hash the plaintext file for integrity, after that I encrypt plaintext file with password which expressing the authenticity
* Hash

  The command sha256sum plaintext.txt > plaintext.hash generates a SHA-256 hash of a file plaintext.txt and stores the result in a file plaintext.hash. 
  ```
  sha256sum plaintext.txt > plaintext.hash
  ```
  ![image](https://github.com/user-attachments/assets/ef4e964b-d3e1-4672-89f8-d42aa680f040)

* Encrypt for safe transferring.
  ```
  openssl enc -aes-256-cbc -salt -in plaintext.txt -out plaintext.enc -pass pass:pw
  ```
  ![image](https://github.com/user-attachments/assets/0d14bb7c-440c-45d8-bfd7-9be61b900fa3)

  This command include:

  `-aes-256` : The AES (Advanced Encryption Standard) algorithm with a 256-bit key size.

  `-cbc` : This stands for Cipher Block Chaining, which is one of the modes of operation for block ciphers. In CBC mode, each block of plaintext is XORed with the previous ciphertext block before being encrypted. This ensures that identical plaintext blocks produce different ciphertext blocks, providing additional security.

  `-salt` : This option adds a random salt to the encryption process

  `-out plaintext.enc` : This specifies the output file where the encrypted data will be saved. In this case, the encrypted file will be stored as plaintext.enc.

  `-pass pass:pw` :  this option provides the password used to derive the encryption key. The pass of this encryption is `pw`

  ![image](https://github.com/user-attachments/assets/a86c1fee-0443-4072-901d-ef2ec5bcfc0c)

## 5. Transfer file
* File the ip address of `nhatan`
  ![image](https://github.com/user-attachments/assets/482f932e-e1a3-4d86-ac77-15bf7b1ee85d)

  ```
  10.0.2.15
  ```
* Transfer to `nhatan` from `seed`
  ```
  scp plaintext.enc plaintext.hash nhatan@10.0.2.15:/home/nhatan

  ```
  This command conclude:
  
  `scp` : Securely transfers files over SSH.

  `plaintext.enc plaintext.hash` : Files to be sent.

  `nhatan@10.0.2.15` : Login as user `nhatan` on the machine with IP 10.0.2.15.
  
  `:/home/nhatan` : Directory on the remote machine where the files will be saved.
  
  ![image](https://github.com/user-attachments/assets/4f4bafca-5aa8-4e93-afb0-607d61cc2e82)

* From `nhatan` decrypt the file

  This file has transferred

  ![image](https://github.com/user-attachments/assets/865abcac-32c0-4f01-b476-9db2b2fc4dbd)

  ```
  openssl enc -d -aes-256-cbc -in plaintext.enc -out decrypted.txt -pass pass:pw
  ```
  This command decrypts a file encrypted with AES-256-CBC

  `openssl enc -d` : Specifies decryption mode.

  `-aes-256-cbc` : Uses the AES algorithm in 256-bit CBC mode.

  `-in plaintext.enc` : The encrypted input file.

  `-out decrypted.txt` : The file where the decrypted output will be saved.

  `-pass pass:pw` : The decryption password (pw in this case).

  ![image](https://github.com/user-attachments/assets/bc81df64-d2e4-4439-84f1-9275cf2fa390)

* Check the content
  ```
  cat decrypted.txt
  ```
  ![image](https://github.com/user-attachments/assets/f7d1246a-f4e5-443b-bdf4-59427856efdd)

  => Task successfully. Explain in details:

  For integrity, i hashed the the plaintext.txt file in sender and also send it to receiver, so let check it to see if it's identical each other
  ![image](https://github.com/user-attachments/assets/20ebfcfa-0900-41bb-a51f-e426c84a7d2f)

  ![image](https://github.com/user-attachments/assets/9ea6ac16-2a18-46d8-9d56-466e5994fa99)

  => Satisfy

  For authenticity, the command to decrypt the file has option enter password that the sender 'seed' sent.

  => Satisfy
  
# Task 2: Transfering encrypted file and decrypt it with hybrid encryption. 
**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:

Overview about what I am going to do:
* `seed` machine will create both private and public key.
* `seed` will send `nhatan` his public key.
* After that on `nhatan`, `nhatan` creates a file name `file.txt`.
* This file will be encrypted by `cbc` with password option and name file.enc
* Password will be set up by secret numbers(random) that later encrypted by public key of `seed`, save as a file name secretkey.encrypted
* Then `nhatan` send both file `file.enc` and `secretkey.encrypted` to `seed`
* `seed` use private key to decrypt the password file, then decrypt the ` file.enc` by `cbc` + this password

## 1. On `Seed` machine
* Create key pair for `Seed` (public key and private key)
  ```
  openssl genrsa -out keypair.pem 2048 
  ```
  This command generates a 2048-bit RSA key pair and aves the private key (with public key embedded) to keypair.pem.
* Next, we extract the public key
  ```
  openssl rsa -in keypair.pem -pubout -out publickey.pem
  ```
  The public key now save in file `publickey.pem`, to see this file use:
  ```
  cat publickey.pem
  ```
  The public key of Seed is:
  ```
    MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAz6dQw0en/OjKxxnz6Sh0
  W+loDKI0C/QpQ9+rqHerCJBwVMpxxVMePq2I3vxgElBUFHBypZcMi7oaTeexeG1g
  pNX5DIXvQEWu6Mg6VWcUW8b6jetLAJRHzBUITwIU4HDeIdSEZVsRwhM9kYWZ8LWU
  JPNZRzE9P4j//h5CHVgVRet8uzdQxK914S/m2aJS5q0lUY2gkqmzJG4l25TWU8Af
  sQHTD17xgE7FQSQiUv56WLQOs5qWRCt8Du3RgRtqvqDH/SqBWR1hYUJwcpwUgB7M
  bOdK86LOFaHrRKp2nI+RX/O7eT7ws21uL/qPozzyRCkFF7Al0RP7KG5IFbro1tUu
  MwIDAQAB
  ```
* Send it to `nhatan`
  ```
  scp publickey.pem nhatan@10.0.2.15:/home/nhatan
  ```
  ![image](https://github.com/user-attachments/assets/6a358b2b-dfe6-4a7d-b95f-3e13855c72bc)

## 2. On `Nhatan` machine
* Check and it has already in `nhatan`
  ![image](https://github.com/user-attachments/assets/07151cbb-5aa0-4c8c-bbdd-9c8041a25b68)
#### Create plain text file
* Create file 
  ```
  echo "This file works for task2" > file.txt
  ```
* Check this file if it exits
  ```
  cat file.txt
  ```
  ![image](https://github.com/user-attachments/assets/b62375e6-c0e5-489a-b2c6-26b630156dd4)

#### File encryption and transfer to `seed`
* Create a `secret key` by random string
  ```
  openssl rand -hex 32 > secretkey
  ```
  This command generates a random 32-byte (256-bit) hexadecimal key and writes it to a file named `secretkey`.

  `openssl rand`: Uses OpenSSL to generate random bytes.

  `-hex`: Specifies that the output should be in hexadecimal format.

  `32`: Generates 32 bytes of random data (64 characters in hexadecimal, as each byte is represented by two hex digits).

  ![image](https://github.com/user-attachments/assets/8b3f526e-b69a-4bd9-961d-cf5eb1a66b8e)

  
* Encrypt `file.txt` by `-aes-256-cbc`
  ```
  openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -pass file:/home/nhatan/secretkey
  ```
  This command is differnt with those above bacause the password now is the file so i use the syntax `-pass file:/home/nhatan/secretkey`

  ![image](https://github.com/user-attachments/assets/37302aae-7d07-4df3-bb44-81689ced9c15)

* Encrypt the secretkey using RSA with `seed`'s public key
  ```
  openssl rsautl -encrypt -inkey publickey.pem -pubin -in secretkey -out secretkey.encrypted
  ```
  ![image](https://github.com/user-attachments/assets/4c96bc34-7145-4abe-8d7f-081e59361cd3)

* Send `file.enc` and `secretkey.encrypted` to `seed`

  Use `ifconfig` on `seed` machine to find ip address

  ![image](https://github.com/user-attachments/assets/b2086d2a-3499-40a2-8249-c6aff8b059c5)

  There it is `10.0.2.4` and then use command below to transfer file to `seed`
  ```
  scp file.enc secretkey.encrypted seed@10.0.2.4:/home/seed
  ```
  ![image](https://github.com/user-attachments/assets/9d0f3ffb-2f03-4c5b-b7a2-d7165b44fb15)

## 3. Back to `seed` machine
* Check `file.enc` and `secretkey.encrypted` existence
  ![image](https://github.com/user-attachments/assets/859710bb-a5a2-4c68-813a-f8b8f1cb8a52)

  => Already existed

* Use private key to decrypt `secretkey.encrypted`
  Decrypt the file.encrypted by using private key in keypair.pem to obtain the file using command
  ```
  openssl rsautl -decrypt -inkey keypair.pem -in secretkey.encrypted -out secretkey.decrypted
  ```
  Explain each parameter in this command:

  `openssl rsautl`: Utility for RSA encryption and decryption.

  `-decrypt`: Specifies decryption mode.
  
  `-inkey keypair.pem`: Provides the private key (stored in keypair.pem) for decryption.

  `-in secretkey.encrypted`: The input file containing the RSA-encrypted data.

  `-out secretkey.decrypted`: The output file where the decrypted data will be saved.

  ![image](https://github.com/user-attachments/assets/04a7e75c-69bb-4625-b505-8f6c220d255b)

* Use `secretkey.decrypted` for the password and `-aes-256-cbc` to decrypt the `file.enc`
  ```
  openssl enc -d -aes-256-cbc -in file.enc -out file.decrypted -pass file:/home/seed/secretkey.decrypted
  ```
  This command is following:

  `openssl enc -d`: Specifies decryption mode.

  `-aes-256-cbc`: Uses the AES algorithm in 256-bit CBC mode for decryption.

  `-in file.enc`: The input file containing the encrypted data.
  
  `-out file.decrypted`: The output file where the decrypted data will be saved.
  
  `-pass file:/home/seed/secretkey.decrypted`: Specifies that the decryption password (symmetric key) should be read from the file /home/seed/secretkey.decrypted.
  
  ![image](https://github.com/user-attachments/assets/b8902d4a-e823-459c-bac4-1e80340756d5)

  => Task completed
# Task 3: Firewall configuration
**Question 1**:
From VMs of previous tasks, install iptables and configure one of the 2 VMs as a web and ssh server. Demonstrate your ability to block/unblock http, icmp, ssh requests from the other host.

**Answer 1**:

This lab, I choose `nhatan` machine to be a web and ssh server, also `seed` machine is a client

## 1. Check IP tables for each machine
* `seed`
  This iptable of `seed` is quite special cause it have already set up a lot of package in it, so it look complicated but we just only pay attention at `Chain INPUT`, `Chain FORWARD` and `Chain OUTPUT`

  ![image](https://github.com/user-attachments/assets/45ad505c-a5c2-4d63-abc5-431a7b954b00)

  
* `nhatan`
  
  ![image](https://github.com/user-attachments/assets/f1a003e4-dd4a-42c7-99be-d143968f7fcc)

## 2. Set up a web and SSH server on `nhatan`
* Install Apache (web server):
  ```
  sudo apt install apache2
  ```
  ![image](https://github.com/user-attachments/assets/8ce42eeb-7cec-4beb-b3a1-007664b8e28a)

* Start and check status of apache2 server
  ```
  sudo systemctl start apache2
  sudo systemctl enable apache2
  ```
  ![image](https://github.com/user-attachments/assets/d0b53f23-3ae5-408f-86cf-b1167b990dc9)

## 3. Check if the server is running from host/client
* Ip address of server `nhatan`
  ```
  ifconfig
  ```
  ![image](https://github.com/user-attachments/assets/087e5581-b4fa-4ac6-8af8-8a344181cab9)

  It is `10.0.2.15`

* From Host `curl` to server
  ```
  curl http://10.0.2.15
  ```
  ![image](https://github.com/user-attachments/assets/bd322f5b-2432-4060-9ffd-3284626a4223)

* From Host `ssh` to server
  ```
  ssh nhatan@10.0.2.15
  ```
  ![image](https://github.com/user-attachments/assets/b58ce881-700f-4f83-8b98-531f41543d4a)

* From Host `ping` to server
  ```
  ping 10.0.2.15
  ```
  ![image](https://github.com/user-attachments/assets/159f4280-8362-4d1c-b0e4-c38957dc971a)

  => Server is running

## 4. Block http, icmp, ssh requests from the other host.
* The ip of host `seed` is `10.0.2.4`
* Block HTTP requests (port 80):
  ```
  sudo iptables -A INPUT -p tcp --dport 80 -s 10.0.2.4 -j DROP
  ```
  This iptables command adds a rule to block incoming traffic on TCP port 80 from a specific source IP address:

  `iptables`: Utility for configuring Linux kernel packet filtering rules.

  `-A INPUT`: Appends the rule to the INPUT chain (for incoming packets).

  `-p tcp`: Specifies that the rule applies to TCP protocol packets.
  
  `--dport 80`: Targets packets destined for port 80 (commonly used for HTTP traffic).

  `-s 10.0.2.4`: Matches packets originating from the source IP address 10.0.2.4. which is the `seed` hosts
  
  `-j DROP`: Specifies the action for matching packets: silently drop them (no response is sent).

  ![image](https://github.com/user-attachments/assets/2e7caa93-5615-49c6-abc3-3b047983a3bc)

  and then we move to `seed` to check if it can access server by HTTP reques
  ```
  curl http://10.0.2.15
  ```
  ![image](https://github.com/user-attachments/assets/7c07743c-2e1e-4fbb-aecb-d6c33c56c0cc)

  => It does not response anything => Block HTTP requests successfully
* Block icmp request
  ```
  sudo iptables -A INPUT -p icmp --icmp-type echo-request -s 10.0.2.4 -j DROP
  sudo iptables -A OUTPUT -p icmp --icmp-type echo-reply -d 10.0.2.4 -j DROP
  ```
  The first rule blocks incoming ping requests from `10.0.2.4`.
  
  The second rule ensures your machine does not send any ping replies to `10.0.2.4`.

  and then we move to `seed` and check by command below
  ```
  ping 10.0.2.15
  ```
  ![image](https://github.com/user-attachments/assets/ea180bd1-5de6-41e4-bb9f-daf8940f1571)

  => It does not response and annouce 100% packet loss => block ok

* Block ssh request by port 22
  ```
  sudo iptables -A INPUT -p tcp --dport 22 -s 10.0.2.4 -j DROP
  ```
  ![image](https://github.com/user-attachments/assets/5352af64-59cd-4ed4-bae0-427233563f14)

  and then we switch to `seed` to check by sending file name plaintext.txt in the task above
  ```
  scp plaintext.txt nhatan@10.0.2.15:/home/nhatan
  ```
  ![image](https://github.com/user-attachments/assets/f7c9e308-a91a-4e48-a1e9-4a69d57e8a8d)

  => The file cannot be sent. We can conclude that it block successful.

## 5. Unblock http, icmp, ssh requests from the other host.
* Unblock HTTP request
  ```
  sudo iptables -D INPUT 1
  ```
  Explain command:
  
  `iptables`: Invokes the iptables utility to manage firewall rules on a Linux-based system.
  
  `-D INPUT`: Deletes a rule from the INPUT chain, which controls incoming network traffic.
  
  `1`: Specifies the rule number to delete from the INPUT chain. In this case, 1 refers to the first rule in the list. See the picture below, the `1` now is DROP HTTP
  
  ![image](https://github.com/user-attachments/assets/0f95fbd1-c87d-4f5e-a3c2-861ae138f487)

  After drop, check the list again:
  ```
  sudo iptables -L
  ```
  ![image](https://github.com/user-attachments/assets/94ae079a-54d3-44b5-b993-515ac8066f4d)

  Moving to `seed` and test by command `curl`
  ```
  curl http://10.0.2.15
  ```
  ![image](https://github.com/user-attachments/assets/0857b5e1-8eab-4e13-80a4-a26243559668)

  => It can curl to the web server so I unclock successfully
* Unclock icmp request
  ```
  sudo iptables -D INPUT 1
  sudo iptables -D OUTPUT 1
  ```
  The command is the same with what we do above.

  `1`: When the http was dropped, now the first rule at `INPUT` is drop icmp. Do the same with `OUTPUT`
  
  ![image](https://github.com/user-attachments/assets/e73836ee-11d0-43e3-b1e1-b6794d1d5d40)

  After drop, check the list again:
  ```
  sudo iptables -L
  ```
  ![image](https://github.com/user-attachments/assets/5ef792a2-c542-4316-a7f6-d17769af26b2)

  On `seed`, we try to ping to `server` using command
  ```
  ping 10.0.2.15
  ```
  ![image](https://github.com/user-attachments/assets/b9825188-da58-4e8c-9374-93e2b0c8808c)

  => All packages are received, ping completed. Unclock icmp ok.
  
* Unclock ssh request
  ```
  sudo iptables -D INPUT 1
  ```
  ![image](https://github.com/user-attachments/assets/3cadaa3c-0dc6-4cc8-aea7-495191f85de1)

  Check if it unclocked by sending file `plaintext.txt` to server from host
  ```
  scp plaintext.txt nhatan@10.0.2.15:/home/nhatan
  ```
  ![image](https://github.com/user-attachments/assets/ebc71b4c-b346-46a8-a87e-18fd47980c65)

  => File is sent. Unclock ssh request successfully.
  
