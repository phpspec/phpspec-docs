PHPSpec でのスペックの記述
==========================

スペック、サンプル、そしてコンテキスト
---------------------------------

振舞駆動開発で使用する用語は、振る舞いを記述することを重視したものです。
これは、テスト駆動開発の用語が
(多くのプログラマにとって直感的ではないために)
引き起こしていたさまざまな誤解を軽減させることでしょう。

スペックとサンプルのふたつの用語は、ほぼ同じ意味で使用しています。
「スペック」は、通常は単一の振る舞いを表します。これは
"なにかをしなければならない (it should do something)"
という単純な文で表すことができます。一方「サンプル」は、
PHPSpec のメソッド全体を表します。つまり「スペック」
を表すメソッドのコードのことです。たとえば以下の例でいうと、コードの中の
``$this-&gt;spec()``
で始まる行のことを「スペック」、
そしてそのスペックが満たす仕様を書いたパブリックメソッド全体のことを
「サンプル」と呼びます。

PHPSpec のサンプルメソッドで記述したスペック
---------------------------------------

PHPSpec のサンプルメソッド:

.. code-block:: php

    public function itShouldHaveScoreOfZero()
    {
        $bowling = new Bowling;
        $bowling->hit(0);
        $this->spec($bowling->score)->should->be(0);
    }

さらに難しい概念が「コンテキスト」
です。要するにコンテキストとは、
振る舞いを定義する際に一般に使用する条件の集まりのことです。
上に示したボウリングの例では、
まずは新しいゲームを開始するところからはじめると仮定しています。
これは、そのクラスのすべてのスペックがが共有するコンテキストとなります。
今後、ゲームが終了した状態だとかゲームの途中の状態なども用意することになるでしょう。
さまざまなコンテキストを用意することで、
条件によって振る舞いがどのように変化するのかを探りやすくなります。

コードを書く前に、まずは要求される振る舞いを定義する
---------------------------------------------------

新しいアプリケーションを開発するにあたり、監査証跡を記録するロギングシステム
が必要となりました。
既存のオープンソースのロガーライブラリを調べてみたところ、
要件を満たすライブラリが見つからないようです。
そこで要件を満たすライブラリを自前で作成することにしました。
実際に作成しはじめる前に、まずその要件をはっきりさせなければなりません。
言いかえれば、そのライブラリがどのように振る舞ってほしいのかをはっきりさせるということです。
他のメンバーと相談した結果、まず最低限の基本機能が確定しました。
それは、メッセージをファイルシステムに記録するということです。

さっさとエディタを立ち上げてコードを書きはじめたい気持ちはわかりますが、
まずは仕様を書くことからはじめましょう。

プレーンテキストで書いた、ファイルシステムロガーのスペック
--------------------------------------------------

New Filesystem Logger (新しいファイルシステムロガー):
- should create a new log file if none currently exists (は、ログファイルが存在しない場合は新規ファイルを作成しなければならない)
- should use an existing log file if one exists without truncating it (は、ログファイルが存在する場合は、既存の内容を残したままそのファイルを使用しなければならない)
- should throw Exception if existing log file not writable (は、ログファイルに書き込めない場合には例外をスローしなければならない)

このシンプルなプレーンテキストの仕様を PHPSpec 形式に変換するには、
新しいコンテキストクラスを作成して
その振る舞いを表すサンプルを定義します。

.. code-block:: php

    class DescribeNewFilesystemLogger extends \PHPSpec\Context
    {
    
        public function itShouldCreateCreateNewLogFileIfNoneExists()
        {
            $this->pending();
        }

        public function itShouldUseAnExistingLogFileIfOneExistsWithoutTruncatingIt()
        {
            $this->pending();
        }

        public function itShouldThrowExceptionIfExistingLogFileNotWriteable()
        {
            $this->pending();
        }
    
    }

