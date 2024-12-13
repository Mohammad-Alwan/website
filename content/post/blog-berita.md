---
date: '2024-12-10T16:17:48+07:00'
draft: false
title: Deploy Aplikasi Blog Berita (Laravel) dengan GitLab CI/CD
ShowToc: true
---
Halo, disini saya mau sharing langkah-langkah deploy aplikasi blog berita berbasis web dengan menggunakan CI/CD GitLab dengan executor GitLab Runner yaitu Docker. Aplikasi ini dibuat dengan framework laravel dan untuk database-nya menggunakan MySQL.

Aplikasi Sistem Informasi Blog Berita Berbasis Web ini dirancang dan dibangun untuk memudahkan operator dalam menyebarkan berita-berita terbaru secara cepat dan up-to-date.

Dari hasil testing, source code dari aplikasi ini berjalan dengan mulus tanpa kendala. Tampilan dari aplikasi modern dan layak untuk digunakan. Menurut saya aplikasi ini memiliki fitur yang sudah cukup lengkap untuk menunjang aplikasi sistem informasi blog dan berita berbasis website.

### Screenshot Aplikasi
![blog berita image](blog-berita.png)


### Fitur Aplikasi
* Panel Admin. Berikut hal-hal yang dapat dikelola oleh admin:
  1. Data Post
  2. Data Category
  3. Data Tags
  4. Data Users
* Dashboard
* LogOut
* dan lain-lain.

### System Requirement
* PHP 7.4
* MySQL database

