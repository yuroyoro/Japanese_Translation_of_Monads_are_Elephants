
Until you experience an adult elephant first hand you won't really understand just how big they can be. If monads are elephants then so far in this series of articles I've only presented baby elephants like List and Option. But now it's time to see a full grown adult pachyderm. As a bonus, this one will even do a bit of circus magic.

大人の象を直接見るまでは本当に象がどれだけ大きいのか理解したことにはなりません。

モナドがもし象であるならこの連載ではまだListとOptionのような幼い象を紹介しただけです。

しかし機は熟しました。大人の象を見ていきましょう。ボーナスとして少しサーカスマジックもさせます。


Functional Programming and IO(関数型プログラミングとIO)


In functional programming there's a concept called referential transparency. Referential transparency means you can call a particular function anywhere and any time and the same arguments will always give the same results. As you might imagine, a referentially transparent function is easier to use and debug than one that isn't.

関数型プログラミングにおいて参照透過性という概念があります。

参照透過性はある関数を同じ引数でいつどこで呼び出したとしても常に同じ結果になるという意味です。

想像のとおり、参照透過である関数はそうでない関数よりも使いやすくデバックしやすいものになります。


There's one area where referential transparency would seem impossible to achieve: IO. Several calls to the same readLine console function may result in any number of different strings depending on things like what the user ate for breakfast. Sending a network packet may end in successful delivery or it might not.

参照透過性を達成できない領域が1つあります。それはIOです。

同じコンソールを1行読み出す関数を何回か呼び出すとユーザーが朝食に何を食べたかのようなことに依存して異なる文字列が返ってくるかもしれません。

ネットワークパケットを送ることは送信に成功したかそうでないかの結果になります。


But we can't get rid of IO just to accomplish referential transparency. A program without IO is just a complicated way to make your CPU hot.

しかし、参照透過性を達成するためにIOを取り除くことはできません。IOなしのプログラミングは単にCPIを熱くさせる複雑な方法でしかありません。


You might guess that monads provide a solution for referentially transparent IO given the topic of this series but I'm going to work my way up from some simple principles. I'll solve the problem for reading and writing strings on the console but the same solution can be extended to arbitrary kinds of IO like file and network.

この連載のトピックを鑑みてモナドは参照透過性のあるIOのための解決方法を提供するものと推測しているかもしれませんが、いくつかの単純な原則から築き上げようと思います。

コンソールから読み書きするために問題を解決しようとしますが、同じ解決方法はファイルやネットワークのような任意の種類のIOにも拡張できます。


Of course, you may not think that referentially transparent IO is terribly important in Scala. I'm not here to preach the one true way of purely functional referential transparency. I'm here to talk about monads and it just so happens that the IO monad is very illustrative of how several monads work.

もちろん、参照透過であるIOがScalaにおいてとても重要であるというこは考えなくてもかまいません。

私は純粋に関数的な参照透過性の1つの真実の手段を説こうとしているわけではありません。

モナドについて話そうとしており、IOモナドがいくつかのモナドがどのように動作しているかについてとても説明に役立つのでそうしているだけです。


The World In a Cup(カップの中の世界)


Reading a string from the console wasn't referentially transparent because readLine depends on the state of the user and "user" isn't one of its parameters. A file reading function would depend on the state of the file system. A function that reads a web page would depend on the state of the target web server, the Internet, and the local network. Equivalent output functions have similar dependencies.

コンソールから文字列を読み出すことは参照透過ではありません。なぜなら1行読みだすことはユーザーの状態に依存し、「ユーザー」はその関数の引数ではないからです。

ファイルを読み出す関数はファイルシステムの状態に依存します。

ウェブページを読み出す関数は対象となるウェブサーバーやインターネット、ローカルネットワークの状態に依存します。

同様に出力する関数も同じような依存性があります。


All this could be summed up by creating a class called WorldState and making it both a parameter and a result for all IO functions. Unfortunately, the world is a big place. My first attempt to write a WorldState resulted in a compiler crash as it ran out of memory. So instead I'll try for something a bit smaller than modeling the whole universe. That's where a bit of circus magic comes in.

これらはすべてWorldStateというクラスを作りすべてのIO関数がパラメータと戻り値の両方ともWorldStateとなるようにすれば話は終わります。

不幸なことに、世界は広いです。WorldStateを書くという最初の目論みはアウトオブメモリーでコンパイラがクラッシュするという結果になります。

ゆえに代わりに全世界をモデル化することよりも少し小さい何かを試してみましょう。

サーカスマジックが出てきました。


The slight-of-hand I'll use is to model only a few aspects of the world and just pretend WorldState knows about the rest of the world. Here are some aspects that would be useful

世界のいくつかの面ををモデル化し、WorldStateが世界の残りをしるように装うためだけに手品を使います。いくつか役に立つ見地があります。


jyukutyoコメント

slight-of-handはsleight of handのタイポ？マジックと手品だし。



