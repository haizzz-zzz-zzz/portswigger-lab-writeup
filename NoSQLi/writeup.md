**Lab: Detecting NoSQL injection**

![alt text](images/1.png)

Đề bài cho biết ứng dụng có lỗ hổng NoSQL injection trong chức năng lọc danh mục sản phẩm. Mục tiêu là khai thác lỗ hổng để hiển thị các sản phẩm chưa được phát hành ra bên ngoài.

Khi ta chọn danh mục và gửi một request hợp lệ thì server trả về mã phản hồi 200 OK bình thường.

![alt text](images/2.png)

Ta thử chèn thêm một dấu nháy đơn `'` vào giá trị tham số lọc thì thấy server lập tức trả về lỗi 500 Internal Server Error.

![alt text](images/3.png)
![alt text](images/4.png)

Nhìn vào thông báo lỗi trả về ta có thể nhận biết được ứng dụng đang sử dụng cơ sở dữ liệu MongoDB và đầu vào không được xử lý an toàn. Bây giờ ta thêm payload chứa biểu thức điều kiện Boolean luôn đúng `'||'1'=='1`.

![alt text](images/5.png)
![alt text](images/6.png)

Server trả về mã 200 OK và danh sách toàn bộ sản phẩm đã được hiển thị, bao gồm cả những sản phẩm chưa phát hành.

![alt text](images/7.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Exploiting NoSQL operator injection to bypass authentication**

![alt text](images/8.png)

Bài lab yêu cầu chúng ta khai thác lỗ hổng NoSQL injection trên MongoDB để vượt qua cơ chế xác thực và đăng nhập vào tài khoản của người dùng `administrator`.

Vào trang đăng nhập và thử gửi thông tin tài khoản mẫu, ta thấy dữ liệu gửi lên server được định dạng dưới dạng JSON.

![alt text](images/9.png)

Ta thử thay đổi trường `password` thành toán tử so sánh `{"$ne": null}` (không bằng null) trong khi giữ nguyên `username` là `wiener`. Server trả về mã 302 chuyển hướng và đăng nhập thành công vào tài khoản `wiener`.

![alt text](images/10.png)
![alt text](images/11.png)

Tuy nhiên khi áp dụng tương tự với `username` là `administrator` thì server lại báo lỗi `Invalid username or password`.

![alt text](images/12.png)

Ta chuyển sang thử nghiệm toán tử `$regex` dùng để tìm kiếm chuỗi khớp với mẫu biểu thức chính quy. Thử với `wien.*` thì đăng nhập được vào tài khoản `wiener`.

![alt text](images/13.png)

Bây giờ ta đổi giá trị `username` thành `{"$regex": "admin.*"}` kết hợp với `password: {"$ne": ""}`. Server trả về mã 302 chuyển hướng đăng nhập.

![alt text](images/14.png)
![alt text](images/15.png)

Hệ thống đã xác thực thành công tài khoản administrator.

Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Exploiting NoSQL injection to extract data**

![alt text](images/16.png)

Đề bài yêu cầu trích xuất mật khẩu của tài khoản `administrator` thông qua lỗ hổng NoSQL injection rồi đăng nhập vào hệ thống để hoàn thành bài lab.

Khi đăng nhập vào tài khoản, ta bắt được một request kiểm tra quyền người dùng tại endpoint `/user/lookup?user=wiener`.

![alt text](images/17.png)

Ta thử gửi tham số với một người dùng không tồn tại `wiener1` thì server báo `Could not find user`. Khi chèn ký tự `'` để phá vỡ cú pháp truy vấn thì server trả về thông báo lỗi `There was an error getting user details`. Điều này xác nhận chức năng có tồn tại lỗ hổng NoSQL injection.

![alt text](images/18.png)
![alt text](images/19.png)

Ý tưởng khai thác là xây dựng một mệnh đề Boolean:
`(Tên user đúng AND Ký tự mật khẩu khớp) OR Biểu thức luôn sai`
- Nếu ký tự đoán đúng, toàn bộ biểu thức trả về `True` và server hiển thị thông tin tài khoản.
- Nếu ký tự đoán sai, biểu thức trả về `False` và server không tìm thấy người dùng.

Ta sử dụng payload: `wiener' && this.password[0]=='a' || '1'=='0` rồi gửi sang tab Intruder để brute-force. Thiết lập Payload Type là Brute Forcer với tập ký tự chữ cái thường, độ dài là 1.

![alt text](images/20.png)

Cấu hình tính năng Grep - Match chuỗi `"username": "wiener"` trong phản hồi để nhận diện lúc nào payload khớp.

![alt text](images/21.png)

Bắt đầu tấn công thì thấy không có kết quả nào khớp chuỗi do payload chưa được mã hóa URL.

![alt text](images/22.png)

Ta tiến hành URL encode lại toàn bộ payload trước khi gửi.

![alt text](images/23.png)

Chạy lại cuộc tấn công, ta đã xác định được ký tự hợp lệ đầu tiên.

![alt text](images/24.png)

Tiếp theo ta kiểm tra độ dài mật khẩu bằng payload:
`wiener' && this.password.length > 0 || '1'=='0`

Đưa vào Intruder để kiểm tra các giá trị tăng dần.

![alt text](images/25.png)

Kết quả cho thấy khi độ dài vượt quá 5 thì xuất hiện thông báo `Could not find user`, chứng minh mật khẩu của `wiener` có độ dài đúng bằng 5 ký tự.

![alt text](images/26.png)

Kiểm tra từng ký tự, ta trích xuất được mật khẩu của `wiener` là `peter`. Phương pháp khai thác đã được kiểm chứng chính xác.

![alt text](images/27.png)
![alt text](images/28.png)

Bây giờ ta áp dụng quy trình trên cho tài khoản `administrator`. Đầu tiên tìm được độ dài mật khẩu của `administrator` là 8 ký tự.

![alt text](images/29.png)

Tiếp tục brute-force trích xuất từng vị trí ký tự, ta tìm ra toàn bộ mật khẩu của `administrator` là `mcmrcwyk`.

![alt text](images/30.png)
![alt text](images/31.png)
![alt text](images/32.png)

Cuối cùng ta quay lại trang đăng nhập, điền tài khoản `administrator` với mật khẩu vừa trích xuất được.

![alt text](images/33.png)
![alt text](images/34.png)

Vậy là hoàn thành bài lab này.

