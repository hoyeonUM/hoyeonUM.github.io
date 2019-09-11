---
layout: post
title: "master 와 slave 의 쿼리실행 계획이 틀려진 이유는 ?"
author: "hoyeonUm"
---

### 옵티마이저는 어떻게 계획을 세울까??
-------------
<br/> 
우리가 쿼리를 실행하면 현대의 똑똑한 옵티마이저는 비용(cost based) 기반의 실행계획을 설립한다.   
이때 비용(cost) 에 해당하는것을 참고하려면 통계정보를 얻어와야하는데 통계정보를 다음과 같은곳에 저장한다.
{% highlight sh%}
use mysql;
show tables like '%stats%';
--------------------------------
[통합 통계 정보 테이블]
column_stats
index_stats
table_stats
 
 
[스토리지 엔진 통계 정보 테이블]
innodb_index_stats
innodb_table_stats

{% endhighlight %}
<br/>
[스토리지 엔진 통계 정보 테이블] 을 사용하려면    
`innodb_stats_persistent` 값이 `ON` 상태값이 되어있어야하는데 다음과 같이 확인할수 있다.

{% highlight sh%}
show variables like 'innodb_stats_persistent';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| innodb_stats_persistent | ON    |
+-------------------------+-------+
{% endhighlight %}

위와 같이 두개의 테이블로 나누어져 있는 이유는 다음과 같다.

{% highlight sh%}
1. 각 스토리지 엔진이 제공하는 통계정보는 내용이 상당히 부족함
2. MariaDB 스토리지 엔진 인터페이스로 통계정보를 전달하기에는 제약이 많음 ( 인덱스가 아닌 컬럼의 통계 정보 수집불가)
3. 통계 정보의 제어가 쉽지 않아서 통계정보가 영구적이지 못하고 단기적으로 관리됨.
{% endhighlight %}
<br/>
<br/>
<br/>
<br/>
### 그렇다면 다음과 같이 2개의 통계정보 테이블이 존재하는데 옵티마이저는 통계를 어떤 정보 테이블을 보고 쿼리 플랜을 작성할까 ?
-------------
{% highlight sh%}
show variables like 'use_stat_tables';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| use_stat_tables | NEVER |
+-----------------+-------+
 
 
1. NEVER  (Default)
 - analyze table 명령어를 입력해도 [통합 통계 정보 테이블] 에 데이터가 수집되지 않는다.
 - 옵티마이저가 실행계획을 수립할때에도 [스토리지 엔진 통계 정보 테이블] 만을 활용하여 실행계획을 수립한다.
2. COMPLEMENTARY
 - analyze table 명령어를 입력할때에 [통합 통계 정보 테이블] 에 데이터가 수집된다.
 - 옵티마이저가 실행계획을 수립할때 [스토리지 엔진 통계 정보 테이블] 통계정보를 우선적으로 사용하되 정보가 부족하거나 없는경우에는 [통합 통계 정보 테이블] 의 정보가 사용된다.
 - MyISAM 이나 MEMORY 스토리지 엔진의 경우에는 통계 정보 테이블이 존재 하지않으므로 이러한 단점을 보완하기 위해 [통합 통계 정보 테이블] 의 정보가 사용된다.
3. PREFERABLY
 - analyze table 명령어를 입력할때에 [통합 통계 정보 테이블] 에 데이터가 수집된다.
 - 옵티마이저가 실행계획을 수립할때 우선적으로 [통합 통계 정보 테이블] 을 활용한다. 만약 [통합 통계 정보 테이블] 이 존재하지 않는 경우에는 [스토리지 엔진 통계 정보 테이블] 을 활용한다.
 
※ 주의 2,3 번인 상태에서 analyze table 명령어를 수행할때 full table scan 이 일어난다.
{% endhighlight %}

따라서 설정을 따로 해주지 않는 이상 옵티마이저는 [스토리지 엔진 통계 정보 테이블] 만을 활용하여 실행계획을 작성하는데 분명 어제까지만 해도 정상적인 실행계획을 갖고있었던 쿼리문이 오늘 배포도 하지않았는데 실행계획이 바뀌었다면 다음과 같은 상황을 의심해볼수있다.  
해당 설정이 ON 일때에는 테이블의 1/16 이 INSERT / DELETE / UPDATE 가 변경되었을때 자동으로 테이블의 상태를 다시 계산하는데 이때 정확하게 계산되지 않는다.

