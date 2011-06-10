「モナドは象だ」 Part4
========================================================================

大人の象を直接見るまでは、本当に象がどれだけ大きいのか理解したことにはなりません。
モナドを象に例えると、この連載ではまだListとOptionのような幼い象を紹介したにすぎません。
しかし機は熟しました。大人の象を見ていきましょう。ボーナスとして少しサーカスマジックもさせます。


関数型プログラミングとIO
------------------------------------------------------------------------

関数型プログラミングには、参照透過性という概念があります。参照透過性は、ある関数を同じ引数で呼び出した結果は、いつどこで呼び出そうとも同じ結果になる、ということを意味します。参照透明な関数は、そうでない関数よりもより簡単に扱えてデバッグしやすいものになります。[#referential_transparency]_

参照透過性を達成できない分野がひとつあります。それはIOです。
コンソールから一行読み出すreadLine関数を何回か呼び出しても、それぞれユーザーが朝食になにを食べたかによって異なる文字列を返すかも知れません。ネットワークへパケットを送ることは、送信が成功するかわかりません。

しかし、参照透明性を保つために完全にIOを取り除くことはできません。IOを行わないプログラムは、単に複雑な方法でCPUを回しているにすぎないからです。

この連載のトピックから、 モナドは参照透明なIOを行うための手段だと思うかも知れません。とりあえずは、単純な原則から構築してみようと思います。文字列をコンソールから読み書きする問題を考えてみます。この問題の解決方法は、同じようにファイルやネットワークのような任意のIOにも拡張して適用することができるからです。

もちろん、参照透過なIOがScalaに置いて重要であるとは思わなくても構いません。純粋に関数的な参照透明が、ただひとうの真実の道であると押しつける訳でもありません。ここではモナドについて話をしているのであって、IOモナドを説明することが、他のモナドがどのように働くかについて説明しやすいから例にあげているだけです。


カップの中の世界
------------------------------------------------------------------------

コンソールから文字列を読み出すことは、参照透過ではありません。readLine関数はユーザーの状態に依存し、「ユーザー」は関数の引数ではないからです。ファイルを読み込む関数はファイルシステムの状態に依存しますし、Webページを読み出す関数は対象のWebサーバーやインターネット、ローカルネットワークの状態に依存します。同様に、出力を行う関数も同じような依存性を持っています。

これは全て、全世界の状態を表すWorldStateというクラスを用意して、全てのIO関数の引数と戻り値をこのWorldState型にすれば話は終わります。しかし、残念ながら世界は広大です。WorldStateを書こうと試みても、メモリ不足でコンパイラがクラッシュするという結果を迎えるだけです。なので、全世界をモデル化するよりはもう小さい単位で試してみましょう。ちょっとしたサーカスマジックが出てきましたよ。

この手品は、世界のいくつかの側面だけをモデル化して、残りはWorldStateが知っているように装います。このWorldStateには、いくつかの役に立つ側面があります。


- 世界の状態は、IO関数の間で変化する
- 成果の状態は、そのまま取り扱うしかない。新しい世界を作りだすような、(val coolWorldState = new WorldState(){def jamesIsBillionaire = true})のような事はできない。
- 世界はどんな瞬間でも厳密にひとつの状態に定まる

3つめの特性は、少しトリッキーなので、まずは特性1と2から考えましょう。



特性1のラフスケッチです。

.. code-block:: scala

  //file RTConsole.scala
  object RTConsole_v1 {
    def getString(state: WorldState) =
      (state.nextState, Console.readLine)

    def putString(state: WorldState, s: String) =
      (state.nextState, Console.print(s) )
  }

getStringとputStringは、scala.Consoleオブジェクトに定義されている低レベルなプリミティブ関数を用います。これらは、世界の状態を引数で受け取って、新しい世界の状態とプリミティブなIOの結果からなるタプルを返します。

ここで、特性2を実装しました。

.. code-block:: scala

  //file RTIO.scala
  sealed trait WorldState{def nextState:WorldState}

  abstract class IOApplication_v1 {
    private class WorldStateImpl(id:BigInt) extends WorldState {
      def nextState = new WorldStateImpl(id + 1)
    }

    final def main(args:Array[String]):Unit = {
      iomain(args, new WorldStateImpl(0))
    }

    def iomain( args:Array[String], startState:WorldState):(WorldState, _)
  }

WorldState型は、seald traitとします。sealed指定されているので、同じファイル内に定義されているクラスしかWorldStateを継承できなくなります。IOApplicationは(WorldStateの)唯一の実装をprivateで定義するため、他の誰もがWorldStateをインスタンス化することはできません。IOApplicationでは、サブクラスで必ず実装する必要のある抽象メソッドiomain関数と、それを呼び出すmain関数(finalなのでオーバライドはできません)が定義されています。これらの定義はすべて、このIOライブラリを利用するプログラマから何を行っているかを隠蔽するための配管と言えます。

これで、Hello worldはこのようになります。


.. code-block:: scala

  // file HelloWorld.scala
  class HelloWorld_v1 extends IOApplication_v1 {
    import RTConsole_v1._

    def iomain( args:Array[String], startState:WorldState) =
      putString(startState, "Hello world")
  }

最悪の特性3
------------------------------------------------------------------------

3つめの特性は、世界はあらゆる瞬間でひとつの状態だけである、と言っています。これは、まだ解決できていません。 なぜなら問題があるからです。

.. code-block:: scala

  class Evil_v1 extends IOApplication_v1 {
    import RTConsole_v1._

    def iomain( args:Array[String], startState:WorldState) = {
      val (stateA, a) = getString(startState)
      val (stateB, b) = getString(startState)
      assert(a == b)
      (startState, b)
    }
  }

ここで、getStringを同じ引数で2回呼び出しています。もしこのgetString関数が参照透過でならば、それぞれの結果のaとbは等しくなるべきですが、当然のことながらユーザーが2回同じ入力を行わない限りそうはなりません。この問題は、「startState」が、それぞれ異なる世界の状態であるstateAとstateBとして同時に見えているからです。


Inside Out
------------------------------------------------------------------------

解決への第1歩として、すべてを裏返してみます。 iomainをWorldStateを取ってWorldStateを返す関数とする代わりに、iomainはそのような関数オブジェクトを返すようにし、mainはiomainが返した関数オブジェクトを実行するようにします。コードはこうなります。

.. code-block:: scala

  //file RTConsole.scala
  object RTConsole_v2 {
    def getString = {state:WorldState => (state.nextState, Console.readLine)}

    def putString(s: String) = {state: WorldState => (state.nextState, Console.print(s))}
  }

getStringとputStringは、もはや文字列をget/putしません。かわりに、WorldStateが渡されるまで実行を「待つ」関数を、毎回作り出して返します。


.. code-block:: scala

  //file RTIO.scala
  sealed trait WorldState{def nextState:WorldState}

  abstract class IOApplication_v2 {

    private class WorldStateImpl(id:BigInt) extends WorldState {
      def nextState = new WorldStateImpl(id + 1)
    }

    final def main(args:Array[String]):Unit = {
      val ioAction = iomain(args)
      ioAction(new WorldStateImpl(0));
    }

    def iomain(args:Array[String]): WorldState => (WorldState, _)
  }


IOApplicationのmain関数は、実行する関数をiomain関数を呼び出して取得して、初期状態のWorldStateを渡して取得した関数を実行します。先ほどのHelloWorldは、WorldStateを引数に取らないようにする以外の変更は行いません。

.. code-block:: scala

  //file HelloWorld.scala
  class HelloWorld_v2 extends IOApplication_v2 {
    import RTConsole_v2._

    def iomain(args:Array[String]) = putString("Hello world")
  }

HelloWorldの中にWorldStateが見つからなくなったので、一見問題は解決したかのように見えます。 しかし、単に隠されているだけだとわかります。


ああ、最悪の特性3
------------------------------------------------------------------------

.. code-block:: scala

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

Evliでは、iomainが正確に期待される関数を返すようになっていますが、未だにこの実装は壊れています。プログラマが任意にIO関数を作成できるようになっている限り、WorldStateが隠蔽されているというトリックが、getStringやputStringを呼び出すことで見破られてしまうからです。


Property 3 Squashed For Good
------------------------------------------------------------------------

プログラマが任意のIO関数を正しいシグニチャで作成できないようにする必要があります。うーん、今何をする必要があるでしょう？

さて、WorldStateで見たように、サブクラスを作成できないようにすることは簡単です。では、関数のシグネチャをtraitに変えてみましょう。

.. code-block:: scala

  sealed trait IOAction[+A] extends Function1[WorldState, (WorldState, A)]

  private class SimpleAction[+A]( expression: => A) extends IOAction[A] ...


WorldStateと異なる点は、IOActionのインスタンスを作成できるようにすることです。例えば、getStringとputStringは異なるファイルに存在するかも知れないのですが、そこから安全に新しいIOActionのインスタンスを生成できるようにする必要があるでしょう。ここで、getStringとputStringは異なる2つの部分に分割されていることを理解しない限り、ちょっとしたジレンマに陥ってしまします。getString/putStringは、プリミティブなIOを行う部分と、入力された世界の状態を次の状態に変える部分の2つから成っています。ちょっとしたファクトリーメソッドで物事を整理する手助けをしましょう。


.. code-block:: scala

  //file RTIO.scala
  sealed trait IOAction_v3[+A] extends Function1[WorldState, (WorldState, A)]

  object IOAction_v3 {

    def apply[A](expression: => A):IOAction_v3[A] = new SimpleAction(expression)

    private class SimpleAction [+A]( expression: => A) extends IOAction_v3[A] {
      def apply(state:WorldState) = (state.nextState, expression)
    }
  }

  sealed trait WorldState{def nextState:WorldState}

  abstract class IOApplication_v3 {

    private class WorldStateImpl(id:BigInt) extends WorldState {
      def nextState = new WorldStateImpl(id + 1)
    }

    final def main(args:Array[String]):Unit = {
      val ioAction = iomain(args)
      ioAction(new WorldStateImpl(0));
    }

    def iomain(args:Array[String]):IOAction_v3[_]
  }


IOActionオブジェクトはSimpleActionを生成する単なるファクトリです。 SimpleActionのコンストラクタは遅延評価の式を引数に取ります。それゆえ、引数の型は「=> A」と表記されています。[#call_by_name]_ 引数に渡した式は、SimpleActionのapplyメソッド[#apply]_ が呼び出されるまで評価されません。そして、SimpleActionのapplyメソッドにはWorldStateを渡す必要があります。返り値は、新しいWorldStateと、式を評価した結果からなるタプルです。

ここで、IOメソッドは次のようになります。

.. code-block:: scala

  //file RTConsole.scala
  object RTConsole_v3 {

    def getString = IOAction_v3(Console.readLine)
    def putString(s: String) = IOAction_v3(Console.print(s))
  }


結局HelloWorldクラスは少しも変わっていません。

.. code-block:: scala

  class HelloWorld_v3 extends IOApplication_v3 {
    import RTConsole_v3._

    def iomain(args:Array[String]) = putString("Hello world")
  }


これで、前に示した'Evil'なIOApplicationを作る手段はなくなりました。プログラマはWorldStateへアクセスできません。全て完全に隠蔽されています。main関数はWorldStateをIOActionのapplyメソッドに渡すだけになり、独自のapplyを定義したIOActionのサブクラスを任意に作成することはできなくなりました。

不幸なことに、結合に問題があります。複数のIOActionを組み合わせることができないため、 「名前は何ですが」「ボブです」「やあボブ」のような単純なことができません。

んー、IOActionは式のためのコンテナであり、モナドはコンテナです。 IOActionは組み合わせる必要があり、モナドは組み合わせ可能です。そうですね、もしかしたら。。。


みなさん、すばらしいIOモナドを紹介します
------------------------------------------------------------------------

IOActionのファクトリメソッドapplyは、引数にA型の式をとりIOAction[A]型を返します。これは、たしかに「unit」のように見えます。実は違うのですが、今のところは同じものだと思っていいです。
もし、このモナドにとってflatMapがどのようなものかわかれば、モナド則によりflatMapとunitを使ってmapの定義を導出することができます。
しかし、flatMapはどうあるべきでしょうか? シグニチャは"def flatMap[B](f: A=>IOAction[B]):IOAction[B]"となりますが、ここで何を行うのしょう?

私たちが今flatMapに望んでいることは、アクションとアクションをつなげた新しい関数を返すことと、呼び出されたときにそのふたつを順番に実行することです。
言い換えると、"getString.flatMap{y => putString(y)}"は新しいIOActionモナドとなり、呼び出されるとまずgetStringアクションを実行して、putStringが返すアクションを実行します。試してみましょう。

.. code-block:: scala

  //file RTIO.scala
  sealed abstract class IOAction_v4[+A] extends Function1[WorldState, (WorldState, A)] {

    def map[B](f:A => B):IOAction_v4[B] = flatMap {x => IOAction_v4(f(x))}

    def flatMap[B](f:A => IOAction_v4[B]):IOAction_v4[B] = new ChainedAction(this, f)

    private class ChainedAction[+A, B]( action1: IOAction_v4[B], f: B => IOAction_v4[A])
      extends IOAction_v4[A] {

      def apply(state1:WorldState) = {
        val (state2, intermediateResult) = action1(state1)
        val action2 = f(intermediateResult)
        action2(state2)
      }
    }
  }

  object IOAction_v4 {
    def apply[A](expression: => A):IOAction_v4[A] = new SimpleAction(expression)

    private class SimpleAction[+A](expression: => A) extends IOAction_v4[A] {

      def apply(state:WorldState) = (state.nextState, expression)
    }
  }

  // the rest remains the same
  sealed trait WorldState{def nextState:WorldState}

  abstract class IOApplication_v4 {
    private class WorldStateImpl(id:BigInt) ...


IOActionのファクトリとSimpleActionはそのままでです。IOActionには、モナドのメソッドを追加しました。モナド則に従い、mapは単にflatMapと今のところそう見なしているunitを用いて定義しました。flatMapは難しい仕事をChainedActionという新しいIOActionの実装クラスに任せています。

このChainedActionでの仕掛けは、applyメソッドにあります。まず、action1を最初のWorldStateで呼び出します。この返り値は、2つめのWorldStateとA型の中間状態の結果(intermediateResult)からなるタプルです。つぎに、接続したい関数(引数fで受け取ったB => IOApplication_v4[A]型の関数)を、action1から得た中間状態(intermediateResult)を渡して呼び出して、action2を作り出します。このaction2に2番目のWorldState(action1で得られた結果)を渡して呼び出し、返ってきたタプルが、flatMapの最終的な結果になります。最初のWorldStateをmain関数から渡されない限り、何も起こらないということを覚えておいてください。


A Test Drive(テスト駆動)
------------------------------------------------------------------------

どうして、どこかの時点でgetStringとputStringという名前をcreateGetStringAction/createPutStringActionのような何を行うか表しているものに変更しないのか、疑問に思うかもしれません。
その答えとして、我々の古い友人である「for」にこれらを突き通してみると何が起こるか見てみましょう。


.. code-block:: scala

  object HelloWorld_v4 extends IOApplication_v4 {
    import RTConsole_v4._
    def iomain(args:Array[String]) = {
      for{
             _ <- putString( "This is an example of the IO monad.");
             _ <- putString("What's your name?");
          name <- getString;
             _ <- putString("Hello " + name)
      } yield ()
    }
  }



まるで、複雑なIOActionを利用するためのミニ言語が「for」とgetString/putStringでできている見えますね。


深呼吸しましょう
------------------------------------------------------------------------

さてここで、今まで行ってきたことをまとめてましょう。IOApplicationは(WorldStateを隠蔽するための)純粋な配管として機能します。ユーザーはIOApplicationを継承して、main関数から呼ばれるiomainメソッドを実装します。
ユーザーがそのサブクラスを作りmainから呼び出されるiomainというメソッドを作成します。そこから何を返すかというと、単体もしくは複数が連結されたIOActionです。このIOActionは、WorldStateが渡されるまで何もせずに「待って」いるだけです。ChainedActionは、連結されたアクションによるWorldStateの変更が、順番に一貫性を持って行われることを保証する責務を持っています。

getString/putStringは、その名前が示すように、実際に文字列を読み書きしません。代わりに、IOActionを作り出します。ですが、IOActionはモナドであるため、それらを「for」で利用すると、まるでgetString/putStringがその名の如く文字列の入出力を行っているかのように見えるのです。

幸先良く、ほぼ完全なモナドとしてのIOActionを用意できました。が、問題が有ります。最初の問題は、unitが世界の状態を変更してしまうので、(例えば m flatMap unit == m のような) モナド則を少し破ってしまっているということです。この場合は隠されているため、些細なことです。が、対応はできます。
2つめの問題は、一般的にIOは失敗することがあり、今のままではその失敗を補足できないことです。


IO Errors
------------------------------------------------------------------------

モナド的な意味で、失敗はゼロとして表現されます。今私たちがやりたいことは、失敗(例外)という固有の概念をこのモナドに導入することです。この点に置いては、今までとは異なるやり方で解説します。インラインでコメントをつけた、このライブライの最終バージョンを書こうと思います。

IOActionオブジェクトは、いくつかのファクトリとプライベートな実装(それらは無名クラスかもしれませんが、名前で説明する方が簡単です)を保持する便利なモジュールとして残っています。SimpleActionも同様であり、IOActionのapplyメソッドはそれらのファクトリです。

.. code-block:: scala

  //file RTIO.scala
  object IOAction {

    private class SimpleAction[+A](expression: => A) extends IOAction[A] {
      def apply(state:WorldState) = (state.nextState, expression)
    }

    def apply[A](expression: => A):IOAction[A] = new SimpleAction(expression)

UnitActionは、unitアクションのためのクラスです。unitアクションは、世界の状態は変更しないで、渡された値を返すだけのアクションです。unitはUnitActionのファクトリメソッドです。SimpleActionと区別しているのは少し奇妙に思うかも知れないですが、これによりモナドの性質の優れたところがわかるかもしれません。

.. code-block:: scala

    private class UnitAction[+A](value: A) extends IOAction[A] {
      def apply(state:WorldState) = (state, value)
    }

    def unit[A](value:A):IOAction[A] = new UnitAction(value)


FailureActionはゼロのためのクラスです。これは、常に例外を送出するIOActionとなります。UserExceptionは起こりえる例外の一種です。failとerrorメソッドは、ゼロを作り出すためのファクトリーメソッドです。failメソッドは文字列を取ってUserExceptionをthrowするIOActionを返すのに対して、ioErrorメソッドは任意の例外を取ってそれをthrowするIOActionを返します。

.. code-block:: scala

    private class FailureAction(e:Exception) extends IOAction[Nothing] {
      def apply(state:WorldState) = throw e
    }

    private class UserException(msg:String) extends Exception(msg)

    def fail(msg:String) = ioError(new UserException(msg))

    def ioError[A](e:Exception):IOAction[A] = new FailureAction(e)
  }


IOActionのflatMapとChainedActionは変わっていません。mapメソッドは、モナド則を満たすためにunitメソッドを呼び出すように変わりました。
また便利なものを2つばかり追加しました。「>>」と「<<」です。flatMapがアクションを返す関数とアクションを順番に結びつけるのに対して、「>>」と「<<」はアクションと他のアクションを順に結びつけます。そこで、「>>」が何を返すのかという疑問があります。「>>」 は"2つめのアクションの結果を返す"アクションを生成するので、 "then"と発音することもできます。ですので、「putString "What's your name" >> getString」はプロンプトを表示しユーザーの入力を返すアクションを生成します。
逆に、「<<」は"最初のアクションの結果を返す"アクションを返すので、"before"と呼ぶこともできます。

.. code-block:: scala

  sealed abstract class IOAction[+A] extends Function1[WorldState, (WorldState, A)] {

    def map[B](f:A => B):IOAction[B] = flatMap {x => IOAction.unit(f(x))}

    def flatMap[B](f:A => IOAction[B]):IOAction[B] = new ChainedAction(this, f)

    private class ChainedAction[+A, B]( action1: IOAction[B],
        f: B => IOAction[A]) extends IOAction[A] {

      def apply(state1:WorldState) = {
        val (state2, intermediateResult) = action1(state1);
        val action2 = f(intermediateResult)
        action2(state2)
      }
    }

    def >>[B](next: => IOAction[B]):IOAction[B] =
      for {
        _      <- this;
        second <- next
      } yield second

    def <<[B](next: => IOAction[B]):IOAction[A] =
      for {
        first <- this;
        _     <- next
      } yield first


ゼロを使えるようにしたので、ただモナド則に従うだけでフィルターメソッドを追加できるようになりました。ただし、ここでは2つの形式でフィルターメソッドを作成しています。1つは、なぜフィルターがマッチしなかったかを示すユーザー指定のメッセージを取ります。対して、もう一つは、シグニチャをScalaが一般的に要求する形式にしておき、("Filter mismatch"という)固定の一般的なエラーメッセージを用いるものです。

.. code-block:: scala

    def filter( p: A => Boolean, msg:String):IOAction[A] =
      flatMap{x => if (p(x)) IOAction.unit(x)
                   else IOAction.fail(msg)}

    def filter(p: A => Boolean):IOAction[A] =
      filter(p, "Filter mismatch")

また、ゼロはモナド的な加算を作成できること意味します。そのための基盤として、HandlingActionを用意します。これは、引数に他のアクションとhandler関数をとり、他のアクションをラップして例外が発生したらhandler関数に例外を渡して処理させる、というアクションです。 onErrorメソッドは、HandlingActionを作成するためのファクトリーメソッドです。そして「or」はモナドにおける加算です。ここでは、アクションがもし例外で失敗したら他のアクションを試す、という意味になります。

.. code-block:: scala

    private class HandlingAction[+A]( action:IOAction[A], handler: Exception => IOAction[A])
      extends IOAction[A] {

      def apply(state:WorldState) = {
        try {
          action(state)
        } catch {
          case e:Exception => handler(e)(state)
        }
      }
    }

    def onError[B >: A]( handler: Exception => IOAction[B]): IOAction[B] =
      new HandlingAction(this, handler)

    def or[B >: A]( alternative:IOAction[B]):IOAction[B] =
      this onError {ex => alternative}
  }

The final version of IOApplication stays the same

IOApplicationの最終バージョンには変化はありません。

.. code-block:: scala

  sealed trait WorldState{def nextState:WorldState}

  abstract class IOApplication {

    private class WorldStateImpl(id:BigInt) extends WorldState {
      def nextState = new WorldStateImpl(id + 1)
    }

    final def main(args:Array[String]):Unit = {
      val ioaction = iomain(args)
      ioaction(new WorldStateImpl(0));
    }

    def iomain(args:Array[String]):IOAction[_]
  }


RTConsoleはほとんど変わっていませんが、printlnと似たようなものであるputLineメソッドを追加しました。またgetStringをvalへ変えました。なぜかというと、常に同じアクションだからです。

.. code-block:: scala

  //file RTConsole.scala
  object RTConsole {

    val getString = IOAction(Console.readLine)

    def putString(s: String) = IOAction(Console.print(s))

    def putLine(s: String) = IOAction(Console.println(s))
  }


では、HelloWorldアプリケーションで、このあたらしい機能を試してみましょう。sayHelloメソッドは文字列からアクションを作成します。文字列が名前として認識できるものであれば、結果は適切な(または不適切な)挨拶を行うアクションになります。そうでなれば失敗するアクションを返します。

askメソッドは、指定した文字列を表示したあと文字列を入力から取得するための便利なメソッドです。「>>」演算子によって、アクションの返り値がgetStringの結果であることが保証されれます。

processStringメソッドは任意の文字列を引数に取ります。"quit"という文字列であればさよならを言うアクションを返します。それ以外なら、sayHelloメソッドを呼び出します。sayHelloが失敗した場合に備えて、結果は「or」によって他のアクションと結合されます。

processsStringは任意の文字列を引数に取り、もしそれが「quit」ならさようならを言うアクションを生成します。他の文字列ならsayHelloを呼び出します。結果はsayHelloが失敗した場合「or」を使って他のアクションと組み合わせます。どちらの場合にせよ、さらにloopアクションに繋がるようになっています。

loopは興味深いことに、 valとして定義されています。defと同様に動作します。 再帰関数であるという意味での完全なループではありませんが、processStringとloopは相互に呼び出し合って定義されているため、再帰的な値として考えることができます。

.. note::

  (訳注:いわゆる相互再帰(トランポリン)に近い形で定義されています。)

iomain関数は、イントロを表示してloopを呼び出すアクションを生成して、アプリケーションを開始します。


警告：このライブラリの実装では、ループによって最終的にスタックが破壊される可能性があります。プロダクションコードでこれを使わないでください。理由はコメントを読んでみてください。

.. code-block:: scala

  object HelloWorld extends IOApplication {
    import IOAction._
    import RTConsole._

    def sayHello(n:String) = n match {
      case "Bob"   => putLine("Hello, Bob")
      case "Chuck" => putLine("Hey, Chuck")
      case "Sarah" => putLine("Helloooo, Sarah")
      case _       => fail("match exception")
    }

    def ask(q:String) = putString(q) >> getString

    def processString(s:String) = s match {
      case "quit" => putLine("Catch ya later")
      case _      => (sayHello(s) or putLine(s + ", I don't know you.")) >> loop
    }

    val loop:IOAction[Unit] =
      for {
        name <- ask("What's your name? ");
        _    <- processString(name)
      } yield ()

    def iomain(args:Array[String]) = {
      putLine( "This is an example of the IO monad.") >>
      putLine("Enter a name or 'quit'") >>
      loop
    }
  }

.. note::

  (訳注)スタックを破壊する可能性があるというのはloopがprocessStringと相互に新しいアクションのインスタンスを作成しながら呼び出しあう形になるからです。Scalaでは、末尾再帰関数はフラットなwhileループになるようにコンパイル時に最適化されますが、ここではいわゆる相互再帰となっているため、loopの呼び出しが繰り返されるたびに新しいIOAcionインスタンスが作成され、スタックに積まれてゆくことになります。よって、最終的にはStackOverFlowを引き起こします。


パート4の結論
------------------------------------------------------------------------

この記事は、IOモナドを「実行を待つアクションというインスタンス」であることをハッキリとさせるために、「IOAction」と呼びました。 ScalaにおけるIOモナドの実践的な価値を少しわかっていただけたでしょう。それで充分です。ここでは参照透過性について説こうとしているわけではありません。ですが、IOモナドはいかなる意味においても明らかにコレクションでないモナドのもっとも単純な一例です。

IOモナドのインスタンスはコンテナのように見えますが、値の代わりに式を格納します。flatMapとmapは、埋め込まれた式をより複雑な式に変換しています。
より利便性の高い概念モデルは、IOモナドをインスタンスを一種の計算や関数と見なすことです。flatMapはより複雑な計算をつり出すために関数に計算を適用する、とも考えられます。


このシリーズの最後に、コンテナと計算モデルを統合する手段を解説しました。しかし私は、モナドがどれだけ役に立つかを、多くのモナド象を使った少し複雑なアプリケーションを見せることで伝えたかったのです。

.. rubric:: 訳注

.. [#referential_transparency] 参照透明ではない関数とは、呼び出し毎に結果が変わる関数です。例えば、ある変数の値をインクリメントして返すような関数は参照透明ではありません。参照透明ではない関数は、副作用を引き起こす可能性を含みます。

.. [#call_by_name] "=> A"は、call by name(名前渡し)と呼ばれる指定方法で、引数を遅延評価します。通常、foo( bar )という呼び出しではbarが評価された結果をfooの引数として渡しますが、fooメソッドの定義が"def foo( v: => String)"のように定義されていると、barをいわゆる無名関数に包んでfooメソッドに渡します。fooの内部では、barを評価することで結果を任意のタイミングで取り出せるようになります。

.. [#apply] "apply"というメソッド名はシンタックスシュガーで、fooObj("bar")というオブジェクトに対するメソッド呼び出しのような書き方で、applyメソッドを呼び出すことができます。ここでは、"IOAction_v3(Console.readLine)"という書き方で"IOAction_v3.apply(Console.readLine)"と同じ結果になります。
