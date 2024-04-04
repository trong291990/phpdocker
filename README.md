## Tổng quan 
Đây là bộ docker-compose dùng để phát triển các ứng dụng trên nền PHP/MySQL. Gồm có:

- PHP 7.4 và PHP 8.1

- MySQL phiên bản mới nhất (Sử dụng MariaDB)

- Nginx 

- Cronjob, Supervisor

## Cách cài đặt 

### Cài đặt chung

- Copy file .env.example sang một file mới và đặt tên là .env, sửa các giá trị nếu muôn 

- Copy file docker-compose.yml.example sang một file mới và đặt tên là docker-compose.yml, sửa các giá trị tại các row có dòng `// need to edit` cho chính xác

### Cài đặt web
- Thêm hai dòng này vào hosts file

```
127.0.0.1 adminer.local
::1 adminer.local
```

### Hoàn thành

- Tại thư mục root (ngang hàng với file docker-composer.yml), gõ lệnh `docker-compose up -d`

- Vào trình duyệt gõ http:://adminer.local:9090. Servername nhập mariadb, username/pass thì tìm ở file .env. Nếu login thành công vào adminer thì đã setup thành công

## Database
- Khi import file Data (SQL) có dung lượng lớn, hãy copy vào thư mục data/shared (được mount vào /var/shared ở container mariadb) và dùng command line để import

## Setup một site mới

Ví dụn muốn setup 1 file mới - dùng laravel:


- Tại máy local, thêm hai dòng này vào hosts file

```
127.0.0.1 mysite.local
::1 mysite.local
```
- Mount tư mục web vào nginx:

Mở file docker-compose.yml, trong service lep, phần volum, hãy thêm đoạn này:e
```
      - FULLPATH_CUA_THU_MUC_LARAVEL:/var/www/sites/mysite:rw
```

Tạo thêm một file có tên là mysite.conf trong thư mục lep/nginx/sites có nội dung sau:

``` 
server {

    listen 80;
    listen [::]:80;

    server_name mysite.local;
    root /var/www/sites/mysite/public;
    index index.php index.html index.htm;

    error_log /var/log/nginx/kk.error.log;

    client_max_body_size 0;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ [^/]\.php(/|$) {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }
}
```

Nếu sử dụng php7.4, thì thay

`fastcgi_pass unix:/run/php/php8.1-fpm.sock;` thành `fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;`

- Restart lại service lep: docker-compose restart lep

- Vào trình duyệt gõ http://mysite.local:9090
