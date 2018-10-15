---
layout: post
title: "node를 사용한 초단위 cron 설정을 진행해보자"
author: "hoyeonUm"
---

#### 환경체크
{% highlight shell bash%}
#노드와 npm 설치 진행
node -v
v8.9.3

npm -v
6.4.0
{% endhighlight %}

#### crontab 설치 
{% highlight shell bash %}
cd /home/crontab
#https://www.npmjs.com/package/cron
npm install cron
{% endhighlight %}

#### 아래코드는 npm 홈페이지에 있는 샘플코드다.
{% highlight js %}
vim test.js

var CronJob = require('cron').CronJob;
new CronJob('* * * * * *', function() {
  console.log('You will see this message every second');
}, null, true, 'America/Los_Angeles');

node test.js

You will see this message every second
You will see this message every second
You will see this message every second
....

{% endhighlight %}
#### npm 링크를타고 들어가보면 다음과 같이 설정 영역이 되어있다.
- Seconds: 0-59
- Minutes: 0-59
- Hours: 0-23
- Day of Month: 1-31
- Months: 0-11 (Jan-Dec)
- Day of Week: 0-6 (Sun-Sat)

###### -리눅스 cron 설정보다 * 이 한개더많다. 앞에는 optional 로 초단위 설정은 줘도 되고 안줘도 된다.

#### 이제 해당 크론탭 데몬이 죽었을때 다시 살려줄수있는 데몬을 활용해보자.
{% highlight shell bash%}
#https://www.npmjs.com/package/forever
#전체적으로 유용하게 쓰일것같아 글로벌인스톨 설치하였다.
npm install forever -g

forever -l /home/logs/forever/test.log -a start test.js

cat /home/logs/forever/test.log
You will see this message every second
You will see this message every second
You will see this message every second
{% endhighlight %} 
###### 다음은 내가자주 쓰는 옵션이다.
- -l 로그파일 위치
- -a 로그파일 append
- list (forever list) 현재목록
- stop index (ex: stop 0) list 로 조회하여 해당 index 를 종료시킴
- stopall 전체 종료
- start 실행
- restartall 전체 재실행
- restart index (ex : restart 0) list 로 조회하여 해당 index 를 재시작시킴
- -c (ex forever start -c "php" test.php (default 값은 node 이다)
