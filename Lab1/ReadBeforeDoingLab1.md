### Lab1 và Lab2 là lab để thực hiện tấn công Buffer-overflow
***
#### Để làm được lab1, cần phải setup như sau: 
* Bước 1: Vào link sau để setup cho docker
  > https://github.com/quang-ute/myprojects
* Bước 2: Vào thư mục lab1 và bật lab1 trên visual studio code
  ```
  cd seclabs/lab1
  $> code .
  ```
* Bước 3: Ánh xạ file vào docker
  ```
  $> docker run -it --privileged -v $HOME/Seclabs:/home/seed/seclabs img4lab
  ```
* Bước 4: Đi vào file lab1 khi đã ánh xạ
  ```
  cd seclabs/lab1
  ```
