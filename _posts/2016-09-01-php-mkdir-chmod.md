---
layout: post
title: "PHP mkdir 권한 문제(umask)"
author: "Chester"
---

파일 업로드 관련 코드를 작성하던 도중 다음과 같은 코드가 있었다.

{% highlight php%}
$path = '/home/test/tmp';
if (is_dir($path) === false) {
    mkdir($path, 0777, true);
}
{% endhighlight %}

위 코드에서 보면 만들고자 하는 디렉토리는 /home/test/tmp 라는 디렉토리이고 recursive 옵션을 true 로 주어서 test 폴더와 tmp 폴더가 만들어지는고 test 와 tmp 폴더의 권한은 777 옵션이 적용되어있는 것을 기대 하였다.

하지만 만들어진 디렉토리를 보니 권한이 755 로 잡혀잇는것이다.

{% highlight sh bash %}
ls -al |grep test_dir
drwxr-xr-x   3 root   root    4096 9월  2 11:11 test_dir
{% endhighlight %}

원인을 찾던도중 다음과 같은 원인을 찾을수 있었다.

##### umask
{% highlight markdown %}
umask 란 ? 권한을 설정할 때 수동적으로 권한을 주지 않고 파일이나 디렉토리가 생성됨가 동시에 지정된 권한이 주어주도록 함.

현재 리눅스PC umask 설정값이 0022 로 잡혀있어서 0777 디렉토리를 생성하고자 할때
0777 - 0022 = 0755
다음과 같은 결과값이 나왔던 것이다.
{% endhighlight %}

이를 코드상에서 0777 의 옵션으로 변경하고자 하면 아래와 같이 코드를 수정하면된다.
{% highlight php%}
$oldumask = umask(0);
$path = '/home/test/tmp';
if (is_dir($path) === false) {
    mkdir($path, 0777, true);
}
umask($oldumask);
{% endhighlight %}



[링크](https://stackoverflow.com/questions/3997641/why-cant-php-create-a-directory-with-777-permissions)
