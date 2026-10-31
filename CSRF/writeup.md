**Lab: CSRF vulnerability with no defenses**

![alt text](images/1.png)

Đề bài yêu cầu khai thác lỗ hổng CSRF trên chức năng đổi email của tài khoản người dùng khi ứng dụng không có bất kỳ cơ chế phòng thủ nào.

Truy cập vào trang đăng nhập và đăng nhập vào tài khoản được cung cấp.

![alt text](images/2.png)

Ta thực hiện đổi email một lần để bắt request đổi email chuẩn.

![alt text](images/3.png)

Quan sát request, ta thấy phiên làm việc chỉ được xác thực bằng duy nhất `Cookie: session`, hoàn toàn không có token CSRF hay cơ chế xác thực nào khác để kiểm tra xem request có thực sự bắt nguồn từ người dùng hay không.

![alt text](images/4.png)

Ta sử dụng tính năng Generate CSRF PoC của Burp Suite để tạo mã khai thác HTML.

![alt text](images/5.png)

Đưa mã HTML lên Exploit Server. Đề bài lưu ý mỗi tài khoản phải có email duy nhất, vì vậy ta sửa email cần đổi thành một địa chỉ mới rồi gửi link cho nạn nhân (Deliver exploit to victim).

![alt text](images/6.png)

Nạn nhân truy cập vào link và bị đổi email thành công. Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: CSRF where token validation depends on request method**

![alt text](images/7.png)

Bài lab này yêu cầu khai thác lỗi CSRF trong cơ chế kiểm tra token, cụ thể là ứng dụng chỉ xác thực token khi request sử dụng phương thức POST, còn khi đổi sang GET thì bỏ qua việc kiểm tra.

Đăng nhập vào tài khoản `wiener:peter`.

![alt text](images/8.png)
![alt text](images/9.png)

Thực hiện đổi email sang `test_001@gmail.com` để lấy request gốc. Quan sát thấy trong body đã có tham số `csrf`.

![alt text](images/10.png)

Ta thử chuyển phương thức của request từ POST sang GET thì thấy server vẫn phản hồi mã 302 chuyển hướng.

![alt text](images/11.png)

Kiểm tra lại giao diện người dùng thì thấy email đã được thay đổi thành công.

![alt text](images/12.png)

Tiếp tục kiểm tra bằng cách xóa hẳn tham số `csrf` trong request GET, server vẫn chấp nhận và trả về 302.

![alt text](images/13.png)
![alt text](images/14.png)

Điều này chứng minh ứng dụng chỉ kiểm tra mã CSRF đối với method POST. Bây giờ ta tạo mã khai thác HTML với phương thức GET (hoặc form method GET) để lưu trữ trên Exploit Server.

![alt text](images/15.png)

Lưu mã khai thác lên Exploit Server.

![alt text](images/16.png)

Thử truy cập trực tiếp bằng link của Exploit Server, email của tài khoản đã được tự động thay đổi.

![alt text](images/17.png)

Đổi lại email mới và gửi exploit cho victim để hoàn thành bài lab.

-----------------------------------------------------------------

**Lab: CSRF where token validation depends on token being present**

![alt text](images/18.png)

Bài lab này kiểm tra một lỗi phổ biến khác trong xác thực CSRF: ứng dụng chỉ kiểm tra token khi tham số đó xuất hiện trong request, nếu người dùng xóa hẳn tham số token đi thì server sẽ bỏ qua kiểm tra hoàn toàn.

Đăng nhập vào tài khoản `wiener`.

![alt text](images/19.png)
![alt text](images/20.png)

Thực hiện thao tác đổi email để bắt request mẫu.

![alt text](images/21.png)

Trong request POST đổi email, ta thử xóa bỏ hoàn toàn tham số `csrf` rồi gửi đi. Server vẫn chấp nhận và phản hồi mã 302.

![alt text](images/22.png)

Kiểm tra thông tin tài khoản thấy email đã được cập nhật thành công mà không cần token CSRF.

![alt text](images/23.png)

Ta soạn thảo mã HTML PoC không chứa thẻ input `csrf`:

