# راهنمای نصب و پیکربندی وب‌سرور Caddy با PHP و MySQL

این راهنما به شما کمک می‌کند تا یک وب‌سرور با Caddy، PHP، و MySQL را بر روی یک سرور Ubuntu نصب و پیکربندی کنید. این دستورالعمل‌ها شامل نصب نرم‌افزارهای مورد نیاز، پیکربندی Caddy برای پشتیبانی از PHP، ایجاد یک پایگاه داده MySQL، و نصب ماژول‌های PHP است. همچنین، نحوه رفع برخی از خطاهای رایج و تنظیم کرون‌جاب را نیز پوشش می‌دهد.

## ۱. به‌روزرسانی سیستم

قبل از هر کاری، باید سیستم خود را به‌روزرسانی کنید تا مطمئن شوید که همه بسته‌ها و نرم‌افزارهای موجود به آخرین نسخه‌ها به‌روزرسانی شده‌اند:

```bash
sudo apt update
sudo apt upgrade -y
```

این دستورات لیست بسته‌ها را به‌روزرسانی کرده و سپس تمام بسته‌های نصب‌شده را به آخرین نسخه‌ها به‌روزرسانی می‌کند.

## ۲. نصب وب‌سرور Caddy

برای نصب وب‌سرور Caddy از دستور زیر استفاده کنید:

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo apt-key add -
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy -y
```

این دستورات مخزن Caddy را به سیستم شما اضافه کرده و سپس Caddy را نصب می‌کنند، همچنین می‌توانید از [داکیومنت رسمی Caddy](https://caddyserver.com/docs/install#) برای نصب بر روی سیستم‌عامل‌های مختلف استفاده کنید.

## ۳. نصب MySQL

### نصب MySQL

برای نصب MySQL و انجام تنظیمات اولیه امنیتی، دستورات زیر را اجرا کنید:

```bash
sudo apt install mysql-server -y
sudo mysql_secure_installation
```

در حین اجرای دستور `mysql_secure_installation`، به ترتیب به سؤالات پاسخ دهید:
- اولین پرسش را با `N` پاسخ دهید. (استفاده از پلاگین اعتبارسنجی رمز عبور)
- سایر سؤالات را با `Y` پاسخ دهید.

### ایجاد کاربر و تنظیمات امنیتی

برای ایجاد یک کاربر جدید MySQL و تنظیم رمز عبور، ابتدا وارد کنسول MySQL شوید:

```bash
sudo mysql
```

سپس دستورات زیر را اجرا کنید تا یک کاربر جدید با دسترسی‌های کامل ایجاد شود:

```sql
CREATE USER 'Your_USER'@'localhost' IDENTIFIED BY 'Your_PASS';
GRANT ALL PRIVILEGES ON *.* TO 'Your_USER'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

**توجه:** `Your_USER` را با نام کاربری دلخواه و `Your_PASS` را با رمز عبور دلخواه خود جایگزین کنید.

## ۴. نصب PHP و ماژول‌های مرتبط

### نصب PHP

برای نصب PHP و ماژول‌های مرتبط مورد نیاز برای اجرای PHP در Caddy، از دستورات زیر استفاده کنید:

```bash
sudo apt install php-fpm php-mysql -y
```

### نصب ماژول‌های اضافی PHP

اگر به ماژول‌های بیشتری نیاز دارید، از دستور زیر استفاده کنید: (پیشنهادی)

```bash
sudo apt install php8.1-amqp php8.1-apcu php8.1-bcmath php8.1-calendar php8.1-ctype php8.1-curl php8.1-intl php8.1-enchant php8.1-exif php8.1-ftp php8.1-gd php8.1-gmp php8.1-igbinary php8.1-imagick php8.1-imap php8.1-intl php8.1-ldap php8.1-mbstring php8.1-memcache php8.1-memcached php8.1-mongodb php8.1-msgpack php8.1-mysql php8.1-oauth php8.1-opcache php8.1-raphf php8.1-redis php8.1-sqlite3 php8.1-tidy php8.1-uuid php8.1-xdebug php8.1-xml php8.1-xmlrpc php8.1-yaml -y
```

**نکته:** تنها ماژول‌هایی را که برای پروژه شما ضروری هستند نصب کنید. نصب ماژول‌های اضافی ممکن است باعث استفاده بیش از حد از منابع سیستم شود.

## ۵. پیکربندی Caddy برای پشتیبانی از PHP

### تنظیمات SSL

وب سرور Caddy به‌صورت پیش‌فرض از SSL پشتیبانی می‌کند و گواهینامه‌های Let’s Encrypt را به‌طور خودکار مدیریت می‌کند. اگر به SSL خود مدیریت شده نیاز دارید، می‌توانید فایل‌های SSL (`fullchain.pem` و `privkey.pem`) را در مسیری قرار دهید و در فایل پیکربندی Caddy مشخص کنید.

### ویرایش فایل پیکربندی Caddy

