<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Deploy Laravel dengan Docker - Nadyaka Shafwana</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background-color: #f8f9fa;
      color: #333;
      line-height: 1.7;
      padding: 20px;
      max-width: 1000px;
      margin: auto;
    }
    h1, h2, h3 {
      color: #2c3e50;
    }
    h1 {
      text-align: center;
      border-bottom: 3px solid #3498db;
      padding-bottom: 10px;
    }
    h2 {
      border-bottom: 2px solid #bdc3c7;
      padding-bottom: 5px;
      margin-top: 30px;
    }
    pre {
      background-color: #2d3436;
      color: #ecf0f1;
      padding: 15px;
      border-radius: 8px;
      overflow-x: auto;
      font-size: 14px;
    }
    code {
      background-color: #ecf0f1;
      color: #e74c3c;
      padding: 2px 6px;
      border-radius: 4px;
      font-family: Consolas, monospace;
    }
    ul, ol {
      margin: 15px 0;
    }
    .footer {
      text-align: center;
      margin-top: 50px;
      color: #7f8c8d;
      font-size: 14px;
    }
    .comment {
      background-color: #fffacd;
      border-left: 4px solid #f1c40f;
      padding: 10px;
      margin: 15px 0;
      font-style: italic;
    }
  </style>
</head>
<body>

  <h1>Deploy Laravel dengan Docker</h1>
  <p><strong>Oleh: Nadyaka Shafwana Salwa Rabbani</strong></p>

  <h2>1. Latar Belakang</h2>
  <p>
    Selama pelaksanaan Praktik Kerja Lapangan (PKL), saya diberikan tugas untuk melakukan deployment aplikasi berbasis Laravel menggunakan Docker. Laravel sebagai framework PHP yang modern dan banyak digunakan, dipilih karena fleksibilitas dan fitur-fiturnya yang mendukung pengembangan aplikasi secara cepat dan terstruktur.
  </p>
  <p>
    Tugas ini tidak hanya menuntut pemahaman terhadap Laravel itu sendiri, tetapi juga kemampuan dalam mengelola infrastruktur aplikasi secara efisien melalui Docker dan Docker Compose. Pelaksanaan exam deploy ini menjadi pengalaman berharga dalam memahami prinsip DevOps dasar, serta meningkatkan kompetensi teknis di bidang backend development dan manajemen infrastruktur aplikasi.
  </p>

  <h2>2. Pengertian</h2>
  <p><strong>Docker</strong> adalah teknologi yang memungkinkan kita mengemas aplikasi beserta semua kebutuhannya (seperti PHP, database, web server) ke dalam satu wadah kecil yang disebut <em>container</em>.</p>
  <p><strong>Laravel</strong> adalah framework PHP yang membantu developer membuat aplikasi web dengan lebih cepat dan rapi. Bayangkan Laravel seperti "pabrik siap pakai" buat bikin website — sudah ada fitur login, database, form, dan lain-lain, jadi nggak perlu bikin dari nol.</p>

  <h2>3. Pengerjaan Exam</h2>

  <h3>A. Setup</h3>
  <ul>
    <li>Ubuntu 22.04 (VirtualBox)</li>
    <li>Dockerfile</li>
    <li>docker-compose.yml</li>
    <li>coding.conf</li>
    <li>build.sh</li>
  </ul>

  <h3>B. Membuat Folder Penempatan</h3>
  <p>Buat folder untuk menampung semua file deployment.</p>
  <pre>mkdir warza
cd warza</pre>

  <h3>C. Menyalin Folder</h3>
  <p>Saya menyalin folder menggunakan WinSCP dari folder exam ke VM <code>pod-managed1</code> menuju folder <code>warza/</code>.</p>

  <h3>D. Membuat Dockerfile</h3>
  <pre>FROM php:7.2-fpm

# Ganti mirror Debian untuk menghindari error 404
RUN sed -i 's/deb.debian.org/archive.debian.org/g' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/archive.debian.org/g' /etc/apt/sources.list && \
    echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid-until

# Install dependensi
RUN apt-get update && apt-get install -y \
    git \
    curl \
    zip \
    unzip \
    libonig-dev \
    libzip-dev \
    libpng-dev \
    && docker-php-ext-install pdo pdo_mysql mbstring zip gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

# Salin kode Laravel dan skrip build
COPY ./spp-laravel/spp /var/www/html
COPY build.sh /build.sh

