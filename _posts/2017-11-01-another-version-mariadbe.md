---
layout: post
title: "mysql 을 두개 설치해보자"
author: "Chester"
---

현재 mysql5.5 -> maria10.2 설치하기에 앞서 리눅스 피시에 maria10.2 설치를 해야하는 상황이 왔다.
다른 가상서버 혹은 docker 로 설치진행해도 되지만 옮겨야하는 데이터디렉토리 용량이 800g가 넘기때문에 그냥 리눅스 피시에 설치를 진행하도록 하겠다.

소스코드 설치로 진행한다.

#### 1.컴파일을 하기위한 라이브러리 설치
{% highlight markdown %}
yum install cmake ncurses ncurses-devel bison gcc gcc-c++ -y
{% endhighlight %}
#### 2.source code 를 다운로드한다.
{% highlight markdown %}
cd /usr/local/src
wget -N https://downloads.mariadb.org/interstitial/mariadb-10.2.7/source/mariadb-10.2.7.tar.gz
tar zxvf mariadb-10.2.7.tar.gz
mkdir build
cd build
{% endhighlight %}

#### 3.compile
{% highlight markdown %}
 cmake ../mariadb-10.2.7 -DWITH_READLINE=1 -DWITH_READLINE=1 -DWITH_SSL=bundled -DWITH_ZLIB=system -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DENABLED_LOCAL_INFILE=1 -DWITH_EXTRA_CHARSETS=all -DWITH_ARIA_STORAGE_ENGINE=1 -DWITH_XTRADB_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_FEDERATEDX_STORAGE_ENGINE=1 -DWITH_PERFSCHEMA_STORAGE_ENGINE=1 -DCMAKE_INSTALL_PREFIX=/usr/local/mariadb-10.2.7 -DMYSQL_DATADIR=/disk/mysql10.2.7
{% endhighlight %}

#### 4.setup
{% highlight markdown %}
make ; make install 
{% endhighlight %}
 
#### 5.설정파일 복사
{% highlight markdown %}
cp /usr/local/mariadb-10.2.7/support-files/my-innodb-heavy-4G.cnf /etc/sql.cnf
{% endhighlight %}

#### 6.계정추가는전버전에서  이미 만들었으므로 생략
{% highlight markdown %}
adduser mysql
{% endhighlight %}

#### 7.alias 설정
{% highlight markdown %}
alias maria10.2start='/usr/local/mariadb-10.2.7/bin/mysqld_safe --defaults-file=/etc/sql.cnf &'
alias maria10.2='mysql --defaults-file=/etc/sql.cnf -uroot -p'
{% endhighlight %}
#### 8.바로적용
{% highlight markdown %}
source ~/.bashrc
{% endhighlight %}

#### 참조링크 : [CentOS 64Bit MariaDB 10 소스 컴파일 설치하기](http://88oy.tistory.com/383)

