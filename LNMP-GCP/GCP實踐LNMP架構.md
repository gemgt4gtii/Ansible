# GCP實踐LNMP架構

## **架構**
- Linux CentOS 7.6.1810
- Nginx 1.14.2
- PHP 7.2.11
- MySQL 8.0.15
- Redis 5.0.3
- PHPMyAdmin
----------
# 一、安裝Nginx 1.14.2
- 更新套件庫
    sudo yum update -y
    sudo yum -y install epel-release
- 建立一個nginx的repo
    sudo vi /etc/yum.repos.d/nginx.repo
    -------------------------------------------------------------------------------------
    [nginx]
    name=nginx repo
    baseurl=http://nginx.org/packages/centos/7/$basearch/
    gpgcheck=0
    enabled=1
    -------------------------------------------------------------------------------------
- 安裝指定版本
    sudo yum install -y nginx-1.14.2
- 查看安裝目錄及設定檔
    nginx -t
- 起動nginx服務
    sudo systemctl start nginx
- 設定開機自動啟動
    sudo systemctl enable nginx
- 在瀏覽器輸入ip測試服務是否啟用
****![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556446686100_Nginx-01.png)

# 二、安裝PHP 7.2.11
- Centos套件庫來源並沒有PHP7.x，所以要自行加入套件來源-epel
    sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
- webtatic
    sudo rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
- PHP
    sudo yum install -y php72w-cli php72w-fpm php72w-common php72w-pdo php72w-mbstring
- 確認安裝php版本(我安裝的是7.2.16)
    php -v
- 啟用 php 服務並 設定開機時啟動
    sudo systemctl start php-fpm 
    sudo systemctl enable php-fpm


# 三、安裝MySQL 8.0.15
- #參照[官網(Download MySQL Yum Repository)](https://dev.mysql.com/downloads/repo/yum/) 找尋指定版本
    ***
    rpm -Uvh https://repo.mysql.com/mysql57-community-release-el7-11.noarch.rpm
    yum -y --enablerepo=mysql80-community install mysql-community-server
    systemctl start mysqld.service
    ***
- 啟動MySQL server
    sudo service mysqld start
- 啟動MySQL
    sudo systemctl start mysqld
- 設定開機自行啟動
    sudo systemctl enable mysqld
- 檢視預設密碼
    grep "A temporary password" /var/log/mysqld.log
    顯示結果：
    [gemgt4gtii@lnmp-lab ~]$ grep "A temporary password" /var/log/mysqld.log
    
    2019-04-28T12:52:01.986505Z 5 \[Note\] [MY-010454] [Server] A temporary password is generated for root@localhost: 9ur%Osgfdzy.
    -------------------------------------------------------------------------------------
- 修改MySQL的root密碼
    mysql -u root -p
   - ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'a_A@123456';
   - ALTER USER USER() IDENTIFIED BY 'a_A@123456';
   - ALTER USER USER() IDENTIFIED BY 'abc';
   - ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password AS '*0D3CED9BEC10A777AEC23CCC353A8C08A633045E';
    systemctl restart mysqld.service
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556498326852_image.png)

![測試修改後的密碼](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556460004956_image.png)

![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556498278640_image.png)

----------
# 防火牆設定：
- 開啟遠端訪問
    sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
- 重新啟動防火牆
    sudo firewall-cmd --reload
    指令說明：
    firewall-cmd：指令               
    -zone=public ：指定區域(zone)為=(自定義)
    -add-port=3306/tcp： 添加端口為=3306/tcp
    -permanent：此規則永久生效
----------
# 設定參數：
## Nginx—default.conf
    sudo vim  /etc/nginx/conf.d/default.conf
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556502366086_image.png)

- 將預設參數修改為：
    server {
        listen       80;
        server_name  localhost;
    
        charset utf-8;
        access_log  /var/log/nginx/access.log  main;
    
        root   /usr/share/nginx/html;
        index  index.php index.html index.htm; 
    
        location / {
            try_files $uri $uri/ =404;
        }
    
        error_page  404              /404.html;
    
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    
        location ~ \.php$ {
            try_files $uri =404;
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    }
----------
## PHP—php.ini、www.conf
- 修改參數-
    sudo vi /etc/php.ini
    搜尋 ;cgi.fix_pathinfo=1，將預設1改為0，並將註解去掉。
    → cgi.fix_pathinfo=0
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556502661792_php_conf.png)

- 修改php.d目錄底下的www.conf預設參數
    sudo vi /etc/php-fpm.d/www.conf
    -----------------------------------修改內容-------------------------------------------
    user = nginx
    group = nginx
    listen.owner = nobody
    listen.group = nobody
    -----------------------------------修改內容-------------------------------------------
    將註解 ; 去掉。
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556504401829_php-fpm-www-conf.png)

- 新增測試頁面 info.php
    sudo vi /usr/share/nginx/html/info.php
    <?php phpinfo(); ?>
- 重新啟動服務
    sudo systemctl restart php-fpm
    sudo systemctl restart nginx
- 測試網頁
    開啟瀏覽器，在網址處輸入
    該主機IP/info.php
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556504865646_php-web.png)

# 將MySQL與PHP進行串接
- 安裝與資料庫進行連接的相關套件
    sudo yum install -y php72w-my*
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556514557621_image.png)

