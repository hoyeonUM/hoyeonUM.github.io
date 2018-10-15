---
layout: post
title: "github 블로그를 jekyll 를 통해 관리해보자"
author: "hoyeonUm"
---

### 환경 centos7

#### 1.필수패키지 설치
{% highlight shell bash %}
yum install gcc-c++ patch readline readline-devel zlib zlib-devel
yum install libyaml-devel libffi-devel openssl-devel make
yum install bzip2 autoconf automake libtool bison iconv-devel sqlite-devel
{% endhighlight %}

#### 2.RVM 설치
{% highlight shell bash %}
curl -sSL https://rvm.io/mpapis.asc | gpg --import -
curl -L get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh
rvm reload
{% endhighlight %}

#### 3.Verify Dependencies
{% highlight shell bash %}
rvm requirements run
{% endhighlight %}

#### 4.ruby 2.2 설치
{% highlight shell bash %}
rvm install ruby-2.2.5
{% endhighlight %}

#### 5.기본 루비버전 설정
{% highlight shell bash %}
rvm use 2.2.5 --default
{% endhighlight %}

#### 6.루비 버전체크
{% highlight shell bash %}
ruby --version

ruby 2.2.5p319 (2016-04-26 revision 54774) [x86_64-linux]
{% endhighlight %}

#### 7.파이썬, nodejs 설치
{% highlight shell bash %}
yum install -y python nodejs
{% endhighlight %}

#### 8.jekyll 설치및 tale.git clone
{% highlight shell bash %}
gem install jekyll
gem install bundler
cd /home
git clone https://github.com/chesterhow/tale.git
{% endhighlight %}

#### 9.추가플러그인다운로드
{% highlight shell bash %}
cd /home/tale
gem install jekyll-paginate
{% endhighlight %}

#### 10.실행
{% highlight shell bash %}
jekyll serve -w -H 1.123.123.123


Configuration file: /home/tale/_config.yml
            Source: /home/tale
       Destination: /home/tale/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.767 seconds.
 Auto-regeneration: enabled for '/home/tale'
    Server address: http://1.123.123.123:4000/tale/
  Server running... press ctrl-c to stop.


여기서 -w 옵션을 주면 
/home/tale/_posts/*.md 파일이 변경될때마다 바로바로 웹페이지에 적용되는것을 확인할수있다.
{% endhighlight %}







