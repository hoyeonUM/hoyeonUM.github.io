---
layout: post
title: "mariadb 에서 connected by 를 써보자!"
author: "Chester"
---

oracle 에서 사용할수있는 connected by 가 mariadb 10.2 버전이후로부터는 지원된다.
####1. 테이블 생성
{% highlight markdown %}
CREATE TABLE `menu` (
  `idx` int(11) NOT NULL AUTO_INCREMENT,
  `parents_idx` int(11) DEFAULT NULL,
  `name` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`idx`),
  KEY `menu_menu_idx_fk` (`parents_idx`),
  CONSTRAINT `menu_menu_idx_fk` FOREIGN KEY (`parents_idx`) REFERENCES `menu` (`idx`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8mb4
{% endhighlight %}

소스코드 설치로 진행한다.

#### 2. 데이터 넣기
{% highlight markdown %}
INSERT INTO `menu` (idx, parents_idx, name) VALUES (1, null, '상품판매');
INSERT INTO `menu` (idx, parents_idx, name) VALUES (3, null, '운영관리');
INSERT INTO `menu` (idx, parents_idx, name) VALUES (4, 3, '사용자관리');
INSERT INTO `menu` (idx, parents_idx, name) VALUES (5, 3, '관리자관리');
INSERT INTO `menu` (idx, parents_idx, name) VALUES (6, 3, '비회원관리');
INSERT INTO `menu` (idx, parents_idx, name) VALUES (7, 1, '메인매뉴판매');
INSERT INTO `menu` (idx, parents_idx, name) VALUES (8, 1, '사이드매뉴판매');
INSERT INTO `menu` (idx, parents_idx, name) VALUES (9, 4, '비밀번호변경');
INSERT INTO `menu` (idx, parents_idx, name) VALUES (10, 4, '아이디변경');
{% endhighlight %}
#### 3. 조회
{% highlight markdown %}
with recursive cte  as
(
  select     idx,
  name,
  parents_idx,
  1 AS level
  from       menu
where parents_idx is null
  union all
  select     p.idx,
  p.name,
  p.parents_idx,
  1+level as level
  from       menu p
  inner join cte
  on p.parents_idx = cte.idx
)
select idx,ifnull( parents_idx, 0) as pidx,name, level from cte
;
{% endhighlight %}

#### 3.결과
{% highlight markdown %}
idx pidx name level
1	0	상품판매	1
3	0	운영관리	1
4	3	사용자관리	2
5	3	관리자관리	2
6	3	비회원관리	2
7	1	메인매뉴판매	2
8	1	사이드매뉴판매	2
9	4	비밀번호변경	3
10	4	아이디변경	3
{% endhighlight %}

#### 4.mariadb 10.1 이하
[참고](https://explainextended.com/2009/03/17/hierarchical-queries-in-mysql/)  
#### 5.기타
그 이외에도 ROW_NUMBER 함수가 추가되어서 PARTITION BY 와 같이쓸수있음.
{% highlight markdown %}
CREATE TABLE student(course VARCHAR(10), mark int, name varchar(10));

INSERT INTO student VALUES 
  ('Maths', 60, 'Thulile'),
  ('Maths', 60, 'Pritha'),
  ('Maths', 70, 'Voitto'),
  ('Maths', 55, 'Chun'),
  ('Biology', 60, 'Bilal'),
   ('Biology', 70, 'Roger');

SELECT 
  RANK() OVER (PARTITION BY course ORDER BY mark DESC) AS rank, 
  DENSE_RANK() OVER (PARTITION BY course ORDER BY mark DESC) AS dense_rank, 
  ROW_NUMBER() OVER (PARTITION BY course ORDER BY mark DESC) AS row_num, 
  course, mark, name 
FROM student ORDER BY course, mark DESC;
+------+------------+---------+---------+------+---------+
| rank | dense_rank | row_num | course  | mark | name    |
+------+------------+---------+---------+------+---------+
|    1 |          1 |       1 | Biology |   70 | Roger   |
|    2 |          2 |       2 | Biology |   60 | Bilal   |
|    1 |          1 |       1 | Maths   |   70 | Voitto  |
|    2 |          2 |       2 | Maths   |   60 | Thulile |
|    2 |          2 |       3 | Maths   |   60 | Pritha  |
|    4 |          3 |       4 | Maths   |   55 | Chun    |
+------+------------+---------+---------+------+---------+
{% endhighlight %}
[출처](https://mariadb.com/kb/en/library/row_number/)