- 出現以上錯誤，改為以下指令—
    sudo yum install -y php72w-my* --skip-broken
- 重新啟動php-fpm
    sudo systemctl restart php-fpm
- 重新載入畫面
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556514839235_image.png)

# 四、redis 5.0.3(以目錄方式啟動服務)
## 編譯安裝

**安裝編譯工具**

----------
- 安裝編譯 gcc 所需要的函式庫：
    sudo yum install -y libmpc-devel mpfr-devel gmp-devel
- 用 `wget` 下載 gcc ：
    wget http://mirrors.concertpass.com/gcc/releases/gcc-6.3.0/gcc-6.3.0.tar.bz2
- 解壓縮 gcc 原始碼：
    tar jxvf gcc-6.3.0.tar.bz2
- 進入 gcc 原始碼目錄：
    cd gcc-6.3.0/
- 下載其他必要的原始碼：
    contrib/download_prerequisites
- 建立編譯用的目錄：
    mkdir ../gcc-build
    cd ../gcc-build
- 使用以下參數執行 `configure`：
    ../gcc-6.3.0/configure -v \
      --enable-languages=c,c++ \
      --disable-multilib \
      --prefix=/usr/local/gcc-6.3.0

或著是：

    ./configure --prefix=/usr/local/gcc-6.3.0

指令：

    --prefix=[軟件安裝目錄]

將 gcc 6.3 安裝在 `/usr/local/gcc-6.3.0` 這個目錄中，如果想安裝在其他的地方，可以自己更改這個路徑。

- 執行 `make` 編譯 gcc 6
    make 

｢編譯完成後，進行安裝：

    sudo mkdir /usr/local/gcc-6.3.0
    sudo chown [用戶]:[用戶] /usr/local/gcc-6.3.0
    make install


----------
- **下載後進入目錄進行編譯(make)**
    wget http://download.redis.io/releases/redis-5.0.3.tar.gz
- **解壓縮**
    tar xzf redis-5.0.4.tar.gz
- **進入壓縮後的目錄**
    cd redis-5.0.4
- **使用make指令進行編譯**
    make ； make install

編輯conf檔

    sudo vi redis.conf
    #將後台運行改為yes
    daeminize no  -> daeminize yes
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556515858305_image.png)

## 設置密碼
- 找到requirepass foobared 字符，在下面添加一行，格式為：
    requirepass [密碼]
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556517202123_redis-conf-pass.png)

    由該conf檔可知道預設的埠號為6379
    #開起指定埠號
    sudo firewall-cmd --permanent --add-port=6379/tcp
    sudo firewall-cmd --reload
## 啟動redis服務
- 進入src目錄裡執行redis-server和redis.conf，檢查埠號確認該服務是否有啟動。
    cd src
    ./redis-server ../redis.conf
    netstat -nlpt
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556517518292_image.png)

- 測試連接：
    到src目錄下執行redis-cli
    輸入：
    auth 密碼
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556517758903_image.png)

----------


----------
# 五、安裝Composer、PHPMyAdmin
## 由於使用yum安裝PHPMyAdmin會出現版本衝突的BUG，所以要使用composer套件進行安裝。
## 安裝Composer([官方指南](https://getcomposer.org/download/))
- 安裝composer
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php -r "if (hash_file('sha384', 'composer-setup.php') === '48e3236262b34d30969dca3c37281b3b4bbe3221bda826ac6a9a62d6444cdb0dcd0615698a5cbe587c3f0fe57a54d8f5') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
    php composer-setup.php
    php -r "unlink('composer-setup.php');"

指令分別為：

1. 複製安裝文件 PHP 檔案
2. 認證下載的文檔是否正確
3. 執行安裝 composer
4. 移除安裝文件
- 完成以後，我們要放到全域環境，這樣才能讓任何專案無須從新安裝 composer。#composer不可在root權限使用。
    sudo mv composer.phar /usr/local/bin/composer
- 由 root 更改權限給使用者
    sudo chown -R z1234:z1234 /usr/share/nginx/html
    sudo chmod 777 /usr/share/nginx/html
## 安裝 phpMyAdmin([官方文件](https://docs.phpmyadmin.net/en/latest/setup.html#installing-from-git))
- 到nginx的html目錄下使用composer指令將phpmyadmin安裝自此，如果安裝在其它目錄的話使用mv指令移至該目錄下即可。
    cd /usr/share/nginx/html
## 方法(一)
    composer create-project phpmyadmin/phpmyadmin
## 方法(二)
    composer create-project phpmyadmin/phpmyadmin --repository-url=https://www.phpmyadmin.net/packages.json --no-dev

#如果出現安裝失敗直接砍到整個目錄即可(phpmyadmin)

![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556862725182_image.png)

- 將 /var/lib/php 將底下的session目錄權限更改為nginx，這樣才可以透過nginx登入phpmyadmin
    cd /var/lib/php
    sudo chown nginx:nginx session/ -R
- 重新啟動php和nginx服務
    sudo service php-fpm restart
- 開啟瀏覽器輸入： 主機IP/phpmyadmin 即可
![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556629199195_phpmyadmin-local.png)


使用root登入

![](https://paper-attachments.dropbox.com/s_9C34C7E752749B1D47E4CDE933BFF2127C3DE17D0DAC2EF88166AC9155B2C1BF_1556629229331_image.png)


