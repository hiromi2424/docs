ルーティング(*Routing*)
#######################

ルーティングは、URLをコントローラのアクションに結びつける(*map*)する機能です。
これは、URLの見ばえをよくし(*pretty URL*)、より柔軟に設定できるように、
CakePHPに備わっているものです。
Apacheのmod\_rewriteが必ず必要というわけではありませんが、
mod\_rewriteを使用すると、アドレスバーに表示されるURLが整ったものになります。

また、CakePHPのルーティングは、パラメータの配列を文字列のURLに戻すことができる、
リバースルーティングの概念を含んでいます。
リバースルーティングを用いることで、全てのコードを更新する必要なく、
容易にアプリケーションのURL構造を再構築することができます。

.. index:: routes.php

.. _routes-configuration:

ルートの設定
============

アプリケーションのルート(*route*)の設定は ``app/Config/routes.php`` で行います。
このファイルは :php:class:`Dispatcher` によってルートが処理される時に読み込まれ、
望みのアプリケーション独自のルートを定義することができます。
このファイルで定義されたルートは、要求されたリクエストがマッチするまで、
上から下に順に処理されます。
これはルートを配置する順番がルートがどう解析されるかに影響し得るということです。
通常、出来る限りルートファイルの一番上に、最も頻繁にアクセスされるルートを
置くようにすることが良い習慣となります。
これは各々のリクエストで、マッチしない多くのルートをチェックする必要を省きます。

ルートは、定義された順序で解析とマッチングが成されます。
二つの同様なルートを定義したとき、
先に定義されたルートは後に定義されたものより高い優先順序を持ちます。
ルートを定義した後で、 :php:meth:`Router::promote()`
を用いてルートの順序を操作することができます。

またCakePHPには、すぐ使い始められるように、デフォルトのルートが付属しています。
後にこれを必要としない時に、無効にすることができます。
デフォルトのルーティングを無効化する手順は、 :ref:`disabling-default-routes`
を見てください。


デフォルトのルーティング
========================

独自のルートを学ぶ前に、CakePHPのデフォルトのルートについて知っておく必要があります。
CakePHPのデフォルトルーティングによって、
どんなアプリケーションでもうまく動くようになっています。
URLの中に、アクション名を置くことで、アクションに直接アクセスすることができます。
URLを使うことでコントローラのアクションにパラメータを渡すこともできます。::

        デフォルトのルートのURLパターン:
        http://example.com/controller/action/param1/param2/param3

/posts/viewというURLは、PostsControllerのview()アクションにマップされます。
/products/view\_clearanceは、
ProductsControllerのview\_clearance()アクションにマップされます。
URLでアクション名が指定されていない場合には、index()メソッドが用いられます。

デフォルトのルーティングでは、URLを使ってアクションにパラメータを渡すこともできます。
例えば、/posts/view/25というリクエストは、
PostsController 上でview(25)として呼ぶのと同じことになります。
また、デフォルトのルーティングはプラグイン、
プレフィックスのルートを、これらの機能の使用を選択した場合、提供します。

ビルトインのルートは ``Cake/Config/routes.php`` にあります。
アプリケーションの :term:`routes.php` ファイルからこれを取り除くことで、
デフォルトのルーティングを無効化できます。

.. index:: :controller, :action, :plugin
.. _connecting-routes:

ルートの定義
============

自ルートの定義により、
アプリケーションを指定したURLで動作させることができるようになります。
独自のルーティングは、 ``app/Config/routes.php``
ファイルで :php:meth:`Router::connect()` メソッドを使用して定義します。

``connect()`` メソッドは３つのパラメータからなっています。
対応させたいURL、独自のルート要素のデフォルトの値、
URLの要素と対応させるための正規表現ルールです。

ルート定義の基本的な形式は次のようになります。::

    Router::connect(
        'URL',
        array('default' => 'defaultValue'),
        array('option' => 'matchingRegex')
    );

最初のパラメータで、制御するURLをルータに伝えます。
URLは、通常のスラッシュで分けられた文字列です。
しかし、ワイルドカード(\*)や、
:ref:`route-elements` を含むことができます。
ワイルドカードを使用すると、
与えられたオプションパラメータ全てを認めるようにする、
とルーターに伝えます。
\*の無いルートは与えられたテンプレートパターンそのものにだけマッチします。

URLを指定した後、 ``connect()`` の次の２つのパラメータを使って、
リクエストが対応した場合に何を行うのかをCakePHPに伝えます。
2番目のパラメータは連想配列です。
配列のキーは、URLのルート要素にちなんだものか、
デフォルト要素(``:controller`` 、 ``:action`` 、 ``:plugin``)にしてください。
配列内の値は、それらのキーに対するデフォルト値です。
connect()の3番目のパラメータを考える前に、基本的な例をいくつか見てみましょう。::

    Router::connect(
        '/pages/*',
        array('controller' => 'pages', 'action' => 'display')
    );

