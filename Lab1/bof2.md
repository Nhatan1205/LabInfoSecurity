### Chi tiết Buffer-overflow trên file bof2.c
***
#### Mục đích của bài này là dùng buffer-overflow để đổi giá trị của biến 'check' thành 0xdeadbeef
Xem qua code ta có thể thấy lỗ hỏng nó đến từ hàm fget(buf, 45, stdin). Vì vậy ta sẽ dùng buffer-overflow để làm tràn biến buf xuống phần của biến check và cái phần được tràn vào biến check sẽ là giá trị 0xdeadbeef

![image](https://github.com/user-attachments/assets/363d2764-dee7-4b4c-895e-db767115cba4)

***
#### Các bước thực hiện
* Bước 1: Biên dịch file
  ```
  gcc -g bof2.c -o bof2.out -fno-stack-protector -mpreferred-stack-boundary=2
  ```
* Bước 2: Điểm qua stackframe

  
  ![image](https://github.com/user-attachments/assets/72d3f459-132e-4053-b7fd-f4ebdea0f608)




  ta thấy mảng buf chứa tối đa 40 bytes nên ta cần truyền vào input 40 kí tự ngẫu nhiên và truyền thêm 4 bytes giá trị 0xdeadbeef nữa để đè lên giá trị hiện tại của biến check là ta có thể thay đổi thành giá trị mới 
* Bước 3: Thực hiền truyền input
  ```
  echo $(python -c "print('a'*40 + '\xef\xbe\xad\xde')") | ./bof2.out
  ```
  Kết quả:
  ![image](https://github.com/user-attachments/assets/d02baa67-03e4-494b-ad39-37fa0634c695)


  => Thành công