The state of the world changes between IO functions.
The world's state is what it is. You can't just create new ones whenever you want (val coolWorldState = new WorldState(){def jamesIsBillionaire = true}).
The world is in exactly one state at any moment in time.

世界の状態はIO関数間で変わる。
世界の状態はそのままのものである。val coolWorldState = new WorldState(){def jamesIsBillionaire = true}のようなことを望んだとしても新しい世界は作れない。
世界はどんな瞬間でも厳密に1つの状態である。

Property 3 is a bit tricky so let's deal with properties 1 and 2 first.

特性3は少しトリッキーなのでまず特性1と2から考えてみましょう。


Here's a rough sketch for property 1

特性1のラフスケッチです。

//file RTConsole.scala
object RTConsole_v1 {
  def getString(state: WorldState) =
    (state.nextState, Console.readLine)
  def putString(state: WorldState, s: String) =
    (state.nextState, Console.print(s) )
}
getString and putString use functions defined in scala.Console as raw primitive functions. They take a world state and return a tuple consisting of a new world state and the result of the primitive IO.

getStringとputStringは低レベルなプリミティブ関数としてscala.Consoleに定義されている関数を使います。世界の状態を引数にとり、世界の新しい状態とプリミティブなIOの結果をで構成されたタプルを返します。


Here's how I'll implement property 2

特性2を実装しました。

//file RTIO.scala
sealed trait WorldState{def nextState:WorldState}

abstract class IOApplication_v1 {
  private class WorldStateImpl(id:BigInt)
      extends WorldState {
    def nextState = new WorldStateImpl(id + 1)
  }
  final def main(args:Array[String]):Unit = {
    iomain(args, new WorldStateImpl(0))
  }
  def iomain(
      args:Array[String],
      startState:WorldState):(WorldState, _)
}
WorldState is a sealed trait; it can only be extended within the same file. IOApplication defines the only implementation privately so nobody else can instantiate it. IOApplication also defines a main function that can't be overridden and calls a function named iomain that must be implemented in a subclass. All of this is plumbing that is meant to be hidden from programmers that use the IO library.

WorldStateはtraitにします。継承できるのは同じファイルにあるものだけです。

IOApplicationはその実装だけをプライベートで定義します。ゆえに誰もインスタンス化することはできません。

IOApplicationはまたオーバーライドできず、サブクラスで必ず実装しなければならないiomain関数を呼び出すものであるmain関数を定義します。

これらはすべてIOライブラリを使うプログラマから隠蔽するための配管です。


Here's what hello world looks like given all this

次のようなhello worldがあります。

// file HelloWorld.scala
class HelloWorld_v1 extends IOApplication_v1 {
  import RTConsole_v1._
  def iomain(
        args:Array[String],
        startState:WorldState) =
    putString(startState, "Hello world")
}









That Darn Property 3(最悪の特性3)

The 3rd property said that the world can only be in one state at any given moment in time. I haven't solved that one yet and here's why it's a problem

3番目の特性は世界はあらゆる瞬間でも1つの状態にだけあるということを言っています。

それはまだ解決できていません。なぜなら問題があるからです。

class Evil_v1 extends IOApplication_v1 {
  import RTConsole_v1._
  def iomain(
      args:Array[String],
      startState:WorldState) = {
    val (stateA, a) = getString(startState)
    val (stateB, b) = getString(startState)
    assert(a == b)
    (startState, b)
  }
}
Here I've called getString twice with the same inputs. If the code was referentially transparent then the result, a and b, should be the same but of course they won't be unless the user types the same thing twice. The problem is that "startState" is visible at the same time as the other world states stateA and stateB.

同じ入力でgetStringを2回呼び出しています。もしこのコードが参照透過であるなら結果であるaとbが同一であるべきですが、同じことを2回入力しない限りもちろんそうはなりません。

問題は「startState」が世界の異なる状態であるstateAとstateBとして同時に可視化されていることです。


Inside Out(裏返す)


As a first step towards a solution, I'm going to turn everything inside out. Instead of iomain being a function from WorldState to WorldState, iomain will return such a function and the main driver will execute it. Here's the code

解決への第1歩として、すべてを裏返してみます。

iomainをWorldStateからWorldStateを返す関数とする代わりに、iomainはそのような関数を返すようにし、mainはそれを実行するようにします。コードはこうです。

//file RTConsole.scala
object RTConsole_v2 {
  def getString = {state:WorldState =>
    (state.nextState, Console.readLine)}
  def putString(s: String) = {state: WorldState =>
    (state.nextState, Console.print(s))}
}
getString and putString no longer get or put a string - instead they each return a new function that's "waiting" to be executed once a WorldState is provided.

getStringとputStringはもはや文字列をgetやputしません。代わりにひとたびWorldStateが与えられると実行を「待つ」新しい関数を毎回返します。

//file RTIO.scala
sealed trait WorldState{def nextState:WorldState}

