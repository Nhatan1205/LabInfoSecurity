### Lab này sẽ dùng John the Ripper để crack mật khẩu. Toàn bộ sẽ được làm theo hướng dẫn của link này

> https://app.box.com/s/rk43iioswot99p0vqfx8d5lirwi04a8y
***
* 1: To view the contents of the passwd file, type:
  ```
  cat /etc/passwd
  ```

* 2: View the permissions on the /etc/passwd file by typing the following command
  ```
  ls -l /etc/passwd
  ```

* 3: To view the contents of the shadow file, type
  ```
  sudo cat /etc/shadow
  ```

* 4: To create a new user named yoda, type the following command in the terminal
  ```
  sudo adduser yoda
  ```
  Enter pasword for yoda is 'green'

* 5: To create a new user named chewbacca, type the following command in the terminal
  ```
  sudo adduser chewbaca
  ```
  Enter pasword for chewbaca is 'green'

* 6: Now, view the changes made to the passwd file by typing the following
  ```
  tail /etc/passwd 
  ```

* 9: Next, examine the alterations to the shadow file by typing the following
  ```
  sudo tail /etc/shadow
  ```

* 10: Examine the entries in the auth.log related to account changes by typing:
  ```
  tail /var/log/auth.log
  ```

* 11: Next, examine the alterations to the shadow file by typing the following:
  ```
  sudo tail -n 2 /etc/shadow 
  ```

* 12: Examine the entries in the auth.log related to account changes by typing:
  ```
  tail /var/log/auth.log
  ```

* 13: To look for specific information about password changes within auth.log, type:
  ```
  cat /var/log/auth.log | grep changed
  ```

* 14: Install john by typing the following command:
  ```
  sudo apt-get install john
  ```

* 15: Type the following command to see available switches for the john command:
  ```
  john
  ```

* 16: Type the following command to attempt to crack the passwords with john:
  ```
  sudo john /etc/shadow 
  ```
  
* 17: To view the password hashes and corresponding passwords, type the following:
  ```
  sudo cat /root/.john/john.pot
  ```
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
  If any account's passwords are changed, john will go through the cracking process again. We
will set chewbaca’s password to a word contained within the password.lst file. 
* 22: Set chewbacca’s password to 'computer' by typing computer twice after typing:
  ```
  passwd chewbaca
  ```
* 23: Type the following to run john again and the new password hash will be loaded
  ```
  john /etc/shadow --wordlist=/usr/share/john/password.lst 
  ```
  Since that word was one of the first few in the dictionary, john was able to crack the password in less than one second. Now we will try one of the last passwords in the list. 
* 24: To view the last 20 lines of the file, type the following command in the terminal:
  ```
  tail password.lst
  ```
* 25: Set chewbaca’s password to '1q2w3e4r' by typing twice after typing:
  ```
  passwd chewbaca
  ```
* 26: Type the following to run john again and the new password hash will be loaded:
  ```
  john /etc/shadow --wordlist=/usr/share/john/password.lst 
  ```
