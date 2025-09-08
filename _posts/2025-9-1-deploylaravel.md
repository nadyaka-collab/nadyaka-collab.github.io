
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Deploy Laravel dengan Docker - Nadyaka Shafwana Salwa Rabbani</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 20px;
            color: #333;
            background-color: #f5f5f5;
        }
        h1, h2, h3 {
            color: #2c3e50;
        }
        h1 {
            text-align: center;
            border-bottom: 2px solid #3498db;
            padding-bottom: 10px;
            margin-bottom: 20px;
        }
        h2 {
            margin-top: 30px;
            border-left: 5px solid #3498db;
            padding-left: 10px;
        }
        ul {
            margin: 10px 0;
            padding-left: 20px;
        }
        li {
            margin: 8px 0;
        }
        pre {
            background-color: #1e1e1e;
            color: #d4d4d4;
            padding: 15px;
            border-radius: 5px;
            overflow-x: auto;
            font-family: 'Courier New', Courier, monospace;
            border: 1px solid #444;
            margin: 15px 0;
        }
        code {
            background-color: #2d2d2d;
            color: #d4d4d4;
            padding: 2px 5px;
            border-radius: 3px;
            font-family: 'Courier New', Courier, monospace;
        }
        .image-container {
            text-align: center;
            margin: 20px 0;
        }
        img {
            max-width: 100%;
            height: auto;
            border: 1px solid #ddd;
            border-radius: 4px;
            padding: 5px;
        }
        .note {
            background-color: #fffde7;
            border-left: 4px solid #ffd600;
            padding: 10px;
            margin: 15px 0;
        }
        .highlight {
            background-color: #f0f0f0;
            padding: 5px 10px;
            border-radius: 3px;
            font-family: 'Courier New', Courier, monospace;
            display: inline-block;
            margin: 0 2px;
        }
    </style>
</head>
<body>

    <h1>Deploy Laravel dengan Docker</h1>
    <p><strong>Oleh: Nadyaka Shafwana Salwa Rabbani</strong></p>

    <h2>1. Latar Belakang</h2>
    <p>Selama pelaksanaan Praktik Kerja Lapangan (PKL), saya diberikan tugas untuk melakukan deployment aplikasi berbasis Laravel menggunakan Docker.</p>
    <p>Laravel sebagai framework PHP yang modern dan banyak digunakan, dipilih karena fleksibilitas dan fitur-fiturnya yang mendukung pengembangan aplikasi secara cepat dan terstruktur. Tugas ini tidak hanya menuntut pemahaman terhadap Laravel itu sendiri, tetapi juga kemampuan dalam mengelola infrastruktur aplikasi secara efisien melalui Docker dan Docker Compose.</p>
    <p>Pelaksanaan exam deploy ini menjadi pengalaman berharga dalam memahami prinsip DevOps dasar, serta meningkatkan kompetensi teknis di bidang backend development dan manajemen infrastruktur aplikasi. Selain itu, kegiatan ini juga mendukung tujuan PKL yaitu memperoleh pengalaman langsung.</p>

    <h2>2. Pengertian</h2>
    <p><strong>Docker</strong> adalah teknologi yang memungkinkan kita mengemas aplikasi beserta semua kebutuhannya (seperti PHP, database, web server) ke dalam satu wadah kecil yang disebut container. Dan Docker tersebut mempunyai banyak keunggulan.</p>
    <p><strong>Laravel</strong> adalah framework PHP (kerangka kerja untuk membuat website) yang membantu developer membuat aplikasi web dengan lebih cepat dan rapi. Bayangkan Laravel seperti "pabrik siap pakai" buat bikin website — sudah ada fitur login, database, form, dan lain-lain, jadi nggak perlu bikin dari nol.</p>

    <h2>3. Pengerjaan Exam</h2>

    <h3>A. Setup</h3>
    <ul>
        <li>Ubuntu 22.04 (VirtualBox)</li>
        <li>Dockerfile</li>
        <li>Docker-compose.yml</li>
        <li>Coding.conf</li>
        <li>build.sh</li>
    </ul>

    <h3>B. Membuat Folder Penempatan</h3>
    <p>Disini kita membuat folder yang dimana akan menjadi tempat untuk semua file yang akan dilakukan deployment.</p>
    <pre>$ mkdir warza