```html
<html>
  <body>
    <form action="https://0aa300e604f940b084b4951a00200031.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="test_0021@gmail.com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

![alt text](images/24.png)

Thử nghiệm truy cập link trên Exploit Server thấy email được đổi tự động.

![alt text](images/25.png)

Cuối cùng thay đổi email và gửi exploit cho victim để hoàn thành bài lab.

![alt text](images/26.png)

-----------------------------------------------------------------

**Lab: CSRF where token is not tied to user session**

![alt text](images/27.png)

Trong bài này ứng dụng có sử dụng token CSRF nhưng lại không ràng buộc token đó với phiên đăng nhập của người dùng. Có nghĩa là ta có thể lấy một token hợp lệ từ tài khoản của mình và dùng nó để thực hiện thao tác thay cho tài khoản nạn nhân.

Đăng nhập vào tài khoản `wiener`.

![alt text](images/28.png)
![alt text](images/29.png)

Khi thử đổi email mà xóa tham số `csrf` thì server báo lỗi `Missing parameter 'csrf'`. Nếu nhập token sai thì báo `Invalid CSRF token`.

![alt text](images/30.png)
![alt text](images/31.png)

Ta bật Intercept trong Burp Suite, gửi request đổi email từ tài khoản `wiener` để lấy một mã CSRF token hợp lệ, sau đó drop request này đi để token chưa bị tiêu thụ.

![alt text](images/32.png)

Kiểm tra tài khoản thứ hai `carlos` hiện đang có email là `carlos1@dontwannacry.com`.

![alt text](images/33.png)

Bắt request đổi email của `carlos` rồi thay mã `csrf` của `wiener` vừa lấy được vào, server lập tức trả về mã 302 chấp nhận yêu cầu.

![alt text](images/34.png)

Email của `carlos` đã được đổi thành `carlos2@dontwannacry.com`.

![alt text](images/35.png)

Ta tiếp tục lấy một token hợp lệ mới từ tài khoản `wiener`, đưa vào mã HTML PoC trên Exploit Server rồi gửi cho victim để hoàn thành lab.

![alt text](images/36.png)

-----------------------------------------------------------------

**Lab: CSRF where token is tied to non-session cookie**

![alt text](images/37.png)

Bài lab này nâng cấp hơn khi token CSRF được ràng buộc với một cookie riêng biệt gọi là `csrfKey` chứ không gắn với cookie phiên `session`. Nếu ứng dụng có thêm một lỗ hổng cho phép set cookie (ví dụ như CRLF injection trong tính năng tìm kiếm), ta có thể ép trình duyệt của nạn nhân nhận `csrfKey` của ta.

Vào tài khoản và quan sát request đổi email.

![alt text](images/38.png)

Ta thấy request chứa cả cookie `csrfKey` và tham số `csrf`.

![alt text](images/39.png)

Lấy một cặp `csrfKey` và `csrf` hợp lệ từ tài khoản của mình và đưa vào request của nạn nhân, server trả về mã 302 thành công.

![alt text](images/40.png)

Kiểm tra thấy email của nạn nhân đã bị đổi.

![alt text](images/41.png)

Để tấn công nạn nhân qua CSRF, ta cần vừa gửi form đổi email vừa tiêm cookie `csrfKey` vào trình duyệt nạn nhân. Quan sát chức năng tìm kiếm có phản hồi header `Set-Cookie` khi tiêm ký tự xuống dòng CRLF `%0d%0a`, ta viết mã HTML PoC như sau:

```html
<html>
  <body>
    <form action="https://0a0f006504e3c02380ef1c5d00d40095.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="pwned@carlos-montoya.net" />
      <input type="hidden" name="csrf" value="ubDXn0VZuMttxghOLimw91hNxB3dDuVv" />
      <input type="submit" value="Submit request" />
    </form>
    <img src="https://0a0f006504e3c02380ef1c5d00d40095.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=uHzdPbkowQhOI6V0bTkD9IHtZg4rJ75w%3b%20SameSite=None" onerror="document.forms[0].submit()">
  </body>
</html>
```

![alt text](images/42.png)

Khi nạn nhân truy cập vào trang, thẻ `img` sẽ kích hoạt request tới chức năng search để ghi cookie `csrfKey`, sau đó sự kiện `onerror` tự động submit form đổi email.

![alt text](images/43.png)

Lấy cặp token mới, cập nhật vào Exploit Server và gửi cho victim để hoàn thành lab.

![alt text](images/44.png)

-----------------------------------------------------------------

**Lab: CSRF where token is duplicated in cookie**

![alt text](images/45.png)

Ứng dụng trong bài này sử dụng cơ chế phòng thủ Double Submit Cookie: mã CSRF token xuất hiện đồng thời trong cookie `csrf` và trong tham số POST `csrf`. Server chỉ kiểm tra xem hai giá trị này có trùng khớp nhau hay không mà không xác thực phía máy chủ.

Đăng nhập vào tài khoản kiểm tra chức năng đổi email.

![alt text](images/46.png)

Quan sát request, ta thấy giá trị của cookie `csrf` và tham số `csrf` trong body hoàn toàn giống hệt nhau.

![alt text](images/47.png)

Nếu chỉ đổi tham số `csrf` trong body thì server sẽ báo `Invalid CSRF token`. Nhưng nếu ta có thể thay đổi đồng thời cả hai giá trị thành cùng một chuỗi tùy ý thì server sẽ chấp nhận.

![alt text](images/48.png)
![alt text](images/49.png)

Thử nghiệm trực tiếp trên trình duyệt cá nhân bằng mã PoC.

![alt text](images/50.png)

Để tấn công victim, ta tiếp tục lợi dụng lỗi chèn header (CRLF injection) ở tham số tìm kiếm để gán cookie `csrf=fake123; SameSite=None` cho trình duyệt của nạn nhân.

![alt text](images/51.png)

Sau đó submit form đổi email với tham số `csrf` cũng mang giá trị `fake123`.

```html
<html>
  <body>
    <form action="https://0a300015043478b180de035200b00099.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="attacker@carlos-montoya.net" />
      <input type="hidden" name="csrf" value="fake123" />
      <input type="submit" value="Submit request" />
    </form>
    <img src="https://0a300015043478b180de035200b00099.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=fake123%3b%20SameSite=None" onerror="document.forms[0].submit()">
  </body>
