# Dockerise PHP application with Nginx and PHP7-FPM
## Tutorial

Buat file `docker/docker-compose.yml` sebagai berikut :

``` diff
-------------------------- docker/docker-compose.yml --------------------------
+version: '2.1'
+services:
+  dev-nginx:
+    restart: always
+    image: nginx:latest
+    container_name: dev-nginx
+    ports:
+      - "8080:80"
```

Jalankan docker compose `docker-compose up`. Setiap kali menjalankan docker, pastikan tidak terdapat output error dari container. Berikut adalah contoh output ketika docker compose berhasil membuat container:
```
Creating dev-nginx ...
Creating dev-nginx ... done
Attaching to dev-nginx
```
Buka localhost:8080 dan berikut adalah hasilnya:

![nginx at localhost:8080](https://raw.githubusercontent.com/valdiearsanur/dockerize-php-nginx/master/readme_asset/1.png)

Selanjutya kita dapat menambahkan project pada nginx. Nginx mempunyai konfigurasi default.conf yang berada pada container. Kita dapat menimpa (override) file tersebut dengan melakukan mapping volume pada file composer. Sebagai contoh kita akan membuat file `./docker/nginx/sites/site.conf` yang akan mengoverride `default.conf` milik container

``` diff
-------------------------- docker/docker-compose.yml --------------------------
     container_name: dev-nginx
     ports:
       - "8080:80"
+    volumes:
+      - ../workspace:/home/workspace:ro
+      - ./nginx/sites/site.conf:/etc/nginx/conf.d/default.conf:ro
```

Setelah melakukan mapping pada volume container, kita dapat membuat file `site.conf` sebagai berikut :

``` diff
-------------------------- docker/nginx/sites/site.conf --------------------------
+server {
+    listen 80;
+    index index.php index.html;
+    server_name localhost;
+    error_log  /var/log/nginx/error.log;
+    access_log /var/log/nginx/access.log;
+    root /home/workspace/project;
+}
```

Pada site.conf, konfigurasi root folder dari webhost kita berada pada `/home/workspace/project` (di container) atau `./workspace/project` pada local file kita. Kita akan menambahkan 1 buat file  untuk memastikan web server kita berhasil terdeploy.

``` diff
-------------------------- workspace/project/index.html --------------------------
+<h1>Hi!</h1>
+I'm running on NGINX 
```

setelah itu deploy tekan ctrl+c pada terminal dan lakukan `docker-compose up` lagi. Hasilnya sebagai berikut:

![nginx at localhost:8080](https://raw.githubusercontent.com/valdiearsanur/dockerize-php-nginx/master/readme_asset/2.png)

Sampai di sini kita sudah berhasil membuat web server, namun NGINX ini tidak mengenali file .php. Selanjutnya kita akan menambakan PHP FPM (FastCGI Process Manager). Kita perlu menambahkan 1 buah container pada file docker compose

``` diff
-------------------------- docker/docker-compose.yml --------------------------
     volumes:
       - ../workspace:/home/workspace:ro
       - ./nginx/sites/site.conf:/etc/nginx/conf.d/default.conf:ro
+    links:
+      - dev-php:fpm_php
+
+  dev-php:
+    restart: always
+    image: php:7-fpm
+    container_name: dev-php
+    volumes:
+      - ../workspace:/home/workspace:ro 
```

selanjutnya kita perlu melakukan konfigurasi pada nginx dengan menambahkan perintah berikut:

``` diff
-------------------------- docker/nginx/sites/site.conf --------------------------
     error_log  /var/log/nginx/error.log;
     access_log /var/log/nginx/access.log;
     root /home/workspace/project;
+
+    location ~ \.php$ {
+        try_files $uri =404;
+        fastcgi_split_path_info ^(.+\.php)(/.+)$;
+        fastcgi_pass fpm_php:9000;
+        fastcgi_index index.php;
+        include fastcgi_params;
+        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
+        fastcgi_param PATH_INFO $fastcgi_path_info;
+    }
 }
```

Untuk memastikan web server kita sudah bisa membaca file php, kita akan menghapus file `index.html` dan membuat file `index.php` yang isinya sebagai berikut:

``` diff
-------------------------- workspace/project/index.php --------------------------
+<?php
+phpinfo(); 
```

setelah itu deploy tekan ctrl+c pada terminal dan lakukan `docker-compose up` lagi. Hasilnya dapat dilihat seperti gambar di bawah:

![nginx at localhost:8080](https://raw.githubusercontent.com/valdiearsanur/dockerize-php-nginx/master/readme_asset/3.png)

Kita sudah berhasil membuat project php kita berjalan di atas container docker, yang artinya kita mempunyai project yang siap untuk di deploy di komputer manapun hanya dengan `docker compose up` tanpa perlu memikirkan versi php, web server, konfigurasi, dsb.

Tambahan:
Sebelumnya kita sudah melakukan konfigurasi log nginx (pada site.conf) sebagai berikut:
```
error_log  /var/log/nginx/error.log;
access_log /var/log/nginx/access.log;
```
File tersebut masih terdapat pada container nginx, sehingga kita tidak dapat secara langsung melihat log. Kita dapat menyimpan file tersebut pada local dengan menambahkan mapping volume pada file `docker-compose.yml`

``` diff
-------------------------- docker-compose.yml --------------------------
     volumes:
       - ../workspace:/home/workspace:ro
       - ./nginx/sites/site.conf:/etc/nginx/conf.d/default.conf:ro
+      - ./nginx/log:/var/log/nginx
     links:
       - dev-php:fpm_php
```

setelah itu deploy tekan ctrl+c pada terminal dan lakukan `docker-compose up` lagi. Setelah itu kita akan melihat file `./docker/nginx/log/access.log` dan `./docker/nginx/log/access.log` yang artinya file log berhasil kita mapping.


Project ini hanyalah contoh sederhana dengan stack standar, yaitu Web Server dan PHP. Kebutuhan project kalian mungkin bisa berbeda-beda, misalnya database, caching, balancer, dsb. Selanjutnya kalian perlu menambahkan container sesuai dengan stack pada project.