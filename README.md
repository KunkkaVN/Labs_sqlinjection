**1. level 1 - SQL injection
----------------------------

Sử dụng bypass: `' or ''='`

**2. Lập trình php, mô phỏng lại lỗ hỏng của level 1
----------------------------------------------------

```sh
<!DOCTYPE html>
<html>
    <head>
        <title></title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
        <?php
            session_start();
            header('Content-Type: text/html; charset=UTF-8');
            if (isset($_POST['dangnhap']))
            {
                $ketnoi['host'] = 'localhost';
                $ketnoi['dbname'] = 'demo';
                $ketnoi['username'] = 'root';
                $ketnoi['password'] = '';
                 @mysql_connect(
                    "{$ketnoi['host']}",
                    "{$ketnoi['username']}",
                    "{$ketnoi['password']}")
                or
                    die("Không thể kết nối database");
                @mysql_select_db(
                "{$ketnoi['dbname']}") 
                or
                    die("Không thể chọn database");
                $username = $_POST['txtUsername'];
                $password = $_POST['txtPassword'];
                if (!$username || !$password) 
                {
                    echo "Vui lòng nhập đầy đủ tên đăng nhập và mật khẩu. <a href='javascript: history.go(-1)'>Trở lại</a>";
                    exit;
                }

                $query = mysql_query("SELECT username, password FROM member WHERE username='$username'");
                 if (mysql_num_rows($query) == 0) 
                {
                    echo "Tên đăng nhập này không tồn tại. Vui lòng kiểm tra lại. <a href='javascript: history.go(-1)'>Trở lại</a>";
                    exit;
                }
                $row = mysql_fetch_array($query);
                if ($password != $row['password']) 
                {
                    echo "Mật khẩu không đúng. Vui lòng nhập lại. <a href='javascript: history.go(-1)'>Trở lại</a>";
                    exit;
                }
                $_SESSION['username'] = $username;
                echo "Xin chào " . $username . ". Bạn đã đăng nhập thành công. <a href='/'>Về trang chủ</a>";
                die();
            }

        ?>
        <form action='dangnhap.php?do=login' method='POST'>
            <table cellpadding='0' cellspacing='0' border='1'>
                <tr>
                    <td>
                        Tên đăng nhập :
                    </td>
                    <td>
                        <input type='text' name='txtUsername' />
                    </td>
                </tr>
                <tr>
                    <td>
                        Mật khẩu :
                    </td>
                    <td>
                        <input type='password' name='txtPassword' />
                    </td>
                </tr>
            </table>
            <input type='submit' name="dangnhap" value='Đăng nhập' />
        </form>
    </body>
</html>
```
**2.khắc phục lỗi sql injection của level 1:
--------------------------------------------

sử dụng hàm addslashes() trước khi get post lấy mật khẩu tài khoản:

$username = addslashes($_POST['txtUsername']);
$password = addslashes($_POST['txtPassword']);
