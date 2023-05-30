# Ví dụ shell
## 1. Cài wordpress bằng shell
- B1: Tạo 1 file với tên là wordpress_install.sh:
```
nano wordpress_install.sh
```
- B2: Thêm vào file wordpress_install.sh:
```
#!/bin/bash

# Cài đặt các gói phụ thuộc
sudo apt-get update
sudo apt-get install -y nginx mysql-server php7.4 php7.4-gd php7.4-mysql php7.4-zip php7.4-fpm

# Khởi động Nginx và MySQL
sudo systemctl start nginx
sudo systemctl start mysql

# Cấu hình MySQL
sudo mysql_secure_installation

# Tạo cơ sở dữ liệu WordPress
sudo mysql -u root -p <<EOF
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'Tung@2402';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT
EOF

# Tải WordPress và giải nén
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo cp -R wordpress/* /var/www/html/

# Cấu hình Nginx
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak

sudo tee /etc/nginx/sites-available/wordpress <<EOF
server {
    listen 80;
    listen [::]:80;
    root /var/www/html;
    index index.php index.html index.htm index.nginx-debian.html;
    # server_name your_domain;

    location / {
        try_files \$uri \$uri/ /index.php?\$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
EOF

sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Cấu hình WordPress
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo sed -i 's/database_name_here/wordpress/g' /var/www/html/wp-config.php
sudo sed -i 's/username_here/wpuser/g' /var/www/html/wp-config.php
sudo sed -i 's/password_here/Tung@2402/g' /var/www/html/wp-config.php

# Phân quyền
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/

# Hoàn tất
echo "Cài đặt WordPress thành công!"
```
- B3: Cấp quyền cho file wordpress_install.sh:
```
sudo chmod +x wordpress_install.sh
```
- B4: Chạy shell
```
./wordpress_install.sh
```
Sau khi cài xong thì chúng ta đăng nhập là xong
## 2. Tạo scritp bắn message đến nhóm trong tele với nội dung message là về thông tin phần cứng của server bao gồm ram, cpu, disk, load average:
- B1: Tạo 1 file mới tên là hardware_info.sh
```
nano hardware_info.sh
```
- B2: Thêm vào file hardware_info.sh dòng sau:

Để lấy token và ip chat của nhóm telegram thì bạn làm theo video sau: https://www.youtube.com/watch?v=JNwEJ5HvLgM&pp=ygUbdG9rZW4gdsOgIGlwIG5ow7NtIHRlbGVncmFt
```
#!/bin/bash

# Cấu hình Telegram Bot và chat_id
TELEGRAM_BOT_TOKEN="token của bạn"
CHAT_ID="ip nhóm của bạn"

# Hàm lấy thông tin phần cứng
get_hardware_info() {
    ram=$(free -h | awk '/^Mem/ { print $2 }')
    cpu=$(grep -m 1 'model name' /proc/cpuinfo | cut -d ':' -f 2)
    disk=$(df -h | awk '$NF == "/" { print $2 }')
    load_avg=$(uptime | awk -F'[a-z]:' '{ print $2 }')

    message="Thông tin phần cứng của server:
RAM: $ram
CPU: $cpu
Disk: $disk
Load Average: $load_avg"

    send_telegram_message "$message"
}

# Hàm gửi message đến nhóm trong Telegram
send_telegram_message() {
    message="$1"
    curl -s -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" -d "chat_id=$CHAT_ID" -d "text=$message" > /dev/null
}

# Gửi thông tin phần cứng đến nhóm trong Telegram
get_hardware_info

# Lặp vô hạn với khoảng cách 3 phút
while true
do
    sleep 180
    get_hardware_info
done
```
- B3: Cấp quyền cho file hardware_info.sh:
```
sudo chmod +x hardware_info.sh
```
- B4: Chạy shell:
```
./hardware_info.sh
```
Nếu bạn muốn để chạy ngầm thì dùng lệnh nohup:
```
nohup ./hardware_info.sh &
```
Sau khi chạy shell thì cứ 3p thì con bot của chúng ta sẽ tự động gửi tin nhắn lên nhóm
