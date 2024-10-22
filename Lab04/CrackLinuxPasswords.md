### Lab này sẽ dùng John the Ripper để crack mật khẩu. Toàn bộ sẽ được làm theo hướng dẫn của link này

> https://app.box.com/s/rk43iioswot99p0vqfx8d5lirwi04a8y
***
* 1: To view the contents of the passwd file, type:
  ```
  cat /etc/passwd
  ```
  ![image](https://github.com/user-attachments/assets/5199f26a-4beb-4f17-9369-b985bff04276)

* 2: View the permissions on the /etc/passwd file by typing the following command
  ```
  ls -l /etc/passwd
  ```
  ![image](https://github.com/user-attachments/assets/8efa3ef5-f0df-4aae-a46a-7b7a5c398582)

* 3: To view the contents of the shadow file, type
  ```
  sudo cat /etc/shadow
  ```
  ![image](https://github.com/user-attachments/assets/643a77de-107b-4e32-9b27-c9337ef13997)

* 4: To create a new user named yoda, type the following command in the terminal
  ```
  sudo adduser yoda
  ```
  Enter pasword for yoda is 'green'
  ![image](https://github.com/user-attachments/assets/388b71a1-2716-4289-af24-d5fe7011bd06)

* 5: To create a new user named chewbacca, type the following command in the terminal
  ```
  sudo adduser chewbaca
  ```
  Enter pasword for chewbaca is 'green'
  ![image](https://github.com/user-attachments/assets/d0a774a9-f551-48b9-b8f6-ae517a32a0b3)

* 6: Now, view the changes made to the passwd file by typing the following
  ```
  tail /etc/passwd 
  ```
  ![image](https://github.com/user-attachments/assets/ba59ae2f-4732-41f6-9668-25f375937bb5)

* 9: Next, examine the alterations to the shadow file by typing the following
  ```
  sudo tail /etc/shadow
  ```
  ![image](https://github.com/user-attachments/assets/62b21922-5844-412a-be60-ae5e819d46f3)

* 10: Examine the entries in the auth.log related to account changes by typing:
  ```
  tail /var/log/auth.log
  ```
  ![image](https://github.com/user-attachments/assets/263ef358-7d57-4a7c-a421-3ce193cdac21)

* 11: Next, examine the alterations to the shadow file by typing the following:
  ```
  sudo tail -n 2 /etc/shadow 
  ```
  ![image](https://github.com/user-attachments/assets/0787a669-d5dc-4a05-ad81-11eb4e4e9846)

* 12: Examine the entries in the auth.log related to account changes by typing:
  ```
  tail /var/log/auth.log
  ```
  ![image](https://github.com/user-attachments/assets/941383a1-fec5-479e-b345-2bae632d67e2)

* 13: To look for specific information about password changes within auth.log, type:
  ```
  cat /var/log/auth.log | grep changed
  ```
  ![image](https://github.com/user-attachments/assets/0edfc3b8-c404-4b67-8f2a-8c23102e2457)

* 14: Install john by typing the following command:
  ```
  sudo apt-get install john
  ```
  ![image](https://github.com/user-attachments/assets/0c98aedb-a8f3-45f4-a3eb-a5f48162fcba)

* 15: Type the following command to see available switches for the john command:
  ```
  john
  ```
  ![image](https://github.com/user-attachments/assets/bbb90557-404a-45c0-ab45-b8d8cf0c7d7f)

* 16: Type the following command to attempt to crack the passwords with john:
  ```
  sudo john /etc/shadow 
  ```
  ![image](https://github.com/user-attachments/assets/cd541b8e-b704-4e3b-93ea-f62abbd1670e)

* 17: To view the password hashes and corresponding passwords, type the following:
  ```
  sudo cat /root/.john/john.pot
  ```
  ![image](https://github.com/user-attachments/assets/9d231e63-4977-4e56-81a9-504357be952c)

* 18: Type the following command to attempt to crack the passwords with john (Crack again)
  ```
  sudo john /etc/shadow 
  ```
  Unfortunately, if you attempt to crack the password again, you will not have success. 
  This is because hashes and their corresponding passwords are stored within the john.pot file;
john will not crack the password hash again. If you want the passwords to be cracked again,
you will need to remove the information stored in the john.pot file. 
* 19: Type the following command to remove the existing john.pot file:
  ```
  sudo rm /root/.john/john.pot
  ```
* 20: Type the following command to re-crack the passwords with john
  ```
  sudo john /etc/shadow
  ```
* 21: To view the first 20 lines of the file, type the following command in the terminal:
  ```
  head -n 20 /usr/share/john/password.lst 
  ```
  ![image](https://github.com/user-attachments/assets/f24ecb02-50bf-4e3f-a390-c42d472b5e83)

  If any account's passwords are changed, john will go through the cracking process again. We
will set chewbaca’s password to a word contained within the password.lst file. 
* 22: Set chewbacca’s password to 'computer' by typing computer twice after typing:
  ```
  sudo passwd chewbaca
  ```
  ![image](https://github.com/user-attachments/assets/febcce5d-5a9b-4e03-be4c-07a536defe4a)

* 23: Type the following to run john again and the new password hash will be loaded
  ```
  sudo john /etc/shadow --wordlist=/usr/share/john/password.lst 
  ```
  ![image](https://github.com/user-attachments/assets/d9e99e81-9938-4272-9abe-3f54b5aa32d6)

  Since that word was one of the first few in the dictionary, john was able to crack the password in less than one second. Now we will try one of the last passwords in the list. 
* 24: To view the last 20 lines of the file, type the following command in the terminal:
  ```
  tail /usr/share/john/password.lst
  ```
  ![image](https://github.com/user-attachments/assets/a4e3e364-d3da-4482-a396-dc01f255dbb6)

* 25: Set chewbaca’s password to '1q2w3e4r' by typing twice after typing:
  ```
  passwd chewbaca
  ```
  ![image](https://github.com/user-attachments/assets/d1e0f861-a429-4e13-8ea6-c8697da66745)

* 26: Type the following to run john again and the new password hash will be loaded:
  ```
  sudo john /etc/shadow --wordlist=/usr/share/john/password.lst 
  ```
  ![image](https://github.com/user-attachments/assets/a015af99-0a36-4ff8-b44e-b7e97cd00b78)