この雛形クラスでは、未確定の (pending)
サンプルが定義されています。
未確定とは、まだ完成していないなどの状態を意味します。
このスペックを NewFilesystemLoggerSpec.php というファイル
(もうひとつのファイル命名規約を用います。先頭の "Describe"
を省略して最後に "Spec" を付加します)
に保存してコマンドラインから実行すると、その出力は次のようになります。

.. code-block:: bash

    ***

    Pending:
      NewFilesystemLogger should create create new log file if none exists
         # No reason given
         # ./NewFilesystemLoggerSpec.php:8

      NewFilesystemLogger should use an existing log file if one exists without truncating it
         # No reason given
         # ./NewFilesystemLoggerSpec.php:13

      NewFilesystemLogger should throw exception if existing log file not writeable
         # No reason given
         # ./NewFilesystemLoggerSpec.php:18

    Finished in 0.058379 seconds
    3 examples, 3 pendings

PHPSpec を実行する際のコマンドラインは次のようになります。

.. code-block:: bash

    $ phpspec NewFileSystemLoggerSpec

先ほど定義した仕様にもとづいて、
これらのサンプルメソッドの中身を作成していきましょう。

    
New Filesystem Logger コンテキストの仕様
-------------------------------------------------

New Filesystem Logger コンテキストの仕様:

.. code-block:: php

    class DescribeNewFilesystemLogger extends \PHPSpec\Context
    {

        public function itShouldCreateCreateNewLogFileIfNoneExists()
        {
            $file = $this->getTmpFileName();
            $logger = new Logger($file);
            $this->spec(file_exists($file))->should->beTrue();
        }

        public function itShouldUseAnExistingLogFileIfOneExistsWithoutTruncatingIt()
        {
            $file = $this->getTmpFileName();
            file_put_contents($file, 'Hello' . "\n");
            $logger = new Logger($file);
            $this->spec(file_get_contents($file))->shouldNot->beEmpty();
        }

        public function itShouldThrowExceptionIfExistingLogFileNotWriteable()
        {
            $file = $this->getTmpFileName();
            file_put_contents($file, 'Hello' . "\n");
            $this->spec('Logger', $file)->should->throw('Exception');
        }

        public function after()
        {
            unlink($this->getTmpFileName());
        }

        public function getTmpFileName()
        {
            return sys_get_temp_dir() . DIRECTORY_SEPARATOR . 'logger_tmp_file.log';
        }

    }
    
これで、最初にプレーンテキストで定義したスペックを実行可能なコードサンプルに落とすことができました。
もちろん、今これを実行しても単に Fatal Error となるだけでしょう。
まだ Logger クラスが存在しないわけですから。
この続きは、また後ほど。

PHPSpec におけるスペックの配置
----------------------------------

先ほど作成した新しいファイルシステムロガーのサンプルを見れば、スペックをどのように作成すればいいのかがわかります。

* すべてのスペックは \PHPSpec\Context のサブクラスに記述し、システムの仕様を表す条件をここに集約する

* コンテキストクラス名の最初は必ず "Describe" となり、その後に内容を表す文を続ける

* コンテキスト内のサンプルメソッド名の最初は必ず "itShould" となり、その仕様を表す説明文をできるだけきちんとした文で書くようにする (現在形で仕様を書くために、"Should" を省略できるようにする可能性もある)

* ``\PHPSpec\Context::spec()`` メソッドを使用して、DSL 経由で使用するオブジェクトやスカラー値を準備する

* ドメイン特化言語 (DSL) は一般的に Expectation (should/shouldNot) と Matcher (beSomething, haveSomething, equals, etc.) で構成される

* 正式なルールではないが、ひとつのサンプルではひとつのスペックのみを扱うようにする - これにより、各スペックが個別の振る舞いを表すようになる

* ``getTmpFileName()`` のように、サンプル以外のメソッドをクラスに追加してヘルパーメソッドとして使用できる

* ``after()`` メソッドおよび ``before()`` メソッドを使用して、各サンプルで共通のフィクスチャを準備できる

