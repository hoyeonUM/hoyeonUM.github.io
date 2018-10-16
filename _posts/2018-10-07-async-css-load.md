---
layout: post
title: "css aysnc load"
author: "hoyeonUm"
---

어느날 웹페이지로딩이 느려 확인해해보니까 css 로드될때까지 화면이 로딩이 되지않는다.
찾아보니 다음과 같은 이유가 있다.

### FOUC
{% highlight markdown%}
CSS 가 로드되기 전에 스타일 없이 페이지가 로드 되는경우가 있다.
이런 경우를 FOUC(Flash Of Unstyled Content) 라고 부른다고 한다. 이를 다루기 위한 방법은
`rel="stylesheet"` 방식을 사용하여 페이지 렌더링 차단으로 로드한다.
DOM 렌더링 및 JavaScript 실행은 모든 렌더링 차단 `rel="stylesheet"` 리소스가 로드되고 CSSOM (CSS Object Model) 로 변환된다고 한다. 
{% endhighlight %}
1. [FOUC 란?](https://ko.wikipedia.org/wiki/FOUC)
2. [chrome 렌더링 과정](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model?hl=ko)

### 해결
{% highlight markdown%}
<link rel="preload" href="test.css" as="style" onload="this.rel='stylesheet'">
{% endhighlight %}

우선순위를 지정하는 방식이지만 rel 옵션이 preload 로 변경되면서 페이지 렌더링 차단이 해제 되고 로딩이 완료 되면서 다시 stylesheet 로 변환되면서
FOUC 를 발생시킨다. 

다음과 같이 구현시 현재버전(69.0.3497.100) 크롬에서 정상 동작하지만 파이어폭스 에서는 동작하지않는다.
[preload 구현](https://www.filamentgroup.com/lab/async-css.html) 을 참조해 preload 옵션을 주었고,
[loadCSS](https://github.com/filamentgroup/loadCSS) 를 사용함으로써 크롬 이외에 브라우저에서도 지원되는걸로 보인다.
