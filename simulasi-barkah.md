#Soal no 19
<br/>
###Seting interface VM 1 jadi ip static
1) copy seting interface sebelum di edit supaya nanti ada backup
sudo cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth1
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/set-ip-statik1.png) 

2) ubah seting dhcp eth0 ke ip static 10.141.0.69
sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/set-ip-statik2.png) 
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/set-ip-statik3.png) 
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/set-ip-statik4.png) 

3) Lakukan restart network setelah perubahan dilakukan
sudo systemctl restart network
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/set-ip-statik5.png) 

4) Cek kembali hasil seting ip
ifconfig eth0
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/set-ip-statik6.png) 
kesimpulan seting sudah berhasil
<br/>
###Login ssh tanpa password
1) Membuat ssh authorized_keys dengan command ssh-keygen dan dibuat tanpa password
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/ssh-keygen.png) 

2) Copy authorized_keys dari lokal ke vm ssh-copy-id -i ~/.ssh/id_rsa.pub centos@192.168.1.89 -p 2269
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/ssh2.png) 

3) Coba login ssh -p 2269 centos@192.168.1.89, berhasil login tanpa password
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/ssh3.png) 
<br/>
###Install webserver nginx dan seting mod rewrite
1) sudo yum install epel-release (menginstall repo EPEL)
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/epel-release.png) 

2) sudo yum install nginx -y (menginstall nginx)
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/install-nginx.png) 

3) Enable dan Start nginx
-sudo systemctl enable nginx (mengaktifkan service nginx supaya ketika system booting service bisa berjalan langsung)
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/enable-nginx.png) 
-sudo systemctl start nginx (menjalankan service nginx)
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/start-nginx.png) 
-sudo systemctl status nginx (pengecekan status nginx)
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/status-nginx.png) 
nginx sudah berjalan

