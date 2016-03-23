---
layout:         post
title:          "The specifications of MySQL index"
description:    "We have to understahd how apply index to query."
keywords:       [MySQL, Index, tuning]
categories:     [Maintainance]
tags:           [MySQL, Tuning, Optimization]
---


MySQLのクエリ最適化において、インデックスがどのように適用されるかを正しく理解する必要があります。
ここでは、インデックスについて学んでいこうと思います。

<!-- more -->

### Not work in range search, fuzzy search and negative operation

否定演算の`!=`や`<>`はインデックスが使用されません。`IN`や`WHERE`で代替しましょう。

{% highlight sql %}
/* NG */  WHERE col_1 != 1
/* OK */  WHERE col_1 IN (2,3,4, ...)

/* NG */  WHERE age != 17 AND age !=19
/* OK */  WHERE age IN (..., 16, 18, 20, ...)
{% endhighlight %}

範囲検索の`BETWEEN`や`<=`、`>=`を使用した場合、それ以降のインデックスが使用されないことがあります。
可能な限り`AND`や`OR`でフィルタできる方が良いです。

{% highlight sql %}
/* NG */  WHERE age>=18 AND age<=20
/* OK */  WHERE　age=18 OR age=19 OR age=20

/* NG */  WHERE col_1 > 1      AND col_2 = 2 AND col_3 = 3
/* OK */  WHERE col_1 IN (2,3) AND col_2 = 2 AND col_3 = 3
{% endhighlight %}

また、曖昧検索では、定数文字列＋前方一致以外ではインデックスが使用されません。

{% highlight sql %}
/* OK */  WHERE col_1 LIKE '文字列%'
/* NG */  WHERE col_1 LIKE '%文字列'
/* NG */  WHERE col_1 LIKE '%文字列%'
/* NG */  WHERE col_1 LIKE val
{% endhighlight %}

WHERE句の全ての`AND`にかかっていないインデックスは使用されませ。インデックスは1つのクエリにつき、1つしか使用されません。
下記例の場合、複合インデックスでなければ`col_1`しか使用されません。

{% highlight sql %}
WHERE col_1 = 1 AND col_2 = 2 OR col_1 = 3 AND col_3 = 4
{% endhighlight %}


### Multi column indexes vs. Index Merge

複数のカラムに対するインデックスの指定は以下の2種類があります。
基本的にインデックスマージの方が遅いです。

複合インデックス
:   1つのインデックスで複数のカラムをキーとする
    
    ~~~ sql
    ALTER TABLE table ADD INDEX multi_index (col_1, col_2, ...)
    ~~~

インデックスマージ
:   1つのインデックスに1つのキーカラムを指定する
    
    ~~~ sql
    ALTER TABLE table add INDEX col1_index (col_1)
    ALTER TABLE table add INDEX col2_index (col_2)
    ~~~

インデックスが使用されるケースは下記のとおりです。

複合インデックス
:   2番目のケースは、`OR`の右辺で判定される場合は`col_1`のインデックスが使用されません。

    ~~~ sql
    /* OK */  WHERE col_1 = 1 AND col_2 = 2
    /* NG */  WHERE col_1 = 1 OR  col_2 = 2
    /* OK */  WHERE col_1 = 1 OR  col_1 = 2 AND col_2 = 3
    ~~~

インデックスマージ
:   3番目のケースは、`AND`で2つのインデックスが結合されるのでその分遅いです。
    
    ~~~ sql
    /* OK */  WHERE col_1 = 1 AND col_2 = 2
    /* OK */  WHERE col_1 = 1 OR  col_2 = 2
    /* OK */  WHERE col_1 = 1 OR  col_1 = 2 AND col_2 = 3
    ~~~


### `ORDER BY` statement

以下のケースにおいては、`ORDER BY`句にインデックスは使用できない。

~~~ sql
ALTER TABLE table ADD INDEX index_1_2_3 (col_1, col_2, col_3)
ALTER TABLE table ADD INDEX index_1 (col_1)
ALTER TABLE table ADD INDEX index_2 (col_2)
ALTER TABLE table ADD INDEX index_3 (col_3)


/* --- Non-sequencial key --- */

/* NG */  ORDER BY col_1, col_3
/* NG */  ORDER BY col_2, col_3
/* NG */  ORDER BY col_3
/* OK */  ORDER BY col_1
/* OK */  ORDER BY col_1, col_2
/* OK */  ORDER BY col_1, col_2, col_3


/* --- Confusion ASC, DESC --- */

/* NG */  ORDER BY col_1     , col_2 DESC
/* OK */  ORDER BY col_1 DESC, col_2 DESC


/* --- Multi key without multi column indexes --- */

ALTER TABLE table DROP INDEX index_1_2_3

/* NG */  ORDER BY col_1, col_2
/* OK */  ORDER BY col_1
/* OK */  ORDER BY col_2


/* --- Use different key than the one used in WHERE statement --- */

/* NG */  WHERE col_1 = 1  ORDER BY col_2
/* OK */  WHERE col_2 = 2  ORDER BY col_2


/* --- Use different key between ORDER BY and GROUP BY statement --- */

/* NG */  GROUP BY col_1  GROUP BY col_2
/* OK */  GROUP BY col_1  GROUP BY col_1
~~~

その他にも、`HAVING`句はインデックスが使用されません。

以上のことに注意しながら、`EXPLAIN`でクエリを分析して最適化していくようにしましょう。
