---
layout: post
lang: id
title: Migrasi Mail
---
Ini adalah catatan rekap migrasi mail pada tanggal 11 December 2008, at 09:40 PM.

Pertama, ini merupakan bagian dari catatan pribadi saya, kalau ada yang salah, mohon ditunjukkan kk. masih cupu kk.  

<!-- more -->

Langkah yang harus dilakukan sebelum migrasi (standar) :

1. daftarkan mx si mail server, dalam hal ini, kita akan langsung pakai mail.students.example.com
2. kasih nama fqdn mail.students.example.com
3. format hardisk ke ufs utk home -> jadinya kita pakai NFS (Network File System)

Nah, mari mulai :

### Install FreeBSD

cara standar aja : pilih2, next2, pilih base, ports hostname dst.

```
yes ethernet adapter
dhcp ato statik
no gateway
yes ssh
no publik ftp
no nfs server
no nfs client
yes timezone
no linux compabiliti
no mouse
no browse
```

#### Postfix

install vim dulu (hehe:D)

kalau jam belom disetting, setting dulu di:

    cp /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
    date 0812012030

install postfix:

    cd /usr/ports/mail/postfix
    make install clean

postfix rupanya abis install perlu bikin file,
misalkan namanya `/etc/periodic.conf`
isinya :
```sh 
daily_clean_hoststat_enable="NO"
daily_status_mail_rejects_enable="NO"
daily_status_include_submit_mailq="NO"
daily_submit_queuerun="NO"
ini terjadi karena postfix menggantikan sendmail sbg MTA.
jadi hapus instruski dari dayly-run scrip yg menyingung ke sendmail agar ga bentrok
Configure main.cf di /usr/local/etc/postfix/main.cf
queue_directory = /var/spool/postfix
command_directory = /usr/local/sbin
daemon_directory = /usr/local/libexec/postfix
mail_owner = postfix
myhostname = mail.students.example.com
mydomain = example.com
myorigin = students.example.com
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, students.example.com
unknown_local_recipient_reject_code = 550
mynetworks_style = host
host digunakan hanya untuk server kita sendiri, tidak untuk relay dari server lain
relay_domains = $mydestination
relayhost = [mx.mxrelay.example.com]
alias_maps = hash:/etc/aliases, hash:/usr/local/mailman/data/aliases
alias_database = hash:/etc/aliases
recipient_delimiter = +
home_mailbox = Maildir/
mail_spool_directory = /var/spool/mail
debug_peer_level = 2
debugger_command =
PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin
xxgdb $daemon_directory/$process_name $process_id & sleep 5
sendmail_path = /usr/local/sbin/sendmail
newaliases_path = /usr/local/bin/newaliases
mailq_path = /usr/local/bin/mailq
setgid_group = maildrop
html_directory = no
manpage_directory = /usr/local/man
sample_directory = /usr/local/etc/postfix
readme_directory = no
```

### Install Dovecot

Untuk menginstall dovecot:
```sh
whereis dovecot
cd /urs/ports/mail/dovecot
make install clean
cd /usr/local/etc
cp dovecot-example.conf deovecot.conf
configure dovecot.conf biar bisa pake ldapnya ldapserv.example.com
cp dovecot-ldap-example.conf dovecot-ldap.conf
```

klo muncul error di `var/maillog` tentang `aliases.db`, jangan lupa eksekusi `newaliases` untuk menghindari error `/etc/aliases.db`

    # newaliases

hapus dulu `/etc/aliases.db` (klo ada)
tes imap dulu

```sh
# telnet localhost 143
..
* OK Dovecot ready.
01 login username password
01 OK Logged in.
02 list "Maildir" "**
03 SELECT INBOX
03
04 FETCH 1:* FLAGS
05 FETCH 1 full
06 FETCH 1 body[text]
07 logout
```

untuk uji coba bisa pake `squirellmail` atau `horde` :D.

Tapi terlebih dahulu install apache22.

### Install Apache

    # cd /usr/ports/www/apache22
    # make instal

abis tu edit `/usr/local/etc/apache22/httpd.conf`

isinya (kurang lebih):

