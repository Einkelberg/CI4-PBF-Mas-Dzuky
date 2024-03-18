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

# 1. Instalasi
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
# 2. Static Page
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



# 3. Page News item dengan Database
### Buat Database 
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
### Model
buat NewsModel.php pada app/Models
dan buat
```shell
<?php

namespace App\Models;

use CodeIgniter\Model;

class NewsModel extends Model
{
    protected $table = 'news'; // untuk mengambil data dari database dengan nama table 'news'

    public function getNews($slug = false) // fungsi mengambil data dari table dengan default $slug=false
    {
        if ($slug === false) {  // jika tidak ada slug, makaambil semua data
            return $this->findAll();
        }

        return $this->where(['slug' => $slug])->first(); // jika $slug = true maka ambil data sesuai $slug tersebut
    }
}
```
### Controller
Buat Controller news.php pada app/controller lalu buat fungsi untuk mengambil data dari model
```shell
<?php

namespace App\Controllers;

use App\Models\NewsModel;

class News extends BaseController
{
    public function index()// ambil semua data karena getNews() tanpa parameter(default)
    {
        $model = model(NewsModel::class);

        $data['news'] = $model->getNews();
    }

    public function show($slug = null)// ambil data sesuai slug karena getNews($slug) punya parameter
    {
        $model = model(NewsModel::class);

        $data['news'] = $model->getNews($slug);
    }
}
```
lalu modify fungsi index untuk mengembalikan view
```shell
public function index()
    {
        $model = model(NewsModel::class);

        $data = [
            'news'  => $model->getNews(),//data dari database
            'title' => 'News archive',//
        ];

        return view('templates/header', $data)
            . view('news/index')
            . view('templates/footer');
    }
```
dan modify juga fungsi show untuk mengembalikan view 
```shell
public function show($slug = null)
    {
        $model = model(NewsModel::class);

        $data['news'] = $model->getNews($slug);

        if (empty($data['news'])) {
            throw new PageNotFoundException('Cannot find the news item: ' . $slug);
        }

        $data['title'] = $data['news']['title'];

        return view('templates/header', $data)
            . view('news/view')
            . view('templates/footer');
    }
```
### Routing
Tambahkan pada routes.php routing untuk coontroller news
```shell
use App\Controllers\News; 
$routes->get('news', [News::class, 'index']);           
$routes->get('news/(:segment)', [News::class, 'show']); 
```
### view
Buat folder dan file pada app/Views/News/index.php dan buat tampilan untuk title perulangan untuk semua news dan juga kondisi jika tidak ada news
```shell
<h2><?= esc($title) ?></h2>

<?php if (! empty($news) && is_array($news)): ?>

    <?php foreach ($news as $news_item): ?>

        <h3><?= esc($news_item['title']) ?></h3>

        <div class="main">
            <?= esc($news_item['body']) ?>
        </div>
        <p><a href="/news/<?= esc($news_item['slug'], 'url') ?>">View article</a></p>

    <?php endforeach ?>

<?php else: ?>

    <h3>No News</h3>

    <p>Unable to find any news for you.</p>

<?php endif ?>
```
dan buat juga app/Views/News/view.php untuk menampilkan data news secara spesifik sesuai slug
```shell
<h2><?= esc($news['title']) ?></h2>
<p><?= esc($news['body']) ?></p>
```
pada localhost:8080/news akan ada tampilan bahwa belum ada news

