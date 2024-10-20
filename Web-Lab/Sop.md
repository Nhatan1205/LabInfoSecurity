#### 4. Tasks  
***
##### 4.1. SOP lab
##### 4.1.1. Setup 
* a) Install VNC viewer on the host computer.
  ![image](https://github.com/user-attachments/assets/f693f5ce-5ebf-4638-8fd0-e648a24e1e92)

* b) In windows Terminal, change working directory to sop. Fire up docker-compose to create docker 
container set for this lab.  
  > docker-compose up -d

  ![image](https://github.com/user-attachments/assets/f344a2aa-b52b-445f-9891-23bbc51838ab)

  This command will bring up 2 containers connecting to the same LAN: The web client (10.9.0.6) 
and a web server (10.9.0.5) 
[web-client] ---- 10.9.0.0/24 -----[web-server] 
The web server is configured with 3 virtual servers running on it.  
* c) Fire up VNC viewer on the host computer to connect to the web client on docker network by 
entering localhost:5900 on the address bar of VNC viewer. After the first connection is 
successfully established, a thumbnail of web-client computer in VNC will show up ready for 
connection.
  ![image](https://github.com/user-attachments/assets/34c9bb6d-dbed-4ab6-9c84-2d5ca7526945)

* d) Attach to the console of web-client docker container by entering: 
  > docker exec -it web-client sh -l
  
  Execute the following command to append hostname configuration lines from /tmp/hosts to the 
end of /etc/hosts file. 
  > cat /tmp/hosts  >> /etc/hosts

  Execute tail command to make sure those 3 lines exist in the /etc/hosts file.
  ``` 
  5c92cca8817f:/# tail -n5 /etc/hosts 
  . . . 
  10.9.0.5    sitea.com 
  10.9.0.5    siteb.com 
  10.9.0.5    parentsite.com
  ```
  ![image](https://github.com/user-attachments/assets/6fb2eaf9-17a6-4f45-ad2f-ac8d523b7cc0)

* e) Fire up Firefox inside the web-client container, enter http://siteb.com or whichever of the above 3 
sites. Home page of those sites will be displayed:
  ![image](https://github.com/user-attachments/assets/4d60a5d1-1731-43bd-b920-9011ddaef57b)
***
##### 4.1.2. Using JS to access DOM objects of the web site
* Load sitea in browser.
  ![image](https://github.com/user-attachments/assets/4be7b6a0-41d8-460a-8d30-4f2c8097cdd1)

* Open Web Developer >> Console Panel (Ctrl + Shift + K)
  ![image](https://github.com/user-attachments/assets/3a5fb223-5c6e-41ba-9f78-929da89d5872)

* Right-click on page > View page source to display source of web page. Take notes of ids in 
body part of page.
  ![image](https://github.com/user-attachments/assets/8720b2b8-05b1-4187-a72b-12d3709ffcc5)

* Goto Console Panel, with javascript:
  * a. Try to access DOM objects of page: document.body, document.body.innerText, 
document.body.innerHTML or whichever that you can.


    ![image](https://github.com/user-attachments/assets/19e190c5-59b1-436b-b153-de740d4f62e7)

  * b. Change the text “Hey,Site A is working” to “Header text is modified via document object”.
    ![image](https://github.com/user-attachments/assets/27926664-baa7-4e32-830b-726f2ebdca07)

  * c. Create a newwin variable for a new tab: 
var newwin = window.open(http://siteb.com,”right”). Can you access document object of 
newwin? Explain why if you could not.
    ![image](https://github.com/user-attachments/assets/56ad5113-d9b6-4c6f-9da1-b9328519aba8)

    I can open siteb on the right tab but cannot access to it because the Sop ban me to do that

* Click siteb.com tab, Ctrl + Shift + K to open Console Panel.
  * d. Enter JS code to set the new location for tab sitea which is the opener of current tab: 
window.opener.location=”http://parentsite.com”.
    ![image](https://github.com/user-attachments/assets/42373992-4daf-4982-ac4c-a6f096c13fe9)

  * e. Try JS code to access document.body object of window.opener. Explain why if you could not
    ![image](https://github.com/user-attachments/assets/527cfd7c-216f-4522-bf03-a691ee955343)

    SOP ban to access.
***
##### 4.3.1. Frame objects
* Access http://parentsite.com then choose parentsite in list of sites on iframe 
picker.
  * a) Can you access window.framea, window.frameb?.
  * b) Can you access document.body of framea, frameb from parentsite?
* Select sitea.com in list of sites on iframe picker.
  * c) Can you access window.parentsite?
  * d) Can you access document.body object of parentsite.com? Which object of parentsite that 
sitea can access? 
*  Select parentsite.com in list of sites on iframe picker, define a variable named hello: var hello=” 
world”
  * e) Can you access hello from sitea, siteb?
***
##### 4.1.4. Sending POST request to a site:
* a) Access http://sitea.com/post.html
* b) Type some text in textbox then click  “Submit query”. What would show up? Explain why you see that.
* c) Repeat step (a), toggle Web console >> Network (Ctrl + Shift + E)
* d) Repeat step (b), examine the request and response header. What can you conclude about POST form request.
***
##### 4.1.5. Access image, stylesheet from other sites 
* a) In the Inspector panel of sitea.com, can you change sitea’s image to image from siteb.com?
* b) In the Inspector panel of sitea.com, can you change the stylesheet to that of siteb.com?
***
##### 4.1.6. SOP applies to web storage 
* Load www.sitea.com in one tab, open Web developer panel > Storage, check that the 
LocalStorage is empty (Storage >> Local Storage).
  * a) Define a new variable named first = 10. You can define either: with javascript in console: window.LocalStorage[“first”] = 100 or Add Item in Storage tab 
  *