$ cd warza</pre>

    <h3>C. Menyalin Folder</h3>
    <p>Saya menyalin folder menggunakan WinSCP dikarenakan mudah untuk digunakan, disini saya menyalin dari folder exam menuju vm pod-managed1 menuju folder warza/</p>

    <h3>D. Membuat File Dockerfile</h3>
    <p>Kita membuat file bernama Dockerfile yang dimana file ini berisikan kumpulan instruksi atau perintah untuk membuat sebuah Docker image.</p>

    <ul>
        <li>Memulai dari image dasar php:7.4-fpm versi PHP yang stabil dan cocok untuk Laravel lama.</li>
        <li>Menginstal dependensi sistem (seperti git, curl, zip) dan ekstensi PHP (pdo_mysql, gd, mbstring) yang wajib ada agar Laravel bisa berjalan sempurna — termasuk upload gambar, koneksi database, dan manipulasi string.</li>
        <li>Menyalin Composer (manajer paket PHP) langsung dari image resminya efisien dan cepat.</li>
        <li>Menyalin seluruh kode aplikasi Laravel dari folder lokal ke dalam container, tepatnya di /var/www/html.</li>
        <li>Menyalin skrip setup.sh dan memberinya izin eksekusi.</li>
        <li>Menjadikan setup.sh sebagai perintah utama yang otomatis jalan saat container dinyalakan.</li>
    </ul>

    <pre>FROM php:7.4-fpm
RUN apt-get update && apt-get install -y \
    git curl zip unzip libonig-dev libzip-dev libpng-dev \
    && docker-php-ext-install pdo pdo_mysql mbstring zip gd

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html
COPY ./spp-laravel/spp /var/www/html
COPY setup.sh /setup.sh
RUN chmod +x /setup.sh

ENTRYPOINT ["/setup.sh"]</pre>

    <h3>E. Membuat File docker-compose.yml</h3>
    <p>File ini mengatur beberapa layanan dalam satu komposisi, agar saat kita menjalankan tidak perlu manual menggunakan docker run.</p>

    <ul>
        <li>Layanan app dibangun dari Dockerfile di direktori saat ini , diberi nama spp-nadyaka, dan membuka port 8000 agar bisa diakses dari browser.</li>
        <li>Layanan db menggunakan image MySQL 5.7, membuat database bernama spp secara otomatis, dan mengimpor struktur + data dari file spp.sql yang dimount ke direktori khusus MySQL (/docker-entrypoint-initdb.d/) agar database langsung terisi saat pertama kali container dijalankan.</li>
        <li>depends_on: - db memastikan container aplikasi hanya jalan setelah MySQL siap, mencegah error koneksi database.</li>
        <li>restart: unless-stopped membuat container otomatis bangkit kembali jika crash, sangat berguna untuk lingkungan produksi atau ujian.</li>
    </ul>

    <pre>services:
  app:
    build: .
    container_name: spp-nadyaka
    ports:
      - "8000:8000"
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: mysql:5.7
    container_name: mysql-perpus
    environment:
      MYSQL_DATABASE: spp
      MYSQL_ROOT_PASSWORD: rootpass
    volumes:
      - ./spp-laravel/database/spp.sql:/docker-entrypoint-initdb.d/spp.sql
    ports:
      - "3306:3306"
    restart: unless-stopped</pre>

    <h3>F. Konfigurasi Nginx (coding.conf)</h3>
    <p>File ini mengatur bagaimana Nginx melayani aplikasi Laravel. Kalau ada yang buka http://localhost:8000/login, Nginx cek dulu file-nya ada atau enggak. Kalau enggak, langsung lempar ke index.php (ini fitur Laravel untuk routing).</p>

    <ul>
        <li>Mendengarkan permintaan di port 8000.</li>
        <li>Menjadikan folder /var/www/html/public sebagai root — sesuai struktur Laravel.</li>
        <li>location / mengatur routing: jika file/folder tidak ditemukan, lempar ke index.php — inilah yang membuat URL seperti /login atau /dashboard tetap bekerja meskipun tidak ada file fisik bernama itu.</li>
        <li>location ~ \.php$ mengarahkan semua request PHP ke service bernama laravel di port 9000 (PHP-FPM) — meskipun dalam kasus ini, karena pakai artisan serve, bagian ini belum aktif. Tapi konfigurasi ini penting jika suatu saat ingin migrasi ke Nginx + PHP-FPM.</li>
    </ul>

    <pre>server {
    listen 8000;
    index index.php index.html;
    root /var/www/html/public;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass laravel:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}</pre>

    <h3>G. Skrip Otomatisasi (build.sh)</h3>
    <p>Ini adalah skrip ajaib yang otomatis menjalankan perintah Laravel saat container hidup.</p>

    <ul>
        <li>Masuk ke folder aplikasi Laravel — tempat semua kode program disimpan.</li>
        <li>Pasang semua library yang dibutuhkan. Pakai perintah composer install --no-dev --optimize-autoloader: Tidak pasang tools buat developer (biar lebih ringan) dan optimalkan autoloader agar aplikasi jalan lebih cepat.</li>
        <li>Siapkan database. Pakai perintah php artisan migrate --force: Buat tabel-tabel di database sesuai struktur Laravel. --force dipakai karena di Docker dianggap lingkungan “produksi”, jadi perlu paksa biar migrasi jalan.</li>
        <li>Nyalakan server Laravel. Pakai perintah php artisan serve --host=0.0.0.0 --port=8000: Server jalan di port 8000 dan bisa diakses dari luar container (misalnya lewat browser di laptop kamu).</li>
    </ul>

    <pre>#!/bin/bash
