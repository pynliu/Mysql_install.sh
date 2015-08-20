# Mysql_install.sh
#编译安装mysql5.6

#!/bin/sh
yum -y groupinstall 'Development Tools'
yum -y install gcc gcc-c++ cmake ncurses-devel perl bison libtool-ltdl-devel autoconf zlib* libxml*
cd /mydata/download
wget http://pan.baidu.com/s/1o6y1Qn4

tar zxf mysql-5.6.4-m7.tar.gz
cd mysql-5.6.4-m7

groupadd mysql
useradd -r -g mysql mysql
mv /etc/my.cnf /etc/my.cnf.bak

mkdir -p /mydata/mysql/{log,backup,innodb_data}

# install 
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/mydata/mysql \
-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysqld.sock \
-DSYSCONFDIR=/etc \
-DMYSQL_USER=mysql \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci 

make -j 4
make install

sed -i "s/enforcing/disable/g" /etc/selinux/config
setenforce 0
chown -R mysql:mysql /usr/local/mysql
chown -R mysql:mysql /mydata/mysql
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql

# my.cnf
cat << EOF >> /etc/my.cnf
[mysqld]
basedir                 = /usr/local/mysql
datadir                 = /mydata/mysql
port                    = 3306

max_connections         = 500
max_allowed_packet      = 20M

log-error               = /mydata/mysql/log/error.log
slow_query_log_file     = /mydata/mysql/log/slow_query.log
innodb_buffer_pool_size = 512M
sql_mode = STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,NO_AUTO_VALUE_ON_ZERO
skip-name-resolve

[mysqld_safe]
open-files-limit = 16384
EOF


/usr/local/mysql/scripts/mysql_install_db --user=mysql --datadir=/mydata/mysql --basedir=/usr/local/mysql --defaults-file=/etc/my.cnf
service mysql start
/usr/local/mysql/bin/mysqladmin -u root password 'password'
ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
/usr/bin/mysql -uroot -padmin -e "grant all on *.* to admin@'%' identified by 'password'"
