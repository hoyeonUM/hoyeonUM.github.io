---
layout: post
title: "tcpdump 와 wireshark 를 이용해 디버깅 하기"
author: "hoyeonUm"
---

서버를 운영하다보면 A 서버에서 B 서버로 API 를 호출하는데 기대값이 아닌 엉뚱한 값이 나오는 경우가 있다.
보통 이경우에는 A 서버의 application layer 에서 input 값을 변환하기 때문에 실제로 B 서버에 요청하는 값은 달라질수가 있다.
이럴때 디버깅을 하기위해서는 디버깅코드를 심어서 배포를 하거나 로그코드를 심어서 배포를 하는경우가 있다.
이럴때 유용하게 사용할수있는 tcpdump 와 wireshark 를 이용해보자.

`단 ssl 적용시 내용 확인 불가하다.`

##### tcpdump 설치
{% highlight sh bash%}
[root@localhost ~]# yum install tcpdump
{% endhighlight %}

##### 노드서버 디렉토리생성 및 세팅
{% highlight js%}
[root@localhost ~]# mkdir /home/node
[root@localhost ~]# cd /home/node
[root@localhost ~]# vim ./A_server.js

let http = require('http');
let url = require('url');

function randomIntFromInterval(min,max)
{
    return Math.floor(Math.random()*(max-min+1)+min);
}

http.createServer(function (req, res) {
  let urlObj = url.parse(req.url, true, false);
  //가공
  if(typeof urlObj.query.param === 'undefined') {
    let error = {
      code : 422,
      message : 'require \'param\' in query string'
    }
    res.write(JSON.stringify(error));
    res.end();
    return;
  }
  let paramLen = urlObj.query.param.length-1;
  let data = urlObj.query.param.substring(randomIntFromInterval(1, paramLen));
  http.get('http://127.0.0.1:8081/?data='+data, (resp) => {
    let data = '';
    resp.on('data', (chunk) => {
      data += chunk;
    });
    resp.on('end', () => {
      res.write(data);
      res.end();
    });
  });
}).listen(8080);


[root@localhost ~]# vim ./B_server.js

let http = require('http');
let url = require('url');


http.createServer(async function (req, res) {
  let urlObj = url.parse(req.url, true, false);
  //가공
  if(typeof urlObj.query.data === 'undefined') {
    let error = {
      code : 422,
      message : 'require \'data\' in query string'
    }
    res.write(JSON.stringify(error));
    res.end();
    return;
  }

  let dataLen = urlObj.query.data.length;
  if (dataLen < 4) {
    let error = {
      code : 422,
      message : 'error'
    }
    res.write(JSON.stringify(error));
    res.end();
    return;
  }

  let response = {
    code : 200,
    message : 'success'
  };

  res.write(JSON.stringify(response));
  res.end();
}).listen(8081);
{% endhighlight %}

위와 같이 서버 2개를 임의로 만든후 호출을 진행하면 아래 두가지의 랜덤 메세지를 받을수있다.
하지만 코드에는 디버깅 할수있는 코드가 없어서 어떨때 에러가 나는지 확인하기가 어렵다.
이럴때 tcpdump 를 사용해보자

##### sample
{% highlight js%}
curl -k http://127.0.0.1:8080/?param=sendParam

{
code: 422,
message: "error"
}

{
code: 200,
message: "success"
}

{% endhighlight %}

아래는 노드서버 두대를 실행시키고 tcpdump 로 패킷캡처를 진행한다음에 윈도우 피시로 가저오는 샘플이다.


{% highlight sh bash %}
#서버를 띄운다.
[root@localhost node]#node A_server.js
[root@localhost node]#node B_server.js

#tcpdump 실행
#로컬호스트에서 로컬호스트로 가는경우에는 이더넷인터페이스 를 lo 로 지정해줘야한다.
tcpdump -i lo port 8080 or port 8081 -w dump.pcap


#5번정도 실행시켜준다.
curl -k http://127.0.0.1:8080/?param=sendParam
curl -k http://127.0.0.1:8080/?param=sendParam
curl -k http://127.0.0.1:8080/?param=sendParam
curl -k http://127.0.0.1:8080/?param=sendParam
curl -k http://127.0.0.1:8080/?param=sendParam

#Ctrl+c 로 tcpdum 종료

#sz 로 윈도우피시로 dump 한걸 가져온다.
sz dump.pcap
{% endhighlight %}

##### 확인방법
{% highlight markdown %}
윈도우 환경 피시를 가지고 있기떄문에 윈도우에서 설치를 진행한다.
전부 next 로 눌러 설치를 진행.

더블클릭을하면 화면에 리스트가 나온다.
시간정렬을 클릭한후 리스트를 살펴보면 GET /?param=sendParam 이라고 보낸 데이터가있다.
아래로 내려가보면 ?data=am 이라고 보낸 HTTP 프로토콜을 확인할수있다. 
데이터가 어떤 형식으로 바뀌어서 넘어갔는지 데이터 송수신이 잘됬는지를 확인할수있다.
{% endhighlight %}

[다운로드 링크](https://www.wireshark.org/download.html)