```sh
ServerRoot "/usr/local"
Listen 80
LoadModule authn_file_module libexec/apache22/mod_authn_file.so
LoadModule authn_dbm_module libexec/apache22/mod_authn_dbm.so
LoadModule authn_anon_module libexec/apache22/mod_authn_anon.so
#banyak banget modulnya ..
#dst

User www
Group www
ServerAdmin if15113@students.example.com
servername mail.students.example.com:80
DocumentRoot "/path/to/forlder/nya"
#...

Options Indexes FollowSymLinks
AllowOverride None
Order allow,deny
Allow from all
#dst
#...
#end of httpd.conf
```


jgn lupa apachenya harus menambahkan module php 5 ke apache
edit `/etc/rc.conf`, tambahin isi :

    apache22_enable="YES"
    apache22_http_accept_enable="YES"


abis tu idupin apache kitah :D

    /usr/local/etc/rc.d/apache22 start

di freebsd katanya harus lakukan ini

    # kldload accf_http

klo ndak, akan ada http ready error
lalu tambahkan ke `/boot/loader.conf` isinya :

    accf_http_load="YES"

memulai apache:

    # apachectl start | restar | graceful | stop

sebelum mulai apache jgn lupa cek sintak

    # apachectl configtest

abis intall apache,lalu install php5

### Install php5

    # whereis php5
    # cd /usr/ports/lang/php5
    # make config
    # make install

abis tu install tambahan2 :

```sh
cd /usr/ports/databases/php5-mysql -> mysql database
cd /usr/ports/www/php5-session sessions
cd /usr/ports/graphics/php5-gd graphics library lama ini..tah va va banyak yg di installnya
```
semuanya di make install clean :D
abis install php5
edit httpd.conf
masukan index.php ke
DirectoryIndex index.php index.html
di akhir baris tambahakn

    AddType application/x-httpd-php .php
    AddType application/x-httpd-php-source .phps

abis tu kopi dan edit file `php.ini`

    # cd /usr/local/etc
    # cp php.ini-recommended php.ini

edit `/usr/local/etc/php.ini`
lokasi penyimpan session .tambahkan di php.ini
unkomen `session.save_path = "/tmp"`
abis tu restart apache
untuk testing, bikin file di `/usr/local/www/apache22/date/infor.php` yang isinya (terserah sih):

### Install RoundCube

    whereis roundcube
    cd /usr/ports/...
    make install clean
    
abis tu bikin database nya

```sh
CREATE DATABASE roundcube_db;
GRANT ALL PRIVILEGES ON roundcube_db.* TO roundcube_user@localhost IDENTIFIED BY 'roundcube_pass';
FLUSH PRIVILEGES;
```

tambahin di httpd.conf:
```sh
Alias /roundcube "/usr/local/www/roundcube/"
Options Indexes FollowSymLinks
AllowOverride All
Order allow,deny
Allow from all
```

edit dua file di folder config:
tambahin :
```sh
$rcmailconfig['dbdsnw'] = 'mysql://roundcube_user:roundcube_pass@localhost/roundcube_db';
$rcmail_config['default_host'] = array('mail.example.com', 'webmail.example.com', 'ssl://mail.example.com:993');
```
setelah itu, eksekusi mysqlnya:
```sh
# cd /usr/local/www/roundcube/SQL
# mysql -u roundcube_user -p roundcube_db <>
```

install squerel:
```sh
#whereis squirrelmail ato locate squirrelmail
#cd /usr/ports/mail/squirrelmail
#make install clean
```
di edit2
dan di masukan alias ke httpd.conf
```sh
Alias /lmail "/usr/local/www/squirrelmail/"
Options None
AllowOverride None
Order allow,deny
Allow from all
```

restart apache22
abis tu test
mantap ga rek?
eh, tunggu

SQUIREL MAKE LDAP BOI..

You can enable LDAP email address lookups in SquirrelMail if you have a functional LDAP server running.
Ensure you have the php5-ldap shared extension installed using this command:

    # pkg_info | grep php5-ldap
    
If you dont get any results, rebuild SquirrelMail:

    # cd /usr/ports/mail/squirrelmail
    # make deinstall
    # make -D WITH_LDAP install clean
To set up LDAP lookups:
    # cd /usr/local/www/squirrelmail
    # ./configure
    [6] Address Books
    [1] Change LDAP Servers
    [ldap] command (?=help) > +
    hostname: localhost
    base: ou=People,dc=example,dc=com
    port: press [enter]
    charset: press [enter]
    name: LDAP: example.com
    maxrows: press [enter]
    binddn: press [enter]
    protocol: 3
    [ldap] command (?=help) > d
    
