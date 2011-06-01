「モナドは象だ」 Part3
========================================================================

この連載では盲目の男たちと象の古い寓話への新しい観点を紹介しました。
盲目の男たちから、それぞれの限られた説明を聞くことで、象に対してのよりよい理解に至るということです。

これまでは、Scalaにおけるモナドの外側を見てきました。そのことはモナドへの深い理解をもたらしてくれましたが、ついに内側を見るときがきました。

象を象足らしめているものは何かというと、DNAです。
モナドは、モナド則という形式が、全てのモナドに共通する「モナドをモナド足らしめるDNA」にあたります。

この記事は一度にすべてを消化するためにとても長いものになっています。おそらくまとめて読まないと意味がわからないでしょう。
また、(リストのように)すでに理解しているモナドに法則を適用しながら読み返すとよいでしょう。


すべてにとって等価である
------------------------------------------------------------------------

モナド則について説明を続ける前に、「f(x) ≡ g(x)」のような3重等号をどのような意味で使っているか、少し形式的に説明せねばなりません。

この意味は、数学者が「=」等号で意味することと同じです。ただの「=」を代入と混同しないようにしているだけです。


よって、左辺と右辺が「等価」である式だと言っているわけです。では、「等価」とはどのような意味でしょうか。

初めに、参照の同一(Scalaのeqメソッド)について話しているわけではありません。参照の同一は私の定義を満たすかもしれませんが、必要条件が強すぎます。

次に、偶然正しく実装されていない限り必ずしも==等式を意味していません。


この「等価」が意味することは、2つのオブジェクト同士が直接的にしろ間接的にしろ、プリミティブな参照の同一性やハッシュコードをベースにした参照、isInstanceOfなどを使って区別できない、ということです。


特に、左辺と右辺のオブジェクトが、内部は微妙に異なりながらも「等価」である、ということがありえます。例えば、あるオブジェクトが右辺と同じ値になったのは、実は間接的に外部(例えばファイルや標準出力など)に依存していたからかもしれません。ここで重要なのは、外から見れば両方のオブジェクトは同じ振る舞いをしなければならないと言うことです。


「等価」についてもう一つ注意することがあります。全ての法則は、暗黙に副作用がないことを前提にしています。

この記事の最後で副作用についてもっと詳しく触れるつもりです。


法則を壊す
------------------------------------------------------------------------


「もし法則xを壊すとどうなるのだろう？」と疑問に思う人がいるかもしれません。
その答えは、どの法則をどのように壊したのかに依存しますが、まず総体的にアプローチします。

数学のある分野で、このような法則があることを思い出してください。
もしaとbとcが有理数であるならかけ算(*)は次の法則に従います。

::

  a * 1 ≡ a
  a * b ≡ b * a
  (a * b) * c ≡ a * (b * c)


たしかに「RationalNumber」のようなクラスを作り*演算子を実装することは簡単です。
しかし、もしこの法則に従わなったら、計算の結果は控えめに言っても混乱させられるものでしょう。
このクラスを計算式に使った人は、間違った答えを得るでしょう。
率直に言って、これらの法則を破ることは難しく、まだ有理数のかけ算のような何かでしかないでしょう。


Monads are not rational numbers. But they do have laws that help define them and their operations. Like arithmetic operations, they also have "formulas" that allow you to use them in interesting ways. For instance, Scala's "for" notation is expanded using a formula that depends on these laws. So breaking the monad laws is likely to break "for" or some other expectation that users of your class might have.

モナドは有理数ではありません。しかしモナド自身とその演算を定義する助けとなる法則を持っています。
算術演算のように、興味深い方法でモナドを使えるような「式」を持ちます。

たとえばScalaの「for」記法はこれらの法則に依存した式を使って展開されます。

だから、あなたのクラスがモナド則を破ることは、「for」記法やユーザーが抱く期待を破る可能性があります。


前置きはここまでです。モナド則を説明するために、もう1つの風変わりな単語ファンクターから始めます。


ファンクターとは何か？
------------------------------------------------------------------------