* ``afterAll()`` メソッドおよび ``beforeAll()`` メソッドを使用して、全サンプルの実行の前後に一度だけ実行する処理を定義できる

* サンプルの内部で例外やエラーを発生させても、それがその他のテストの実行を妨げることはない

New Filesystem Logger の仕様を実装するコード
-------------------------------------------------------------

PHPSpec で書いた仕様をもとに、
その仕様を満たすロガーの実装を始めましょう。
きっとリファクタリングのことを考える人もおられるのでしょうが、
ここではまず、仕様を満たす必要最小限のコードを書くことだけを考えます。

ファイルシステムロガーの実装
---------------------------------------

これが実装です:

.. code-block:: php

    class Logger
    {

        protected $_file = null;

        public function __construct($file)
        {
            if (!file_exists($file)) {
                $f = fopen($file, 'w');
                fclose($f);
            } elseif (file_exists($file) && is_writeable($file)) {
                $this->_file = $file;
            } else {
                throw new Exception('log file is not writeable');
            }
        }

    }
      
次に、これら以外にどんな振る舞いがあるのかを考えて
それを表すスペックを書いていきましょう。
Exception クラスを継承した Logger_Exception クラスを作成しますか?
ファイルのチェックをもう少し厳しくしますか?
ファイルの処理を新たなサブクラスに移したり、
あるいはストラテジークラスを使用したりしますか?

何をやるにしても、コードを書き始める前にまずスペックを書くようにします。
小さなことからコツコツと進め、少しずつクラスを作成していくようにしましょう。
また、仕様以上のことをコードに書かないよう心がけましょう。
ファイル処理を別のクラスに抽出することにしたとしても、
(その価値が十分あると保証できる場合を除いて)
すぐに新しいクラスの仕様を考えることはありません。というのも、
もとのスペックにおいても
ロガーを作成する際にファイルを指定するということが網羅されているからです。
この場合は新たな振る舞いを追加するのではなく、
単にその振る舞いに関する実装を透過的に変更するということになります。

スペック用のドメイン特化言語 (DSL)
---------------------------------------

PHPSpec では、振る舞いを表すサンプルを書く際に専用のドメイン特化言語
(DSL) を使用します。この DSL はできるだけ自然な
(かつ文法的に正しい) 英語に近い形式で書けるように作られており、
直感的に使用することができます。また、読んで理解するのも簡単になります。

DSL の基本的な形式は、Expectation (should あるいは
should not) と Matcher (be, beAnInstanceOf, equal, etc.)
を用意して、それを新規スペックに渡した値やオブジェクトに関連づけるというものです。
こうすることで、比較的読みやすい文章ができあがります。
ほんの少し手を加えるだけで、普通の英語 (あるいはその他の言語!) に変換することができます。
変換の手間が最小限であること、そして私たちが実際に頭で考える内容に近いこと
などから、スペックの内容をレビューしたり修正したりするのも常に簡単です。

スペック DSL の例: Bowling は Logger のインスタンスであってはならない
-------------------------------------------------------------

スペックの例:

.. code-block:: php

    $bowling = $this->spec(new Bowling);
    $bowling->shouldNot->beAnInstanceOf('Logger');

実際の値 (Actual Value)
----------------------

PHPSpec のサンプルメソッドで DSL のインスタンスを作成するには、
``\PHPSpec\Context::spec()`` を使用します。
このメソッドには、次の 3 種類のパラメータを渡すことができます。

* スカラー値 (文字列、整数値、論理値、浮動小数点数値、あるいは配列)
* オブジェクト
* オブジェクトの名前とコンストラクタへのパラメータ

Actual Term: スカラーの例
----------------------------

例:

.. code-block:: php

    $this->spec('i am a string')->should-beString();
    $this->spec(567)->should->equal(567);
    $this->spec(array(1, 2, 3))->shouldNot->beEmpty();
      

Actual Term: オブジェクトの例
----------------------------

例:

.. code-block:: php

    $this->spec(new Bowling)->should->beAnInstanceOf('Bowling');

    $bowling = new Bowling;
    $this->spec($bowling)->shouldNot->havePlayers();
      