4) Seting mod rewrite
sudo vi /etc/nginx/nginx.conf
>user nginx;
>worker_processes auto;
>error_log /var/log/nginx/error.log;
>pid /run/nginx.pid;
>include /usr/share/nginx/modules/*.conf;
>
>events {
>    worker_connections 1024;
>}
>
>http {
>    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
>                      '$status $body_bytes_sent "$http_referer" '
>                      '"$http_user_agent" "$http_x_forwarded_for"';
>
>    access_log  /var/log/nginx/access.log  main;
>
>    sendfile            on;
>    tcp_nopush          on;
>    tcp_nodelay         on;
>    keepalive_timeout   65;
>    types_hash_max_size 4096;
>
>    include             /etc/nginx/mime.types;
>    default_type        application/octet-stream;
>
>    # Load modular configuration files from the /etc/nginx/conf.d directory.
>    # See http://nginx.org/en/docs/ngx_core_module.html#include
>    # for more information.
>    include /etc/nginx/conf.d/*.conf;
>
>    server {
>        listen       80;
>        #listen       [::]:80;
>        server_name  192.168.1.89;
>        root         /usr/share/nginx/html;
>	index	index.html index.html;
>
>        # Load configuration files for the default server block.
>        include /etc/nginx/default.d/*.conf;
>
>	location / {
>        try_files $uri $uri/ =404;
>    	}
>	
>        error_page 404 /404.html;
>        error_page 500 502 503 504 /50x.html;
>        location = /50x.html {
>        }
>
>	location ~ \.php$ {
>        try_files $uri =404;
>	fastcgi_split_path_info ^(.+\.php)(/.+)$;
>        fastcgi_pass unix:/var/run/php/php80-fpm.sock;
>        fastcgi_index index.php;
>        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
>        include fastcgi_params;
>   	 } 
>   	 }
>   	}

5) Reload service nginx (sudo service nginx reload)
<br/>
###Install php-fpm 7.3, 7.4, 8.0 dan membuat folder website dengan phpinfo untuk masing-masing versi php
1) proses install php-fpm
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/install-remi-repo.png) 
- sudo yum -y install yum-utils
- sudo yum-config-manager --disable 'remi-php*'
- sudo yum-config-manager --enable remi-php73
- sudo yum-config-manager --enable remi-php74
- sudo yum-config-manager --enable remi-php80
- sudo yum-config-manager --disable 'remi-php*'
- sudo yum-config-manager --enable remi-php73
- sudo yum-config-manager --enable remi-php74
- sudo yum-config-manager --enable remi-php80
- sudo yum install php php73-fpm php74-fpm php80-fpm -y

2) seting config pada php-fpm yang sudah di install
lakukan backup sebelum edit
- sudo cp /etc/opt/remi/php73/php-fpm.d/www.conf /etc/opt/remi/php73/php-fpm.d/www.conf-bak
- sudo vi /etc/opt/remi/php73/php-fpm.d/www.conf
perubahan yang dilakukan
>listen = 127.0.0.1:9000 menjadi listen = /var/run/php/php73-fpm.sock
>;listen.owner = nobody menjadi listen.owner = nginx
>
>;listen.group = nobody menjadi listen.group = nginx
>
>;listen.mode = 0666 menjadi listen.mode = 0666
>user = apache menjadi user = nginx
>
>group = apache menjadi group = nginx

Kemudian save dan exit.


sebelum enable dan start harus membuat folder <strong>php</strong> di /var/run karena setelah dicek folder tersebut belum ada, setelah itu kalukan perintah
- sudo systemctl enable php73-php-fpm (tujuan nya supaya langsung aktif ketika server booting) setelah itu cek status php73
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/php73.png
) 
selanjutnya lakukan hal sama untuk php74 dan php80 dengan copy config dari php73 dan di modifikasi bagian www.conf.
untuk php74 yang berbeda pada bagian
>listen = 127.0.0.1:9000 menjadi listen = /var/run/php/php74-fpm.sock

untuk php80 yang berbeda pada bagian
>listen = 127.0.0.1:9000 menjadi listen = /var/run/php/php80-fpm.sock

![](https://github.com/batarok/barkah-simulasi-2022/blob/main/php74-1.png) 
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/php74-2.png
) 
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/php80-1.png
) 
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/php80-2.png
) 

3) Membuat folder website dan mengisinya dengan file phpinfo
- membuat folder
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/folder-web.png
)
- membuat file website dan melakukan copy ke folder website lain 
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/file%20website.png
) 
Untuk isi file index.php nya
><?php phpinfo(); ?>

4) melakukan seting nginx website 73 74 & 80
nginx website 73
>location ~ /73 {
>        try_files /73/$uri /73/$uri/ /73/index.php?q=$uri&$args;
>
>		location ~* \.php$ {
>		fastcgi_pass unix:/var/run/php/php73-fpm.sock;
>		include fastcgi_params;
>        	fastcgi_index index.php;
>		fastcgi_split_path_info ^(.+\.php)(/.+)$;
>		fastcgi_param PATH_INFO             $fastcgi_path_info;
>        	fastcgi_param PATH_TRANSLATED       $document_root$fastcgi_path_info;
>        	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
>        	 }
>	
>	}

nginx website 74
>location ~ /74 {
>        try_files /74/$uri /74/$uri/ /74/index.php?q=$uri&$args;
>
>               location ~* \.php$ {
>                fastcgi_pass unix:/var/run/php/php74-fpm.sock;
>                include fastcgi_params;
>                fastcgi_index index.php;
>                fastcgi_split_path_info ^(.+\.php)(/.+)$;
>                fastcgi_param PATH_INFO             $fastcgi_path_info;
>                fastcgi_param PATH_TRANSLATED       $document_root$fastcgi_path_info;
>                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
>                 }
>
>        }

nginx website 80
>location ~ /80 {
>        try_files /80/$uri /80/$uri/ /80/index.php?q=$uri&$args;
>
>                location ~* \.php$ {
>                fastcgi_pass unix:/var/run/php/php80-fpm.sock;
>                include fastcgi_params;
>                fastcgi_index index.php;
>                fastcgi_split_path_info ^(.+\.php)(/.+)$;
>                fastcgi_param PATH_INFO             $fastcgi_path_info;
>                fastcgi_param PATH_TRANSLATED       $document_root$fastcgi_path_info;
>                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
>                 }
>
>        }

untuk konfigurasi tersebut digabungkan dalam file nginx.conf yang ada di /etc/nginx/nginx.conf, jadi keseluruhan file config seperti ini;
>user nginx;
>worker_processes auto;
>error_log /var/log/nginx/error.log;
>pid /run/nginx.pid;
>
>include /usr/share/nginx/modules/*.conf;
>
>events {
>    worker_connections 1024;
>}
>
>http {
>    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
>                      '$status $body_bytes_sent "$http_referer" '
>                      '"$http_user_agent" "$http_x_forwarded_for"';
>
>    access_log  /var/log/nginx/access.log  main;
>
>    sendfile            on;
>    tcp_nopush          on;
>    tcp_nodelay         on;
>    keepalive_timeout   65;
>    types_hash_max_size 4096;
>
>    include             /etc/nginx/mime.types;
>    default_type        application/octet-stream;
>    include /etc/nginx/conf.d/*.conf;
>
>    server {
>        listen       80;
>        #listen       [::]:80;
>        server_name  192.168.1.89;
>        root         /usr/share/nginx/html;
>	index	index.html;
>
>        include /etc/nginx/default.d/*.conf;
>
>	location / {
>        try_files $uri $uri/ =404;
>    	}
>	
>	location ~ /73 {
>        try_files /73/$uri /73/$uri/ /73/index.php?q=$uri&$args;
>
>		location ~* \.php$ {
>		fastcgi_pass unix:/var/run/php/php73-fpm.sock;
>		include fastcgi_params;
>        	fastcgi_index index.php;
>		fastcgi_split_path_info ^(.+\.php)(/.+)$;
>		fastcgi_param PATH_INFO             $fastcgi_path_info;
>        	fastcgi_param PATH_TRANSLATED       $document_root$fastcgi_path_info;
>        	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
>        	 }
>	
>	}
>
>	
>	location ~ /74 {
>        try_files /74/$uri /74/$uri/ /74/index.php?q=$uri&$args;
>
>               location ~* \.php$ {
>                fastcgi_pass unix:/var/run/php/php74-fpm.sock;
>                include fastcgi_params;
>                fastcgi_index index.php;
>                fastcgi_split_path_info ^(.+\.php)(/.+)$;
>                fastcgi_param PATH_INFO             $fastcgi_path_info;
>                fastcgi_param PATH_TRANSLATED       $document_root$fastcgi_path_info;
>                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
>                 }
>
>        }
>
>	location ~ /80 {
>        try_files /80/$uri /80/$uri/ /80/index.php?q=$uri&$args;
>
>                location ~* \.php$ {
>                fastcgi_pass unix:/var/run/php/php80-fpm.sock;
>                include fastcgi_params;
>                fastcgi_index index.php;
>                fastcgi_split_path_info ^(.+\.php)(/.+)$;
>                fastcgi_param PATH_INFO             $fastcgi_path_info;
>                fastcgi_param PATH_TRANSLATED       $document_root$fastcgi_path_info;
>                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
>                 }
>
>        }
>	
>        error_page 404 /404.html;
>        error_page 500 502 503 504 /50x.html;
>        location = /50x.html {
>        }
>
>	location ~ \.php$ {
>        try_files $uri =404;
>        fastcgi_pass unix:/var/run/php/php80-fpm.sock;
>        fastcgi_index index.php;
>        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
>        include fastcgi_params;
>   	 }
>	
>
>    }
>}

Untuk link akses website
- [http://192.168.1.89:869/73](http://192.168.1.89:869/73)
- [http://192.168.1.89:869/74](http://192.168.1.89:869/74) 
- [http://192.168.1.89:869/80](http://192.168.1.89:869/80) 
<br/>
###Seting interface ke 2 eth1
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/eth1.png
) 
di ubah menjadi
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/eth1-1.png
) 
- sudo systemctl restart network
- ifconfig
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/eth1-2.png
) 
- Telnet ke ip 172.16.8.1 port 80
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/telnet-1.png
) 
Telnet berhasil
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/telnet-2.png
) 

-install wget
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/install-wget.png
) 
-download file website http://172.16.8.1/web.tar.gz
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/download-file-web1.png
) 
-ekstrak file web
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/ekstrak-file-web.png
) 
-url website [http://192.168.1.89:869/index.php](http://192.168.1.89:869/index.php)  karena konfigurasi index utama pada nginx masih index.html

###install mysql dan phpmyadmin
1 sudo yum install mysql php-mysql -y
2 install phpmyadmin
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/install-phpmyadmin.png
)
untuk http://192.168.1.89:869/phpMyAdmin belum bisa diakses, untuk solusi menggunakan adminer
-[http://192.168.1.89:869/admin](http://192.168.1.89:869/admin) 
![](https://github.com/batarok/barkah-simulasi-2022/blob/main/adminer.png) 