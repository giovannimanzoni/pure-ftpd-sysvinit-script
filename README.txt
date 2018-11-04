REMOVE SYSTEMD
=====================
http://without-systemd.org/wiki/index.php/How_to_remove_systemd_from_a_Debian_Stretch_installation
-> Using sysvinit instead of systemd in Debian Stretch
-> Optional steps


APT
==================
apt update && apt upgrade && apt install pkg-config gcc build-essential libmariadb-client-lgpl-dev libssl-dev psmisc


LIBSODIUM
==================
cd /root
git clone https://github.com/jedisct1/libsodium --branch stable
cd libsodium/
./configure
make && make check
make install

#tip for libsodium
echo $(pkg-config --cflags libsodium) -> -I/usr/local/include
echo $(pkg-config --libs libsodium) -> -L/usr/local/lib -lsodium



PRO-FTPD
================
echo "/bin/false" >> /etc/shells   # /bin/false or what you want, MUST exist in /etc/shells
groupadd ftpgroup
useradd -u 2001 -d /dev/null -s /bin/false -c "pureftpd user" -g www-data ftpuser
wget ftp://ftp.pureftpd.org/pub/pure-ftpd/releases/pure-ftpd-1.0.47.tar.gz
tar -zxvf pure-ftpd-1.0.47.tar.gz
rm pure-ftpd-1.0.47.tar.gz 
mkdir -p /etc/ssl/pure-ftpd/private # if you want use certificate for connection to ftp server
cd pure-ftpd-1.0.47/


./configure --with-language=italian --with-ftpwho  --with-mysql= --with-paranoidmsg --with-ratios --with-boring --with-bonjour --with-throttling --with-rfc2640 --with-sysquotas --with-tls --with-certfile=/etc/ssl/pure-ftpd/private/pure-ftpd.pem --without-iplogging --without-shadow --without-usernames --prefix=/usr/local/ --sysconfdir=/etc CFLAGS="-I/usr/local/include -O2 -fomit-frame-pointer -fgcse -Os" LDFLAGS="-L/usr/local/lib -lsodium"

make install-strip


CREATE USER 'pureftpd'@'FTP_SERVER_IP' IDENTIFIED BY 'YOUR_PASSWORD';
CREATE DATABASE pureftpd;
GRANT SELECT ON pureftpd.* TO 'pureftpd'@'FTP_SERVER_IP' IDENTIFIED BY 'YOUR_PASSWORD';
FLUSH PRIVILEGES;
USE pureftpd;

CREATE TABLE ftpd (
User varchar(255) NOT NULL default '',
status enum('0','1') NOT NULL default '0',
Password varchar(255) NOT NULL default '',
Dir varchar(255) NOT NULL default '',
ULBandwidth int(6) NOT NULL default '0',
DLBandwidth int(6) NOT NULL default '0',
comment tinytext NOT NULL,
ipaccess varchar(15) NOT NULL default '*',
QuotaSize int(7) NOT NULL default '0',
QuotaFiles int(11) NOT NULL default 0,
PRIMARY KEY (User),
UNIQUE KEY User (User)
) ENGINE=InnoDB;
exit;



INSERT QUERY ACCESS for virtual user
=========================================
#connect to your db and insert yopur query. For example one hash for the password H3ll0Th1s1sMyP4ssw0rd:
SCRYPT: $7$C6..../....7u9edvfXJ9dXqnWqDbsYMf8cIAIAwWcUOFHdkxM/ZS6$Tiv9SJlv7CiZ5OON4v7hAQOn400fSZQ9L.PscyEFVJC

#DO NOT SET 65536 KiB of memory for Argon2, pure-ftp can not handle it
ARGON2I: $argon2i$v=19$m=32768,t=4,p=1$c29tZXNhbHQ$v9etopt7JhNz9K0XNBdEwkiUQXtHx5qQrCv1N7zT5SM

so

USE pureftpd; 
INSERT INTO `ftpd` (`User`, `status`, `Password`, `Dir`, `ULBandwidth`, `DLBandwidth`, `comment`, `ipaccess`, `QuotaSize`, `QuotaFiles`) VALUES ('adminuser', '1', '$argon2i$v=19$m=32768,t=4,p=1$c29tZXNhbHQ$v9etopt7JhNz9K0XNBdEwkiUQXtHx5qQrCv1N7zT5SM', '/var/www/', '50000', '50000', '', '*', '100000', '0'); 
exit;



FOR DEBUG
===============
ps aux | grep ftp
netstat -tnulp | grep pure-ftpd
tail -n 100 /var/log/syslog
cat /var/log/messages
/usr/local/sbin/pure-ftpd /etc/pure-ftpd.conf &

