---
layout: post
title: "PHP 대용량 엑셀다운로드"
author: "hoyeonUm"
---

#### 소개
{% highlight shell markdown%}
300 백만건 이상의 엑셀 다운로드 파일 요청이 들어와서 기능을 검토하던 도중 다운로드 크기가 200MB 를 넘어 버리는 현상이 발생해 서버 설정에 잡아둔 메모리를 초과하는 현상이 나타났다. ini_set 설정으로 ini_set('memory_limit', -1) 로 진행 하려했지만 서버에 무리가 갈거 같아서 라이브러리들을 찾는 도중 괜찮은 라이브러리가 있어 메모해둡니다.

공식 홈페이지에서는 데이터 기록시 한번에 한줄 혹은 여러줄씩 데이터를 읽어서 메모리에 저장해 둔다음 파일을 쓰고 메모리를 해제 하는 방식을 채택한다. (읽기 기능도 마찬가지로 한줄 혹은 여러줄씩 데이터를 읽어오고 해제 한다고 합니다.)
{% endhighlight %}
[github](https://github.com/box/spout)

#### 공식 홈페이지 CODE
{% highlight php %}
composer require box/spout

use Box\Spout\Writer\WriterFactory;
use Box\Spout\Common\Type;

$writer = WriterFactory::create(Type::XLSX); // for XLSX files
//$writer = WriterFactory::create(Type::CSV); // for CSV files
//$writer = WriterFactory::create(Type::ODS); // for ODS files

$writer->openToFile($filePath); // write data to a file or to a PHP stream
//$writer->openToBrowser($fileName); // stream data directly to the browser

$writer->addRow($singleRow); // add a row at a time
$writer->addRows($multipleRows); // add multiple rows at a time

$writer->close();
{% endhighlight %}

#### DB 연동
{% highlight php %}
use Box\Spout\Writer\WriterFactory;
use Box\Spout\Common\Type;

//데이터를 전부 가져오지 않고 fetch 처리하여 순차적으로 읽어들여옴
$list = $db->fetch();

$writer = WriterFactory::create(Type::XLSX);
$writer->openToFile('/home/tmp/test.xls');
$writer->addRow([
            '이름',
            '전화번호',
            '성별',
            '직급',
        ]);

foreach ($list as $row) {
$writer->addRow([
	$row['name'],
	$row['tel'],
	$row['sex'],
	$row['position'],
]);
}
$writer->close();
//file download... /home/tmp/test.xls
{% endhighlight %}

아직까진 넓이 지정 기능이 없다.
넓이 지정을 위해서 [pull request](https://github.com/box/spout/pull/532/files) 를 참조하여 확장해 사용하였다.
