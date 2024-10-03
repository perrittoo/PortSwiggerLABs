# HTTP Host header attacks

# Bản chất
Lỗ hổng này xảy ra khi server không nhận biết đúng hoặc nhận biết sai, xử lý sai Host header từ các request lên server. Bằng cách nào đó như kiểu là obfuscate hoặc override, sử dụng 1 số header ít sử dụng khác mà có thể làm cho server hiểu lầm và từ đó xử lý sai cách.

# Lab 1: Basic password reset poisoning
Bài lab này bị lỗi ở chỗ là khi truy cập vào chức năng `forgot-password`, thì web sẽ sử dụng Host header để làm nơi nhận cái token lấy password mới.

Vậy mình sẽ sử dụng Burp Intercept để sửa đổi request lên server như sau:
Mình sẽ đến màn hình nhập username hoặc email người cần đổi mật khẩu. Nhập username là `carlos` nhé.
![image](https://hackmd.io/_uploads/B1Am_o5AA.png)

Khi đến POST /forgot-password thì thay đổi Host header thành exploit server mà bài lab cung cấp
![image](https://hackmd.io/_uploads/S1UO_j5AR.png)

Sau bước đó bỏ intercept đi là được. Đến exploit server vào access log bạn sẽ thấy có request sau:
![image](https://hackmd.io/_uploads/B1b2uo9CR.png)

Nó chứa token để xác nhận rằng user `carlos` là người đổi mật khẩu. Giờ dán token đó vào browser thôi.
![image](https://hackmd.io/_uploads/Sk0ltj9AA.png)

Vào đến đây nhập mật khẩu mới cho carlos và đăng nhập là ta solve bài lab.

# Lab 2: Password reset poisoning via middleware
Với các kĩ thuật mà trong docs của PortSwigger có đó là cung cấp 1 Host header bất kì để xem có response `Invalid Host header` hay không, rồi là sử đổi 1 chút hoặc toàn bộ Host header, inject 1 Host header mới, ... thì mình thấy đều không sử dụng được
![image](https://hackmd.io/_uploads/S1ZqjsqRC.png)

Và mình đọc lại phần inject host override header có phần inject 1 header khác `X-Forwarded-Host` và các header khác `X-Forwarded-Server`, `X-HTTP-Host-Override`, `Forwarded`. 

Vậy mình sẽ thử với các header này để chèn exploit server của mình vào, và do bài lab có bảo là carlos sẽ không cẩn thận mà nhấp vào link forgot-password cho nên sẽ có token về access log của server. 

![image](https://hackmd.io/_uploads/B1VvaFoA0.png)
Thêm header đó vào trong request và chờ token trong access log.
![image](https://hackmd.io/_uploads/SJ2K6toC0.png)

Giờ dán token vào url và tạo mật khẩu mới cho carlos rồi đăng nhập thôi
![image](https://hackmd.io/_uploads/SynaTFoAR.png)



# Lab 3: Web cache poisoning via ambiguous requests
Bài lab này chứa lỗ hổng web cache poisoning dựa trên sự khác biệt trong các bộ cache và backend xử lí request mơ hồ. 
Mình thấy dấu hiệu của các lỗ hổng về web cache liên quan đến header `X-Cache` này và cách mà backend sẽ xử lý với những dữ liệu được lưu trữ trong bộ cache

![image](https://hackmd.io/_uploads/BklVbAqCR.png)
Có 1 endpoint thú vị ở đây `/resources/js/tracking.js`
![image](https://hackmd.io/_uploads/S1HobC5AC.png)

File `tracking.js` này sẽ lấy 1 đường dẫn tương đối từ `/resourses/images/tracker.gif`, vậy sẽ ra sao nếu mình thay đổi host header thành server exploit của mình có chứa 1 đường dẫn tương tự

Exploit server sẽ như sau
![image](https://hackmd.io/_uploads/S1WHQ0c0C.png)
Chỉnh sửa Host header như sau
![image](https://hackmd.io/_uploads/rkyomR5CC.png)

Lỗ hổng này khá là mơ hồ cho nên mình check solution để làm case study luôn


# Lab 4: Host header authentication bypass
Bài lab đưa ra giả định về đặc quyền của user dựa trên HTPT Host header, truy cập vào admin panel và xóa user `carlos`

Khi truy cập vào /admin thì chắc chắn bị chặn 
`Admin interface only available to local users`. Vậy mình thử thay Host header thành localhost xem sao.
![image](https://hackmd.io/_uploads/BkxEJh9RA.png)

Mình truy cập được admin panel luôn.
Vậy giờ truy cập `/admin/delete?username=carlos` nữa thôi là được
![image](https://hackmd.io/_uploads/SyZY1ncRR.png)

# Lab 5: Routing-based SSRF
Bài lab này nói về lỗ hổng SSRF dựa trên định tuyến thông qua Host header. Mình có thể khai thác bằng cách truy cập vào mạng nội bộ dựa trên dải IP `192.168.0.0/24` khi đó mình cần brute force IP từ `192.168.0.0` đến `192.168.0.255`

![image](https://hackmd.io/_uploads/SJUGq65RA.png)
Lúc này mình nghĩ đến là có thể là Host header hoặc là 1 header khác là `X-Forwarded-For` cho nên brute force 2 cái cùng 1 lúc với Battering ram luôn. Nhưng mà chạy xong không thấy có gì xảy ra. Nhìn kĩ lại trong request thì Host header của mình lại trở về ban đầu là domain bài lab, lí do là mình quên tắt tính năng update Host header to match target.

Tắt nó đi và thành công tìm ra internal IP của web. 
![image](https://hackmd.io/_uploads/HyqTq69CA.png)
Không giống mấy bài lab trên chỉ cần request đến `/admin/delete?username=carlos` là solve còn bài lab này dùng POST request với csrf token.

![image](https://hackmd.io/_uploads/Hy28jpqRA.png)
Sửa như sau là được
![image](https://hackmd.io/_uploads/B1jjopcCR.png)

# Lab 6: SSRF via flawed request parsing
Bài này cũng giống bài lab trên nhưng cần phải sử dụng 1 kĩ thuật khác và mình phải tìm ra nó.

Sửa Host header sẽ bị lỗi

![image](https://hackmd.io/_uploads/Hk3I269CR.png)

Mình sẽ thử với header `X-Forwarded-For` xem sao. Không được và mình sẽ đọc lại docs để xem các kĩ thuật khác

![image](https://hackmd.io/_uploads/SJ0n3TcAA.png)

Kĩ thuật sửa Host header không khả thi rồi. Còn các kĩ thuật sau: 
1. chèn thêm 1 Host header nữa và bị lỗi
![image](https://hackmd.io/_uploads/rJx86TcC0.png)
2. Chèn 1 URL tuyệt đối vào trong request line
![image](https://hackmd.io/_uploads/rkHn6aq00.png)

Không giống với việc chỉnh sửa Host header sẽ bị forbidden, còn với kĩ thuật này là bị connection time out. Vậy mình sẽ thử kiểm tra sâu hơn (nhớ tắt Update Host header đi).

Thành công nhận được response 200 
![image](https://hackmd.io/_uploads/S1IQAacCC.png)
Tìm ra thành công internal IP của bài lab thì làm theo giống bài lab trước là được.
![image](https://hackmd.io/_uploads/S1ntCT50A.png)

# Lab 7: Host validation bypass via connection state attack
Bài lab cũng bị lỗ hổng SSRF dựa trên định tuyến thông qua Host header. Nhưng frontend đã có những sự xác minh mạnh mẽ với host header, nó lại đưa ra giả định rằng tất cả request trên 1 connection dựa trên request đầu tiên nó nhận được. 
Và bài lab cung cấp cho chúng ta IP của nó luôn `192.168.0.1/admin`

Những lỗ hổng kiểu như này giống như kiểm soát truy cập access control. Nó chỉ kiểm tra, xác minh request đầu tiên của 1 connection và những request phía sau thì không check kĩ càng.
Vậy ở đây mình sẽ sử dụng tính năng send a group tab in a single connection. 

Request đầu tiên sẽ là dummy request, request phía sau là request payload thật. 
![image](https://hackmd.io/_uploads/ByAzg09RA.png)
![image](https://hackmd.io/_uploads/S12Ql05AA.png)

Request thứ hai đã thành công truy cập với response code 200. Giờ thực hiện giống 2 bài lab trên thôi.
![image](https://hackmd.io/_uploads/rJeFeA9RC.png)