一般的に「モナド」と「ファンクター」のような単語で始まる記事はすぐに、ギリシャ文字の海が襲ってきます。
なぜなら「モナド」も「ファンクター」も圏論と呼ばれる数学の分野に置ける抽象的な概念だからです。 またそれらを完全に説明するには数学的な手法が必要です。
幸運なことに、私の仕事はそれらを完全に説明することではなくScalaでそれらをカバーすることだけです。

In Scala a functor is a class with a map method and a few simple properties. For a functor of type M[A], the map method takes a function from A to B and returns an M[B]. In other words, map converts an M[A] into an M[B] based on a function argument. It's important to think of map as performing a transformation and not necessarily having anything to do with loops. It might be implemented as a loop, but then again it might not.

Scalaにおいてファンクターはmapメソッドといくつかの単純な特性を持つクラスです。
M[A]型のファンクターでは、mapメソッドは"A型からB型への関数"を引数に取りM[B]型を返します。 言い換えると、mapメソッドはは引数の関数に基づいてM[A]型をM[B]型へ変換します。

これは重要なことなのですが、mapは変換を行うものとして考え、必ずしもループで何かをするものものとして考えないことです。 もしかしたらループとして実装されるかもしれませんが、そうでなくてもかまわないのです。

mapのシグネチャは次のようなものです。

.. code-block:: scala

  class M[A] {
   def map[B](f: A => B):M[B] = ...
  }


First Functor Law: Identity(ファンクターの第1法則：同一性)
_____________________________________________________________________



次のようなidentity関数を作ります。

.. code-block:: scala

  def identity[A](x:A) = x


This obviously has the property that for any x

明らかに、いかなるxに対しても以下の特性を持ちます。

::

    identity(x) ≡ x


これ以上何もしませんが、それがポイントです。(どんな型であっても)その引数を何も変えずに返すだけです。

ゆえにファンクターの第1法則はこうなります。あらゆるファンクターmにとって

::

  F1.  m map identity ≡ m           // or equivalently *
  F1b. m map {x => identity(x)} ≡ m // or equivalently
  F1c. m map {x => x} ≡ m


言い換えると、何もしないということは何も変えないということになります。すばらしい！

しかしながら、左辺の式が異なるオブジェクトを返し、それが異なる内部構造を持っていさえする場合があるということを思い出してください。
単にそれらを見分けない限りですが。


もし法則に従わないファンクターを作成し、このあとに続くことが真にならないとします。
なぜこれが混乱することなのか見てみましょう。mはリストを装っているとします。

::

  F1d. for (x <- m) yield x ≡ m

Second Functor Law: Composition(第2法則：コンポジション)
_____________________________________________________________________

The second functor law specifies the way several "maps" compose together.

2つ目のファンクター法則は、いくつかの「map」を一緒に組み合わせる方法を定義します。

::

  F2. m map g map f ≡ m map {x => f(g(x))}

This just says that if you map with g and then map with f then it's exactly the same thing as mapping with the composition "f of g." This composition law allows a programmer to do things all at once or stretch them out into multiple statements. Based on this law, a programmer can always assume the following will work.

これは単に、gでmapした結果をさらにfでmapすると、「gとf」の合成関数でmapする
もしgをmapしそれからfをmapすると、まさに「gのf」というコンポジションでmapすることと同じであるということを言っています。

このコンポジションの法則でプログラマは一度にすべてを実行するか複数のステートメントにそれらを伸ばすことができます。

この法則に基づいて、プログラマはいつも次のことが動作すると仮定するでしょう。

val result1 = m map (f compose g)
val temp = m map g
val result2 =  temp map f
assert result1 == result2
In "for" notation this law looks like the following eye bleeder

「for」表記法においてこの法則は次のような見づらいものになります。