Actual Term: オブジェクト名とコンストラクタのパラメータの例
-----------------------------------------------------

例:

.. code-block:: php

    $this->spec('Bowling', new Player('Joe'), new Player('Jim'))->should->havePlayers();
      
期待する内容 (Expectation (Should or Should Not))
------------------------------------------------

英語と同様、あらゆる期待は大きく二つに分類できます。
失敗することを期待するものと、成功することを期待するものです。
実際の値が一致してほしいのか一致してほしくないのかに応じて、
DSL で ``should`` あるいは
``shouldNot`` のいずれかを使用します。

以下のサンプルは、どれも成功するはずです。

Expectation Term: さまざまなサンプル
------------------------------------------

コード例:

.. code-block:: php

    $spec->( array() )->should->beEmpty();
    $spec->('Bowling')->shouldNot->havePlayers();
    $spec->('i am a string')->should->match("/^[a-z ]$/");
    $spec->(is_int('string'))->shouldNot->beTrue();
      
条件 (Matcher)
----------------

ユニットテストのフレームワークがアサーション (表明) に頼っているのに対して、
PHPSpec は期待 (Expectation Term) と条件 (Matcher) に責任を分担させています。
Matcher はシンプルなオブジェクトで、実際の値と期待内容を
DSL のメソッドで比較します。そしてマッチしたか否かを返します。
Matcher の形式は ``\PHPSpec\Matcher``
インターフェイスで定義されているので、独自の Matchers
を書くこともできます (現在この機能は未完成です)。

PHPSpec フレームワークには、すでにさまざまな Matcher
が用意されています [注意: 中にはまだ開発途中のものもあります]。

Matcher とは、一般にスペックの最後に追加されるものです。
先ほどごらんいただいた例でもそのようになっています。

PHPSpec に含まれる Matcher
----------------------------

すべての Matcher は、boolean 値を返します。
したがって、スペックを記述する「流れるようなインターフェイス」
においては一番最後にコールすることになります。
``NULL`` とされているパラメータは、
通常は不要であることを意味します
(Matcher の名前から、期待する内容は暗黙のうちに決まります)。

PHPSpec の Matcher
------------------

