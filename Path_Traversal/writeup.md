**Lab: File path traversal, simple case**
![alt text](image/1.png)
Truy cập và lab ta thấy một trang web bán hàng quen thuộc của PortSwigger
![alt text](image/3.png)
Chức năng duy nhất mà trang web này có là xem sản phẩm
![alt text](image/2.png)
Ta có thể tìm thấy đường dẫn đến các file ảnh sảm của trang web trông dev tool. Điểm này có thể dẫn đến lỗ hổng path traversal
![alt text](image/4.png)

Tiếp theo ta bắt req truy cấp đến file ảnh bất kì rồi thay đổi đường dẫn là có kết quả
![alt text](image/5.png) 
Vậy là hoàn thành bài lab này
------------------------------------------------------------------
**Lab: File path traversal, traversal sequences blocked with absolute path bypass**
![alt text](image/6.png)
Đề bài có nói là `Ứng dụng chặn các chuỗi duyệt nhưng coi tên tệp được cung cấp là tên tương đối so với thư mục làm việc mặc định`. Vì vậy nên các req có `../` đã bị chặn nhưng vẫn chấp nhận các tệp trong thư mục mặc định
![alt text](image/7.png)
Vì vậy ta chỉ việc truy cập thẳng vào `/etc/paswd` là hoàn thành lab
------------------------------------------------------------------
**Lab: File path traversal, traversal sequences stripped non-recursively**
![alt text](image/8.png)
Đề bài của bài lab này nói rằng `Ứng dụng sẽ loại bỏ các chuỗi duyệt thư mục khỏi tên tệp do người dùng cung cấp trước khi sử dụng`. Vậy nên các chuỗi `../` sẽ bị loại bỏ khi ta gửi req
![alt text](image/9.png)
Tuy nhiên chung ta chỉ cần chỉnh sửa sao cho sau khi trang web loại bỏ các chuỗi duyệt thư mục mà vẫn có hiệu lực
![alt text](image/10.png)
------------------------------------------------------------------
**Lab: File path traversal, traversal sequences stripped with superfluous URL-decode**
![alt text](image/11.png)
Bài lab này đã `chặn các đầu vào chứa chuỗi tấn công duyệt thư mục` tuy nhiên nó `thực hiện giải mã URL của đầu vào trước khi sử dụng`. Vậy chung ta cần mã hóa URL payload 
![alt text](image/12.png)
------------------------------------------------------------------
**Lab: File path traversal, validation of start of path**
![alt text](image/13.png)
Bài lab này duyệt đường dẫn của file bắt đầu từ thư mục gốc
![alt text](image/14.png)

Vì vậy ta chỉ vần thêm đường dẫn của thư mục gốc vào payload


![alt text](image/15.png)
------------------------------------------------------------------

**Lab: File path traversal, validation of file extension with null byte bypass**
![alt text](image/16.png)
Có hai điều cần chú ý trong bài lab này.

1. Bài lab xác thực phần mở rộng tệp bằng cách bỏ qua ký tự null

2. Bài lab sẽ kiểm tra xem tên tệp được cung cấp có kết thúc bằng phần mở rộng tệp mong muốn hay không

Vậy có nghĩ là trang web sẽ kiểm tra xem đầu vào có phải file hình ảnh hay không và có cớ chês bỏ qua kí tự null
vậy ta chỉ cần thêm đuôi `.png` để vượt qua `2` và tận dụng `1` để bài lab không hiểu nhầm ý nghĩa của payload
![alt text](image/17.png)