cd /var/www/html
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan serve --host=0.0.0.0 --port=8000</pre>

    <h3>H. Jalankan Compose</h3>
    <p>Disini saya menjalankan docker compose agar file yang kita buat tadi berjalan dan otomatis membuat container.</p>
    <pre>$ docker-compose up --build</pre>
    <p>Tunggu hingga selesai, lalu...</p>

    <h3>I. Mengatur Permission pada Container (Opsional)</h3>
    <p class="note">Kenapa disini saya bilang opsional, karena saat saya docker compose up, sudah langsung bisa diakses. Dan apabila ada error saat mengakses web, biasanya ada kendala terkait permission di container, jadi bisa mengikuti langkah-langkah di bawah ini.</p>

    <p>Jika sudah masuk ke dalam container laravel_spp:</p>
    <pre>$ docker exec -it laravel_spp /bin/bash</pre>

    <p>Langkah selanjutnya mengubah permission dengan chmod & chown:</p>
    <pre>$ chown -R www-data:www-data storage bootstrap/cache
$ chmod -R 755 /var/www/html
$ exit</pre>

    <p><strong>Penjelasan:</strong></p>
    <ul>
        <li>chown -R www-data:www-data storage bootstrap/cache: Perintah ini mengubah pemilik (owner) dari folder storage dan bootstrap/cache ke pengguna dan grup www-data.</li>
        <li>chmod -R 755 /var/www/html: Perintah ini memberikan izin (permission) pada semua file dan folder di /var/www/html.</li>
    </ul>

    <h3>J. Pengujian Web</h3>
    <p>Disini akan muncul halaman login yang dimana saya memasukan email dan password dengan:</p>
    <ul>
        <li><strong>Email:</strong> admin@example.com</li>
        <li><strong>Password:</strong> password</li>
    </ul>

    <p>Lalu jika setelah login akan muncul halaman dashboard yang merupakan halaman utama website SPP PAUD.</p>

![_config.yml]({{ site.baseurl }}/images/login.png)
![_config.yml]({{ site.baseurl }}/images/dasbord.png)

 <h2>4. Kesimpulan</h2>
    <p>Setelah menjalankan perintah docker-compose up, maka:</p>
    <ul>
        <li>Container MySQL berjalan dengan database spp yang sudah diimpor dari spp.sql.</li>
        <li>Container Laravel melakukan migrasi dan seeding data.</li>
        <li>Nginx menerima request dan mengarahkan ke PHP-FPM.</li>
    </ul>

    <p><strong>--- Dokumen Selesai ---</strong></p>

</body>
</html>
