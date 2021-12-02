## **Praktikum Modul 2**

oleh : Axel Gavan & Charisma Ilham Saputra

1. **Rubah LXC landing dengan ubuntu focal (destroy n create, same ip, same name)**

- Pertama kita cek lxc menggunakan

```
 lxc-ls -f
```

-  lalu kita hapus ubuntu landing menggunakan command dibawah, dan buat ubuntu focal dengan lxc-create

```
 lxc-destroy ubuntu_landing
 
 sudo lxc-create -n ubuntu_landing -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```

gambar

- Setelah membuat ubuntu focal, gunakan command dibawah (lxc-start untuk memulai ubuntu_landing, lxc-attach untuk membuka ubuntu_landing, dan apt install nano untuk menginstall nano yang digunakan untuk mengedit config ).

```
 lxc-start ubuntu_landing
 lxc-attach ubuntu_landing
 apt install nano net-tools curl
```

 gambar



- Masuk ke nano lalu atur IP ubuntu_landing menjadi 10.0.3.103

```
nano /etc/netplan/10-lxc.yaml
netplan apply
```

gambar



- Atur autostart lxc, menggunakan command echo

```
echo "lxc.start.auto = 1" >> /var/lib/lxc/ubuntu_landing/config
lxc-ls -f
```

gambar

- install ssh

```
apt-get install openssh-server -y
```

gambar

- konfigurasi ssh

```
nano /etc/ssh/sshd_config
```

gambar

- buat password baru

```
service sshd restart
passwd
```

gambar

- cek ssh

```
ssh root@10.0.3.103
```

gambar

2. **Rubah LXC php7 dengan ubuntu focal (destroy n create, same ip, same name)**

- menghapus container ubuntu php 7.4

```
lxc-destroy ubuntu_php7.4 -f
```

gambar

- buat container lxc ubuntu versi focal

```
lxc-create -n ubuntu_php7.4 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```

gambar

- Setelah membuat ubuntu focal, gunakan command dibawah (lxc-start untuk memulai ubuntu_landing, lxc-attach untuk membuka ubuntu_landing, dan apt install nano untuk menginstall nano yang digunakan untuk mengedit config ).

```
 lxc-start ubuntu_landing
 lxc-attach ubuntu_landing
 apt install nano net-tools curl
```

gambar

- konfigurasi menjadi IP statik

```
nano /etc/netplan/10-lxc.yaml
```

gambar

```
netplan apply
```

gambar

- install ssh

```
lxc-start ubuntu_php7.4
lxc-attach ubuntu_php7.4
apt-get install openssh-server -y
```

gambar

- konfigurasi ssh

```
nano /etc/ssh/sshd_config
```

gambar

- buat password baru

```
service sshd restart
passwd
```

gambar

- cek ssh

```
ssh root@10.0.3.103
```

gambar

3. **vm.local/**

- masuk ansible untuk install laravel

```
cd ~/ansible/
mkdir laravel/
cd laravel/
```

gambar

- membuat hosts untuk lxc

```
nano hosts
ubuntu_landing ansible_host=lxc_landing.dev ansible_ssh_user=root ansible_become_pass=1234
```

gambar

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

gambar

- instalasi

```
ansible-playbook -y hosts nginxphp.yml -k
```

gambar

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

gambar

- instalasi

```
ansible-playbook -y hosts installcomposer.yml -k
```

gambar

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

gambar

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

gambar

- instalasi

```
ansible-playbook -y hosts config.yml -k
```

gambar

- buka vm.local untuk mengecek keberhasilan

gambar

4. **vm.local/blog**

- masuk ansible untuk install wordpress

```
cd ~/ansible/
mkdir wordoress/
cd wordpress/
```

gambar

- membuat hosts untuk lxc

```
nano hosts
ubuntu_php7.4 ansible_host=lxc_php7.dev ansible_ssh_user=root ansible_become_pass=1234
```

gambar

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

gambar

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

gambar

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

gambar

- jalankan ansible untuk menginstall

```
ansible-playbook -i hosts installwordpress.yml -k
```

gambar

- buka vm.local/blog/ untuk mengecek

gambar

##### Soal tambahan

1. Laravel

- masuk pada konfigurasi file lxc_landing

```
cd ~/ansible/laravel/
nano lxc_landing.dev
```

gambar

- ubah menjadi seperti gambar dibawah

gambar

- buat ansible dengan nama "soal2.yml"

```
nano soal2.yml
```

gambar

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

gambar

