# CodeIgniter 4 Application Starter

# What is CodeIgniter?

CodeIgniter is a PHP full-stack web framework that is light, fast, flexible and secure.
More information can be found at the [official site](https://codeigniter.com).

This repository holds a composer-installable app starter.
It has been built from the
[development repository](https://github.com/codeigniter4/CodeIgniter4).

More information about the plans for version 4 can be found in [CodeIgniter 4](https://forum.codeigniter.com/forumdisplay.php?fid=28) on the forums.

You can read the [user guide](https://codeigniter.com/user_guide/)
corresponding to the latest version of the framework.

# Instalasi
Untuk instalasi CI4 diperlukan composer terinstall pada device.
pada cmd pilih folder yang diingikan dengan 
```shell
cd D:\direktori tujuan
```
lalu pada folder yang diinginkan mulai instalasi dengan cara mengetikan ini pada terminal
```shell
  composer create-project codeigniter4/appstarter nama-project
```
# Running App
untuk menjalankan app CI4 dapat dilakukan dari visual studio code, dengan cara
buka folder ci yang terinstall pada vscode lalu klik ctrl+` untuk membuka terminal lalu ketikan

```shell
  php spark serve
```
lalu app dapat dibuka melalui web dengan alamat (http://localhost:8080/)
# Membuat Page
sebelum coding sebaiknya pada file env ubah ENVIRONMENT ke develpment untuk mempermudah debugging dan testing dan juga ubah  DATABASE supaya tersambung ke database masing masing
lalu ubah nama menjadi .env
```shell
 CI_ENVIRONMENT = development
```
```shell
 database.default.hostname = localhost
 database.default.database = ci4
 database.default.username = root
 database.default.password = 
 database.default.DBDriver = MySQLi
```
# Static Page
### Controller
pada app/controller buat file Pages.php
dan buat controller dengan function view sesuaing dengan kode dibawah
```shell
<?php

namespace App\Controllers;
use CodeIgniter\Exceptions\PageNotFoundException; //untuk exception

class Page extends BaseController
{
    public function view($page = 'home')
    {
            if (! is_file(APPPATH . 'Views/pages/' . $page . '.php')) {// kondisi untuk menangkap error not found
            throw new PageNotFoundException($page);
        }

        $data['title'] = ucfirst($page);

        return view('templates/header', $data)
            . view('pages/' . $page)
            . view('templates/footer');
    }
}

```

### Views

template header and footer
untuk template setiap page yang akan dibuat buatlah kedua file tersebut lalu isi sesuai dengan header atau footer
 app/Views/templates/header.php
 ```shell
<!doctype html>
<html>
<head>
    <title>CodeIgniter Tutorial</title>
</head>
<body>

    <h1><?= esc($title) ?></h1>
 ```
 app/Views/templates/footer.php

 ```shell

    <em>&copy; 2022</em>
</body>
</html>
 ```
Sekarang pada app/Views/pages //buat foldernya jika belum ada
buat about.php
lalu buat html untuk menyatakan bahwa page itu merupakan page about
```shell
    <h1>About</h1>

```
### Route
pada app/config/routes.php
buat route baru yang akan memanggil fungsi pada controller tadi
dan jangan lupa untuk use App\Controllers\Pages
```shell
use App\Controllers\Pages;

$routes->get('pages', [Pages::class, 'index']);
$routes->get('(:segment)', [Pages::class, 'view']);
```
### Test
coba akses page tersebut dengan cara
http://localhost:8080/home

![home](https://github.com/Einkelberg/CI4-PBF-Mas-Dzuky/assets/127199885/38f63102-5d69-4ccd-9f3c-1a09c50f02ce)

http://localhost:8080/about

![about](https://github.com/Einkelberg/CI4-PBF-Mas-Dzuky/assets/127199885/2f21b04e-27a4-45a4-af35-9fa70f0c44b8)



#Page News item dengan Database
###Buat Database 
Buat database ci4tutorial lalu buat table dengan kode sql berikut
```
CREATE TABLE news (
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    title VARCHAR(128) NOT NULL,
    slug VARCHAR(128) NOT NULL,
    body TEXT NOT NULL,
    PRIMARY KEY (id),
    UNIQUE slug (slug) //url identifier
);
```
###Model
buat NewsModel.php pada app/Models
dan buat
```shell
<?php

namespace App\Models;

use CodeIgniter\Model;

class NewsModel extends Model
{
    protected $table = 'news';

    public function getNews($slug = false)
    {
        if ($slug === false) {
            return $this->findAll();
        }

        return $this->where(['slug' => $slug])->first();
    }
}
```
###Controller
###Routing
###view
