## **Praktikum Modul 2**

oleh : Axel Gavan & Charisma Ilham Saputra

1. **Rubah LXC landing dengan ubuntu focal (destroy n create, same ip, same name)**

- Pertama kita cek lxc menggunakan

```
 lxc-ls -f
```
![Picture1](https://user-images.githubusercontent.com/93067446/144472039-3493e071-54d2-4cb4-ab36-ded4d039d278.png)


-  lalu kita hapus ubuntu landing menggunakan command dibawah, dan buat ubuntu focal dengan lxc-create

```
 lxc-destroy ubuntu_landing
 
 sudo lxc-create -n ubuntu_landing -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```

![Picture2](https://user-images.githubusercontent.com/93067446/144471909-2df68395-a757-4739-b668-a40e368d5558.png)


- Setelah membuat ubuntu focal, gunakan command dibawah (lxc-start untuk memulai ubuntu_landing, lxc-attach untuk membuka ubuntu_landing, dan apt install nano untuk menginstall nano yang digunakan untuk mengedit config ).

```
 lxc-start ubuntu_landing
 lxc-attach ubuntu_landing
 apt install nano net-tools curl
```

- Masuk ke nano lalu atur IP ubuntu_landing menjadi 10.0.3.103

```
nano /etc/netplan/10-lxc.yaml
netplan apply
```
![Picture3](https://user-images.githubusercontent.com/93067446/144472323-7f2b3ba8-5b41-4c5b-8f5f-465c6df6e372.png)


![Picture4](https://user-images.githubusercontent.com/93067446/144472188-3dea5b9e-a019-4b01-be26-eae96187bc90.png)


- Atur autostart lxc, menggunakan command echo

```
echo "lxc.start.auto = 1" >> /var/lib/lxc/ubuntu_landing/config
lxc-ls -f
```
- install ssh

```
apt-get install openssh-server -y
```

![Picture5](https://user-images.githubusercontent.com/93067446/144472404-f4044f58-18c6-47fa-9b7e-d56302eab30b.png)

- konfigurasi ssh

```
nano /etc/ssh/sshd_config
```

![Picture6](https://user-images.githubusercontent.com/93067446/144472454-92f4d5bc-7f17-4443-be82-3ab6735b1598.png)


- buat password baru

```
service sshd restart
passwd
```

![Picture7](https://user-images.githubusercontent.com/93067446/144472517-8dd93610-88db-4e20-823b-47839a938437.png)


- cek ssh

```
ssh root@10.0.3.103
```

![Picture8](https://user-images.githubusercontent.com/93067446/144472562-04467811-d490-49d6-8cb7-95494da0a770.png)


2. **Rubah LXC php7 dengan ubuntu focal (destroy n create, same ip, same name)**

- menghapus container ubuntu php 7.4

```
lxc-destroy ubuntu_php7.4 -f
```

![Picture9](https://user-images.githubusercontent.com/93067446/144472625-757a36a8-adf3-4392-b3be-16e03e5ff1fe.png)


- buat container lxc ubuntu versi focal

```
sudo lxc-create -n ubuntu_php7.4 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```

![Picture10](https://user-images.githubusercontent.com/93067446/144472749-9a5fa4b1-f918-4f5d-9d1d-89521b5b1042.png)


- Setelah membuat ubuntu focal, gunakan command dibawah (lxc-start untuk memulai ubuntu_landing, lxc-attach untuk membuka ubuntu_landing, dan apt install nano untuk menginstall nano yang digunakan untuk mengedit config ).

```
 lxc-start ubuntu_landing
 lxc-attach ubuntu_landing
 apt install nano net-tools curl
```

![Picture11](https://user-images.githubusercontent.com/93067446/144472804-669e5ecb-dab8-40ae-9ee2-064f65f0d8f9.png)


- konfigurasi menjadi IP statik

```
nano /etc/netplan/10-lxc.yaml
```

![Picture12](https://user-images.githubusercontent.com/93067446/144472862-cb3e7403-52e1-4fb1-8db9-f31c6947ec17.png)


```
netplan apply
```

![Picture13](https://user-images.githubusercontent.com/93067446/144472903-a6b0d522-5f20-434d-9c4e-3519b050c6f4.png)


- install ssh

```
lxc-start ubuntu_php7.4
lxc-attach ubuntu_php7.4
apt-get install openssh-server -y
```

![Picture14](https://user-images.githubusercontent.com/93067446/144472945-bafcd4c3-c674-4779-81e0-bd0427773e2c.png)


- konfigurasi ssh

```
nano /etc/ssh/sshd_config
```

![Picture15](https://user-images.githubusercontent.com/93067446/144472972-194554c5-f1da-498f-9206-536610bd501a.png)


- buat password baru

```
service sshd restart
passwd
```

![Picture16](https://user-images.githubusercontent.com/93067446/144473005-61334f06-a97e-4a71-bba1-00470cab0723.png)


- cek ssh

```
ssh root@10.0.3.103
```

![Picture17](https://user-images.githubusercontent.com/93067446/144473028-8807c2fc-d569-4d7e-85c0-aa2519fc6aab.png)


3. **vm.local/**

- masuk ansible untuk install laravel

```
cd ~/ansible/
mkdir laravel/
cd laravel/
```

![Picture18](https://user-images.githubusercontent.com/93067446/144474272-83eca012-81ef-4e44-8bca-999c75967ffb.png)


- membuat hosts untuk lxc

```
nano hosts
ubuntu_landing ansible_host=lxc_landing.dev ansible_ssh_user=root ansible_become_pass=1234
```

![Picture19](https://user-images.githubusercontent.com/93067446/144473212-56f4f7ae-1bd7-4ce8-afac-f27c93dad3fb.png)


- buat direktori yang akan dijalankan pada folder php dam install di nginxphp.yml

```
---
- hosts: all
  become : yes
  tasks:
    - name: install nginx nginx extras
      apt:
       pkg:
         - nginx
         - nginx-extras
       state: latest
    - name: start nginx
      service:
       name: nginx
       state: started
    - name: menginstall tools
      apt:
       pkg:
         - curl
         - software-properties-common
         - unzip
       state: latest
         - name: "Repo PHP 7.4"
       apt_repository:
         repo="ppa:ondrej/php"
    - name: "Updating the repo"
      apt: update_cache=yes
    - name: Installation PHP 7.4
      apt: name=php7.4 state=present
    - name: install php untuk laravel
      apt:
       pkg:
          - php7.4-fpm
              - php7.4-mysql
              - php7.4-mbstring
              - php7.4-xml
              - php7.4-bcmath
              - php7.4-json
              - php7.4-zip
              - php7.4-common
                   state: present
```

![Picture20](https://user-images.githubusercontent.com/93067446/144474351-7ad43adf-30ef-40a9-9ade-b8461b1e1ac8.png)


- instalasi

```
ansible-playbook -y hosts nginxphp.yml -k
```

![Picture21](https://user-images.githubusercontent.com/93067446/144473305-87f01120-8d74-4ffc-8ce2-2b2ad70cb59a.png)


- buat folder installcomposer.yml

```
---
-hosts: all
  become : yes
  tasks:
   - name: Download and install Composer
     shell: curl -sS https://getcomposer.org/installer | php
     args:
      chdir: /usr/src/
      creates: /usr/local/bin/composer
      warn: false
   - name: Add Composer to global path
     copy:
      dest: /usr/local/bin/composer
      group: root
      mode: '0755'
      owner: root
      src: /usr/src/composer.phar
      remote_src: yes
   - name: Composer create project
     become_user: root
     composer:
      command: create-project
      arguments: laravel/laravel landing 
      working_dir: /var/www/html
      prefer_dist: yes
     environment:
        COMPOSER_NO_INTERACTION: "1"
   - name: mengkopi file .env.example jadi .env
     copy:
      dest: /var/www/html/landing/.env.example
      src: /var/www/html/landing/.env
      remote_src: yes
   - name: mengganti konfigurasi .env
     lineinfile:
      path: /var/www/html/landing/.env
      regexp: "{{ item.regexp }}"
      line: "{{ item.line }}"
      backrefs: yes
     loop:
      - { regexp: '^(.*)DB_HOST(.*)$', line: 'DB_HOST=10.0.3.200' }
      - { regexp: '^(.*)DB_DATABASE(.*)$', line: 'DB_DATABASE=landing' }
      - { regexp: '^(.*)DB_USERNAME(.*)$', line: 'DB_USERNAME=arafah' }
      - { regexp: '^(.*)DB_PASSWORD(.*)$', line: 'DB_PASSWORD=1234 ' }
      - { regexp: '^(.*)APP_URL(.*)$', line: 'APP_URL=http://vm.local' }
      - { regexp: '^(.*)APP_NAME=(.*)$', line: 'APP_NAME=landing' }
   - name: Composer install ke landing
     composer:
       command: install
       working_dir: /var/www/html/landing
     environment:
       COMPOSER_NO_INTERACTION: "1"
   - name: generate php artisan
     args:
      chdir: /var/www/html/landing
     shell: php artisan key:generate
   - name: mengganti permission storage
     file:
      path: /var/www/html/landing/storage
      mode: 0777
      recurse: yes
```

![Picture22](https://user-images.githubusercontent.com/93067446/144474401-463dff3c-cdb2-4bdc-9458-a6ebeae1ef41.png)


- instalasi

```
ansible-playbook -y hosts installcomposer.yml -k
```

![Picture23](https://user-images.githubusercontent.com/93067446/144473377-91141ff7-3c8f-4c2d-b9f1-829d37c86b10.png)


- mengatur lxc_landing.dev

```
server {
        listen 80;

        root /var/www/html/landing/public;
        index index.php index.html index.htm;
        server_name lxc_landing.dev;
    
        error_log /var/log/nginx/landing_error.log;
        access_log /var/log/nginx/landing_access.log;
    
        client_max_body_size 100M;
        location / {
                try_files $uri $uri/ /index.php$args;
        }
        location ~\.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:run/php/php7.4-fpm.sock;
                fastcgi_param SCRIPTFILENAME $document_root$fastcgi_script_name;
        }
}
```

![Picture24](https://user-images.githubusercontent.com/93067446/144473428-2b27dae6-84dc-4a86-8fed-fb2c0fe94c3a.png)


- buat folder config.yml

```
---
- hosts: all
  become : yes
  vars:
    domain: 'lxc_landing.dev'
  tasks:
   - name: stop apache2
     service:
      name: apache2
      state: stopped
      enabled: no
   - name: Write {{ domain }} to /etc/hosts
     lineinfile:
      dest: /etc/hosts
      regexp: '.*{{ domain }}$'
      line: "127.0.0.1 {{ domain }}"
      state: present
   - name: ensure nginx is at the latest version
     apt: name=nginx state=latest
   - name: start nginx
     service:
      name: nginx
      state: started
   - name: copy the nginx config file 
     copy:
      src: ~/ansible/laravel/lxc_landing.dev
      dest: /etc/nginx/sites-available/lxc_landing.dev
   - name: Symlink lxc_landing.dev
     command: ln -sfn /etc/nginx/sites-available/lxc_landing.dev /etc/nginx/sites-enabled/lxc_landing.dev
     args:
      warn: false
   - name: restart nginx
     service:
      name: nginx
      state: restarted
   - name: restart php7
     service:
      name: php7.4-fpm
      state: restarted
   - name: curl web
     command: curl -i http://lxc_landing.dev
     args:
      warn: false
```

![Picture25](https://user-images.githubusercontent.com/93067446/144473474-29667818-97c1-4eee-b28c-3959ca565a8c.png)


- instalasi

```
ansible-playbook -y hosts config.yml -k
```

![Picture26](https://user-images.githubusercontent.com/93067446/144473494-6f2ec918-6e70-408a-b976-179244da846c.png)


- buka vm.local untuk mengecek keberhasilan

![Picture27](https://user-images.githubusercontent.com/93067446/144473521-57679036-df6c-44e6-b278-92d003be8ef8.png)


4. **vm.local/blog**

- masuk ansible untuk install wordpress

```
cd ~/ansible/
mkdir wordoress/
cd wordpress/
```

![1](https://user-images.githubusercontent.com/93067446/144474487-0ad2f01d-d982-485e-802d-8b7e5b371f28.png)


- membuat hosts untuk lxc

```
nano hosts
ubuntu_php7.4 ansible_host=lxc_php7.dev ansible_ssh_user=root ansible_become_pass=1234
```

![Picture28](https://user-images.githubusercontent.com/93067446/144473587-c52d2678-5bf8-481f-b6c8-51d11013b377.png)


- buat direktori untuk tasks

```
---
- hosts: all
  vars:
    domain: 'lxc_php7.dev'
  tasks:
   - name: delete apt chache
     become: yes
     become_user: root
     become_method: su
     command: rm -vf /var/lib/apt/lists/*

   - name: install requirement
     become: yes
     become_user: root
     become_method: su
     apt: name={{ item }} state=latest update_cache=true
     with_items:
      - nginx
      - nginx-extras
      - curl
      - wget
      - php7.4
      - php7.4-fpm
      - php7.4-curl
      - php7.4-xml
      - php7.4-gd
      - php7.4-opcache
      - php7.4-mbstring
      - php7.4-zip
      - php7.4-json
      - php7.4-cli
      - php7.4-mysqlnd
      - php7.4-xmlrpc
      - php7.4-curl

   - name: wget wordpress
     shell: wget -c http://wordpress.org/latest.tar.gz

   - name: tar latest.tar.gz
     shell: tar -xvzf latest.tar.gz

   - name: copy folder wordpress
     shell: cp -R wordpress /var/www/html/blog

   - name: chmod
     become: yes
     become_user: root
     become_method: su
     command: chmod 775 -R /var/www/html/blog/

   - name: copy .wp-config.conf
     copy:
      src=~/ansible/wordpress/wp.conf
      dest=/var/www/html/blog/wp-config.php

   - name: copy wordpress.conf
     copy:
      src=~/ansible/wordpress/wordpress.conf
      dest=/etc/nginx/sites-available/{{ domain }}
     vars:
      servername: '{{ domain }}'

   - name: Symlink wordpress.conf
     command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
  
   - name: restart nginx
     become: yes
     become_user: root
     become_method: su
     action: service name=nginx state=restarted

   - name: Write {{ domain }} to /etc/hosts
     lineinfile:
      dest: /etc/hosts
      regexp: '.*{{ domain }}$'
      line: "127.0.0.1 {{ domain }}"
      state: present

   - name: enable module php mbstring
     command: phpenmod mbstring

   - name: restart php
     become: yes
     become_user: root
     become_method: su
     action: service name=php7.4-fpm state=restarted

   - name: restart nginx
     become: yes
     become_user: root
     become_method: su
     action: service name=nginx state=restarted
```

![Picture29](https://user-images.githubusercontent.com/93067446/144473636-85cc96e8-ad7d-44b1-adc4-2aa828bc94f7.png)


- masuk pada template wp.conf

```
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/support/article/editing-wp-config-php/
 *
 * @package WordPress
 */

define( 'WP_HOME', 'http://vm.local/blog' );
define( 'WP_SITEURL', 'http://vm.local/blog' );

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'blog' );

/** MySQL database username */
define( 'DB_USER', 'admin' );

/** MySQL database password */
define( 'DB_PASSWORD', 'SysAdminSas0102' );

/** MySQL hostname */
define( 'DB_HOST', '10.0.3.200:3306' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
 define( 'AUTH_KEY',         'put your unique phrase here' );
 define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
 define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
 define( 'NONCE_KEY',        'put your unique phrase here' );
 define( 'AUTH_SALT',        'put your unique phrase here' );
 define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
 define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
 define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/

/**
 * WordPress database table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
 $table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
 define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */



/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

![Picture30](https://user-images.githubusercontent.com/93067446/144473739-337522ae-9054-4789-971e-85086555c4f5.png)


- masuk template wordpress.conf

```
server {
     listen 80;
     listen [::]:80;

     # Log files for Debugging
     access_log /var/log/nginx/wordpress-access.log;
     error_log /var/log/nginx/wordpress-error.log;
    
     # Webroot Directory for Laravel project
     root /var/www/html/blog;
     index index.php index.html index.htm;
    
     # Your  Name
     server_name lxc_php7.dev;
    
     location / {
             try_files $uri $uri/ /index.php?$query_string;
     }
    
     # PHP-FPM Configuration Nginx
     location ~ \.php$ {
             try_files $uri =404;
             fastcgi_split_path_info ^(.+\.php)(/.+)$;
             fastcgi_pass unix:/run/php/php7.4-fpm.sock;
             fastcgi_index index.php;
             fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
             include fastcgi_params;
     }
}
```

![Picture31](https://user-images.githubusercontent.com/93067446/144473769-c346e0de-a46c-4d8e-8ae7-656f3f9a1920.png)


- jalankan ansible untuk menginstall

```
ansible-playbook -i hosts installwordpress.yml -k
```

![Picture32](https://user-images.githubusercontent.com/93067446/144473800-1431320c-61d9-4baf-aad8-e27333b297e3.png)


- buka vm.local/blog/ untuk mengecek

![Picture33](https://user-images.githubusercontent.com/93067446/144473818-dd473add-c118-40c4-9c7b-e862c3f3ba36.png)


##### Soal tambahan

1. Laravel

- masuk pada konfigurasi file lxc_landing

```
cd ~/ansible/laravel/
nano lxc_landing.dev
```

![Picture34](https://user-images.githubusercontent.com/93067446/144473852-eff50f9a-86dc-4a1c-bf8e-52d1db91cf90.png)


- ubah menjadi seperti gambar dibawah


![Picture35](https://user-images.githubusercontent.com/93067446/144473870-e960278f-12e9-4de9-8d94-8344889b3a41.png)


- buat ansible dengan nama "soal2.yml"

```
nano soal2.yml
```

![Picture36](https://user-images.githubusercontent.com/93067446/144473944-9e0dfef6-6c03-4e29-a685-8b01251a4b8c.png)


- jalankan ansible "tambahan.yml"

```
---
- hosts: all
  become : yes
  tasks:
   - name: mengganti php sock
     lineinfile:
      path: /etc/php/7.4/fpm/pool.d/www.conf
      regexp: '^(.*)listen =(.*)$'
      line: 'listen = 127.0.0.1:9001'
      backrefs: yes
   - name: copy the nginx config file 
     copy:
      src: ~/ansible/laravel/lxc_landing.dev
      dest: /etc/nginx/sites-available/lxc_landing.dev
   - name: Symlink lxc_landing.dev
     command: ln -sfn /etc/nginx/sites-available/lxc_landing.dev /etc/nginx/sites-enabled/lxc_landing.dev
     args:
      warn: false
   - name: restart nginx
     service:
      name: nginx
      state: restarted
   - name: restart php7
     service:
      name: php7.4-fpm
      state: restarted
   - name: curl web
     command: curl -i http://lxc_landing.dev
     args:
      warn: false
```

![Picture37](https://user-images.githubusercontent.com/93067446/144473987-753abcba-15e7-4f1e-a552-d23911e96b76.png)

![Picture38](https://user-images.githubusercontent.com/93067446/144474016-1dd1ab13-3e0e-4f92-96db-6812f9fd0f26.png)