RUN chmod +x /build.sh

ENTRYPOINT ["/build.sh"]</pre>

  <p><strong>Penjelasan Singkat:</strong></p>
  <ul>
    <li><code>FROM php:7.2-fpm</code>: Gunakan image PHP 7.2 dengan FPM.</li>
    <li><code>RUN</code>: Instal paket dan ekstensi PHP.</li>
    <li><code>COPY --from=composer</code>: Salin Composer dari image resmi.</li>
    <li><code>WORKDIR</code>: Set direktori kerja ke <code>/var/www/html</code>.</li>
    <li><code>ENTRYPOINT ["/build.sh"]</code>: Jalankan skrip saat container dimulai.</li>
  </ul>

  <h3>E. Membuat docker-compose.yml</h3>
  <pre>version: '3.8'

services:
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

  <p><strong>Penjelasan:</strong></p>
  <ul>
    <li><code>build: .</code>: Build image dari Dockerfile di direktori saat ini.</li>
    <li><code>container_name</code>: Beri nama container.</li>
    <li><code>ports</code>: Mapping port host ke container.</li>
    <li><code>depends_on</code>: Service <code>app</code> bergantung pada <code>db</code>.</li>
    <li><code>restart: unless-stopped</code>: Auto-restart container.</li>
  </ul>

  <h3>F. Konfigurasi Nginx (coding.conf)</h3>
  <pre>server {
    listen 8000;
    index index.php index.html;
    server_name localhost;
    root /var/www/html/public;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~ /\.ht {
        deny all;
    }
}</pre>

  <p><strong>Penjelasan:</strong></p>
  <ul>
    <li><code>listen 8000</code>: Nginx mendengarkan port 8000.</li>
    <li><code>root /var/www/html/public</code>: Direktori public Laravel.</li>
    <li><code>try_files</code>: Routing otomatis ke <code>index.php</code>.</li>
    <li><code>fastcgi_pass app:9000</code>: Kirim request PHP ke service <code>app</code>.</li>
  </ul>

  <div class="comment">
    Catatan: Pastikan service Nginx ditambahkan di <code>docker-compose.yml</code> jika belum ada.
  </div>

  <h3>G. Skrip Otomatisasi (build.sh)</h3>
  <pre>#!/bin/bash

cd /var/www/html

php artisan key:generate
php artisan migrate
php artisan db:seed
php-fpm</pre>

  <p><strong>Penjelasan:</strong></p>
  <ul>
    <li><code>key:generate</code>: Generate aplikasi key Laravel.</li>
    <li><code>migrate</code>: Jalankan migrasi database.</li>
    <li><code>db:seed</code>: Isi data awal.</li>
    <li><code>php-fpm</code>: Jalankan PHP-FPM agar container tetap hidup.</li>
  </ul>

  <h3>H. Menjalankan Docker Compose</h3>
  <pre>docker-compose up --build -d</pre>
  <p>Tunggu hingga proses selesai.</p>

  <p>Masuk ke container:</p>
  <pre>docker exec -it spp-nadyaka /bin/bash</pre>

  <p>Atur permission:</p>
  <pre>chown -R www-data:www-data storage bootstrap/cache
chmod -R 755 /var/www/html
exit</pre>

  <h3>I. Pengujian Web</h3>
  <p>Buka di browser:</p>
  <pre>http://localhost:8000/login</pre>
  <p><strong>Login:</strong></p>
  <ul>
    <li><strong>Email:</strong> admin@example.com</li>
    <li><strong>Password:</strong> password</li>
  </ul>
  <p>Setelah login, akan muncul halaman dashboard SPP PAUD.</p>

  <h2>4. Kesimpulan</h2>
  <p>Setelah menjalankan <code>docker-compose up</code>:</p>
  <ul>
    <li>Container MySQL berjalan dengan database <code>spp</code> yang sudah diimpor dari <code>spp.sql</code>.</li>
    <li>Container Laravel melakukan migrasi dan seeding data.</li>
    <li>Nginx menerima request dan mengarahkan ke PHP-FPM.</li>
  </ul>
  <p>Proses ini menunjukkan integrasi sempurna antara Laravel, Docker, dan MySQL untuk deployment aplikasi web modern.</p>

  <div class="footer">
    &copy; 2025 Nadyaka Shafwana Salwa Rabbani | Dokumentasi PKL - Deploy Laravel dengan Docker
  </div>

</body>
</html>