### Required Software
- Git. ( [Install Git](https://git-scm.com/downloads) )
- Docker. ( [Install Docker](https://docs.docker.com/engine/install/) ) 
- GitLab-Runner. ( [Install GitLab-Runner](https://docs.gitlab.com/runner/install/) )

### Topologi
![Topologi](test.png)

> - Docker in Docker adalah konsep menjalankan Docker dalam Docker container. Atau lebih jelasnya adalah menjalankan Docker daemon di dalam Docker container. 
> -  Bridge adalah Network driver yang memungkinkan container untuk terhubung satu sama lain dan dengan host. 

### Arsitektur GitLab CI/CD executor docker yang digunakan
![arsitektur-gitlab](arsitektur-gitlab.png)


Langsung saja untuk tutorialnya, Berikut langkah-langkahnya:


1. Pertama teman-teman harus memiliki akun GitLab. Karena GitLab inilah yang akan digunakan untuk CI/CD nanti. Teman-teman sign-up jika belum memiliki akun atau bisa langsung sign-in jika sudah memiliki akun. Setelah itu, buat proyek kosong pada GitLab.

Pada menu GitLab klik **New Project** <span>&#8594;</span>  **Create blank Project** <span>&#8594;</span>  isi <span>&#8594;</span>  **Create Project**
![create-project](create-project.png)

2. Selain GitLab, teman-teman juga harus memiliki source code dari aplikasi tersebut untuk dideploy. Untuk itu teman-teman bisa donwload source code pada link [berikut](https://drive.google.com/file/d/1mK3_WhvYdBzUaPFvkBVimYUiS4ltpB2y/view) atau bisa langsung clone saja pada environment teman-teman.
   ```bash
   git clone https://github.com/Mohammad-Alwan/blogberita.git
   ```

3. Setelah itu, kita bisa langung mulai saja langkah-langkahnya. Setelah step tadi, teman-teman bisa pindah ke folder atau direktori hasil dari clone dengan menggunakan perintah berikut:
   ```bash
   cd blogberita 
   ````

4. Teman-teman akan melihat satu file, dan file tersebut adalah souce code dari aplikasi yang akan dideploy. File yang akan teman-teman liat berformat 7z. Untuk ekstrak bisa ikuti langkah berikut:
   ```bash
   7z x aplikasi-blog-berita-berbasis-web-laravel.7z
   ```
   Jika sudah maka akan terlihat beberapa file dan direktori yang tersedia setelah proses ekstrak tadi. Kita akan berfokus pada dua direktori yaitu direktori blog dan direktori database. Langsung saja teman-teman pindah saja file "blog.sql" dalam direktori database ke direktori blog. Karena file tersebut adalah dump.sql yang akan digunakan pada database. Lakukan step ini karena step ini akan berkaitan dengan step-step berikutnya

5. Untuk pindah file lakukan perintah seperti berikut:
   ```bash
   mv database/blog.sql blog/
   ```

 6. Selanjutnya pindah ke direktori blog dengan menggunakan perintah **cd blog** dan lakukan command **git init** pada terminal untuk menginisialisasi direktori blog sebagai repository lokal.
    ```bash
    git init
    ```

7. Hubungkan repository lokal dengan repository (project) pada GitLab yang pada step awal sudah dibuat. Tadi pada saat awal kita clone untuk mendapatkan source code, maka remote repository kita akan ke mengarah ke github. Karena kita akan menggunakan GitLab, ubah remote repository ke project GitLab. Untuk itu copy/salin url repository di GitLab dengan cara masuk ke **project yang sudah dibuat** <span>&#8594;</span> **Clone** <span>&#8594;</span> **Copy** url http.
![remote](git-remote.png)  

   ```bash
   git remote set-url origin <URL>
   ```
> URL yang didapatkan teman-teman mungkin berbeda dengan saya, karena ini GitLab  hasil hosting sebuah perusahaan. Tapi pada intinya teman-teman copy saja url pada GitLab teman-teman.

8. Buat dua docker file untuk webserver dan database.
- Buat dockerfile dengan nama file _Dockerfile_ menggunakan perintah berikut:

   ```bash
   vim Dockerfile
   ```

   ```dockerfile
   FROM php:7.4-apache
   WORKDIR /var/www/html
   COPY . /var/www/html
   
   RUN apt-get update && apt-get install -y \
       libpng-dev \
       zlib1g-dev \
       libxml2-dev \
       libzip-dev \
       libonig-dev \
       && docker-php-ext-configure gd \
       && docker-php-ext-install -j$(nproc) gd \
       && docker-php-ext-install pdo_mysql \
       && docker-php-ext-install mysqli \
       && docker-php-ext-install zip \
       && docker-php-source delete
   
   ENV COMPOSER_ALLOW_SUPERUSER=1
   RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
   RUN chown -R www-data:www-data /var/www/html \
       && a2enmod rewrite
   
   RUN composer update
   COPY .env.example .env
   RUN sed -i 's/DB_HOST=.*/DB_HOST=192.168.0.13/' /var/www/html/.env
   RUN sed -i 's/DB_DATABASE.*/DB_DATABASE=blog/' /var/www/html/.env
   RUN sed -i 's/DB_USERNAME.*/DB_USERNAME=alwan/' /var/www/html/.env
   RUN sed -i 's/DB_PASSWORD.*/DB_PASSWORD=alwan/' /var/www/html/.env
   
   RUN php artisan key:generate
   CMD php artisan key:generate && php artisan migrate && php artisan db:seed && php artisan serve --host=0.0.0.0 --port=80


>  Pada Dockerfile diatas di bagian mengganti isi file _.env_ bisa dikonfigurasikan sesuai environment dan keinginan teman-tema Lokasi atau alamat dari host database yang akan digunakan image aplikasi tersebut.
>
> DB_HOST - Lokasi atau alamat dari host database yang akan digunakan image aplikasi tersebut.
>
> DB_DATABASE - Nama database yang akan digunakan oleh aplikasi tersebut.
>
> DB_USERNAME - Nama pengguna atau username yang akan digunakan oleh aplikasi untuk mengakses database.
>
> DB_PASSWORD - Kata sandi atau password dari pengguna yang akan digunakan aplikasi untuk mengautentikasi koneksi ke database.
   
- Buat docker file dengan nama file Dockerfilemysql menggunakan perintah berikut:
  ```bash
  vim Dockerfilemysql
  ```
  ```dockerfile
  FROM mysql:5.7
  ADD blog.sql /docker-entrypoint-initdb.d
  ```

9. Buat file docker compose dengan isi perintah menjalankan container dengan port 8000 di lokal dan port 80 di dalam container. Container yang akan dijalankan dengan docker compose ini nantinya akan menggunakan network yang sama dengan database (pembuatan network ada pada step selanjutnya). Docker compose ini akan menggunakan image yang dibuild/dihasilkan dari docker file dengan nama _Dockerfile_. Buat docker compose dengan perintah berikut : 
   ```bash
   vim docker-compose.yaml
   ```
   ```yaml
   services:
     my_app:
       image: index.docker.io/$DOCKER_USER/blogberita-mohammadalwanadb:1.0
       container_name: blogberita
       ports:
         - "8000:80"
       networks:
         - blogberita
   
   
   networks:
     blogberita:
       external: true
   ```
10. Buat pipeline dengan nama file _.gitlab-ci.yml_. File ini yang nanti akan mendeploy secara otomatis aplikasi blog berita dan juga menjalankan databasenya. Perintah pada file ini diantaranya build image dari file _Dockerfile_ dan _Dockerfilemysql_, membuat network bridge dengan nama _blogberita_, menjalankan MySql dengan menggunakan image hasil build dari file _Dockerfilemysql_, memberhentikan semua service yang dijalankan dengan _docker-compose.yml_ serta menghapus semua image yang terkait dan yang terakhir menjalankan service yang ada pada _docker-compose.yml_ di background. Untuk membuat pipeline gunakan perintah berikut:

    ```bash
    vim .gitlab-ci.yml
    ``` 
    ```yaml
    stages:
      - build-webserver
    
    variables:
      DOCKER_IMAGE: index.docker.io/$DOCKER_USER/blogberita-mohammadalwanadb:1.0
    
    
    build-webserver:
      stage: build-webserver
      script:
        - 'docker login -u $DOCKER_USER -p $DOCKER_PASSWORD  "docker.io"'
        - 'docker build -t $DOCKER_IMAGE .'
        - 'docker build -t imagesmysql:1.0 -f Dockerfilemysql .'
        - 'docker network create -d bridge blogberita'
        - 'docker run -d --name mysql \
           -e MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD \
           -e MYSQL_DATABASE=$MYSQL_DATABASE \
           -e MYSQL_USER=$MYSQL_USER \
           -e MYSQL_PASSWORD=$MYSQL_PASSWORD \ 
           -v /home/student/blogberita/blog/data-db:/var/lib/mysql \
            --network blogberita -p 3306:3306 imagesmysql:1.0'
        - 'docker push $DOCKER_IMAGE'
        - 'docker-compose down --rmi all' 
        - 'docker-compose up -d'
      only:
        - main
    ```
11. Buat direktori untuk volume database. Jadi karena nanti mysql berjalan sebagai container, direktori yang akan dibuat ini akan terhubung dengan penyimpanan data mysql. Untuk membuat direktori gunakan  dengan perintah berikut:
    ```bash
    mkdir data-db
    ```

12. Apabila semua step diatas sudah dilakukan, kita bisa langsung masukkan semua perubahan ke repository lokal. Tapi sebelum memasukkan semua perubahan ke repository lokal, teman-teman bisa gunakan perintah cek semua perubahan yang sudah dilakukan dengan perintah **git status**. Jika sudah, Untuk memasukkan semua perubahan ke repository lokal, gunakan perintah berikut:
    ```bash
    git add .
    ```
    ```bash
    git commit . -m "<messages>"
    ```

13. Setelah sebelumnya kita sudah melakukan konfigurasi pada terminal, selanjutnya konfigurasi kita lakukan pada GitLab. Pertama, atur value dari variable yang kita gunakan pada file _.gitlab-ci.yml_ dan _docker-compose.yaml_ . Untuk itu, masuk ke project yang sudah dibuat pada **GitLab** <span>&#8594;</span> **Settings** <span>&#8594;</span> **CI/CD** <span>&#8594;</span> **Variables** <span>&#8594;</span> **Add Variables**. Berikut variable-nya:

> DOCKER_USER - username docker hub. Jadi teman-teman dipastikan harus memiliki akun pada docker hub. Karena kita akan menggunakan base image dari docker hub dan juga nanti kita akan push image ke docker hub. Untuk sign up akun docker hub bisa pada link berikut. 
>
> DOCKER_PASSWORD - Kata sandi (password) dari user docker hub.
>
> MYSQL_ROOT_PASSWORD -   Kata sandi (password) user root MySQL.
>
> MYSQL_DATABASE - Nama database (yang akan dibuat) pada MySQL
>
> MYSQL_USER - User (yang akan dibuat) pada MySQL.
>
> MYSQL_PASSWORD - kata sandi (password) yang akan digunakan oleh user yang sudah dibuat.

![variable](variable.png)

14. Atur GitLab Runner dengan executor Docker. Caranya sebagai berikut:
- **Project pada GitLab** <span>&#8594;</span> **Setting** <span>&#8594;</span> **CI/CD** <span>&#8594;</span> **Runners** <span>&#8594;</span> **Copy Registration token**
![token](token.png)
- Paste hasil copy pada GitLab tadi di terminal ke bagian setelah " --registration-token " pada perintah dibawah. Contohnya sebagai berikut:
    ```bash
    sudo gitlab-runner register -n --url <GitLab-URL> \
    --registration-token [TOKEN] --executor docker --description "Docker Runner" \
    --docker-image docker:latest --docker-volumes /var/run/docker.sock:/var/run/docker.sock
    ```
- Verifikasi atau periksa executor dan pastikan tidak ada masalah
    ```bash
    sudo gitlab-runner verify
    ```
15. Push repository lokal teman-teman ke repository GitLab. Sebelum itu periksa branch yang digunakan teman-teman dengan mengeceknya menggunakan **git branch**. Jika branch yang digunakan adalah main, maka cara untuk push-nya dengan cara dibawah ini. Jika bukan tinggal disesuaikan dengan branch teman-teman.
    ```bash
    git push origin main
    ```
16. Pastikan pipeline sudah running. Untuk itu cek dengan cara berikut:
- **Project pada GitLab** <span>&#8594;</span> **Build** <span>&#8594;</span> **Pipelines**
    ![pipeline-running](pipeline-running.png)

17. Pastikan juga pipeline berjalan dengan lancar dan berhasil. Jika berhasil maka statusnya akan berubah, dari **running** menjadi **passed**. Untuk melihat proses berjalannya pipeline, bisa dilihat pada bagian **Jobs**. Dibawah ini adalah contoh pipeline sudah berhasil.
![pipeline-hasil1](pipeline-hasil1.png)
![pipeline-hasil2](pipeline-hasil2.png)

18. Pastikan juga pada terminal teman-teman status dari Container adalah running. Cek dengan menggunakan perintah berikut:
    ```bash
    docker ps	
    ```
    ![docker-ps](docker-ps.png)

19. Lakukan tunneling dan cek pada port 8000.
![hasil-berita](hasil-berita.png)

20. Login menggunakan:
	- Email : admin@gmail.com
	- Password : admin123
![login](login.png)
![panel-admin](panel-admin.png)
