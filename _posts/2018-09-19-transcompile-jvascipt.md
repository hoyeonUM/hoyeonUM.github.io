---
layout: post
title: "JavaScript Transcompile 을 진행해보자"
author: "Chester"
---

예전 진행한 프로젝트에서 function 을 arrow function 으로 변환하는 도중 다음과 같은코드가 있었다.

{% highlight js%}
let someThingObject = {
  something : 'test' ,
  getSomthing : function () {
    return this.somthing;
  }
}

someThingObject.getSomthing(); //test
{% endhighlight %}

해당코드를 다음과 같이 변환하고있는데 일반 function 에서는 잘되던 this 값이 arrow function 으로 이동되면서는 전달되지 않는 문제가 생겼다.
arrow function 은 this 를 that 이나 bind 함수 없이 쓸수있도록 도와준다고 알고있었는데 적용이 안되어서 테스트를 진행해 보았다.
{% highlight js%}
let someThingObject = {
  something : 'test' ,
  getSomthing : () => {
    return this.somthing;
  }
}
someThingObject.getSomthing(); //undefined
{% endhighlight %}

node 랑 npm 은 설치 되어있다고 가정하고 진행한다.

#### setUp
{% highlight bash sh%}
npm install babel-cli -g

/usr/bin/babel-doctor -> /usr/lib/node_modules/babel-cli/bin/babel-doctor.js
/usr/bin/babel -> /usr/lib/node_modules/babel-cli/bin/babel.js
/usr/bin/babel-node -> /usr/lib/node_modules/babel-cli/bin/babel-node.js
/usr/bin/babel-external-helpers -> /usr/lib/node_modules/babel-cli/bin/babel-external-helpers.js

cd /home
mkdir ./babeltest
cd ./babeltes

//babel preset setup
npm install --save-dev babel-preset-es2015t

//babelrc
vim ./.babelrc
{
  "presets":["es2015"]
}

{% endhighlight %}

기초세팅종료.


#### test.js 
{% highlight js%}
vim ./test.js

let testObj = {
  testVal : '1234',
  callArrow : () => {
    console.log(this.testVal)
  },
  callFunction : function () {
    console.log(this.testVal);
  },
  callFunctionInArrow : function() {
    setTimeout(() => {
        console.log(this.testVal);
    });
  },
  callFunctionInFunction : function() {
    setTimeout(function () {
      console.log(this.testVal);
    });
  },
  callArrowInArrow : () => {
    setTimeout(() => {
        console.log(this.testVal);
    });
  },
  callArrowInFunction : () => {
    setTimeout(function() {
        console.log(this.testVal);
    });
  },
}
testObj.callArrow(); //undefined
testObj.callFunction(); //1234
testObj.callFunctionInArrow(); //1234
testObj.callFunctionInFunction(); //undefined
testObj.callArrowInArrow(); //undefined
testObj.callArrowInFunction(); //undefined
{% endhighlight %}

#### babel compile
{% highlight js%}
babel ./test.js -o ./test_compile.js

vim ./test_compile.js

'use strict';

var testObj = {
  testVal: '1234',
  callArrow: function callArrow() {
    console.log(undefined.testVal);
  },
  callFunction: function callFunction() {
    console.log(this.testVal);
  },
  callFunctionInArrow: function callFunctionInArrow() {
    var _this = this;

    setTimeout(function () {
      console.log(_this.testVal);
    });
  },
  callFunctionInFunction: function callFunctionInFunction() {
    setTimeout(function () {
      console.log(this.testVal);
    });
  },
  callArrowInArrow: function callArrowInArrow() {
    setTimeout(function () {
      console.log(undefined.testVal);
    });
  },
  callArrowInFunction: function callArrowInFunction() {
    setTimeout(function () {
      console.log(this.testVal);
    });
  }
};
testObj.callArrow(); //undefined
testObj.callFunction(); //1234
testObj.callFunctionInArrow(); //1234
testObj.callFunctionInFunction(); //undefined
testObj.callArrowInArrow(); //undefined
testObj.callArrowInFunction(); //undefined

{% endhighlight %}

위와 같이 애초에 this 값이 undefined 로 변환되서 저장되는것을 확인할수있다.

화살표 함수에서 this 를 찾을수 없는 이유는
{% highlight markdown%}
화살표 함수는 자기 자신의 this 를 가지고있지않는다.(bind 와 같은 행위를 할수없음) 따라서
화살표 함수 내에서 this 에 대한 호출이 일어나면 this에 대한 바인딩을 찾기위해 화살표 함수의 범위를 찾기 시작한다.
일반 함수 (function a () {}) 와 global scope 만이 this 를 소유할수 있기때문에 화살표 함수는 object 안에서의 this 를 찾을수 없다. 따라서 최상위 (global scope) 의 this 를 참조하게 된다.
{% endhighlight %}
[참조](https://github.com/anirudh-modi/JS-essentials/blob/master/ES2015/Functions/Arrow%20functions.md#how-this-is-different-for-arrow-functions)