+---------------------------------------+--------------------------------------------------------------------+
| Matcher メソッド                      | 説明                                                               |
+=======================================+====================================================================+
| bool be (mixed $expected)             | ``equal()`` と同じ意味で、英語っぽく書くために用意されています。   |
+---------------------------------------+--------------------------------------------------------------------+
| bool beEqualTo (mixed $expected)      | ``equal()`` と同じ意味で、英語っぽく書くために用意されています。   |
+---------------------------------------+--------------------------------------------------------------------+
| bool equal (mixed $expected)          | 期待する内容と等しいかどうかを調べます。                           |
|                                       | スカラー値、オブジェクトのクラス、配列の内容など、                 |
|                                       | 種類に応じて適切な比較を行います。                                 |
+---------------------------------------+--------------------------------------------------------------------+
| bool beTrue (null $expected)          | 実際の値を ``TRUE`` と比較します。                                 |
+---------------------------------------+--------------------------------------------------------------------+
| bool beFalse (null $expected)         | 実際の値を ``FALSE`` と比較します。                                |
+---------------------------------------+--------------------------------------------------------------------+
| bool beNull (null $expected)          | 実際の値が ``NULL`` かどうかを調べます。                           |
+---------------------------------------+--------------------------------------------------------------------+
| bool beEmpty (mixed $expected)        | 実際の値が空かどうかを (``empty()`` で) 調べます。                 |
+---------------------------------------+--------------------------------------------------------------------+
| bool beSet (null $expected)           | 実際の値が設定されているかどうかを (``isset()`` で) 調べます。     |
+---------------------------------------+--------------------------------------------------------------------+
| bool beAnInstanceOf (string $expected)| 実際の値がオブジェクトであり、                                     |
|                                       | かつ指定したクラスのインスタンスであるかどうかを調べます。         |
+---------------------------------------+--------------------------------------------------------------------+
| bool beInt (null $expected)           | 実際の値が整数値かどうかを調べます。                               |
|                                       | 厳格なチェックを行うので、数値形式の文字列は整数値と見なしません。 |
+---------------------------------------+--------------------------------------------------------------------+
| bool beArray (null $expected)         | 実際の値が配列かどうかを調べます。                                 |
+---------------------------------------+--------------------------------------------------------------------+
| bool beString (null $expected)        | 実際の値が文字列かどうかを調べます。                               |
+---------------------------------------+--------------------------------------------------------------------+
| bool beFloat (null $expected)         | 実際の値が浮動小数点数値かどうかを調べます。                       |
+---------------------------------------+--------------------------------------------------------------------+
| bool beObject (null $expected)        | 実際の値がオブジェクトかどうかを調べます。                         |
|                                       | どのようなクラスのオブジェクトなのかは調べません。                 |
+---------------------------------------+--------------------------------------------------------------------+
| bool beGreaterThan (mixed $expected)  | 実際の値が指定した値より大きい (``>``)                             |
|                                       | かどうかを調べます。                                               |
+---------------------------------------+--------------------------------------------------------------------+
| bool beLessThan (mixed $expected)     | 実際の値が指定した値より小さい                                     |
|                                       | (``<``) かどうかを調べます。                                       |
+---------------------------------------+--------------------------------------------------------------------+
| bool beGreaterThanOrEqualTo           | 実際の値が指定した値以上                                           |
| (mixed $expected)                     | (``>=``) かどうかを調べます。                                      |
+---------------------------------------+--------------------------------------------------------------------+
| bool beLessThanOrEqualTo (mixed       | 実際の値が指定した値以下                                           |
| $expected)                            | (``<=``) かどうかを調べます。                                      |
+---------------------------------------+--------------------------------------------------------------------+

叙述型の Matcher (Predicate Matchers)
--------------------------------------

叙述型の Matcher
とは、指定したオブジェクトから実際の値を取得する Matcher です。
オブジェクトを調べ、 ``isSomething()``
あるいは ``hasSomething()``
形式のメソッドをコールします。
すでに先ほどの DSL の例でこれを使用しており、
DSL のメソッド ``havePlayers()``
は ``Bowling::hasPlayers()``
メソッドをコールします。コールしたメソッドの戻り値である boolean
値が ``TRUE`` かどうかを調べ、
その結果を返します。

        
クラスと叙述型の Matcher のコール例
----------------------------------------------

例:

.. code-block:: php

    class Insect { // 昆虫

        public function isInsect() { // …は、昆虫です
            return true;
        }

        public function hasWings() { // …は、羽根を持っています
            return true;
        }

    }

    class Flea extends Insect { // ノミ

        public function hasWings() {
            return false; // ノミは血を吸う虫で、羽根を持ちません
        }

    }

    class DescribeFlea extends \PHPSpec\Context {

        public function itShouldBeAnInsect()
        {
            $flea = new Flea;
            $this->spec($flea)->should->beAnInsect(); // Flea::isInsect() == TRUE
        }

        public function itShouldHaveNoWings()
        {
            $flea = new Flea;
            $this->spec($flea)->shouldNot->haveWings(); // Flea::hasWings() == FALSE
        }
    }
        

叙述型の Matcher メソッドとして、DSL 内では
``be()`` 、 ``beA()`` 、
``beAn()`` 形式を使用することができます。
これらはそれぞれ文法的に正しくなるように区別されているだけであり、
それ以外はまったく同じです。 ``have(), haveA() および haveAn()``
も同様です。同じような規則でオブジェクトのメソッドを探します
(オブジェクトのメソッド名が文法的に正しいものである必要があります!)。
将来的には、他の形式にもこのような叙述型を取り入れる予定です。
何かよい案があれば、ぜひ教えてください。