![empty news](https://github.com/Einkelberg/CI4-PBF-Mas-Dzuky/assets/127199885/0a10c2d1-e8f5-43c8-b4db-8e0bb8aad2ab)


# 4. Create News Items

### Enabling csrf
buka app/Config/Filters.php dan ubah method menjadi post => csrf
```shell
    public array $methods = [
        'post' => ['csrf'],
    ];
```
csrf digunakan untuk alasan security yang akan mengirimkan token setiap kali form diisi memastikan bahwa form tersebut merupakan authorized form

### Route
pada routes.php tambahkan route baru yang akan digunakan untuk mengakses form dan juga mengirim data baru dengan method post
```shell
$routes->get('news/new', [News::class, 'new']);
$routes->post('news', [News::class, 'create']);
```

### Buat Form
Buat file create.php pada app/Views/news dan buat form untuk menginput title dan body dari news
```shell
<h2><?= esc($title) ?></h2>

<?= session()->getFlashdata('error') ?>/
<?= validation_list_errors() ?> // untuk melihat error yang ditimbulkan dari form

<form action="/news" method="post">
    <?= csrf_field() ?> // form akan mengirimkan csrf token

    <label for="title">Title</label>
    <input type="input" name="title" value="<?= set_value('title') ?>">
    <br>

    <label for="body">Text</label>
    <textarea name="body" cols="45" rows="4"><?= set_value('body') ?></textarea>
    <br>

    <input type="submit" name="submit" value="Create news item">
</form>
```
### Controller
buat fungsi baru pada controller news untuk menuju form
```shell
public function new()
    {
        helper('form');

        return view('templates/header', ['title' => 'Create a news item'])
            . view('news/create')
            . view('templates/footer');
    }
```
dan juga tambahkan fungsi untuk membuat news
```shell
public function create()
    {
        helper('form');

        $data = $this->request->getPost(['title', 'body']);

        
        if (! $this->validateData($data, [ //cek data berdasar yang dikirimkan form
            'title' => 'required|max_length[255]|min_length[3]',
            'body'  => 'required|max_length[5000]|min_length[10]',
        ])) {
            // kondisi jika gagal
            return $this->new();
        }

        $post = $this->validator->getValidated();

        $model = model(NewsModel::class);

        $model->save([ //menyimpan data dari form
            'title' => $post['title'],
            'slug'  => url_title($post['title'], '-', true),
            'body'  => $post['body'],
        ]);

        return view('templates/header', ['title' => 'Create a news item'])
            . view('news/success')
            . view('templates/footer');
    }
```
### Model
Pada NewsModel tambahkan allowed field supaya dapat ditambahkan valuenya
```shell
    protected $allowedFields = ['title', 'slug', 'body'];
```
### Views
buat success.php pada folder app/Views/news dan gunakan page itu untuk menampilkan pesan berhasil seperti
```shell
<p>News item created successfully.</p>
```
![success](https://github.com/Einkelberg/CI4-PBF-Mas-Dzuky/assets/127199885/da3ad7a9-e10a-4d85-b207-51abd193c5f3)

# 5. Application Struncture
Setelah menginstal CI4 akan ada beberapa folder yang dapat ditemukan didalam folder seperti app, public, writable, test dan vendor atau system

### App 
Directory app adalah dimana kebannyakan kode dibuat dan directory app juga mempunnyai structure yang mudah digunakan.
```shell
app/
    Config/         Menimpan konfigurasi
    Controllers/    Pengendali alur kerja
    Database/       Menympan Database Migrasi dan seed
    Filters/        Menyimpan class yang difilter supaya bisa dijalankan setelah atau sebelum Controller
    Helpers/        Menyipan fungsi bantuan yang dapat berdiri sendiri
    Language/       Berisi beberapa bantuan untuk beberapa bahasa tertentu
    Libraries/      Class penting yang tidak dapat dikelompokan ke class lain
    Models/         Model yang mempresentasikan entity pada database
    ThirdParty/     ThirdParty libraries yang bisa digunakan
    Views/          HTML file yang nantinya akan ditampilkan ke client
```
semua file tersebut dapat dikeluarkan dari app dengan cara mengganti konfigurasi dari app/Config/Constants.php

### System
Directory yang merupakan bagian dari framework itu sendiri berisi class yang tidak boleh diubah tetapi bisa di extend untuk digunakan.
### Public
Directory ini tersambung dengan browser yang mencegah akses browser langsung ke code. Directory ini berisi .htaccess, index.php dan gambar/css/js/dll
### Writable
Directory ini berisi logs, file cache , dan apapun yang user upload dari website. Menjadikan direktory lain lebih bersih dan juga mencegah penulisan pada code dari user
### Test
Directory ini digunakan untuk menyimpan file test dan juga menyimpan segala jenis mock class dan utilities. Directory ini tidak perlu diupload ke server.
### Mengubah Dierctory
Jika ingin mengubah directory maka harus dilakuakan dari dari app/Config/Paths.php


# 6. MVC
MVC merupakan sebuah pola mengorganisir file  dimana data, presentasi dan alur kerja merupakan bagian terpisah.

### Views
Views biasanya terisi dari HTML dan beberapa PHP  dimana Views ini berfungsi menapilkan komponen yang didapat dari Controller.
Views biasanya tersimpan di app/Views dan untuk pengorganisiannya diharapkan untuk setiap views yang menngunakan Controller yag sama maka dijadikan satu folder
Contoh :
app/Views/user/profile.php.
app/Views/admin/profile.php.

### Models
Model berfungsi untuk menjaga sebuah data pada aplikasi seperti pengguna, admin, transaksi.
Model berguna untuk menentukan aturan mengambil dan menginput ke database dan juga handling penympanan dan mengambilan data dari database.

### Controllers
Controller menerima input dan menentukan apa yang akan dilakukan, seperti mengirimkan input user dari views lalu dikirimkan ke model untuk disimpan.
Controller juga mengurus http request, autentikasi, security dll.


# 7.
