---
layout: post
title: "javascript regex toggle working"
author: "hoyeonUm"
---


#### 사건의시작
{% highlight html%}
let pattern = /[,';"\\]|(#\$%)/gi;
if(pattern.test(text)) {
  ..
}
{% endhighlight %}

위와같은 정규표현식이있어서 해당 정규표현식에 걸리면 중단되는 코드였다.
딱히 i (대소문자 구별) 과 g (global match) 를 의식해서 넣은건 아니지만 기존 정규표현식에 추가하다보니 의식하지 않고 다음패턴을 추가하려고했다.

{% highlight html%}
let pattern = /[,';"\\]|(#\$%)/gi;
console.log(pattern.test('안녕;')); // true
console.log(pattern.test('안녕;')); // false
console.log(pattern.test('안녕;')); // true
console.log(pattern.test('안녕;')); // false
console.log(pattern.test('안녕;')); // true
console.log(pattern.test('안녕;')); // false
{% endhighlight %}
다음과 같이 토글형태로 결과값을 return 해 주고있었다. 원인은 다음과 같았다.

RegExp는 일치항목이 발생한 마지막 인덱스를 추가한다.
따라서 이후 매치에서는 0이 아닌 마지막에 사용된 인덱스부터 찾아서 시작한다.


{% highlight html%}
let pattern = /[,';"\\]|(#\$%)/gi;
console.log(pattern.test('안녕;')); // true
console.log(pattern.lastIndex); //3
console.log(pattern.test('안녕;')); // false
console.log(pattern.lastIndex); //0
console.log(pattern.test('안녕;')); // true
console.log(pattern.lastIndex); //3
console.log(pattern.test('안녕;')); // false
console.log(pattern.lastIndex); //0

//;은 3번째에 있으므로 강제로 2로 주어도 true 가 나오는것을 확인할수있다.
pattern.lastIndex = 2;
pattern.test('안녕;')


let pattern = /[,';"\\]|(#\$%)/i;
pattern.test('안녕;') //ture
console.log(pattern.lastIndex); //0
pattern.test('안녕;') //ture
console.log(pattern.lastIndex); //0
pattern.test('안녕;') //ture
console.log(pattern.lastIndex); //0

pattern.lastIndex = 10;
pattern.test('안녕;') //ture
console.log(pattern.lastIndex); //10
{% endhighlight %}
다음과 같이 g 옵션을 주지않은것은 lastIndex 의 영향을 받지않는다.

[참조링크](https://stackoverflow.com/questions/1520800/why-does-a-regexp-with-global-flag-give-wrong-results)