install mysql versi 1
```sh
# cd /usr/ports/databases/mysql50-server
# make -D BUILD_OPTIMIZED install clean
# rehash
# mysql_install_db --user=mysql
# mysqld_safe &
# mysqladmin -u root password 'localpassword'
# mysqladmin -u root -h host.example.com password 'remotepassword'
```
di `etc/rc.conf` add `mysql_enable="YES"`

    # /usr/local/etc/rc.d/mysql-server restart

testing:

    # mysqlshow -p

pastikan folder /tmp :

    # chown root:wheel /tmp
    # chmod 777 /tmp
    # chmod =t /tmp

add new user,full akses

    mysql> grant all on database.* to user@localhost identified by 'password';

install mysql versi 2:

```sh
#cd /usr/ports/databases/mysql50-server
#make install clean
#rehash
#mysql_install_db .user=mysql
#chown -R mysql:mysql /var/db/mysql
```

untuk testing, coba jalankan:

    #/usr/local/bin/mysqld_safe -user=mysql &

klo ndak ada eror berarti berhasil :D
masukin `mysql_enable=.YES. ke /etc/rc.conf`
ganti password root nya mysql:

    #mysqladmin -u root password passwordhary
    #/usr/local/bin/mysqladmin -u root -h mail.example.com password passwordhary
passwordhary adalah password yg diinginkan
omedeto :D

### install mailman

konfigurasi mailman
```sh
#cd /usr/ports/mail/mailman
#make MAIL_GID=mailman install clean -> agak lama, banyak warning..kenapa nih?
#vim /usr/local/mailman/Mailman/mm_cfg.py
#tambahkan SMTHOST='mail.students.example.com'
MTA = 'Postfix'
```
edit httpd.conf tambahkan :
```sh
ScriptAlias /mailman "/usr/local/mailman/cgi-bin"
Alias /pipermail "/usr/local/mailman/archives/public"
Options FollowSymLinks ExecCGI
AllowOverride None
Order allow,deny
Allow from all
Options Indexes MultiViews FollowSymLinks
AllowOverride None
Order allow,deny
Allow from all
```
jalanlan `./genaliases` yg ada di bin mailman
pastikan apa yg jadi punya mailman
```sh
#cd /usr/local/mailman/data
#chown mailman aliase*
#chmod g+w alias*
#vi /usr/local/etc/postfix/master.cf
```
tambahkan:
```sh
mailman unix - n n - - pipe
flags=FR user=mailman:mailman
argv=/usr/local/mailman/postfix-to-mailman-2.1.py ${nexthop} ${user}
```
edit /usr/local/etc/postfix/main.cf, tambahkan

    hash:/usr/local/mailman/data/aliases pada bagian alias_maps = /etc/aliases

donlot postfix-to-mailman-2.1.py dari :

    http://www.gurulabs.com/goodies/downloads.php
    http://www.gurulabs.com/downloads/postfix-to-mailman-2.1.py

kemudian letakkan file tsb di /usr/local/mailman/ dan edit

    # vim /usr/local/mailman/postfix-to-mailman-2.1.py

Edit parameter berikut:

    MailmanHome = "/usr/local/mailman"; # Mailman home directory.
    MailmanOwner = "postmaster@mail2.ai.example.com";

Untuk memastikan daftar alias dari Postfix, gunakan perintah-perintah di bawah
ini:

    # /usr/local/sbin/postalias /etc/mail/aliases
    # /usr/local/sbin/postalias /etc/aliases
    # /usr/local/sbin/postalias /usr/local/etc/postfix/aliases

Setelah itu kita reload postfix dan restart apache:

    # postfix reload
    # apachectl restart

Akhirnya kita coba jalankan mailman:

    # /usr/local/etc/rc.d/mailman start

Untuk membuat list pertama kali kita lakukan seperti berikut:

    # cd /usr/local/mailman/
    # bin/newlist mailman
    Enter the email of the person running the list: admin@mail.example.com
    Password:
    # bin/config_list -i data/sitelist.cfg mailman

Kita perlu juga menambahkan maintenance mailman ke dalam cron:

    # cd /usr/local/mailman/cron
    # crontab -u mailman crontab.in
    # cd /usr/local/mailman
    # bin/mailmanctl start

