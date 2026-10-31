**Lab: Web shell upload via obfuscated file extension**

![alt text](images/1.png)

Đề bài cho biết trang web có chức năng upload ảnh nhưng bị lỗi cấu hình blacklist phần mở rộng tệp. Mục tiêu của chúng ta là upload một web shell PHP để đọc nội dung file `/home/carlos/secret`.

Đầu tiên ta đăng nhập vào tài khoản được cung cấp `wiener:peter`.

![alt text](images/2.png)

Ta thử upload một file ảnh hợp lệ lên hệ thống.

![alt text](images/3.png)

Ảnh được upload thành công. Tiếp theo khi thử gửi một file web shell với phần mở rộng `.php` thì server lập tức từ chối.

![alt text](images/4.png)

Thử upload file cấu hình `.htaccess` để ghi đè quy tắc MIME type nhưng cũng bị server chặn.

![alt text](images/5.png)

Ta thử kết hợp cả hai đuôi `.png` và `.php` xem server có hỗ trợ xử lý cả hai định dạng hay không. Kết quả là upload thành công.

![alt text](images/6.png)

Tuy nhiên khi truy cập vào file thì thấy server chỉ xử lý như một file ảnh bình thường chứ không thực thi mã PHP.

![alt text](images/7.png)

Ta thử đặt tên `pwn.php.php` để xem server có kiểm tra theo kiểu cắt bỏ (trim) chuỗi `.php` một lần hay không.

![alt text](images/8.png)

Tiếp tục thử với `pwn.p.phphp` xem cơ chế trim có xử lý đệ quy hay không.

![alt text](images/9.png)

Bây giờ ta thử kỹ thuật chèn ký tự null byte `%00` vào tên file (`pwn.php%00.png`). Cơ chế tiếp nhận chuỗi sẽ nhận toàn bộ tên file, nhưng khi xử lý hệ thống tệp ở tầng dưới thì ký tự null sẽ kết thúc chuỗi và lưu file thành `pwn.php`. Request gửi đi thành công với mã 200 OK.

![alt text](images/10.png)

Khi truy cập vào file `pwn.php` thì thấy đoạn mã PHP không còn hiển thị nguyên văn nữa, chứng tỏ server đã thực thi code PHP.

![alt text](images/11.png)

Bây giờ ta chỉnh sửa nội dung file `pwn.php` để đọc file hệ thống `/etc/passwd`.

![alt text](images/12.png)

Server đã trả về nội dung của file `/etc/passwd`.

![alt text](images/13.png)

Cuối cùng ta thay đổi payload để trích xuất nội dung file `/home/carlos/secret` rồi submit đáp án.

![alt text](images/14.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Web shell upload via path traversal**

![alt text](images/15.png)

Đề bài cho biết trang web có tính năng upload ảnh nhưng thư mục lưu trữ ảnh bị vô hiệu hóa quyền thực thi mã phía máy chủ. Tuy nhiên ta có thể tận dụng lỗ hổng path traversal để lưu file web shell sang một thư mục khác có quyền thực thi. Mục tiêu vẫn là đọc file `/home/carlos/secret`.

Ta đăng nhập vào tài khoản `wiener:peter`.

![alt text](images/16.png)

Thử upload một file web shell `.php` lên server thì nhận được phản hồi 200 OK ngay.

![alt text](images/17.png)

Nhưng khi truy cập trực tiếp vào file trong thư mục ảnh thì server chỉ trả về nội dung text của file chứ không chạy code. Điều này cho thấy thư mục chứa ảnh không có quyền thực thi PHP, ta cần đưa file ra thư mục cha.

![alt text](images/18.png)

Ta thử kỹ thuật path traversal bằng cách chèn `../` vào tham số `filename`, nhưng có vẻ server đã lọc bỏ chuỗi `../`.

![alt text](images/19.png)

Tiếp theo ta thử mã hóa URL chuỗi path traversal thành `..%2f` (hoặc double URL encode `..%252f`). Lần này file đã được upload thành công sang thư mục cấp trên `/files/avatars/..`.

![alt text](images/20.png)

Truy cập vào file vừa upload, server đã thực thi mã PHP và trả về nội dung của `/etc/passwd`. Việc còn lại là sửa payload để đọc `/home/carlos/secret` và submit kết quả.

![alt text](images/21.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Remote code execution via polyglot web shell upload**

![alt text](images/22.png)

Bài lab này yêu cầu khai thác tính năng upload ảnh để thực thi mã từ xa. Điểm đặc biệt là ứng dụng có cơ chế kiểm tra nội dung tệp xem có thực sự là một file ảnh hợp lệ hay không trước khi lưu. Ta cần đọc nội dung file `/home/carlos/secret`.

Đăng nhập vào tài khoản `wiener:peter`.

![alt text](images/23.png)

Khi thử upload một file web shell `.php` thông thường thì server từ chối ngay vì phát hiện nội dung không phải file ảnh.

![alt text](images/24.png)

Thử đổi tên thành `.php.jpg` cũng vẫn bị chặn, chứng tỏ server kiểm tra header/nội dung ảnh chứ không chỉ dựa vào phần mở rộng tệp.

![alt text](images/25.png)

Khi gửi một file có nội dung ảnh thật nhưng đổi đuôi thành `.php` thì server lại trả về 200 OK.

![alt text](images/26.png)

Từ đây ta có ý tưởng tạo một file ảnh đa hình (polyglot) vừa là ảnh hợp lệ vừa chứa mã PHP bên trong metadata. Ta sử dụng công cụ ExifTool để nhúng đoạn code PHP vào trường comment của bức ảnh.

![alt text](images/27.png)

Sau khi tạo xong file polyglot, ta upload file lên server.

![alt text](images/28.png)

Gửi request truy cập file ảnh vừa upload, server đã thực thi đoạn code PHP nhúng bên trong và hiển thị nội dung file `/etc/passwd`.

![alt text](images/29.png)

Bây giờ ta đổi payload thành lệnh đọc file `/home/carlos/secret`, lấy mã bí mật và submit đáp án để giải quyết lab.

![alt text](images/30.png)

Vậy là hoàn thành lab.