abstract class IOApplication_v2 {
  private class WorldStateImpl(id:BigInt)
      extends WorldState {
    def nextState = new WorldStateImpl(id + 1)
  }
  final def main(args:Array[String]):Unit = {
    val ioAction = iomain(args)
    ioAction(new WorldStateImpl(0));
  }
  def iomain(args:Array[String]):
    WorldState => (WorldState, _)
}
IOApplication's main driver calls iomain to get the function it will execute, then executes that function with an initial WorldState. HelloWorld doesn't change too much except it no longer takes a WorldState.

IOApplicationのmain関数は実行する関数を取得するためにiomainを呼び出します。それから内部のWorldStateとともに州の関数を実行します。

HelloWorldはもはやWorldStateを引数にしないこと以外ほとんど変更しません。

//file HelloWorld.scala
class HelloWorld_v2 extends IOApplication_v2 {
  import RTConsole_v2._
  def iomain(args:Array[String]) =
    putString("Hello world")
}
At first glance we seem to have solved our problem because WorldState is nowhere to be found in HelloWorld. But it turns out it's just been buried a bit.

一見問題を解決したように見えます。なぜならWorldStateはHelloWorldのどこにも見つからないからです。

しかし、単に隠されているだけだとわかります。


Oh That Darn Property 3(ああ、最悪の特性3)

class Evil_v2 extends IOApplication_v2 {
  import RTConsole_v2._
  def iomain(args:Array[String]) = {
    {startState:WorldState =>
      val (statea, a) = getString(startState)
      val (stateb, b) = getString(startState)
      assert(a == b)
      (startState, b)
    }
  }
}
Evil creates exactly the kind of function that iomain is supposed to return but once again things are broken. As long as the programmer can create arbitrary IO functions he or she can see through the WorldState trick.

Evilはiomainが戻すと改訂してる関数を正確に作成していますが、またしても物事は壊れています。プログラマが自由にIO関数を作成する限り、その人はWorldStateのトリックをIO関数を通じて見れるからです。


Property 3 Squashed For Good(特性3はよいもののために押しつぶされる)


All we need to do is prevent the programmer from creating arbitrary functions with the right signature. Um...we need to do what now?

我々に必要なものはプログラマが正しいシグネチャで自由に関数を作れないようにすることだけです。うーん、今何をする必要があるでしょう？


Okay, as we saw with WorldState it's easy to prevent programmers from creating subclasses. So let's turn our function signature into a trait.

さて、WorldStateを見てきてサブクラスを作成できないようにすることは簡単です。ゆえに関数のシグネチャをtraitに変えてみましょう。

sealed trait IOAction[+A] extends
  Function1[WorldState, (WorldState, A)]

private class SimpleAction[+A](
   expression: => A) extends IOAction[A]...
jyukutyoコメント