Terakhir kita perlu mengatur password admin untuk mailman sbb:

    # bin/mmsitepass
    Password:
    # bin/mmsitepass -c
    Password:
    # ./withlist -l -r fix_url nama milis --urlhost=mail.students.example.com

jalankan daemon mailman:

    # ./bin/mailmanctl start

periksa permission

    # ./bin/check_perms

untuk memperbaiki permission :

    # ./bin/check_perms -f

masukin ke rc.conf

    maiman_enable=.YES.

exports data milis :

kopi kan data2 yg perlu ke server baru :

    # cd /var/lib/mailman
    # tar -cpvW -f /root/mmdata ./data
    # tar -cpvW -f /home/mmbackup/mmarchives ./archives -> gede filenya boi...
    # tar -cpvW -f /root/mmlists ./lists

di server baru :

    # cd /usr/local/mailman
    # tar -xvf mmdata
    # tar -xvf mmarchives
    # tar -xvf mmlists
    # chown -R www:mailman .

optional keknya `chmod -R a+rx,g+ws `.

jalankan :

    # ./withlist -l -a -r fix_url
    #./genaliases

horeee..

### Install Horde

sekarang kita coba-coba install horde
ikuti semua yg ada di docs/INSTALL
yg jelas harus install php terlebih dahulu

    # cd /usr/ports/mail/pear-Mail_mimeDecode/
    # cd /usr/ports/mail/pear-Mail_Mime
    # cd /usr/ports/sysutils/pear-Log
    # cd /urs/ports/tex./php5-dom (lupa saya, tar di tambahin lagi
    # make install clean :D

install ldap authentication

    # cd /usr/ports/net/nss_ldap
    # cd /usr/ports/security/pam_ldap
    # make install clean

nampaknya ada error, maka waktu itu saya lakukan hal di bawah ini

    # cd /usr/ports/ports-mgmt/portupgrade
    # make install clean

misalkan nss_ldap ga bisa di install gara2 mismacth checksum dan md5

    # cd /usr/ports/net/nss_ldap
    # rm diskinfo
    # portupgrage nss_ldap
    # make install clean

enjoyyyy

edit `nss_ldap.conf`,kurang lebih :

    host 202.202.202.30
    base dc=ldapserv,dc=ab,dc=cde,dc=ac,dc=id
    uri ldap://202.202.202.30:389/
    ldap_version 3
    binddn cn=Manager,dc=ldapserv,dc=ab,dc=cde,dc=ac,dc=id
    bindpw password
    rootbinddn cn=Manager,dc=ab,dc=cde,dc=ac,dc=id
    port 389
    nss_base_passwd ou=Users,dc=ldapserv,dc=ab,dc=cde,dc=ac,dc=id?one

abis tu edit yg mau dipake ldap,di sini kita pake imap ama pop3
tambahkan baris :

    auth sufficient /usr/local/lib/pam_ldap.so no_warn try_first_pass

tambahan-tambahan:

#### install fortune

    #cd /usr/scr/games/fortune
    #make all install

lokasi fortune ada di `/usr/games/fortune`
masukkan ke `.login`

    [ -x /usr/games/fortune ] && /usr/games/fortune -s

software yang perlu diinstall:
`alpine` -> untuk baca mail, merupakan tipe baru dari pine yang telah ditinggalkan
setelah install alpine, bakal muncul error2 ttg sekuriti, sbb:

<pre><code>
The SECURITY PROBLEM came about because the server advertised the
AUTH=PLAIN SASL authentication mechanism outside of a TLS-encrypted session,
in violation of RFC 4616. This message is just a warning,
and in fact occurred after the server had disconnected.
</code></pre>

`zinc` -> untuk chat
dan banyak lagi -> mari kita list
update
dah jalan mailnya :D
rupanya ada error di nfs nya..perlu ditambahin beberapa hal
ehm..
dalam kasus ini, harddisk ada 202.202.202.25 dan server ada di 202.202.202.8
di 202.202.202.25 (kebetulan linux FC3,cmiiw) setting /etc/exports:

    /home 202.202.202.8(rw,sync,no_root_squash)

di 202.202.202.8 setting

    nfs_client_enable=.YES.
    nfs_client_flafs=.-n 4.
    rpc_lockd_enable=.YES.
    rpc_statd_enable=.YES.


sekian dulu ya..
nanti di edit lagih..
insyaAllah