{% highlight sh%}
show global variables like 'innodb_stats_auto_recalc';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| innodb_stats_auto_recalc | ON    |
+--------------------------+-------+
{% endhighlight %}
<br/>
다음과 같은 시스템변수가 테이블 상태값을 계산하기위해 존재한다.

{% highlight sh%}
show variables like 'innodb_stats_transient_sample_pages';                                             
+-------------------------------------+-------+
| Variable_name                       | Value |
+-------------------------------------+-------+
| innodb_stats_transient_sample_pages | 8     |
+-------------------------------------+-------+
자동으로 통계 정보 수집이 실행될때 8개의 페이지만 임의로 샘플링해서 분석하고 그 결과를 통계정보로 활용함.
 
 
show variables like 'innodb_stats_persistent_sample_pages';
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| innodb_stats_persistent_sample_pages | 20    |
+--------------------------------------+-------+
analyze table 통계 정보 수집이 실행될때 20개의 페이지만 임의로 샘플링해서 분석하고 그 결과를 통계정보로 활용함.
{% endhighlight %}

만약 기본상태값 그대로 가지고 대량으로 데이터가 변경되었을때 8개의 페이지만 샘플링되어서 상태값에 업데이트가 되었다고 가정하면 이에대한 통계정보는 매우 부정확할 확률이 높다.  
그래서 테이블에 인덱스가 많고 데이터의 변경이 빈번하게 일어나는 경우라면 시스템변수 변경이 아닌 다음과 같이 설정을 변경할수있다.
{% highlight sh%}
CREATE TABLE `test` (
 `field1` varchar(200)
) ENGINE=InnoDB STATS_AUTO_RECALC = 0, STATS_SAMPLE_PAGES = 2000; 
  
ALTER TABLE `test` STATS_AUTO_RECALC = 0, STATS_SAMPLE_PAGES = 2000;
#(샘플페이지 갯수는 많을수록 정확도가 높아짐)
{% endhighlight %}

<br/>
### 결론
-------------

{% highlight sh%}
MariaDB 의 기본설정은 테이블 통계정보를 수집하고 있으며 (Cost 측정을위함) 대량으로 테이블이 변경되었을때에는 테이블 상태값을 업데이트하는데
이때 매우 적은수의 페이지를 가지고 테이블 상태값을 업데이트한다. 따라서 매우 부정확한 정보로 갱신이 되었을때는 옵티마이저가 실행계획을 잘못 수립할수 있다.
 
 
이걸 방지하기 위해서는 테이블단위로는 위의 예제와 같이 만들어주고 ALTER 나 CREATE DDL 을 통해 STATS_AUTO_RECALC = 0 설정을 맞추어주어야한다.  
최초로 테이블을 생성할때에는  `innodb_stats_persistent` 시스템변수를 참고하여 테이블을 생성한다고한다. 
MariaDB > SHOW VARIABLES LIKE 'innodb_stats_persistent';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| innodb_stats_persistent | ON    |
+-------------------------+-------+
 
 
하지만 해당 옵션을 꺼버리는 순간 analyze table 옵션으로도 [스토리지 엔진 통계 정보 테이블] 갱신되지 않으므로 설정을 ON 으로 두고
자동 갱신을 사용하지 않을 테이블에 한해서 STATS_AUTO_RECALC  을 변경해서 사용해야 한다.
 
 
 
 
※ 즉 테이블별로 STATS_AUTO_RECALC  에 대한 설정을 0 으로 변경하고 STATS_SAMPLE_PAGES 의 수를 적절하게 조절하여 시스템 점검이나 시스템이 한가한 시간에 주기적으로 상태를 업데이트해주는것이 가장 좋은방법이다.
{% endhighlight %}


[참조링크1](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)  
[참조링크2](http://www.yes24.com/Product/Goods/12653486)
