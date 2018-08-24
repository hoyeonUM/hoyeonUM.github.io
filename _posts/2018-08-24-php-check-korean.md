---
layout: post
title: "php 에서 한글포함 여부를 체크해보자"
author: "Chester"
---

#### 1. 환경체크
{% highlight bash%}
php -v
PHP 7.2.9 (cli) (built: Aug 15 2018 08:05:45) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.9, Copyright (c) 1999-2018, by Zend Technologie


php -m |grep mbstring
mbstring
{% endhighlight %}

#### 2. 코드샘플 실행 
{% highlight php %}
<?php
$notIncludeKR = [
    'a',
    'b',
    1,
    -1,
    'A',
    'B',
    'C',
    '数',
    '数-色色',
    '+1',
    '|',
    '^-$',
    '1234',
    '[]',
    '!',
    '@',
    '#'
];
$includeKR = [
    '[ㄱ]',
    '数-가',
    '数-가-色',
    '★-가-色',
    '色-★-가',
    '가-★-色',
    '★-ㄱ-色',
    '★-ㄴ-色',
    '★-ㅣ-色',
    '★-ㅋ-色',
];
//case1
function utf8Ord($ch)
{
    $len = strlen($ch);
    if ($len <= 0) {
        return false;
    }
    $h = ord($ch{0});
    if ($h <= 0x7F) {
        return $h;
    }
    if ($h < 0xC2) {
        return false;
    }
    if ($h <= 0xDF && $len > 1) {
        return ($h & 0x1F) << 6 | (ord($ch{1}) & 0x3F);
    }
    if ($h <= 0xEF && $len > 2) {
        return ($h & 0x0F) << 12 | (ord($ch{1}) & 0x3F) << 6 | (ord($ch{2}) & 0x3F);
    }
    if ($h <= 0xF4 && $len > 3) {
        return ($h & 0x0F) << 18 | (ord($ch{1}) & 0x3F) << 12 | (ord($ch{2}) & 0x3F) << 6 | (ord($ch{3}) & 0x3F);
    }

    return false;
}

function hasKorean($value)
{
    $c = utf8Ord($value);
    if (0x1100 <= $c && $c <= 0x11FF) {
        return true;
    }
    if (0x3130 <= $c && $c <= 0x318F) {
        return true;
    }
    if (0xAC00 <= $c && $c <= 0xD7A3) {
        return true;
    }

    return false;
}

//case2
function charAt($str, $pos, $encoding = "UTF-8")
{
    return mb_substr($str, $pos, 1, $encoding);
}

function hasKoreanV2($ch)
{
    for ($i = 0; $i < mb_strlen($ch); $i++) {
        if (preg_match("/[\xA1-\xFE][\xA1-\xFE]/", charAt($ch, $i))) {
            return true;
        }
    }
    return false;
}

function hasKoreanV3($ch)
{
    return preg_match('/(\p{Hangul})/u', $ch) == 1;
}


echo 'hasKorean method';
echo PHP_EOL;
//하위 케이스에서 echo 가 찍히면 안됨.
foreach ($notIncludeKR as $value) {
    if (hasKorean($value)) {
        echo $value;
        echo PHP_EOL;
    }
};

foreach ($includeKR as $value) {
    if (!hasKorean($value)) {
        echo $value;
        echo PHP_EOL;
    }
}
echo 'hasKorean method END';
echo PHP_EOL;


echo 'hasKorean method V2';
echo PHP_EOL;
//하위 케이스에서 echo 가 찍히면 안됨.
foreach ($notIncludeKR as $value) {
    if (hasKoreanV2($value)) {
        echo $value;
        echo PHP_EOL;
    }
};

foreach ($includeKR as $value) {
    if (!hasKoreanV2($value)) {
        echo $value;
        echo PHP_EOL;
    }
}
echo 'hasKoreanV2 method END';
echo PHP_EOL;


echo 'hasKorean method V3';
echo PHP_EOL;
//하위 케이스에서 echo 가 찍히면 안됨.
foreach ($notIncludeKR as $value) {
    if (hasKoreanV3($value)) {
        echo $value;
        echo PHP_EOL;
    }
};

foreach ($includeKR as $value) {
    if (!hasKoreanV3($value)) {
        echo $value;
        echo PHP_EOL;
    }
}
echo 'hasKoreanV3 method END';
{% endhighlight %}
#### 3.결과
{% highlight shell bash %}
hasKorean method
[ㄱ]
数-가
数-가-色
★-가-色
色-★-가
★-ㄱ-色
★-ㄴ-色
★-ㅣ-色
★-ㅋ-色
hasKorean method END
hasKorean method V2
[ㄱ]
★-ㄱ-色
★-ㄴ-色
★-ㅣ-色
★-ㅋ-色
hasKoreanV2 method END
hasKorean method V3
hasKoreanV3 method END
{% endhighlight %}

{% highlight plaintext %}
결과와 같이 hasKorean에서는 -가 들어간 문자 및 특수문자가 들어간 문자들을 걸러내지못하였고 hasKoreanV2 에서는 위에 hasKorean 을 보완하여 mbstring 함수를 사용하여 문자열을 하나씩 잘라서 for 문을 돌려서 한글자씩 문자열 체크를 진행하였다. 그럼에도 걸러지지 못하는 것들이 있어서 Hangul 이라는 정규표현식을 사용하여 통과하였다.

php 5.1 이상부터 지원된다고한다.

hasKoreanV2 에서는 hasKorean 에 사용된 utf8Ord 을 사용하면 mbstring 에서 잘라온 문자열 코드를 다르게 인식하여서 같은 코드를 사용할수없었다. 처음엔 간단하게 생각하여 javascript 와같이 str.split('') === explode('', str) 이런식으로 생각하여 문자열을 자르려고 시도를 했지만 기대값을 보여주지 못하였고 str[0], str[1] 과 같이 문자열에 배열로 진입하니까 문자열이 깨져서 나와 mbstring 을 사용하여 문자열을 분리하였다.
{% endhighlight %}