</html>
```

![alt text](images/52.png)

Lưu mã PoC vào Exploit Server và gửi cho nạn nhân.

![alt text](images/53.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: SameSite Lax bypass via method override**

![alt text](images/54.png)

Mục tiêu của bài lab này là vượt qua cơ chế bảo vệ cookie `SameSite=Lax`. Khi cookie đặt `SameSite=Lax`, các request dạng POST từ trang ngoài sẽ không gửi kèm cookie phiên, nhưng request dạng GET điều hướng cấp cao nhất (top-level navigation) vẫn được gửi cookie.

Đăng nhập vào tài khoản để lấy request đổi email.

![alt text](images/55.png)
![alt text](images/56.png)
![alt text](images/57.png)

Nếu ta gửi request bằng mã PoC POST thông thường từ Exploit Server thì phiên làm việc bị ngắt và nạn nhân bị logout do trình duyệt không gửi kèm cookie session.

![alt text](images/58.png)

Thử đổi request sang method GET thông thường thì server từ chối vì chức năng chỉ chấp nhận POST.

![alt text](images/59.png)

Tuy nhiên, ứng dụng lại hỗ trợ tham số ghi đè phương thức `_method=POST`. Khi gửi bằng GET kèm theo `_method=POST`, server vẫn xử lý như một request POST hợp lệ và trả về 302.

![alt text](images/60.png)

Kiểm tra tài khoản thấy email đã được thay đổi thành công.

![alt text](images/61.png)
![alt text](images/62.png)

Ta tạo mã HTML PoC sử dụng form method GET kết hợp với input ẩn `_method=POST`:

```html
<html>
  <body>
    <form action="https://0ade00bd03b23a26aa32c99500100076.web-security-academy.net/my-account/change-email">
      <input type="hidden" name="email" value="pwned@gmail.com" />
      <input type="hidden" name="_method" value="POST" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

![alt text](images/63.png)

Lưu file trên Exploit Server và kiểm tra tính năng.

![alt text](images/64.png)

Request điều hướng GET kích hoạt thành công việc gửi cookie session và đổi email.

![alt text](images/65.png)

Gửi exploit cho victim để giải quyết lab.

![alt text](images/66.png)

-----------------------------------------------------------------

**Lab: SameSite Strict bypass via client-side redirect**

![alt text](images/67.png)

Bài lab cấu hình cookie `SameSite=Strict`, nghĩa là trình duyệt sẽ không bao giờ gửi cookie trong bất kỳ cross-site request nào. Để vượt qua, ta phải tìm một lỗi chuyển hướng phía máy khách (client-side redirect) ngay trên chính tên miền mục tiêu để biến request thành same-site.

Đăng nhập vào tài khoản và kiểm tra cấu hình cookie.

![alt text](images/68.png)
![alt text](images/69.png)

Thao tác đổi email gửi request POST trả về 302.

![alt text](images/70.png)
![alt text](images/71.png)

Nếu dùng mã HTML thông thường từ site khác, request bị từ chối do không có cookie phiên.

![alt text](images/72.png)
![alt text](images/73.png)

Khi chuyển method sang GET, server vẫn chấp nhận thay đổi email qua URL `/my-account/change-email?email=...`.

![alt text](images/74.png)
![alt text](images/75.png)

Bây giờ ta tìm kiếm lỗ hổng chuyển hướng trên trang web. Quan sát chức năng đăng bình luận bài viết, sau khi gửi comment, ứng dụng hiển thị trang xác nhận rồi dùng JavaScript để chuyển hướng người dùng quay lại bài viết.

![alt text](images/76.png)
![alt text](images/77.png)

Đoạn mã chuyển hướng trong file JavaScript:

```javascript
redirectOnConfirmation = (blogPath) => {
    setTimeout(() => {
        const url = new URL(window.location);
        const postId = url.searchParams.get("postId");
        window.location = blogPath + '/' + postId;
    }, 3000);
}
```

