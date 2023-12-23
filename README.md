# PEMOGRAMAAN WEB

## Nama : Ramadhan Ardi Iman Prakoso
## NIM : 312210722
## Kelas : TI. 22. A.3

# PRAKTIKUM 12

## MEMBUAT TABEL : USER LOGIN

![menambahkan_gambar](img/BUAT%20TABEL%2013.png)

Kalian bisa langsung saja membuat sebuah tabel pada PHPMyAdmin dengan mengklik tombol MySQL yang ada diatas kemudian masukan kode dibawah kemudian klik kirim.

```mysql
CREATE TABLE user (
    id INT(11) auto_increment,
    username VARCHAR(200) NOT NULL,
    useremail VARCHAR(200),
    userpassword VARCHAR(200),
    PRIMARY KEY(id)
);
```

## MEMBUAT MODEL USER

Selanjutnya buatlah model untuk memproses data login dengan membuat sebuah file baru pada direktori app/Models dengan nama UserModel.php yang diisi dengan kode berikut.

```php
<?php

namespace App\Models;

use CodeIgniter\Model;

class UserModel extends Model
{
    protected $table = 'user';
    protected $primaryKey = 'id';
    protected $useAutoIncrement = true;
    protected $allowedFields = ['username', 'useremail', 'userpassword'];
}
```

## MEMBUAT CONTROLLER USER

Buatlah controller baru dengan nama User.php pada direktori app/Controllers. Kemudian tambahkan method index() untuk menampilkan daftar user dan method login() untuk proses login dengan memasukan kode berikut.

```php
<?php

namespace App\Controllers;

use App\Models\UserModel;

class User extends BaseController
{
    public function index()
    {
        $title = 'Daftar User';
        $model = new UserModel();
        $users = $model->findAll();
        return view('user/index', compact('users', 'title'));
    }

    public function login()
    {
        helper(['form']);
        $email = $this->request->getPost('email');
        $password = $this->request->getPost('password');
        if (!$email)
        {
            return view('user/login');
        }

        $session = session();
        $model = new UserModel();
        $login = $model->where('useremail', $email)->first();
        if ($login)
        {
            $pass = $login['userpassword'];
            if (password_verify($password, $pass))
            {
                $login_data = [
                    'user_id' => $login['id'],
                    'user_name' => $login['username'],
                    'user_email' => $login['useremail'],
                    'logged_in' => TRUE,
                ];
                $session->set($login_data);
                return redirect('admin/artikel');
            }
            else
            {
                $session->setFlashdata("flash_msg", "Password salah.");
                return redirect()->to('/user/login');
            }
        }
        else
        {
            $session->setFlashdata("flash_msg", "email tidak terdaftar.");
            return redirect()->to('/user/login');
        }
    }
}
```

## MEMBUAT VIEW LOGIN

Buatlah direktori baru dengan nama user pada direktori app/views, kemudian buatlah sebuah file baru dengan nama login.php menggunakan kode berikut.

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
    <link rel="stylesheet" href="<?= base_url('/style.css');?>">
</head>
<body>
    <div id="login-wrapper">
        <h1>Sign In</h1>
        <?php if(session()->getFlashdata('flash_msg')):?>
            <div class="alert alert-danger"><?=
session()->getFlashdata('flash_msg') ?></div>
        <?php endif;?>
        <form action="" method="post">
            <div class="mb-3">
                <label for="InputForEmail" class="form-label">Email
address</label>
                <input type="email" name="email" class="form-control"
id="InputForEmail" value="<?= set_value('email') ?>">
            </div>
            <div class="mb-3">
                <label for="InputForPassword"
class="form-label">Password</label>
                    <input type="password" name="password"
class="form-control" id="InputForPassword">
                </div>
                <button type="submit" class="btn
btn-primary">Login</button>
            </form>
    </div>
</body>
</html>
```

## MEMBUAT DATABASE SEEDER

Database ini digunakan untuk membuat data dummy. Untuk mencobanya kita perlu memasukan data user dan password kedalam database. Untuk itu buatlah database seeder untuk tabel user. Bukalah CLI kemudia tulis perintah berikut.

```
php spark make:seeder UserSeeder
```
![menambahkan_gambar](img/DATABASE%20SEEDER%2013.png)

Selanjutnya, bukalah file UserSeeder.php yang berada pada direktori app/Database/Seeds/UserSeeder.php kemudian isi dengan kode berikut ini.

```php
<?php

namespace App\Database\Seeds;

use CodeIgniter\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run()
    {
        $model = model('UserModel');
        $model->insert([
            'username' => 'admin',
            'useremail' => 'admin@email.com',
            'userpassword' => password_hash('admin123', PASSWORD_DEFAULT),
        ]);
    }
}
```

Selanjutnya buka kembali CLI dan ketikan perintah berikut.

```
php spark db:seed UserSeeder
```

Setelah menambahkan perintah tersebut bukalah browser untuk melakukan pengecekan dengan menggunakan URL http://localhost:8080/user/login seperti inilah tampilan yang dihasilkan.

![menambahkan_gambar](img/COBA%20AKSES%2013.png)


## MENAMBAHKAN AUTH FILTER

Untuk halaman admin selanjutnya buatlah file baru dengan nama Auth.php pada direktori app/Filters. Kemudian tambahkan kode berikut ini.

```php
<?php namespace App\Filters;

use CodeIgniter\HTTP\RequestInterface;
use CodeIgniter\HTTP\ResponseInterface;
use CodeIgniter\Filters\FilterInterface;

class Auth implements FilterInterface
{
    public function before(RequestInterface $request, $arguments = null)
    {
        // jika user belum login
        if(! session()->get('logged_in')){
            // maka redirct ke halaman login
            return redirect()->to('/user/login');
        }
    }

    public function after(RequestInterface $request, ResponseInterface
$response, $arguments = null)
    {
        // Do something here
    }
}
```

Selanjutnya bukalah file app/config/Filters.php kemudian tambahkan kode berikut ke dalamnya.

```php
'auth' => App\Filters\Auth::class
```

Setelahnya buka file app/Config/Routes.php dan sesuaikanlah kodenya seperti gambar dibawah.

![menambahkan_gambar](img/SESUAIKAN%20KODE%2013.png)

## PERCOBAAN AKSES MENU ADMIN

Bukalah URL dengan alamat http://localhost:8080/user/login dan ketika alamat tersebut diakses maka akan muncul tampilan halaman login seperti berikut.

![menambahkan_gambar](img/COBA%20AKSES%2013.png)

Setelah mendapat tampilan diatas, loginlah menggunakan email adress dan password yang sudah disetting sebelumnya dan inilah tampilan yang dihasilkan ketika kalian sudah berhasil login.

![menambahkan_gambar](img/ADMIN%2012.png)

## FUNGSI LOGOUT

Tambahkanlah method logout pada Controller User seperti berikut:

```php
    public function logout()
    {
        session()->destroy();
        return redirect()->to('/user/login');
    }
```