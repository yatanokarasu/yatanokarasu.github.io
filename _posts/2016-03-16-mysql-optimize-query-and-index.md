---
layout:         post
title:          "Optimize MySQL query and index"
description:    "If your query is slow, you may have to review your query and index."
keywords:       [MySQL, Index, tuning]
categories:     [Maintainance]
tags:           [MySQL, Tuning, Optimization]
---

If MySQL query is slow, we may have to review structure of the query and index.
This post brings together information how to optimize query and index.
For details, the following pages will be helpful studying about this topic.

*   [http://nippondanji.blogspot.jp/2009/03/mysqlexplain.html]()
*   [http://www.slideshare.net/infinite_loop/mysql-indexexplain]()

もしMySQLクエリが遅いのなら、クエリの構造とインデックスを見直したほうが良いです。
ここではクエリの最適化とインデックスについてまとめていきます。
詳しくは、上記のページが勉強になります。

<!-- more -->

# Usage EXPLAIN statement

<u>EXPLAIN</u> statement provides information about how MySQL executes queries.  
<u>EXPLAIN</u>句は、クエリがどのように実行されているかを調べることができます。
実行結果以下の様になります(クエリは[コチラ](http://nippondanji.blogspot.jp/2009/03/mysqlexplain.html)から引用)。

{% highlight mysql %}
mysql> EXPLAIN SELECT * FROM Country, (SELECT * FROM City WHERE Population > 1000000) AS C1 WHERE Country.Code = C1.CountryCode;
+----+-------------+------------+--------+---------------+---------+---------+----------------+------+-------------+
| id | select_type | table      | type   | possible_keys | key     | key_len | ref            | rows | Extra       |
+----+-------------+------------+--------+---------------+---------+---------+----------------+------+-------------+
|  1 | PRIMARY     | <derived2> | ALL    | NULL          | NULL    | NULL    | NULL           |  237 |             |
|  1 | PRIMARY     | Country    | eq_ref | PRIMARY       | PRIMARY | 3       | C1.CountryCode |    1 |             |
|  2 | DERIVED     | City       | ALL    | NULL          | NULL    | NULL    | NULL           | 4079 | Using where |
+----+-------------+------------+--------+---------------+---------+---------+----------------+------+-------------+
3 rows in set (0.00 sec)
{% endhighlight %}

出力内容の詳細は[公式ドキュメント](https://dev.mysql.com/doc/refman/5.6/ja/explain-output.html)が詳しいですが、
自分なりにまとめてみたいと思います。

#### Table of output columns

|  column        |  means                                   |
|:---------------|:-----------------------------------------|
| id             | identifier                               |
| select_type    | Type of SELECT statement                 |
| table          | Table name                               |
| type           | Type of JOIN                             |
| possible_keys  | Selectable Possible Keys                 |
| key            | Selected Key                             |
| key_len        | Selected Key Length                      |
| ref            | The columns compared by keys             |
| row            | Number of rows                           |
| Extra          | Additional information to resolve query  |
{: rules="groups"}


より細かい説明は以下のようです。

id
:   SELECT文の識別子。クエリ内で連番になります。NULLの場合もあるようです。

select_type
:   SELECT文の種類。主なものは以下表のとおり。他はちょっとよくわからなかった。。。要勉強。。。

    | select_type   | means                               |
    |:--------------|:------------------------------------|
    | SIMPLE        | 単純なSELECT(UNIONやサブクエリを含まない)   |
    | PRIMARY       | もっとも外側のSELECT。最初にfetchされるテーブル |
    | UNION         | UNION内の2つめ以降にfetchされるテーブル      |
    | UNION RESULT  | UNIONの実行結果                        |
    | SUBQUERY      | サブクエリ内の最初のクエリ                     |
    | DERIVED       | 派生テーブルSELECT(FROM句内のサブクエリ)        |
    {: rules="groups"}

table
:   参照しているテーブルの名前。具体的なテーブル名でなく、以下の値になることもあるようです。
    
    \<union ___M___, ___N___>
    :   `id`値が ___M___ と ___N___ の和集合への参照を意味します。
    
    \<derived ___N___>
    :   `id`値が ___N___ の派生テーブル(FROM句内のサブクエリ)結果への参照を意味します。
    
    \<subquery ___N___>
    :   `id`値が ___N___ のサブクエリ結果への参照を意味します。

type
:   この値は個人的にとても重要だと思っています。この値を見ると、どのクエリが遅いのかが分かります。
    
    const
    :   主キーまたはユニークインデックスでの検索です。一発で値が検索できるので最速です。
    
    eq_ref
    :   JOINする際に主キーやユニークインデックスが使用されている場合にこの値になるようです。
        __const__ に次いで高速です。
    
    ref
    :   主キーやユニークインデックスではないインデックスを使って`=`や`<=>`を使って比較した場合です。
    
    range
    :   インデックスを使って特定の範囲検索をした場合です。
        `=`、`<>`、`>`、`>=`、`<`、`<=`、`IS`、`NULL`、`<=>`、`BETWEEN`、`IN()`演算を使って
        キーカラムと定数を比較する場合です。
    
    index
    :   インデックスって名前から良さそうですが、やってることはフルインデックススキャンです。
        つまり、インデックス全体をスキャンするためとっても遅いです。
    
    ALL
    :   名前の通りでず、フルテーブルスキャンです。全検索なので激遅です。
        早々の改善が必要です。

possible_keys
:   テーブルアクセスに使用可能なキー一覧です。

key
:   possible_keysの中から実際に選択されたキーです。

key_len
:   選択されたキーの長さです。キー長が短い方が早いようです。

ref
:   キーと比較されている値やカラムの種類です。定数の場合は`const`です。

rows
:   そのクエリによりフェッチされる行数見積りです。確実な行数ではないようです。

Extra
:   typeフィールドの次に重要だそうです。クエリの改善方法に役立つ追加情報が表示されます。
    クエリーを可能な限り高速にしたい場合は`Using filesort`と`Using temporary`に注意する
    必要があるそうです。
    
    Using filesort
    :   まずクイックソートしてからJOINするようです。なので遅いです。
    
    Using index
    :   インデックスだけでクエリを解決できることを示しているようです。
    
    Using temporary
    :   クエリ実行結果からソート(`ORDER BY`)したり、重複排除(`DISTINCT`)したりするために
        一時的に保持される表が必要なことを示しています。
    
    Using where
    :   公式ドキュメントを見てもよくわかりませんでしたが、`WHERE`句が指定されていても
        インデックスだけでは全てに適用できない場合を示しているようです。



[コチラ](http://nippondanji.blogspot.jp/2009/03/using-filesort.html)では以下の様に
まとめられていました。

> * Using filesortがEXPLAINに出ていないときはインデックスを用いてソートが行われている。（最も高速。）
> * Using filesortだけがEXPLAINの最初の行に出ている時は、先にソートしてからJOINが実行されている。
> * Using temporary; Using filesortがEXPLAINの最初の行に出ている時は、先にJOINしてからソートされている。（最も遅い。）