CakePHPのroutes.phpファイルの中に、このルートが書かれています。
このルートは、 ``/pages/`` ではじまるすべてのURLにマッチし、
それを ``PagesController();`` の ``display()`` アクションに渡します。
例えば、/pages/productsというリクエストは、
``PagesController->display('products')`` にマップされます。

貪欲なアスタリスク(*greedy star*)と言える ``/*`` に加えて、
``/**`` という後端アスタリスク(*trailing star*)記法もあります。
後端に二重のアスタリスクを用いると、URLの残りの部分を単一の引数として捕捉します。
これは ``/`` を含む引数を用いたい場合便利です。::

    Router::connect(
        '/pages/**',
        array('controller' => 'pages', 'action' => 'show')
    );

``/pages/the-example-/-and-proof`` というURLは、
単一の引数として ``the-example-/-and-proof`` という結果になります。

.. versionadded:: 2.1

    後端二重アスタリスクは2.1で追加されました。

:php:meth:`Router::connect()` の二番目の引数を用いて、
ルートのデフォルト値を作るルーティングパラメータを与えることができます。::

    Router::connect(
        '/government',
        array('controller' => 'pages', 'action' => 'display', 5)
    );

この2つ目の例は、 ``connect()``
の2番目のパラメータでデフォルトパラメータを定義する方法を示しています。
もし、種々のお客様向けに異なる製品がある場合などに、
ルートを作成することを考えることができるでしょう。
この例では、 ``/pages/display/5`` としなくても、
``/government`` という URLでアクセスできるようになります。

.. note::

    代替のルートを定義することができますが、
    デフォルトのルートは引き続き機能します。
    これはコンテンツが二つのURLになってしまうという状況を作り得ます。
    デフォルトのルートを無効化し、自身で定義したURLのみを与えるには、
    :ref:`disabling-default-routes` を見てください。

他の一般的なルーターの使い方は、コントローラの別名(*alias*)を定義することです。
通常の ``/users/some_action/5`` という URL の代わりに、
``/cooks/some_action/5`` でアクセスさせたいとしましょう。
このようなルートの設定は、次のようにすることで簡単に実現できます。::

    Router::connect(
        '/cooks/:action/*', array('controller' => 'users')
    );

この設定は、ルーターが ``/cooks/`` で始まるURLを、
全てusersコントローラに送ることを表しています。
呼ばれるアクションは ``:action`` パラメータの値に依存します。
:ref:`route-elements` を使うことによって、
ユーザー入力や変数を受け入れる様々なルートを作成できます。
上記のルートは貪欲なアスタリスクも使っています。
貪欲なアスタリスクは :php:class:`Router` に、
このルートが与えられる追加の固定的(*positional*)
なパラメータを受け入れるべきであることを指示します。
これらの引数は :ref:`passed-arguments`
配列で利用することができるようになります。

URLが生成されるときも、ルートの設定が適用されます。
上述の例にあるルートが最初にマッチした場合、URL として
``array('controller' => 'users', 'action' => 'some_action', 5)``
のように指定すると、/cooks/someAction/5というURLが生成されます。

ルートの中で独自の名前を持つパラメータを使うつもりならば、
:php:meth:`Router::connectNamed()`
を用いてルーターにそれを伝える必要があります。
上述の例にあるルートに ``/cooks/some_action/type:chef``
のようなURLをマッチさせたい場合は、次のようにします。::

    Router::connectNamed(array('type'));
    Router::connect(
        '/cooks/:action/*', array('controller' => 'users')
    );

.. _route-elements:

ルート要素
----------

さらに柔軟な追加機能として、独自のルート要素を指定できます。
この機能により、コントローラアクション用のパラメータが、
URLのどこにあるのかを定義できるようになります。
リクエストがあると、このルート要素は、
コントローラの ``$this->request->params`` の中に入ります。
これは、名前付きパラメータとは異なる扱いとなります。
以下の違いに注意してください。
名前付きパラメータ（/コントローラ/アクション/名前:値）は、
``$this->request->params['named']`` に入りますが、ルート要素のデータは、
``$this->request->params`` の中に入ります。
独自のルート要素を定義する際には、任意で正規表現を指定することができます。
これはURLが正しく作られているか、そうでないかをCakePHPが判断できるようにします。
正規表現を使わない選択をとる場合、
``/`` でない全てがパラメータの一部として扱われます。::

    Router::connect(
        '/:controller/:id',
        array('action' => 'view'),
        array('id' => '[0-9]+')
    );

これは、 ``/コントローラー名/:id`` という形で、
どんなコントローラからでもモデルを表示(*view*)できるようにする簡素な例です。
connect()に指定されているURLには２つのルート要素、
``:controller`` と ``:id`` があります。
``:controller`` 要素はCakePHPのデフォルトのルート要素なので、
ルーターがURL内のコントローラ名をマッチし特定することができます。
``:id`` 要素は独自のルート要素なので、
connect()の３番目のパラメータ内の正規表現で、
どうやってマッチさせるのかを明示する必要があります。

.. note::

    ルート要素に使われるパターンはグループキャプチャを含んではなりません。
    これを含んでいると、ルーターは正しく動作しないでしょう。

この route が定義されると、 ``/apples/5`` というリクエストは、
``/apples/view/5`` としたのと同じことになります。
どちらも、ApplesControllerのview()メソッドを呼びます。
view()メソッド内では、 ``$this->request->params['id']``
を使って渡されたIDにアクセスする必要があることになるでしょう。

もし、アプリケーション内でコントローラが1つだけで、
コントローラ名をURL中に表示したくない場合、
そのコントローラのアクションにURLをマッピングすることができます。
例えば、 ``home`` コントローラのアクションにURLをマッピングする
(URL を ``/home/demo`` の代わりに ``/demo`` にする)には、以下のようにします。::

    Router::connect('/:action', array('controller' => 'home'));

大文字小文字を考慮しないURLを提供したいなら、
正規表現のインライン修飾子(*inline modifier*)を使うことができます。::

    Router::connect(
        '/:userShortcut',
        array('controller' => 'teachers', 'action' => 'profile', 1),
        array('userShortcut' => '(?i:principal)')
    );

もうひとつ例をあげます。もうあなたはルーティングの達人でしょう::

    Router::connect(
        '/:controller/:year/:month/:day',
        array('action' => 'index'),
        array(
            'year' => '[12][0-9]{3}',
            'month' => '0[1-9]|1[012]',
            'day' => '0[1-9]|[12][0-9]|3[01]'
        )
    );

ややこしいものですが、ルートが非常に強力であることを示す例です。
このURLでは、4つの要素があります。
最初のものはよく知っているもので、
コントローラ名が来ることをCakePHPに伝えるデフォルトのルート要素です。

次に、いくつかのデフォルト値を指定します。
コントローラ名に関係なく、index()アクションが呼ばれるようにします。

最後に、数値形式の年(*year*)、月(*month*)、日(*day*)
にマッチする正規表現を指定します。
ここで、この正規表現では括弧(グループ化)がサポートされてないことに注意してください。
なお、上記のように、その他の正規表現を使うことはできますが、
括弧でグループ化することはできません。

このルートを設定すると、 ``/articles/2007/02/01`` 、 ``/posts/2004/11/16``
等にマッチし、 ``$this->request->params`` に日付パラメータを伴って、
対応するコントローラのindex()アクションにリクエストを処理します。

CakePHPでは特別な意味を持つルート要素がいくつかあり、
その特別な意味を必要としないならこれらを使ってはいけません。

* ``controller`` ルートのコントローラ名に使われます。
* ``action`` ルートのコントローラアクション名に使われます。.
* ``plugin`` コントローラが配置されているプラグイン名に使われます。
* ``prefix`` :ref:`prefix-routing` に使われます。
* ``ext`` :ref:`file-extensions` ルーティングに使われます。

アクションにパラメータを渡す
----------------------------

:ref:`route-elements` を用いてルートを定義するとき、
ルート要素を引数渡し(*passed arguments*)にしたいことがあるでしょう。
:php:meth:`Router::connect()` の3番目の引数を用いて、
どのルート要素が渡される引数として利用できるべきかも定義できます。::

    // SomeController.php
    public function view($articleId = null, $slug = null) {
        // ここに何かしらコードを書く
    }

    // routes.php
    Router::connect(
        '/blog/:id-:slug', // E.g. /blog/3-CakePHP_Rocks
        array('controller' => 'blog', 'action' => 'view'),
        array(
            // 上記のアクションで":id"を$articleIDにマッピングするため、順番は重要です。
            'pass' => array('id', 'slug'),
            'id' => '[0-9]+'
        )
    );

これで今や、リバースルーティングの能力のおかげで、
以下のようにURL配列に渡すことができます。ルートで定義されているので、
CakeはURLをどのように作成したらよいかがわかっています。::

    // view.ctp
    // これは /blog/3-CakePHP_Rocks へのリンクを返します
    echo $this->Html->link('CakePHP Rocks', array(
        'controller' => 'blog',
        'action' => 'view',
        'id' => 3,
        'slug' => 'CakePHP_Rocks'
    ));

ルートごとの名前付きパラメータ
------------------------------

:php:meth:`Router::connectNamed()`
を用いてグローバル規模で名前付きパラメータを制御できる一方、
``Router::connect()`` の3番目の引数を使って、
ルートレベルで名前付きパラメータの挙動を制御することもできます。::

    Router::connect(
        '/:controller/:action/*',
        array(),
        array(
            'named' => array(
                'wibble',
                'fish' => array('action' => 'index'),
                'fizz' => array('controller' => array('comments', 'other')),
                'buzz' => 'val-[\d]+'
            )
        )
    );

上記のルート定義は ``named`` キーを用いて、
種々の名前付きパラメータがどう扱われるべきかを定義します。
以下でいろいろなルールそれぞれを1つずつみていきましょう。

* 「wibble」は補足的な情報がありません。
  このルートにマッチするURLにこれがあると、
  常にパースされるということを意味します。
* 「fish」は条件の配列で、「action」キーをもっています。
  これはfishがアクションがindexのときのみ、
  名前付きパラメータとしてパースされることを意味します。
* 「fizz」も条件の配列を持っています。
  しかしながら、これは二つのコントローラをもっています。
  これは配列内の名前のうち一つにコントローラがマッチしたときのみ、
  「fizz」がパースされることを意味します。
* 「buzz」は文字列条件をもっています。
  文字列条件は正規表現の一部として扱われます。
  パターンにマッチしたbuzzの値のみパースされます。

名前付きパラメータが使用されていて、与えられた条件に適合しない場合、
名前付きパラメータの代わりに引数渡しとして扱われます。

.. index:: admin routing, prefix routing
.. _prefix-routing:

プレフィックスル－ティング(*Prefix Routing*)
--------------------------------------------

たいていのアプリケーションには、
権限のあるユーザだけが情報を変更できる管理画面が必要です。
そのために ``/admin/users/edit/5``
のような特別なURLを準備することがよくあります。
CakePHPでは、コア設定ファイル内のRouting.prefixesにプレフィックスを設定することで、
プレフィックスルーティングを有効にできます。
また、プレフィックスはルータに関連しますが、 ``app/Config/core.php``
で設定することに留意してください。::

    Configure::write('Routing.prefixes', array('admin'));

コントローラ(controller)内では、プレフィックスとして ``admin_``
をメソッドの前につけたすべてのアクションが呼び出されます。
usersを例にすると、 ``/admin/users/edit/5`` というURLへのアクセスで、
``UsersController`` の ``admin_edit`` メソッドが、引数5で呼び出されます。
ビューファイルは ``app/View/Users/admin_edit.ctp`` となります。

/adminというURLを、pagesコントローラの ``admin_index``
アクションにマップするには次のようなルートを使います。::

    Router::connect('/admin', array('controller' => 'pages', 'action' => 'index', 'admin' => true));

``Routing.prefixes`` に値を追加することによって、
複数のプレフィックスを使うようにルーターを設定することもできます。もし、::

    Configure::write('Routing.prefixes', array('admin', 'manager'));

と設定すると、Cakeはadminとmanagerの両方のプレフィックスに対するルートを自動的に生成します。
各々の設定されたプレフィックスのために、以下のルートが生成されるでしょう。::

    Router::connect("/{$prefix}/:plugin/:controller", array('action' => 'index', 'prefix' => $prefix, $prefix => true));
    Router::connect("/{$prefix}/:plugin/:controller/:action/*", array('prefix' => $prefix, $prefix => true));
    Router::connect("/{$prefix}/:controller", array('action' => 'index', 'prefix' => $prefix, $prefix => true));
    Router::connect("/{$prefix}/:controller/:action/*", array('prefix' => $prefix, $prefix => true));

adminルーティングのように、
全てのプレフィックス付きアクションはプレフィックス名を接頭辞としてつけるべきです。
``/manager/posts/add`` が ``PostsController::manager_add()``
にマッピングされるからです。

加えて、現在のプレフィックスはコントローラのメソッドから
``$this->request->prefix`` を通して取得できます。

プレフィックスルートを使うときに重要な点は、
HTMLヘルパーを使用してリンクを作成するなら、
プレフィックスを用いた呼び出しの管理も楽になるということです。
HTMLヘルパーを使ったリンクの例は次のようになります。::

    // プレフィックスがつけられたルートに行く
    echo $this->Html->link('Manage posts', array('manager' => true, 'controller' => 'posts', 'action' => 'add'));

    // プレフィックスとお別れする
    echo $this->Html->link('View Post', array('manager' => false, 'controller' => 'posts', 'action' => 'view', 5));

.. index:: plugin routing

プラグインルーティング
----------------------

プラグインルーティングは **plugin** キーを使います。
プラグインを示すリンクを作ることができますが、
pluginキーをURL配列に追加してください。::

    echo $this->Html->link('New todo', array('plugin' => 'todo', 'controller' => 'todo_items', 'action' => 'create'));

逆に、進行中のリクエストがプラグインのリクエストで、
プラグイン無しのリンクを生成したいときは、以下のようにできます。::

    echo $this->Html->link('New todo', array('plugin' => null, 'controller' => 'users', 'action' => 'profile'));

``plugin => null`` と設定することによって、
ルータに作りたいリンクがプラグインの一部ではないということを伝えることができます。

.. index:: file extensions
.. _file-extensions:

ファイル拡張子
--------------

ルートで異なるファイル拡張子を扱う場合は、
ルート設定ファイルに特別な1行が必要です。::

    Router::parseExtensions('html', 'rss');

これは全てのマッチするファイル拡張子を削除して、
残りの部分を解析するようにルータに指示します。

もし/page/title-of-page.htmlのようなURLを作りたいのなら、
以下に例示するようなルートを作ることになります。::

    Router::connect(
        '/page/:title',
        array('controller' => 'pages', 'action' => 'view'),
        array(
            'pass' => array('title')
        )
    );

そして、ルートに遡って合致するリンクを作成するには、
単純にに次のようにします。::

    $this->Html->link(
        'Link title',
        array('controller' => 'pages', 'action' => 'view', 'title' => 'super-article', 'ext' => 'html')
    );

ファイル拡張子は、 :php:class:`RequestHandlerComponent` によって、
コンテンツの種類に基づいて自動的にビューの切り替えをするのに使用されます。
詳しい情報はRequestHandlerComponentの章を見てください。

.. _route-conditions:

ルートのマッチング時の補助的な条件の使用
----------------------------------------

ルートを作成するとき、
一定のURLを特定のリクエスト・環境設定の元に制限したい場合があるでしょう。
この良い例として、 :doc:`rest` ルーティングがあります。
:php:meth:`Router::connect()` の ``$defaults``
引数で補足的な条件を指定することができます。
CakePHPはデフォルトで3つの環境条件を公開していますが、
:ref:`custom-route-classes` を使って更に追加することができます。
ビルトインのオプションは以下の通りです。

- ``[type]`` 特定のコンテンツの種類(*content type*)のリクエストのみにマッチします。
- ``[method]`` 特定のHTTP動詞(*verbs*)を伴うリクエストのみにマッチします。
- ``[server]`` $_SERVER['SERVER_NAME'] が与えられた値にマッチする時のみマッチします。

``[method]`` オプションを使い独自のレストフルなルートを作成する方法の、
簡単な例をここに示します。::

    Router::connect(
        "/:controller/:id",
        array("action" => "edit", "[method]" => "PUT"),
        array("id" => "[0-9]+")
    );

上記のルートは ``PUT`` リクエストのみにマッチします。
これらの条件を用いて、独自のRESTルーティングや、
その他のリクエストデータに依存する情報を作成することができます。

.. index:: passed arguments
.. _passed-arguments:

passed引数
==========

passed引数はリクエストを作り上げるときに、
その他の引数またはパスセグメントとして使用されます。
これらはコントローラのメソッドにパラメータを渡すためによく使われます。::

    http://localhost/calendars/view/recent/mark

上記の例では、 ``recent`` と ``mark`` は
``CalendarsController::view()`` へのpassed引数です。
passed引数は三つの方法でコントローラに渡されます。
一つは、アクションのメソッドが呼び出されたときの引数としてです。
二つ目は、順番付けされた配列として
``$this->request->params['pass']`` を利用できます。
最後は二つ目と同じ方法で ``$this->passedArgs`` が利用できます。
また、カスタムルートを使うとき、
passed引数に特定のパラメータを入れさせることもできます。

以下の様なコントローラ・アクションをもち、前述のURLを参照した場合、::

    CalendarsController extends AppController {
        public function view($arg1, $arg2) {
            debug(func_get_args());
        }
    }

以下の出力を得ることでしょう::

    Array
    (
        [0] => recent
        [1] => mark
    )

これと同じデータを コントローラ、ビュー、ヘルパーの
``$this->request->params['pass']`` と ``$this->passedArgs``
でも利用できます。
pass配列の値は呼ばれたURLに現れる順番を元に数値添字で配置されます。::

    debug($this->request->params['pass']);
    debug($this->passedArgs);

上記のどちらも以下の出力をします::

    Array
    (
        [0] => recent
        [1] => mark
    )

.. note::

    $this->passedArgsには、名前付き引数が連想配列として、
    passed引数と混ぜられて含まれていることがあります。

:term:`routing array` を使ったURLの生成時、
配列に文字列キー無しの値としてpassed引数を与えます。::

    array('controller' => 'posts', 'action' => 'view', 5)

``5`` は数値キーをもつので、passed引数として扱われます。

.. index:: named parameters

.. _named-parameters:

名前付きパラメータ
==================

URLを使ってパラメータに名前を付けて、その値を渡すことができます。
``/posts/view/title:first/category:general`` というリクエストでは、
PostsControllerのview()アクションが呼ばれます。
このアクションでは、titleとcategoryというパラメータの値を、
``$this->params['named']`` の中に見つけることができます。
また、 ``$this->passedArgs`` の中でも利用することができます。
どちらの方法でも、その名前をインデックスとして使って、
名前付きパラメータにアクセスすることができます。
名前付きパラメータが省略された場合はセットされません。


.. note::

    名前付きパラメータとして解析されるものは、
    :php:meth:`Router::connectNamed()` によって制御されます。
    What is parsed as a named parameter is controlled by
    :php:meth:`Router::connectNamed()`.  If your named parameters are not
    reverse routing, or parsing correctly, you will need to inform
    :php:class:`Router` about them.

Some summarizing examples for default routes might prove helpful::

    URL to controller action mapping using default routes:

    URL: /monkeys/jump
    Mapping: MonkeysController->jump();

    URL: /products
    Mapping: ProductsController->index();

    URL: /tasks/view/45
    Mapping: TasksController->view(45);

    URL: /donations/view/recent/2001
    Mapping: DonationsController->view('recent', '2001');

    URL: /contents/view/chapter:models/section:associations
    Mapping: ContentsController->view();
    $this->passedArgs['chapter'] = 'models';
    $this->passedArgs['section'] = 'associations';
    $this->params['named']['chapter'] = 'models';
    $this->params['named']['section'] = 'associations';

When making custom routes, a common pitfall is that using named
parameters will break your custom routes. In order to solve this
you should inform the Router about which parameters are intended to
be named parameters. Without this knowledge the Router is unable to
determine whether named parameters are intended to actually be
named parameters or routed parameters, and defaults to assuming you
intended them to be routed parameters. To connect named parameters
in the router use :php:meth:`Router::connectNamed()`::

    Router::connectNamed(array('chapter', 'section'));

Will ensure that your chapter and section parameters reverse route
correctly.

When generating urls, using a :term:`routing array` you add named
parameters as values with string keys matching the name::

    array('controller' => 'posts', 'action' => 'view', 'chapter' => 'association')

Since 'chapter' doesn't match any defined route elements, it's treated
as a named parameter.

.. note::

    Both named parameters and route elements share the same key-space.
    It's best to avoid re-using a key for both a route element and a named
    parameter.

Named parameters also support using arrays to generate and parse
urls.  The syntax works very similar to the array syntax used
for GET parameters.  When generating urls you can use the following
syntax::

    $url = Router::url(array(
        'controller' => 'posts',
        'action' => 'index',
        'filter' => array(
            'published' => 1
            'frontpage' => 1
        )
    ));

The above would generate the url ``/posts/index/filter[published]:1/filter[frontpage]:1``.
The parameters are then parsed and stored in your controller's passedArgs variable
as an array, just as you sent them to :php:meth:`Router::url`::

    $this->passedArgs['filter'] = array(
        'published' => 1
        'frontpage' => 1
    );

Arrays can be deeply nested as well, allowing you even more flexibility in
passing arguments::

    $url = Router::url(array(
        'controller' => 'posts',
        'action' => 'search',
        'models' => array(
            'post' => array(
                'order' => 'asc',
                'filter' => array(
                    'published' => 1
                )
            ),
            'comment' => array(
                'order' => 'desc',
                'filter' => array(
                    'spam' => 0
                )
            ),
        ),
        'users' => array(1, 2, 3)
    ));

You would end up with a pretty long url like this (wrapped for easy reading)::

    posts/search
      /models[post][order]:asc/models[post][filter][published]:1
      /models[comment][order]:desc/models[comment][filter][spam]:0
      /users[]:1/users[]:2/users[]:3

And the resulting array that would be passed to the controller would match that
which you passed to the router::

    $this->passedArgs['models'] = array(
        'post' => array(
            'order' => 'asc',
            'filter' => array(
                'published' => 1
            )
        ),
        'comment' => array(
            'order' => 'desc',
            'filter' => array(
                'spam' => 0
            )
        ),
    );

.. _controlling-named-parameters:

Controlling named parameters
----------------------------

You can control named parameter configuration at the per-route-level
or control them globally.  Global control is done through ``Router::connectNamed()``
The following gives some examples of how you can control named parameter parsing
with connectNamed().

Do not parse any named parameters::

    Router::connectNamed(false);

Parse only default parameters used for CakePHP's pagination::

    Router::connectNamed(false, array('default' => true));

Parse only the page parameter if its value is a number::

    Router::connectNamed(array('page' => '[\d]+'), array('default' => false, 'greedy' => false));

Parse only the page parameter no matter what::

    Router::connectNamed(array('page'), array('default' => false, 'greedy' => false));

Parse only the page parameter if the current action is 'index'::

    Router::connectNamed(
        array('page' => array('action' => 'index')),
        array('default' => false, 'greedy' => false)
    );

Parse only the page parameter if the current action is 'index' and the controller is 'pages'::

    Router::connectNamed(
        array('page' => array('action' => 'index', 'controller' => 'pages')),
        array('default' => false, 'greedy' => false)
    );


connectNamed() supports a number of options:

* ``greedy`` Setting this to true will make Router parse all named params.
  Setting it to false will parse only the connected named params.
* ``default`` Set this to true to merge in the default set of named parameters.
* ``reset`` Set to true to clear existing rules and start fresh.
* ``separator`` Change the string used to separate the key & value in a named
  parameter. Defaults to `:`

Reverse routing
===============

Reverse routing is a feature in CakePHP that is used to allow you to
easily change your url structure without having to modify all your code.
By using :term:`routing arrays <routing array>` to define your urls, you can
later configure routes and the generated urls will automatically update.

If you create urls using strings like::

    $this->Html->link('View', '/posts/view/' + $id);

And then later decide that ``/posts`` should really be called
'articles' instead, you would have to go through your entire
application renaming urls.  However, if you defined your link like::

    $this->Html->link(
        'View',
        array('controller' => 'posts', 'action' => 'view', $id)
    );

Then when you decided to change your urls, you could do so by defining a
route.  This would change both the incoming URL mapping, as well as the
generated urls.

When using array urls, you can define both query string parameters and
document fragments using special keys::

    Router::url(array(
        'controller' => 'posts',
        'action' => 'index',
        '?' => array('page' => 1),
        '#' => 'top'
    ));

    // will generate a url like.
    /posts/index?page=1#top

.. _redirect-routing:

Redirect routing
================

Redirect routing allows you to issue HTTP status 30x redirects for
incoming routes, and point them at different urls. This is useful
when you want to inform client applications that a resource has moved
and you don't want to expose two urls for the same content

Redirection routes are different from normal routes as they perform an actual
header redirection if a match is found. The redirection can occur to
a destination within your application or an outside location::

    Router::redirect(
        '/home/*',
        array('controller' => 'posts', 'action' => 'view'),
        array('persist' => true)
    );

Redirects ``/home/*`` to ``/posts/view`` and passes the parameters to
``/posts/view``.  Using an array as the redirect destination allows
you to use other routes to define where a url string should be
redirected to.  You can redirect to external locations using
string urls as the destination::

    Router::redirect('/posts/*', 'http://google.com', array('status' => 302));

This would redirect ``/posts/*`` to ``http://google.com`` with a
HTTP status of 302.

.. _disabling-default-routes:

Disabling the default routes
============================

If you have fully customized all your routes, and want to avoid any
possible duplicate content penalties from search engines, you can
remove the default routes that CakePHP offers by deleting them from your
application's routes.php file.

This will cause CakePHP to serve errors, when users try to visit
urls that would normally be provided by CakePHP but have not
been connected explicitly.

.. _custom-route-classes:

Custom Route classes
====================

Custom route classes allow you to extend and change how individual
routes parse requests and handle reverse routing. A route class
should extend :php:class:`CakeRoute` and implement one or both of
``match()`` and/or ``parse()``. ``parse()`` is used to parse requests and
``match()`` is used to handle reverse routing.

You can use a custom route class when making a route by using the
``routeClass`` option, and loading the file containing your route
before trying to use it::

    Router::connect(
         '/:slug',
         array('controller' => 'posts', 'action' => 'view'),
         array('routeClass' => 'SlugRoute')
    );

This route would create an instance of ``SlugRoute`` and allow you
to implement custom parameter handling.

Router API
==========

.. php:class:: Router

    Router manages generation of outgoing urls, and parsing of incoming
    request uri's into parameter sets that CakePHP can dispatch.

.. php:staticmethod:: connect($route, $defaults = array(), $options = array())

    :param string $route: A string describing the template of the route
    :param array $defaults: An array describing the default route parameters.
        These parameters will be used by default
        and can supply routing parameters that are not dynamic.
    :param array $options: An array matching the named elements in the route
        to regular expressions which that element should match.  Also contains
        additional parameters such as which routed parameters should be
        shifted into the passed arguments, supplying patterns for routing
        parameters and supplying the name of a custom routing class.

    Routes are a way of connecting request urls to objects in your application.
    At their core routes are a set or regular expressions that are used to
    match requests to destinations.

    Examples::

        Router::connect('/:controller/:action/*');

    The first parameter will be used as a controller name while the second is
    used as the action name. The '/\*' syntax makes this route greedy in that
    it will match requests like `/posts/index` as well as requests like
    ``/posts/edit/1/foo/bar`` .::

        Router::connect('/home-page', array('controller' => 'pages', 'action' => 'display', 'home'));

    The above shows the use of route parameter defaults. And providing routing
    parameters for a static route.::

        Router::connect(
            '/:lang/:controller/:action/:id',
            array(),
            array('id' => '[0-9]+', 'lang' => '[a-z]{3}')
        );

    Shows connecting a route with custom route parameters as well as providing
    patterns for those parameters. Patterns for routing parameters do not need
    capturing groups, as one will be added for each route params.

    $options offers three 'special' keys. ``pass``, ``persist`` and ``routeClass``
    have special meaning in the $options array.

    * ``pass`` is used to define which of the routed parameters should be
      shifted into the pass array.  Adding a parameter to pass will remove
      it from the regular route array. Ex. ``'pass' => array('slug')``

    * ``persist`` is used to define which route parameters should be automatically
      included when generating new urls. You can override persistent parameters
      by redefining them in a url or remove them by setting the parameter to
      ``false``.  Ex. ``'persist' => array('lang')``

    * ``routeClass`` is used to extend and change how individual routes parse
      requests and handle reverse routing, via a custom routing class.
      Ex. ``'routeClass' => 'SlugRoute'``

    * ``named`` is used to configure named parameters at the route level.
      This key uses the same options as :php:meth:`Router::connectNamed()`

.. php:staticmethod:: redirect($route, $url, $options = array())

    :param string $route: A route template that dictates which urls should
        be redirected.
    :param mixed $url: Either a :term:`routing array` or a string url
        for the destination of the redirect.
    :param array $options: An array of options for the redirect.

    Connects a new redirection Route in the router.
    See :ref:`redirect-routing` for more information.

.. php:staticmethod:: connectNamed($named, $options = array())

    :param array $named: A list of named parameters. Key value pairs are accepted where
        values are either regex strings to match, or arrays.
    :param array $options: Allows control of all settings:
        separator, greedy, reset, default

    Specifies what named parameters CakePHP should be parsing out of
    incoming urls. By default CakePHP will parse every named parameter
    out of incoming URLs. See :ref:`controlling-named-parameters` for
    more information.

.. php:staticmethod:: promote($which = null)

    :param integer $which: A zero-based array index representing the route to move.
        For example, if 3 routes have been added, the last route would be 2.

    Promote a route (by default, the last one added) to the beginning of the list.

.. php:staticmethod:: url($url = null, $full = false)

    :param mixed $url: Cake-relative URL, like "/products/edit/92" or
        "/presidents/elect/4" or a :term:`routing array`
    :param mixed $full: If (bool) true, the full base URL will be prepended
        to the result. If an array accepts the following keys

           * escape - used when making urls embedded in html escapes query
             string '&'
           * full - if true the full base URL will be prepended.

    Generate a URL for the specified action. Returns an URL pointing
    to a combination of controller and action. $url can be:

    * Empty - the method will find the address to the actual controller/action.
    * '/' - the method will find the base URL of application.
    * A combination of controller/action - the method will find the url for it.

    There are a few 'special' parameters that can change the final URL string that is generated:

    * ``base`` - Set to false to remove the base path from the generated URL.
      If your application is not in the root directory, this can be used to
      generate URLs that are 'cake relative'. Cake relative URLs are required
      when using requestAction.
    * ``?`` - Takes an array of query string parameters
    * ``#`` - Allows you to set URL hash fragments.
    * ``full_base`` - If true the :php:const:`FULL_BASE_URL` constant will
      be prepended to generated URLs.

.. php:staticmethod:: mapResources($controller, $options = array())

    Creates REST resource routes for the given controller(s).  See
    the :doc:`/development/rest` section for more information.

.. php:staticmethod:: parseExtensions($types)

    Used in routes.php to declare which :ref:`file-extensions` your application
    supports.  By providing no arguments, all file extensions will be supported.

.. php:staticmethod:: setExtensions($extensions, $merge = true)

    .. versionadded:: 2.2

    Set or add valid extensions. To have the extensions parsed, you are still
    required to call :php:meth:`Router::parseExtensions()`.

.. php:staticmethod:: defaultRouteClass($classname)

    .. versionadded:: 2.1

    Set the default route to be used when connecting routes in the future.

.. php:class:: CakeRoute

    The base class for custom routes to be based on.

.. php:method:: parse($url)

    :param string $url: The string url to parse.

    Parses an incoming url, and generates an array of request parameters
    that Dispatcher can act upon. Extending this method allows you to customize
    how incoming URLs are converted into an array.  Return ``false`` from
    URL to indicate a match failure.

.. php:method:: match($url)

    :param array $url: The routing array to convert into a string URL.

    Attempt to match a URL array.  If the URL matches the route parameters
    and settings, then return a generated string URL.  If the URL doesn't
    match the route parameters, false will be returned.  This method handles
    the reverse routing or conversion of URL arrays into string URLs.

.. php:method:: compile()

    Force a route to compile its regular expression.


.. meta::
    :title lang=en: Routing
    :keywords lang=en: controller actions,default routes,mod rewrite,code index,string url,php class,incoming requests,dispatcher,url url,meth,maps,match,parameters,array,config,cakephp,apache,router