Hàm này lấy trực tiếp giá trị tham số `postId` từ URL và nối vào đường dẫn chuyển hướng mà không kiểm tra hợp lệ.

![alt text](images/78.png)

Ta có thể áp dụng kỹ thuật Path Traversal vào tham số `postId` để điều hướng trang web tới `/my-account/change-email`.

![alt text](images/79.png)

Thử nghiệm trên trình duyệt với tham số `postId=../my-account/change-email?email=test%40gmail.com&submit=1`.

![alt text](images/80.png)
![alt text](images/81.png)

Mã hóa URL các tham số con và chạy thử trên trình duyệt.

![alt text](images/82.png)

Trang web tự động chuyển hướng và thực hiện đổi email thành công cùng với cookie `SameSite=Strict`.

![alt text](images/83.png)

Soạn mã HTML trên Exploit Server để điều hướng nạn nhân tới URL chuyển hướng trên trang web chính.

![alt text](images/84.png)
![alt text](images/85.png)

Gửi payload cho victim để hoàn thành lab.

![alt text](images/86.png)

-----------------------------------------------------------------

**Lab: CSRF where Referer validation depends on header being present**

![alt text](images/87.png)

Bài lab này bảo vệ CSRF bằng cách kiểm tra header `Referer` trong request, nhưng cơ chế kiểm tra chỉ được thực hiện khi header `Referer` có mặt. Nếu request không có header này thì server sẽ bỏ qua xác thực.

Đăng nhập vào tài khoản và thực hiện đổi email.

![alt text](images/88.png)
![alt text](images/89.png)
![alt text](images/90.png)

Khi tạo form CSRF từ Exploit Server và gửi đi, server chặn lại với thông báo `Invalid referer header`.

![alt text](images/91.png)

Để ngăn trình duyệt gửi header `Referer` khi submit form, ta thêm thẻ meta `<meta name="referrer" content="never">` (hoặc `no-referrer`) vào trang HTML khai thác:

```html
<html>
  <body>
    <form action="https://0acf00f3043c59b58017034600fe00dc.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="pwned@gmail.com" />
      <input type="submit" value="Submit request" />
      <meta name="referrer" content="never">
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit(); 
    </script>
  </body>
</html>
```

![alt text](images/92.png)

Thử nghiệm trên Exploit Server, request gửi đi không còn chứa header `Referer` và email đã được đổi thành công.

Gửi exploit cho victim để hoàn thành bài lab.

![alt text](images/93.png)

-----------------------------------------------------------------

**Lab: CSRF with broken Referer validation**

![alt text](images/94.png)

Bài lab này kiểm tra header `Referer` một cách lỏng lẻo bằng biểu thức kiểm tra xem chuỗi tên miền của ứng dụng có xuất hiện ở bất kỳ đâu trong header `Referer` hay không, thay vì kiểm tra tên miền gốc.

Đăng nhập vào tài khoản và bắt request đổi email.

![alt text](images/95.png)
![alt text](images/96.png)

Nếu xóa hẳn header `Referer` hoặc để domain khác, server sẽ báo lỗi `Invalid referer header`.

![alt text](images/97.png)
![alt text](images/98.png)

Tuy nhiên, khi ta thêm tên miền của lab vào sau query parameter của header `Referer` (ví dụ `https://example.com/test?https://0aa0001f03710643809712c0007e0021.web-security-academy.net`) thì server lại trả về mã 302 chấp nhận request.

![alt text](images/99.png)

Kiểm tra thấy email đã được thay đổi.

![alt text](images/100.png)

Để trình duyệt gửi toàn bộ đường dẫn query của Exploit Server trong header Referer, ta cần cấu hình header phản hồi `Referrer-Policy: unsafe-url` trên Exploit Server.

![alt text](images/101.png)

Ban đầu nếu trong mã khai thác có lệnh `history.pushState('', '', '/');`, đoạn code này sẽ cắt bỏ query string trên thanh địa chỉ dẫn đến mất chuỗi domain của lab.

![alt text](images/102.png)
![alt text](images/103.png)

Ta điều chỉnh lại mã JavaScript, sử dụng `history.pushState` để gán query string chứa tên miền của lab vào URL trước khi submit:

```html
<html>
  <body>
    <form action="https://0aa0001f03710643809712c0007e0021.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="pwned_final@gmail.com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/test?0aa0001f03710643809712c0007e0021.web-security-academy.net');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

![alt text](images/104.png)

Lưu mã trên Exploit Server kèm theo header `Referrer-Policy: unsafe-url`. Khi gửi cho victim, header `Referer` gửi đi sẽ chứa chuỗi tên miền của lab và vượt qua bộ lọc.

![alt text](images/105.png)

Vậy là hoàn thành bài lab này.