فایل پیکربندی Caddy را برای فعال‌سازی HTTPS و پشتیبانی از PHP ویرایش کنید. این فایل معمولاً در مسیر `/etc/caddy/Caddyfile` قرار دارد:

```bash
sudo nano /etc/caddy/Caddyfile
```

فایل را به صورت زیر پیکربندی کنید:

```caddy
yourdomain.com {
    root * /usr/share/phpmyadmin
    php_fastcgi unix//run/php/php8.1-fpm.sock
    file_server

    @phpmyadmin {
        path /phpmyadmin
    }
    rewrite @phpmyadmin /phpmyadmin/

    log {
        output file /var/log/caddy/errors.log
        format json
        level ERROR
    }

    encode gzip
}
```

**نکته:** مسیر `php_fastcgi unix//run/php/php8.1-fpm.sock` باید با نسخه PHP نصب‌شده در سیستم شما مطابقت داشته باشد.

### بررسی و راه‌اندازی مجدد Caddy

پس از انجام تغییرات، پیکربندی Caddy را بررسی و سرویس را راه‌اندازی مجدد کنید:

```bash
sudo systemctl reload caddy
```

### ۶. ایجاد پایگاه داده و دسترسی‌ها

ابتدا phpMyAdmin را با دستور زیر نصب کنید:

```bash
sudo apt install phpmyadmin -y
```

برای ایجاد یک پایگاه داده در MySQL و اختصاص دسترسی‌های لازم به یک کاربر، ابتدا وارد MySQL شوید:

```bash
sudo mysql
```

سپس دستورات زیر را اجرا کنید:

```sql
CREATE DATABASE mydatabase;
GRANT ALL PRIVILEGES ON mydatabase.* TO 'Your_USER'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

اکنون می‌توانید با رفتن به آدرس `yourdomain.com/phpmyadmin` به phpMyAdmin دسترسی داشته باشید.


## نکات اضافی

### ۱. رفع خطاها

#### نصب اکستنشن Soap

برای رفع خطای `Uncaught Error: Class "SoapClient" not found`، اکستنشن soap را نصب و فعال کنید:

```bash
sudo apt install php-soap -y
sudo systemctl restart php8.1-fpm
sudo systemctl reload caddy
```

### ۲. تنظیم دسترسی‌ها

برای تنظیم دسترسی‌های صحیح به فایل‌ها و دایرکتوری‌ها، دستورات زیر را اجرا کنید:

```bash
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

### ۳. تغییر گروه مالک و اعمال بیت Setgid

دستورات زیر به ترتیب برای تغییر گروه مالک دایرکتوری و اعمال بیت Setgid روی آن استفاده می‌شوند:

```bash
sudo chgrp -R www-data /var/www/html/
sudo chmod g+s /var/www/html/
```

### ۴. تنظیم کرون‌جاب

برای اجرای یک اسکریپت PHP به صورت منظم با استفاده از کرون‌جاب، دستور زیر را در کرون‌جاب وارد کنید:

```bash
crontab -e
 * * * * * /usr/bin/php /var/www/html/test/test.php
```

### ۵. بررسی لاگ‌ها

برای بررسی مشکلات احتمالی، لاگ‌های Caddy و PHP-FPM را چک کنید:

```bash
sudo tail -f /var/log/caddy/access.log
sudo tail -f /var/log/php8.1-fpm.log
```

### ۶. مدیریت آپلود فایل‌های حجیم

#### تغییر تنظیمات PHP

##### ویرایش فایل `php.ini`

فایل `php.ini` را ویرایش کنید:

```bash
sudo nano /etc/php/8.1/fpm/php.ini
```

مقادیر زیر را تغییر دهید:

```ini
upload_max_filesize = 10M
post_max_size = 12M
```

##### ریستارت PHP-FPM

برای اعمال تغییرات، PHP-FPM را ریستارت کنید:

```bash
sudo systemctl restart php8.1-fpm
```

#### تغییر تنظیمات Caddy

##### ویرایش فایل پیکربندی Caddy

فایل پیکربندی Caddy را ویرایش کنید:

```bash
sudo nano /etc/caddy/Caddyfile
```

خط زیر را اضافه کنید:

```caddy
yourdomain.com {
    ...
    client_max_body_size 10M
    ...
}
```

##### ریستارت Caddy

برای اعمال تغییرات، Caddy را ریستارت کنید:

```bash
sudo systemctl reload caddy
```

### ۷. نصب و راه‌اندازی UFW (فایروال)

برای افزایش امنیت سرور، توصیه می‌شود از فایروال UFW استفاده کنید. ابتدا UFW را نصب و سپس پورت‌های مورد نیاز را باز کنید:

```bash
sudo apt install ufw -y
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow پورت دلخواه
sudo ufw enable
```

پس از اعمال این تنظیمات، UFW را فعال کنید تا سرور شما در برابر ترافیک‌های ناخواسته محافظت شود.

**نکته:** اگر برنامه‌ای دیگری دارید که به پورت‌های دیگری نیاز دارد، باید آن پورت‌ها را نیز باز کنید.
