---
layout: post
title: "ECMAScript vs JavaScript"
author: "Chester"
---

#### ECMAScript
{% highlight markdown%}
ECMA International에 의해 표준화되고 TC39위원회가 감독하는 언어. 이 용어는 일반적으로 표준 자체를 지칭하기 위해 사용됨
{% endhighlight %}

#### JavaScript
{% highlight markdown%}
ECMAScript 표준 구현에 일반적으로 사용되는 이름
이 용어는 특정 버전의 ECMAScript 표준과 관련이 없으며 특정 ECMAScript 버전의 전체 또는 일부를 구현한것을 지칭하기 위해 사용될수있음.
1995년 넷스케이프 네비게이터의 초기버전인 라이브 스크립트를 1년후 자바스크립트로 이름을 바꾸기를 원함 당시 자바의 인기를 활용하기위함,
1996년에 Netscape 표준화를 위해 JavaScript 라는 명칭으로 ECMA International 에 제출함.

이때부터 ECMAScript 라는 새로운 표준이 생김

즉 우리가 현재 사용하는 Javascript 의 표준 스펙이 생김.
{% endhighlight %}

#### ES spec 
{% highlight markdown%}
ECMAScript 5 (ES5) : 2009년에 표준화된 ECMAScript 제 5 판 해당표준은 최신브라우저에서 매우완벽하게 구현되어있음.

ECMAScript 6 (ES6)/ ECMAScript 2015 (ES2015):  2015 년에 표준화 된 ECMAScript 제 6 판.이 표준은 대부분의 최신 브라우저에서 거의 대부분 구현되어있음
let , Class, arrow function 등의 기능이 확장되었다.

ECMAScript 2016 : ES7 Firefox 54 이후 버전과 WebKit은 ES7 이후 명세를 100% 지원하고 있다.
1) 제곱연산자 `5 ** 3;` //125
2) includes
[1, 2, 3].includes(1); // true
[1, 2, 3].includes(1, 1); // false
이 추가되었다.

ECMAScript 2017 :  ES8의 중요한 명세인 비동기 함수 문법(async와 await)은 Firefox 52 이후 버전과 Chrome 55 이후 버전에서 이미 구현돼 있다
async , await 이 추가되었다. (비동기처리) 
{% endhighlight %}