「sealed」はクラスに設定できる修飾詞です。
* sealedとされたクラスは、同一ファイル内のクラスからは継承できますが、別ファイル内で定義されたクラスでは継承できません。
[Scala

Unlike WorldState we do need to create IOAction instances. For example, getString and putString are in a separate file but they would need to create new IOActions. We just need them to do so safely. It's a bit of a dilemma until we realize that getString and putString have two separate pieces: the piece that does the primitive IO and the piece that turns the input world state into the next world state. A bit of a factory method might help keep things clean, too.

WorldStateと異なりIOActionインスタンスを生成する必要があります。たとえば、getStringとputStringは別のファイルにありますが新しいIOActionを生成する必要があるでしょう。

我々は安全にそれをする必要があるだけです。我々がgetStringとputStringが2つの別のものだと理解しない限り少しジレンマがあります。

プリミティブなIOをするものと入力した世界の状態を次の世界の状態に変えるものです。

少しのファクトリメソッドがものごとを整理する手助けになるでしょう。

//file RTIO.scala
sealed trait IOAction_v3[+A] extends
  Function1[WorldState, (WorldState, A)]

object IOAction_v3 {
  def apply[A](expression: => A):IOAction_v3[A] =
    new SimpleAction(expression)

  private class SimpleAction [+A](
      expression: => A) extends IOAction_v3[A] {
    def apply(state:WorldState) =
      (state.nextState, expression)
  }
}

sealed trait WorldState{def nextState:WorldState}

abstract class IOApplication_v3 {
  private class WorldStateImpl(id:BigInt)
      extends WorldState {
    def nextState = new WorldStateImpl(id + 1)
  }
  final def main(args:Array[String]):Unit = {
    val ioAction = iomain(args)
    ioAction(new WorldStateImpl(0));
  }
  def iomain(args:Array[String]):IOAction_v3[_]
}
The IOAction object is just a nice factory to create SimpleActions. SimpleAction's constructor takes a lazy expression as an argument, hence the "=> A" annotation. That expression won't be evaluated until SimpleAction's apply method is called. To call SimpleAction's apply method, a WorldState must be passed in. What comes out is a tuple with the new WorldState and the result of the expression.

IOActionオブジェクトはSimpleActionを生成する単なるファクトリです。

SimpleActionのコンストラクタは遅延評価の式を引数に取ります。それゆえ「=> A」の注釈となります。

この式はSimpleActionのapplyメソッドが呼び出されるまで評価されません。SimpleActionのapplyメソッドを呼び出すためには、WorldStateが渡されなければなりません。結果は新しいWorldStateと式の結果のタプルです。



jyukutyoコメント

「:」と型名の間に「=>」を入れることで、遅延評価渡し、すなわち無評価で実引数を渡すこともできる
Scala4階：関数

Here's what our IO methods look like now

今IOメソッドは次のようになります。

//file RTConsole.scala
object RTConsole_v3 {
  def getString = IOAction_v3(Console.readLine)
  def putString(s: String) =
    IOAction_v3(Console.print(s))
}

jyukutyoコメント

applyは特殊なメソッド。IOAction_v3(Console.readLine)の呼び出しはobject IOAction_v3のapplyメソッド呼び出しとなる。

applyメソッドを使うと，Array(...)の様に，返り値を持った関数の様にも使える．
object Length {
def apply(s: String): int = s.length
}
println(Length("Foo"))
とすれば，3が帰ってくる．Length.apply()とLength()が等価．
利点は簡単で，importすれば，objectを関数代わりに使えるという事．ユーティリティメソッド代わりにできるかな．
航海日誌%282007-09-08%29
applyについて詳しい説明とサンプルはScalaのAdvanced Exampleを写経する(7)-custom 'apply/update' - Fight the Future じゅくのblogへ。


And finally our HelloWorld class doesn't change a bit

結局HelloWorldクラスは少しも変わっていません。

class HelloWorld_v3 extends IOApplication_v3 {
  import RTConsole_v3._
  def iomain(args:Array[String]) =
    putString("Hello world")
}
A little thought shows that there's no way to create an Evil IOApplication now. A programmer simply has no access to a WorldState. It has become totally sealed away. The main driver will only pass a WorldState to an IOAction's apply method, and we can't create arbitrary IOAction subclasses with custom definitions of apply.

ちょっとした見解として、今Evil IOApplicationを生成する手段はありません。

プログラマは単純にWorldStateへアクセスできません。すべて隠蔽されています。

main関数はWorldStateをIOActionのapplyメソッドに渡すだけであり、独自のapplyを定義した任意のIOActionのサブクラスを作成することはできません。


Unfortunately, we've got a combining problem. We can't combine multiple IOActions so we can't do something as simple as "What's your name", Bob, "Hello Bob."

不幸なことに、結びついた問題があります。複数のIOActionを組み合わせることができないため、「名前は何ですが」「ボブです」「やあボブ」のような単純なことができません。


Hmmmm, IOAction is a container for an expression and monads are containers. IOAction needs to be combined and monads are combinable. Maybe, just maybe...

んー、IOActionは式のためのンテナであり、モナドはコンテナです。

IOActionは組み合わせる必要があり、モナドは組み合わせ可能です。そうですね、もしかしたら。。。











Ladies and Gentleman I Present the Mighty IO Monad(みなさん、すばらしいIOモナドを紹介します)


The IOAction.apply factory method takes an expression of type A and returns an IOAction[A]. It sure looks like "unit." It's not, but it's close enough for now. And if we knew what flatMap was for this monad then the monad laws would tell us how to create map using it and unit. But what's flatMap going to be? The signature needs to look like def flatMap[B](f: A=>IOAction[B]):IOAction[B]. But what does it do?

IOActionのファクトリメソッドapplyは引数にA型の式をとりIOAction[A]を返します。たしかに「unit」のように見えます、そうではないのですが、今はだいたい同じでいいです。

もしflatMapがこのモナドのためにすることを知っているならモナド則は我々にそれとunitを使ってマップを作成する方法を教えてくれます。

しかし何がflatMapであるべきでしょう？シグネチャはdef flatMap[B](f: A=>IOAction[B]):IOAction[B]のようなものを必要とします。しかし、それは何をするのでしょう？


What we want it to do is chain an action to a function that returns an action and when activated causes the two actions to occur in order. In other words, getString.flatMap{y => putString(y)} should result in a new IOAction monad that, when activated, first activates the getString action then does the action that putString returns. Let's give it a whirl.

それにやらせたいことはアクションを返す関数のための連鎖したアクションであり、活性化したとき2つのアクションを順に実行するものです。言い換えると、getString.flatMap{y => putString(y)}は新しいIOActionモナドとなるべきであり、活性化されると最初にgetStringアクションを活性化しそれからputStringが返すアクションを実行します。試してみましょう。

//file RTIO.scala
sealed abstract class IOAction_v4[+A] extends
    Function1[WorldState, (WorldState, A)] {
  def map[B](f:A => B):IOAction_v4[B] =
    flatMap {x => IOAction_v4(f(x))}
  def flatMap[B](f:A => IOAction_v4[B]):IOAction_v4[B]=
    new ChainedAction(this, f)

  private class ChainedAction[+A, B](
      action1: IOAction_v4[B],
      f: B => IOAction_v4[A]) extends IOAction_v4[A] {
    def apply(state1:WorldState) = {
      val (state2, intermediateResult) =
        action1(state1);
      val action2 = f(intermediateResult)
      action2(state2)
    }
  }
}

object IOAction_v4 {
  def apply[A](expression: => A):IOAction_v4[A] =
    new SimpleAction(expression)

  private class SimpleAction[+A](expression: => A)
      extends IOAction_v4[A] {
    def apply(state:WorldState) =
      (state.nextState, expression)
  }
}

// the rest remains the same
sealed trait WorldState{def nextState:WorldState}

abstract class IOApplication_v4 {
  private class WorldStateImpl(id:BigInt) ...

The IOAction factory and SimpleAction remain the same. The IOAction class gets the monad methods. Per the monad laws, map is just defined in terms of flatMap and what we're using as unit for now. flatMap defers all the hard work to a new IOAction implementation called ChainedAction.

IOActionのファクトリとSimpleActionは変わっていません。IOActionはモナドメソッドを加えました。モナド則に従って、mapは単にflatMapとさしあたりunitとして使っているものを利用して実装しています。flatMapはChainedActionという新しいIOActionの実装への難しい責務を任せています。


The trick in ChainedAction is its apply method. First it calls action1 with the first world state. This results in a second world state and an intermediate result. The function it was chained to needs that result and in return the function generates another action: action2. action2 is called with the second world state and the tuple that come out is the end result. Remember that none of this will happen until the main driver passes in an initial WorldState object.

ChainedActionにおけるトリックはそのapplyメソッドです。初めaction1を1番目の世界の状態で呼び出します。

これは2番目の世界の状態と中間結果が結果となります。それがつなぐ関数は結果を必要とし、戻り値として他のアクションであるaction2を生成します。

action2は2番目の席あの状態とともに呼び出し、出てくるタプルを最終結果とします。

内部的なWorldStateオブジェクトをmain関数に渡さない限りこのどれもが起こらないことを覚えておいてください。


A Test Drive(テスト駆動)


At some point you may have wondered why getString and putString weren't renamed to something like createGetStringAction/createPutStringAction since that's in fact what they do. For an answer, look at what happens when we stick 'em in our old friend "for".

getStringとputStringがcreateGetStringAction/createPutStringActionのような何かになぜ名前を変更しないのかある時点で不思議に思うかもしれません。

実際それらがやっていることがそうだからです。

object HelloWorld_v4 extends IOApplication_v4 {
  import RTConsole_v4._
  def iomain(args:Array[String]) = {
    for{
        _ <- putString(
            "This is an example of the IO monad.");
        _ <- putString("What's your name?");
        name <- getString;
        _ <- putString("Hello " + name)
    } yield ()
  }
}
It's as if "for" and getString/putString work together to create a mini language just for creating a complex IOActions.

これはまるで「for」とgetString/putStringが複雑なIOActionを作成するためだけにミニ言語を作成しようとともに動いているかのようです。


Take a Deep Breath(深呼吸しましょう)


Now's a good moment to sum up what we've got. IOApplication is pure plumbing. Users subclass it and create a method called iomain which is called by main. What comes back is an IOAction - which could in fact be a single action or several actions chained together. This IOAction is just "waiting" for a WorldState object before it can do its work. The ChainedAction class is responsible for ensuring that the WorldState is changed and threaded through each chained action in turn.

さて我々が成し遂げたことを総括するいい機会です。IOApplicationは純粋な配管です。

ユーザーがそのサブクラスを作りmainから呼び出されるiomainというメソッドを作成します。

IOActionを言い換えましょう。実際IOActionは単一のアクションでも連結したいくつかのアクションでもあるかもしれません。このIOActionはその作業をする前にWorldStateオブジェクトを「待って」いるだけです。

ChainedActionクラスはWorldStateが順番に各連結したアクションを通じて変更され通されたことを保証する責務があります。


getString and putString don't actually get or put Strings as their names might indicate. Instead, they create IOActions. But, since IOAction is a monad we can stick it into a "for" statement and the result looks as if getString/putString really do what they say the do.

getStringとputStringはその名前が指し示すように実際にStringを取得したり設定したりしません。代わりにIOActionを生成します。しかしIOActionはモナドであるため、それを「for」構文に突き通すことができますし結果はまるでgetString/putStringが実際に名前の通りのことをやっているかのように見えます。


It's a good start; we've almost got a perfectly good monad in IOAction. We've got two problems. The first is that, because unit changes the world state we're breaking the monad laws slightly (e.g. m flatMap unit === m). That's kinda trivial in this case because it's invisible. But we might as well fix it.

いいスタートです。IOActionにおけるほぼ完璧なできのモナドを手に入れました。問題が2つあります。1つめはunitが世界の状態を変更してしまうのでモナド則を少し破っているということです(たとえばm flatMap unit === m)。この場合見えないことなので取るに足らないことです。しかし、これも対応できるでしょう。


The second problem is that, in general, IO can fail and we haven't captured that just yet.

2つめの問題は、一般にIOは失敗し、それをすぐに捉えることができないということです。


IO Errors(IOエラー)


In monadic terms, failure is represented by a zero. So all we need to do is map the native concept of failure (exceptions) to our monad. At this point I'm going to take a different tack from what I've been doing so far: I'll write one final version of the library with comments inline as I go.

モナド的な見地では、失敗はゼロで表現します。ゆえに必要なことは失敗(例外)の固有の概念をモナドに結びつけるだけです。この点では、これまでやってきたこととは異なる方針をとります。インラインでコメントをつけたこのライブラリの最終バージョンを書きます。


The IOAction object remains a convenient module to hold several factories and private implementations (which could be anonymous classes, but it's easier to explain with names). SimpleAction remains the same and IOAction's apply method is a factory for them.

IOActionオブジェクトはいくつかのファクトリとプライベートな実装(それらは無名クラスかもしれませんが、名前で説明する方が簡単です)を保持する便利なモジュールとして残っています。SimpleActionも同様であり、IOActionのapplyメソッドはそれらのファクトリです。

//file RTIO.scala
object IOAction {
  private class SimpleAction[+A](expression: => A)
      extends IOAction[A] {
    def apply(state:WorldState) =
      (state.nextState, expression)
  }

  def apply[A](expression: => A):IOAction[A] =
    new SimpleAction(expression)
UnitAction is a class for unit actions - actions that return the specified value but don't change the world state. unit is a factory method for it. It's kind of odd to make a distinction from SimpleAction, but we might as well get in good monad habits now for monads where it does matter.

UnitActionはunitアクションのためのクラスです。unitアクションは指定された値を返すが世界の状態は変更しないアクションです。unitはそれのためのファクトリメソッドです。SimpleActionと区別させることはやや奇妙ですが、その上に我々は今優れたモナドにおけるモナドを重要足らしめているところのものの性質がわかるかもしれません。

  private class UnitAction[+A](value: A)
      extends IOAction[A] {
    def apply(state:WorldState) =
      (state, value)
  }

  def unit[A](value:A):IOAction[A] =
    new UnitAction(value)
FailureAction is a class for our zeros. It's an IOAction that always throws an exception. UserException is one such possible exception. The fail and ioError methods are factory methods for creating zeroes. Fail takes a string and results in an action that will raise a UserException whereas ioError takes an arbitrary exception and results in an action that will throw that exception.

FailureActionはゼロのためのクラスです。それは常に例外をスローするIOActionです。UserExceptionはそういう例外の1つです。failとioErrorメソッドはゼロを生成するファクトリメソッドですioErrorが任意の例外を引数に取りその例外をスローするアクションを返すのに対して、failは文字列を引数に取りUserExceptionを発生させるアクションを返します。

  private class FailureAction(e:Exception)
      extends IOAction[Nothing] {
    def apply(state:WorldState) = throw e
  }

  private class UserException(msg:String)
    extends Exception(msg)

  def fail(msg:String) =
    ioError(new UserException(msg))
  def ioError[A](e:Exception):IOAction[A] =
    new FailureAction(e)
}
IOAction's flatMap, and ChainedAction remain the same. Map changes to actually call the unit method so that it complies with the monad laws.

IOActionのflatMapとChainedActionは変わっていません。mapは実はunitメソッドを呼び出すように変わりました。モナド則を満たすためです。


I've also added two bits of convenience: >> and <<. Where flatMap sequences this action with a function that returns an action, >> and << sequence this action with another action.

また便利なものを2つばかり追加しました。>>と<<です。flatMapはアクションを返す関数とともにこのアクションを順序づけるのに対して、>>と<<はこのアクションを他のアクションとともに順序づけます。


It's just a question of which result you get back. >>, which can be pronounced "then", creates an action that returns the second result, so 'putString "What's your name" >> getString' creates an action that will display a prompt then return the user's response.

どちらの結果を戻すのかという疑問があります。>>は、「then」と発音しますが、2つ目の結果を返すアクションを生成するので、「putString "What's your name" >> getString」はプロンプトを表示しユーザーのレスポンスを戻すアクションを生成します。


Conversely, <<, which can be called "before" creates an action that will return the result from the first action.

逆に、<<は、「before」と呼びますが、最初のアクションから結果を戻すアクションを生成します。

sealed abstract class IOAction[+A]
    extends Function1[WorldState, (WorldState, A)] {
  def map[B](f:A => B):IOAction[B] =
    flatMap {x => IOAction.unit(f(x))}
  def flatMap[B](f:A => IOAction[B]):IOAction[B]=
    new ChainedAction(this, f)

  private class ChainedAction[+A, B](
      action1: IOAction[B],
      f: B => IOAction[A]) extends IOAction[A] {
    def apply(state1:WorldState) = {
      val (state2, intermediateResult) =
        action1(state1);
      val action2 = f(intermediateResult)
      action2(state2)
    }
  }

  def >>[B](next: => IOAction[B]):IOAction[B] =
    for {
      _ <- this;
      second <- next
    } yield second

  def <<[B](next: => IOAction[B]):IOAction[A] =
    for {
      first <- this;
      _ <- next
    } yield first
Because we've got a zero now, it's possible to add a filter method by just following the monad laws. But here I've created two forms of filter method. One takes a user specified message to indicate why the filter didn't match whereas the other complies with Scala's required interface and uses a generic error message.

今ゼロを得たので、モナド則に従うだけでフィルターメソッドを追加することができます。しかしここで2つの形式のフィルターメソッドを作成しました。1つはフィルターがマッチしなかった理由を示すためにユーザーが指定したメッセージを引数に取りますが、もう1つはSからが必要とするインターフェースを満たし一般的なエラーメッセージを使います。

  def filter(
      p: A => Boolean,
      msg:String):IOAction[A] =
    flatMap{x =>
      if (p(x)) IOAction.unit(x)
      else IOAction.fail(msg)}
  def filter(p: A => Boolean):IOAction[A] =
    filter(p, "Filter mismatch")
A zero also means we can create a monadic plus. As some infrastructure for creating it, HandlingAction is an action that wraps another action and if that action throws an exception then it sends that exception to a handler function. onError is a factory method for creating HandlingActions. Finally, "or" is the monadic plus. It basically says that if this action fails with an exception then try the alternative action.

ゼロはまたモナド的な加算を作成することができるということを意味します。それを作成するためのある基盤として、HandlingActionは他のアクションをラップしそのアクションが例外をスローすればその例外をハンドラーに渡すというアクションです。

onErrorはHandlingActionを生成するファクトリメソッドです。最後に、「or」はモナド的な加算です。それは基本的にこのアクションがもし例外とともに失敗すれば代わりのアクションを試すということを述べています。

  private class HandlingAction[+A](
      action:IOAction[A],
      handler: Exception => IOAction[A])
      extends IOAction[A] {
    def apply(state:WorldState) = {
      try {
        action(state)
      } catch {
        case e:Exception => handler(e)(state)
      }
    }
  }

  def onError[B >: A](
      handler: Exception => IOAction[B]):
      IOAction[B] =
    new HandlingAction(this, handler)

  def or[B >: A](
      alternative:IOAction[B]):IOAction[B] =
    this onError {ex => alternative}
}
The final version of IOApplication stays the same

IOApplicationの最終バージョンも変わりません。

sealed trait WorldState{def nextState:WorldState}

abstract class IOApplication {
  private class WorldStateImpl(id:BigInt)
      extends WorldState {
    def nextState = new WorldStateImpl(id + 1)
  }
  final def main(args:Array[String]):Unit = {
    val ioaction = iomain(args)
    ioaction(new WorldStateImpl(0));
  }
  def iomain(args:Array[String]):IOAction[_]
}
RTConsole stays mostly the same, but I've added a putLine method as an analog to println. I've also changed getString to be a val. Why not? It's always the same action.

RTConsoleはほとんど変わっていませんが、printlnの類似のものとしてputLineメソッドを追加しました。またgetStringをvalへ変えました。なぜ？常に同じアクションだからです。

//file RTConsole.scala
object RTConsole {
  val getString = IOAction(Console.readLine)
  def putString(s: String) =
    IOAction(Console.print(s))
  def putLine(s: String) =
    IOAction(Console.println(s))
}
And now a HelloWorld application to exercise some of this new functionality. sayHello creates an action from a string. If the string is a recognized name then the result is an appropriate (or inappropriate) greeting. Otherwise it's a failure action.

さあHelloWorldアプリケーションでこの新しい機能性のいくつかを試してみましょう。

sayHelloは文字列から空くshんを生成します。もし文字列が名前として認識できるなら結果は適切な(もしくは不適切な)あいさつになります。そうでなければアクションは失敗します。


Ask is a convenience method that creates an action that will display a specified string then get one. The >> operator ensures that the action's result will be the result of getString.

askは指定した文字列を表示してそれを取得するアクションを生成する便利なメソッドです。>>演算子はアクションの結果がgetStringの結果であることを確かめます。


processsString takes an arbitrary string and, if it's 'quit' then it creates an action that will say goodbye and be done. On any other string sayHello is called. The result is combined with another action using 'or' in case sayHello fails. Either way the action is sequenced with the loop action.

processsStringは任意の文字列を引数に取り、もしそれが「quit」ならさようならを言うアクションを生成します。他の文字列ならsayHelloを呼び出します。結果はsayHelloが失敗した場合「or」を使って他のアクションと組み合わせます。


Loop is interesting. It's defined as a val just because it can be - a def would work just as well. So it's not quite a loop in the sense of being a recursive function, but it is a recursive value since it's defined in terms of processString which in turn is defined based on loop.

ループは興味深いです。そうすることができるという理由だけでvalとして定義しています。defと同様に動作します。ゆえに再帰関数であるという意味においては完全にはループではありませんが、再帰的な値であるので今度はループに基づくものとして定義されているprocessStringの見地から定義されます。


The iomain function kicks everything off by creating an action that will display an intro then do what the loop action specifies.

iomain関数はイントロを表示しループアクションが指定することを実行するアクションを生成してすべてを開始します。


Warning: because of the way the library is implemented this loop will eventually blow the stack. Do not use it in production code. Read the comments to see why.

警告：ライブラリを実装した手段により、このループは最終的にスタックを破壊するかもしれません。プロダクションコードでこれを使わないでください。理由はコメントを読んでみてください。

object HelloWorld extends IOApplication {
  import IOAction._
  import RTConsole._

  def sayHello(n:String) = n match {
    case "Bob" => putLine("Hello, Bob")
    case "Chuck" => putLine("Hey, Chuck")
    case "Sarah" => putLine("Helloooo, Sarah")
    case _ => fail("match exception")
  }

  def ask(q:String) =
    putString(q) >> getString

  def processString(s:String) = s match {
    case "quit" => putLine("Catch ya later")
    case _ => (sayHello(s) or
        putLine(s + ", I don't know you.")) >>
        loop
  }

  val loop:IOAction[Unit] =
    for {
      name <- ask("What's your name? ");
      _ <- processString(name)
    } yield ()

  def iomain(args:Array[String]) = {
    putLine(
        "This is an example of the IO monad.") >>
    putLine("Enter a name or 'quit'") >>
    loop
  }
}

jyukutyoコメント

理由というのはこのコメントだと思う。

As for loop not being tail recursive - well, it can't be. The reason is a bit subtle. Loop isn't quite a normal loop. Instead, ultimately, it's an instance of ChainedAction. That's where the real problem is: as the library is designed ChainedAction's apply method cannot be tail recursive since its tail call must be to some arbitrary IOAction's apply method rather than to its own apply method.
One Div Zero: Monads are Elephants Part 4
要は、loopが末尾再帰ではないからというのが理由らしい。ループは単なるループではなくて、結局はChainedActionのインスタンスである。ChainedActionのapplyメソッドは末尾再帰にできない。自身のapplyメソッドではなく任意のIOActionのapplyメソッド呼び出しでなければならない、と。

末尾再帰でない以上、その関数呼び出しから戻ってきた後の処理の情報をスタックに保存するので、再帰が深くなればスタックの使用量が増え、いつかはあふれるということ。


Conclusion for Part 4(パート4の結論)


In this article I've called the IO monad 'IOAction' to make it clear that instances are actions that are waiting to be performed. Many will find the IO monad of little practical value in Scala. That's okay, I'm not here to preach about referential transparency. However, the IO monad is one of the simplest monads that's clearly not a collection in any sense.

この記事ではインスタンスが実行を待つアクションであることをはっきりさせるためにIOモナドを「IOAction」と呼びました。ScalaにおけるIOモナドの実践的な価値を少しわかっていただけたでしょう。それで大丈夫です。私はここで参照透過性について説こうとしているわけではありません。しかしながら、IOモナドはいかなる意味においても明らかにコレクションでないモナドのもっとも単純な1つです。


Still, instances of the IO monad can be seen as containers. But instead of containing values they contain expressions. flatMap and map in essence turn the embedded expressions into more complex expressions.

まだIOモナドのインスタンスをコンテナとして見るかもしれません。しかし値を格納する代わりにIOモナドは式を格納します。flatMapとmapは本質的に埋め込まれた式をより複雑な式へ変換します。


Perhaps a more useful mental model is to see instances of the IO monad as computations or functions. flatMap can be seen as applying a function to the computation to create a more complex computation.

おそらくより役に立つ観念的なモデルはIOモナドのインスタンスを計算や関数としてみることです。flatMapはより複雑な計算を生成するために計算に関数を適用するものとして見ることができます。


In the last part of this series I'll cover a way to unify the container and computation models. But first I want to reinforce how useful monads can be by showing an application that uses an elephantine herd of monads to do something a bit more complicated.

この連載の最後のパートではコンテナと計算モデルを統合する手段をカバーします。しかし最初により少し複雑な何かをするためにモナドの象の群れを使ったアプリケーションを見せることによってモナドがどれだけ役に立つのかを述べたいです。
