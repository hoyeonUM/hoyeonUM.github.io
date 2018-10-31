---
layout: post
title: "hg before commit"
author: "hoyeonUm"
---

hg before commit hook 을 걸기위해 방법을 찾아보자

#### .hg/hgrc
{% highlight php%}
[hooks]
pretxncommit.psrcheck = php /workspace/psrcheck.php
{% endhighlight %}

위와 같이 설정을 해두면 commit을 하기전에 check 를 해줄수있다.
근데 만약 파일이 10000개가 넘으면 개인 커밋인데도 check 하기에 너무 오래걸린다.
그래서 psrcheck.php 에 `sleep(1000)` 문구를 걸어서 커밋하기전 저장되는장소를 찾아보았다.


.hg/store/journal
해당파일에 commit 될 목록이 표시되고있었다.

#### vim /workspace/psrcheck.php
{% highlight php%}
$journalPath = 'path/.hg/store/journal';
$phpcsPath = 'path/phpcs';

$readLine = file_get_contents($journalPath);
preg_match_all('/\/test.*php/', $readLine, $target);
if (empty($target[0])) {
    exit();
}

$target = $target[0];
array_walk($target, function (&$v) use ($workingDir) {
$v= $workingDir.$v;
});
$concatTarget = implode(' ', $target);

$command = $phpcsPath.' -v --standard=PSR2'.$concatTarget;
exec($command, $output);
$buffer = [];
foreach ($output as $key => $value) {
    preg_match("/FILE:.*php$/", $value, $regex);
    if (count($regex) > 0) {
        foreach ($output as $k => $v) {
            if ($key > $k) {
                continue;
            }
            echo $v;
            echo PHP_EOL;
        }
        exit(1);
    };
}
{% endhighlight %}
exit(1) 을 만나면 commit 이 정상처리되지않는다.
