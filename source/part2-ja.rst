「モナドは象だ」 Part2
========================================================================

パート1では盲目の男たちと象の寓話を通じてScalaのモナドを紹介しました。


ふつうこの話は、男たちがそれぞれ象がどんなものか理解が深まっていないという意味で使います。 しかし、私は別の捉え方を示しました。もし全員の経験を聞いていたなら、すぐに象とはどんなものか、驚くほどよく理解できるに違いないと。

パート2では、「for内包表記」というScalaのモナドに関するシンタックスシュガーを説明することで、その動物をもう少し突いてみるつもりです。


A Little "For"(もう少し「for」を)
------------------------------------------------------------------------

とても単純な「for」は次のようなものです。

.. code-block:: scala

  val ns = List(1, 2)
  val qs = for (n <- ns) yield n * 2
  assert (qs == List(2, 4))

「for」は「for [each] n [in] ns yield n * 2」と読み替えられます。ループのように見えますが、そうではなく我々の古い友人であるmapが潜んでいます。

.. code-block:: scala

  val qs = ns map {n => n * 2}

ここでのルールは単純です。

.. code-block:: scala

  for (x <- expr) yield resultExpr

を展開すると [#for_pattern]_

.. code-block:: scala

  expr map {x => resultExpr}

そして覚えておいてほしいのですが、これは次と等価になります。

.. code-block:: scala

  expr flatMap {x => unit(resultExpr)}


さらに「for」を
------------------------------------------------------------------------

式がひとつだけの「for」はあまり面白くありません。もう少し付け加えてみましょう。

.. code-block:: scala

  val ns = List(1, 2)
  val os = List (4, 5)

  val qs = for ( n <- ns;
                 o <- os) yield n * o

  assert (qs == List (1*4, 1*5, 2*4, 2*5))

この「for」は「"for [each] n [in] ns [and each] o [in] os yield n * o」と読み替えられます。
この「for」の形はネストしたループにも見えますが、どちらかというとmapとflatMapです。

.. code-block:: scala

  val qs = ns flatMap {n =>
           os     map {o => n * o }}


なぜこれが動作するのか理解するために時間を費やすことはムダではありません。
どのように計算されるか見てみましょう(イタリック体の赤字が太字の緑字に変換されます)。

.. TODO 色付ける

::

  val qs = ns flatMap {n => os map {o => n * o }}
  val qs = ns flatMap {n => List(n * 4, n * 5)}
  val qs = List(1 * 4, 1 * 5, 2 * 4, 2 * 5


「さらに式を増やしてみる」
------------------------------------------------------------------------

段階を上げてみましょう。

.. code-block:: scala

  val qs =
     for (n <- ns;
          o <- os;
          p <- ps) yield n * o * p


この「for」は次のように展開されます。

.. code-block:: scala

  val qs = ns flatMap {n =>
           os flatMap {o =>
          {ps     map {p => n * o * p}}}}


これは先ほどの「for」とまったく同じように見えます。なぜならルールを繰り返しただけだからです。

.. code-block:: scala

  for(x1 <- expr1;...x <- expr)
     yield resultExpr

これは、次のように展開されます。

.. code-block:: scala

  expr1 flatMap {x1 =>
        for(...;x <- expr) yield resultExpr
  }


このルールはmapを使う式が1つだけになるまで繰り返し適用されます。以下は、コンパイラがどのように"val qs = for..."という式を展開するかを示したものです(先ほどと同様に、赤字斜体が太字の緑に変換されます)。

.. TODO 色付ける

::

  val qs = for (n <- ns; o <- os; p <- ps)
           yield n * o * p

  val qs = ns flatMap {n =>
           for(o <- os; p <- ps)
           yield n * o * p}

  val qs = ns flatMap {n =>
           os flatMap {o =>
           for(p <- ps) yield n * o * p}}

  val qs = ns flatMap {n =>
           os flatMap {o =>
          {ps map {p => n * o * p}}}



命令の「for」
------------------------------------------------------------------------

単に副作用がある関数を呼び出したい場合のために、「for」の命令型のバージョンが用意されています。 yieldステートメントを取るだけです。

.. code-block:: scala

  val ns = List(1, 2)
  val os = List (4, 5)

  for (n <- ns;
       o <- os)  println(n * o)

展開のルールはyieldをベースにしたバージョンとすごく似ていますが、flatMapやmapの代わりにforeachを使います。

.. code-block:: scala

  ns foreach {n =>
  os foreach {o => println(n * o) }}

Now, you don't have to implement foreach if you don't want to use the imperative form of "for", but foreach is trivial to implement since we already have map.

さて、もし命令形の「for」を使うつもりがなければ、モナドにforeachを実装する必要はありません。ですが、既にmapがあるのでforeachは簡単に実装できます。

.. code-block:: scala

  class M[A] {
     def map[B](f: A => B) : M[B] = ...
     def flatMap[B](f: A => M[B]) : M[B] = ...
     def foreach[B](f: A => B) : Unit = {
         map(f)
         ()  // Scalaでは()はUnit型を表す。ここでは明示的に()を返しているが、この行は無くてもよい
     }
  }

foreachは、単にmapを呼び出した結果を返さずに捨ててしまっているだけだと言えます。この方法は、最適な実行効率を実現する方法ではないかもしれません。ですので、Scalaでは独自の方法でforeachを実装できるようになっています。


「for」のフィルタリング
------------------------------------------------------------------------

これまで、モナドはいくつかのキーとなるコンセプトで構築してきました。これら3つのメソッド、map、flatMap、forEachによって、「for」が実行できることのほとんどすべてが実現できます。

しかし、Scalaの「for」ステートメントはもう1つ機能があります。「if」ガードです。例を示します。

.. code-block:: scala

  val names = List("Abe", "Beth", "Bob", "Mary")

  val bNames = for (bName <- names;
     if bName(0) == 'B'
  ) yield bName + " is a name starting with B"

  assert(bNames == List(
     "Beth is a name starting with B",
     "Bob is a name starting with B"))


「if」ガードはfilterと呼ばれるメソッドに変換されます。filterは述語関数(引数を取りtrueかfalseを返す関数)を引数に取り、述語にマッチしない要素を除いた新しいモナドを作ります。

上記のforステートメントは次のようなものに変換されます。

.. code-block:: scala

  val bNames =
     (names filter { bName => bName(0) == 'B' })
     .map { bName =>
        bName + " is a name starting with B"
     }


最初にリストが名前がBで始まるものだけを残すフィルターにかけられます。それからそのフィルターにかけられたリストは" is a name..."を追加する無名関数を使ってmapされます。

すべてのモナドがフィルターできるわけではありません。コンテナにたとえて言うと、フィルターによってすべての要素が削除するかもしれないですし、空にできないコンテナもあります。

そのようなモナドではfilterメソッドを作成する必要はありません。Scalaは「for」式で「if」ガードを使わない限りエラーとしません。

filterについてはもう少し話すことがあります。次回は、純粋にモナド的な観点でfilterをどのように定義するか、どの種類のモナドがフィルターできないかといったことをお話しします。


Part2の結論
------------------------------------------------------------------------

「for」はモナドを簡便に方法です。リストなどのコレクションと組み合わせるときにhは、特に便利な文法になっています。
しかし「for」はそれよりもっと普遍的です。mapやflatMap、foreach、filterに展開されます。

その中でもmapとflatMapはあらゆるモナドで定義されるべきです。

foreachメソッドは、モナドを命令的に扱いたいときに定義します。実装は簡単です。
filterは定義してもよいモナドもありますがそうでないものもあります。

「m map f」は「m flatMap {x => unit(x)}」として実装できます。
「m foreach f」はmapを用いて実装するか、flatMapを利用して「m flatMap {x => unit(f(x));()}」として実装します。

「m filter p」でさえ、flatMapを使って実装することができます(次回方法を示します)。flatMapは、まさに象という動物の心臓なのです。


モナドは象だということを思い出してください。これまでにお見せしたモナドというものは、コレクションであるという側面が強調されていました。

パート4では、コレクションでないモナドや抽象的な手段においてのみコンテナであるようなモナドを紹介します。
その前に、すべてのモナドが満たすべきいくつかの特性、モナド則をパート3で解説する必要があります。

In the mean time, here's a cheat sheet showing how Haskell's do and Scala's for are related.

話は変わりますが、ここでHaskellのdoとScalaのforがどのように関連しているかを示すチートシートを載せておきます。

+----------------------------+------------------------------+
| Haskell                    |  Scala                       |
+============================+==============================+
| do var1 <- expn1           |  for {var1 <- expn1;         |
|    var2 <- expn2           |     var2 <- expn2;           |
|    expn3                   |     result <- expn3          |
|                            |  } yield result              |
+----------------------------+------------------------------+
| do var1 <- expn1           |  for {var1 <- expn1;         |
|    var2 <- expn2           |     var2 <- expn2;           |
|    return expn3            |  } yield expn3               |
+----------------------------+------------------------------+
| do var1 <- expn1 >> expn2  |  for {_ <- expn1;            |
|    return expn3            |     var1 <- expn2            |
|                            |  } yield expn3               |
+----------------------------+------------------------------+


.. rubric:: 脚注

.. [#for_pattern] Scalaの仕様では、「for」はパターンマッチングを用いて展開します。実際の仕様では展開の規則として、「<-」の左側にパターンを書くことを許可しています。このことについて深く解説すると、記事の主題が大きくぼやけてしまいます。