F2b. for (y<- (for (x <-m) yield g(x)) yield f(y) ≡ for (x <- m) yield f(g(x))



Functors and Monads, Alive, Alive Oh(ファンクターとモナドはまだまだ続く)


As you may have guessed by now all monads are functors so they must follow the functor laws. In fact, the functor laws can be deduced from the monad laws. It's just that the functor laws are so simple that it's easier to get a handle on them and see why they should be true.

もう想像したかもしれませんが、すべてのモナドはファンクターです。それゆえモナドはファンクターの法則に従います。

実際、ファンクターの法則はモナド則から演繹されます。ファンクターの法則は単純なのでそれを理解しなぜ真になるのか理解することは簡単です。


As a reminder, a Scala monad has both map and flatMap methods with the following signatures

思い出してほしいのですが、Scalaのモナドは次のシグネチャでmapとflatMapの両方のメソッドを持っています。

class M[A] {
 def map[B](f: A => B):M[B] = ...
 def flatMap[B](f: A=> M[B]): M[B] = ...
}
Additionally, the laws I present here will be based on "unit." "unit" stands for a single argument constructor or factory with the following signature

その上、ここで示した法則は「unit」に基づいています。「unit」は単一引数のコンストラクタかファクトリを表します。

次のようなシグネチャです。

def unit[A](x:A):M[A] = ...
"unit" shouldn't be taken as the literal name of a function or method unless you want it to be. Scala doesn't specify or use it but it's an important part of monads.

「unit」はそうしたいと思わない限り関数やメソッドのリテラル名を引数に取るべきではありません。

Scalaはそれを条件づけたり使ったりしませんが、これはモナドの重要な部分です。


Any function that satisfies this signature and behaves according to the monad laws will do. Normally it's handy to create a monad M as a case class or with a companion object with an appropriate apply(x:A):M[A] method so that the expression M(x) behaves as unit(x).

このシグネチャを満たしモナド則に従って振る舞うあらゆる関数はそう動作します。

通常、式M(x)がunit(x)として振る舞うようにモナドMを適切なapply(x:A):M[A]メソッドを持つケースクラスや随行オブジェクトとして作成するのは便利です。


The Functor/Monad Connection Law: The Zeroth Law(ファンクター/モナドの結合法則：0番目の法則)


In the very first installment of this series I introduced a relationship

このシリーズの一番初めの前置きで関連をこのように紹介しました。

FM1. m map f ≡ m flatMap {x => unit(f(x))}
This law doesn't do much for us alone, but it does create a connection between three concepts: unit, map, and flatMap.

この法則は単独ではあまり意味を成しませんが、3つのコンセプトunitとmap、flatMapの結びつきを作り出します。


This law can be expressed using "for" notation pretty nicely

この法則は「for」表記法を使ってかなりうまく表現されるでしょう。

FM1a. for (x <- m) yield f(x) ≡ for (x <- m; y <- unit(f(x))) yield y

Flatten Revisited(flattenふたたび)


In the very first article I mentioned the concept of "flatten" or "join" as something that converts a monad of type M[M[A into M[A], but didn't describe it formally. In that article I said that flatMap is a map followed by a flatten.

一番最初の記事で「flatten」または「join」のコンセプトをM[M[A野茂などをM[A]に変換するものとして言及しましたが、正しく記述していません。

その記事ではflatMapがflattenに従ったmapだと言いました。

FL1. m flatMap f ≡ flatten(m map f)
This leads to a very simple definition of flatten

これは非常に単純なflattenの定義を導きます。

flatten(m map identity) ≡ m flatMap identity // substitute identity for f
FL1a. flatten(m) ≡ m flatMap identity // by F1
So flattening m is the same as flatMapping m with the identity function. I won't use the flatten laws in this article as flatten isn't required by Scala but it's a nice concept to keep in your back pocket when flatMap seems too abstract.

ゆえにmをflattenすることはmをidentity関数とともにflatMapすることと同じです。Scalaではflattenは必要でないのでこの記事ではflattenの法則を使いたくありませんが、flatMapが抽象化され過ぎであるときのためにflatternは知っておくとよい概念です。









The First Monad Law: Identity(第一のモナド則：同一性)
_____________________________________________________________________


The first and simplest of the monad laws is the monad identity law

モナド則の最初の、そしてもっとも単純な法則はモナドの同一性の法則です。

M1. m flatMap unit ≡ m // or equivalently
M1a. m flatMap {x => unit(x)} ≡ m
Where the connector law connected 3 concepts, this law focuses on the relationship between 2 of them. One way of reading this law is that, in a sense, flatMap undoes whatever unit does. Again the reminder that the object that results on the left may actually be a bit different internally as long as it behaves the same as "m."

結合法則が3つの概念をつなげているのに対して、この法則はそのうち2つの関係に焦点を合わせています。この法則を読み解く1つの方法は、ある意味ではflatMapはunitが行ったことを元に戻しているだけです。もう1度思い出してほしいのが左辺の結果となるオブジェクトは実はそれが「m」として同じように振る舞う限り内部的に少し異なっているかもしれないということです。


Using this and the connection law, we can derive the functor identity law

これと結合法則を使ってファンクターの同一法則を導出することができます。

m flatMap {x => unit(x)} ≡ m // M1a
m flatMap {x => unit(identity(x))}≡ m // identity
F1b. m map {x => identity(x)} ≡ m // by FM1
jyukutyoコメント

ファンクターの第1法則は「m map identity ≡ m」。map {x => identity(x)} と map identity は等価。


The same derivation works in reverse, too. Expressed in "for" notation, the monad identity law is pretty straight forward

同じ導出は逆からでも有効です。「for」表記法で表せば、モナドの同一法則はとても明白です。

M1c. for (x <- m; y <- unit(x)) yield y ≡ m

The Second Monad Law: Unit(モナドの第2法則：Unit)


Monads have a sort of reverse to the monad identity law.

モナドの同一法則に対するある種の逆法則もあります。

M2. unit(x) flatMap f ≡ f(x) // or equivalently
M2a. unit(x) flatMap {y => f(y)} ≡ f(x)
The law is basically saying that unit(x) must somehow preserve x in order to be able to figure out f(x) if f is handed to it. It's in precisely this sense that it's safe to say that any monad is a type of container (but that doesn't mean a monad is a collection!).

この法則は基本的に次のようなことを言っています。もしfがxを扱うなら、f(x)を算定できるようにするためにunit(x)はなんとかしてxを保存しなければならないということです。

正確にこの意味においてあらゆるモナドはコンテナ型であると言っても安全です(しかしこれはモナドがコレクションであるとは言っていません！)。


In "for" notation, the unit law becomes

「for」表記法では、unit法則はこうなります。

M2b. for (y <- unit(x); result <- f(y)) yield result ≡ f(x)
This law has another implication for unit and how it relates to map

この法則はunitについてそれがどのようにmapと関連するかということも暗示します。

unit(x) map f ≡ unit(x) map f // no, really, it does!
unit(x) map f ≡ unit(x) flatMap {y => unit(f(y))} // by FM1
M2c. unit(x) map f ≡ unit(f(x)) // by M2a
In other words, if we create a monad instance from a single argument x and then map it using f we should get the same result as if we had created the monad instance from the result of applying f to x. In for notation

言い換えると、もし単一引数xでモナドインスタンスを生成しそれをfを使ってmapすると、xにfを適用した結果からモナドを生成したときと同じ結果を得るべきです。

for表記法では、

M2d. for (y <- unit(x)) yield f(y) ≡ unit(f(x))
The Third Monad Law: Composition(モナド第3法則：コンポジション)

The composition law for monads is a rule for how a series of flatMaps work together.

モナドにとってのコンポジション法則は一連のflatMapがどのように一緒に動作するかのルールです。

M3. m flatMap g flatMap f ≡ m flatMap {x => g(x) flatMap f} // or equivalently
M3a. m flatMap {x => g(x)} flatMap {y => f(y)} ≡ m flatMap {x => g(x) flatMap {y => f(y) }}
It's the most complicated of all our laws and takes some time to appreciate. On the left side we start with a monad, m, flatMap it with g. Then that result is flatMapped with f. On the right side, we create an anonymous function that applies g to its argument and then flatMaps that result with f. Finally m is flatMapped with the anonymous function. Both have same result.

これはすべての法則の中でもっとも複雑であり、認めるには時間がかかります。左辺ではモナドmで始めそれをgでflatMapします。それからその結果をfでflatMapします。

右辺では引数にgを適用しその結果をfでflatMapする無名関数を作成します。結局mは無名関数でflatMapされます。

両方とも同じ結果になります。


In "for" notation, the composition law will send you fleeing in terror, so I recommend skipping it

「for」表記法ではコンポジション法則は恐れをなして逃げてしまいそうなので、飛ばすことをお勧めします。

M3b. for (a <- m;b <- g(a);result <- f(b)) yield result ≡ for(a <- m; result <- for(b < g(a); temp <- f(b)) yield temp) yield result
From this law, we can derive the functor composition law. Which is to say breaking the monad composition law also breaks the (simpler) functor composition. The proof involves throwing several monad laws at the problem and it's not for the faint of heart

この法則から、ファンクターのコンポジション法則を導出できます。それはモナドのコンポジション法則を破ると同時に(より単純な)ファンクターのコンポジションも破ることになるということです。証明はいくつかのモナド則を問題に投入することを含んでいますが、気弱な人は見なくても結構です。

m map g map f ≡ m map g map f // I'm pretty sure
m map g map f ≡ m flatMap {x => unit(g(x))} flatMap {y => unit(f(y))} // by FM1, twice
m map g map f ≡ m flatMap {x => unit(g(x)) flatMap {y => unit(f(y))}} // by M3a
m map g map f ≡ m flatMap {x => unit(g(x)) map {y => f(y)}} // by FM1a
m map g map f ≡ m flatMap {x => unit(f(g(x))} // by M2c
F2. m map g map f ≡ m map {x => f(g(x))} // by FM1a
Total Loser Zeros(完全な敗者はいない)


List has Nil (the empty list) and Option has None. Nil and None seem to have a certain similarity: they both represent a kind of emptiness. Formally they're called monadic zeros.

リストはNil(空のリスト)を持ちOptionはNoneを持っています。NilとNoneにはある類似性があります。両方ともある種の空を表しています。正式にはモナド的なゼロと呼ばれるものです。


A monad may have many zeros. For instance, imagine an Option-like monad called Result. A Result can either be a Success(value) or a Failure(msg). The Failure constructor takes a string indicating why the failure occurred. Every different failure object is a different zero for Result.

モナドは多くのゼロを持っているかもしれません。例えばResultと呼ぶOptionのようなモナド想像してみてください。ResultはSuccess(value) かFailure(msg)のどちらかです。Failureのコンストラクタはなぜ失敗が起こったかを示す文字列を引数に取ります。すべての異なるfailureオブジェクトはResultにとって異なるゼロとなります。


A monad may have no zeros. While all collection monads will have zeros (empty collections) other kinds of monads may or may not depending on whether they have a concept of emptiness or failure that can follow the zero laws.

モナドはゼロを持たないかもしれません。すべてのコレクションモナドはゼロ(空のコレクション)を持ちますが、他の種類のモナドはゼロの法則に従う空や失敗の概念に依存するかもしれませんし、しないかもしれません。


The First Zero Law: Identity(第1のゼロの法則：同一性)
_____________________________________________________________________


If mzero is a monadic zero then for any f it makes sense that

もしmzeroがモナド的なゼロであればあらゆるfの場合で次のことが成り立ちます。

MZ1. mzero flatMap f ≡ mzero
Translated into Texan: if t'ain't nothin' to start with then t'ain't gonna be nothin' after neither.

テキサスなまりに変換しましょう。もし？？？


This law allows us to derive another zero law

この法則はほかのゼロの法則を導出します。

mzero map f ≡ mzero map f // identity
mzero map f ≡ mzero flatMap {x => unit(f(x)) // by FM1
MZ1b. mzero map f ≡ mzero // by MZ1
So taking a zero and mapping with any function also results in a zero. This law makes clear that a zero is different from, say, unit(null) or some other construction that may appear empty but isn't quite empty enough. To see why look at this

ゆえにゼロにどんな関数をmapしても結果はゼロになります。この法則は次のことを明らかにします。ゼロはunit(null)や空のように見えるが完全に空ではない他の構造とは異なるということです。なぜなのか見てみましょう。

unit(null) map {x => "Nope, not empty enough to be a zero"} ≡ unit("Nope, not empty enough to be a zero")










The Second Zero Law: M to Zero in Nothing Flat(第2のゼロの法則：すぐにMをゼロにする)


The reverse of the zero identity law looks like this

ゼロの同一法則を逆にすると次のようになります。

MZ2. m flatMap {x => mzero} ≡ mzero
Basically this says that replacing everything with nothing results in nothing which um...sure. This law just formalizes your intuition about how zeros "flatten."

基本的にこれはすべてを何もないものに置き換えると結果は何もないものになります。うーん。たしかに。

この法則はゼロをどのように「flatten」するかについての直感を承認しただけです。


The Third and Fourth Zero Laws: Plus(第3と第4のゼロの法則：加算)


Monads that have zeros can also have something that works a bit like addition. For List, the "plus" equivalent is ":::" and for Option it's "orElse." Whatever it's called its signature will look this

ゼロを持つモナドはまた加算と少し似た何かを持ちます。リストでは「plus」は「:::」と等価であり、Optionでは「orElse」です。

何度呼ばれようとそのシグネチャは次のようなものです。

class M[A] {
   ...
   def plus(other:M[B >: A]): M[B] = ...
}
jyukutyoコメント

orElse [B >: A](alternative : => Option[B]) : Option[B]

If the option is nonempty return it, otherwise return the result of evaluating an alternative expression.


Plus has the following two laws which should make sense: adding anything to a zero is that thing.

加算には次の2つの法則があります。どんなものにゼロを加算してもそのままだということです。

MZ3. mzero plus m ≡ m
MZ4. m plus mzero ≡ m
The plus laws don't say much about what "m plus n" is if neither is a monadic zero. That's left entirely up to you and will vary quite a bit depending on the monad. Typically, if concatenation makes sense for the monad then that's what plus will be. Otherwise, it will typically behave like an "or," returning the first non-zero value.

加算の法則は「m plus n」の両方ともがモナド的にゼロである場合についてのことは何も言っていてません。

そのことについては完全にあなた次第であり、依存するモナドによってかなり変わります。

通常は、もしモナドに取って連結が意味を成すならそれが加算が意味することになります。

そうでなければ、通常「or」のように振る舞い、ゼロでない最初の値を返します。


Filtering Revisited(フィルタリング再び)

In the previous installment I briefly mentioned that filter can be seen in purely monadic terms, and monadic zeros are just the trick to seeing how. As a reminder, a filterable monad looks like this

前回の連載では純粋にモナド的な意味でフィルターについて簡潔に言及しました。モナド的なゼロはどのようになるのかは単なるトリックです。

思い出してほしいのですが、フィルタリングできるモナドは次のようなものです。

class M[A] {
   def map[B](f: A => B):M[B] = ...
   def flatMap[B](f: A=> M[B]): M[B] = ...
   def filter(p: A=> Boolean): M[A] = ...
}
The filter method is completely described in one simple law

フィルターモナドは単純な1つの法則で完全に記述できます。

FIL1. m filter p ≡ m flatMap {x => if(p(x)) unit(x) else mzero}
We create an anonymous function that takes x and either returns unit(x) or mzero depending on what the predicate says about x. This anonymous function is then used in a flatMap. Here are a couple of results from this

xを引数にとり述語がxについて返すことに依存してunit(x)かmzeroを返す無名関数を作ります。

この無名関数はflatMapで使われます。ここから2,3の結論が出ます。

m filter {x => true} ≡ m filter {x => true} // identity
m filter {x => true} ≡ m flatMap {x => if (true) unit(x) else mzero} // by FIL1
m filter {x => true} ≡ m flatMap {x => unit(x)} // by definition of if
FIL1a. m filter {x => true} ≡ m // by M1
So filtering with a constant "true" results in the same object. Conversely

ゆえに定数「true」でフィルタリングすると同じオブジェクトとなります。逆に

m filter {x => false} ≡ m filter {x => false} // identity
m filter {x => false} ≡ m flatMap {x => if (false) unit(x) else mzero} // by FIL1
m filter {x => false} ≡ m flatMap {x => mzero} // by definition of if
FIL1b. m filter {x => false} ≡ mzero // by MZ1
Filtering with a constant false results in a monadic zero.

定数「false」でフィルタリングするとモナド的なゼロになります。


Side Effects(副作用)

Throughout this article I've implicitly assumed no side effects. Let's revisit our second functor law

この記事を通じて私は暗黙的に副作用がないことを前提にしてきました。2つめのファンクターの法則に戻ってみましょう。

m map g map f ≡ m map {x => (f(g(x)) }
If m is a List with several elements, then the order of the operations will be different between the left and right side. On the left, g will be called for every element and then f will be called for every element. On the right, calls to f and g will be interleaved. If f and g have side effects like doing IO or modifying the state of other variables then the system might behave differently if somebody "refactors" one expression into the other.

もしmがいくつかの要素があるリストであるなら、左辺と右辺では操作の順序が異なるでしょう。

左辺においてgを各要素で呼び出してからfを各要素で呼び出します。

右辺ではfとgを交互に呼び出します。もしfとgにIOしたり他の変数の状態を変更するような副作用があれば、誰かがある式をほかの式に「リファクタリング」してしまうとシステムはことなる振る舞いをするかもしれません。


The moral of the story is this: avoid side effects when defining or using map, flatMap, and filter. Stick to foreach for side effects. Its very definition is a big warning sign that reordering things might cause different behavior.

この話の教訓は次のようなものです。mapやflatMap、filterを定義したり使ったりするときは副作用を避けるということです。

まさにその定義は順序を変えてしまうと異なる振る舞いを起こすかもしれないという大きな警告です。


Speaking of which, where are the foreach laws? Well, given that foreach returns no result, the only real rule I can express in this notation is

そういえば、foreachの法則はどこにあるんでしょう？えーと、何の結果も返さないものであるforeachが与えられると、この表記法で説明できるたった1つのほんとうのルールは、

m foreach f ≡ ()
Which would imply that foreach does nothing. In a purely functional sense that's true, it converts m and f into a void result. But foreach is meant to be used for side effects - it's an imperative construct.

これはforeachが何もしないということを暗示しています。純粋に関数的な意味ではこれは真実であり、mとfをvoidの結果に変換します。

しかしforeachは副作用のために使われるということを意味しています、これは命令的な構造です。


Conclusion for Part 3(パート3の結論)

Up until now, I've focused on Option and List to let your intuition get a feel for monads. With this article you've finally seen what really makes a monad a monad. It turns out that the monad laws say nothing about collections; they're more general than that. It's just that the monad laws happen to apply very well to collections.

今まで、モナドの感触を直感でつかんでもらおうとOptionとListに焦点を合わせてきました。

この記事では最終的にモナドを真にモナドたらしめているものは何かということを観ていきます。

モナド則はコレクションについて何も言っていないということが結局わかります。コレクションよりも普遍的なものなのです。

モナド則はコレクションに対してたまたまとてもうまく適用できるというだけなのです。


In part 4 I'm going to present a full grown adult elephant er monad that has nothing collection-like about it and is only a container if seen in the right light.

パート4では完全に育った大人の象を紹介し、モナドがまったくコレクション的なものではなく正しい意味でとらえるなら単なるコンテナであるということを紹介します。


Here's the obligatory Scala to Haskell cheet sheet showing the more important laws

ここで必須の、より重要な法則を示すScaalからHaskellへのチートシートを出します。

   Scala  Haskell
FM1  m map f ≡ m flatMap {x => unit(f(x))}  fmap f m ≡ m >>= \x -> return (f x)
M1  m flatMap unit ≡ m  m >>= return ≡ m
M2  unit(x) flatMap f ≡ f(x)  (return x) >>= f ≡ f x
M3  m flatMap g flatMap f ≡ m flatMap {x => g(x) flatMap f}  (m >>= f) >>= g ≡ m >>= (\x -> f x >>= g)
MZ1  mzero flatMap f ≡ mzero  mzero >>= f ≡ mzero
MZ2  m flatMap {x => mzero} ≡ mzero  m >>= (\x -> mzero) ≡ mzero
MZ3  mzero plus m ≡ m  mzero 'mplus' m ≡ m
MZ4  m plus mzero ≡ m  m 'mplus' mzero ≡ m
FIL1  m filter p ≡ m flatMap {x => if(p(x)) unit(x) else mzero}  mfilter p m ≡ m >>= (\x -> if p x then return x else mzero)

