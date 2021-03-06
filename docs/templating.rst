.. Templates
   =========

テンプレート
==================

.. Flask leverages Jinja2 as template engine.  You are obviously free to use
   a different template engine, but you still have to install Jinja2 to run
   Flask itself.  This requirement is necessary to enable rich extensions.
   An extension can depend on Jinja2 being present.

FlaskはテンプレートエンジンとしてJinja2を使っています。
違うテンプレートエンジンを使うことは自由ですが、Flaskを実行するためにJinja2もインストールしなければいけません。
この仕様は豊富な数の拡張機能を可能とする必要があります。
拡張機能は既にあるように、Jinja2に依存させることができます。

.. This section only gives a very quick introduction into how Jinja2
   is integrated into Flask.  If you want information on the template
   engine's syntax itself, head over to the official `Jinja2 Template
   Documentation <http://jinja.pocoo.org/2/documentation/templates>`_ for
   more information.

この章は、Jinja2をどのようにFlaskに統合しているかを簡単に紹介しています。
テンプレートエンジンのシンタックスの詳しい情報が必要な場合、
公式の `Jinja2 Template Documentation <http://jinja.pocoo.org/2/documentation/templates>`_
を見て下さい。

.. Jinja Setup
   -----------

Jinjaのセットアップ
-------------------------

.. Unless customized, Jinja2 is configured by Flask as follows:

カスタマイズした場合を除き、Jinja2はFlaskによって以下のように設定されます。 :

.. autoescaping is enabled for all templates ending in ``.html``,
   ``.htm``, ``.xml`` as well as ``.xhtml``
.. a template has the ability to opt in/out autoescaping with the
   ``{% autoescape %}`` tag.
.. Flask inserts a couple of global functions and helpers into the
   Jinja2 context, additionally to the values that are present by
   default.

- 自動エスケープは、ファイル名の最後が、 ``.html`` 、 ``.htm`` 、 ``.xml`` 、
  ``.xhtml`` などで終わるすべてのテンプレートで有効化されます。
- テンプレートでは、 ``{% autoescape %}`` タグで自動エスケープをオプトイン/アウト
  する機能があります。
- Flaskはグローバル関数やヘルパー関数に、
  デフォルトで表示される値に追加するためのJinja2のコンテキストを挿入することができます。

.. Standard Context
   ----------------

標準のコンテキスト
-------------------

.. The following global variables are available within Jinja2 templates
   by default:

以下のグローバル変数は、デフォルトでJinja2テンプレート内で有効です。 :

.. data:: config
   :noindex:

   .. The current configuration object (:data:`flask.config`)

   現在の設定オブジェクト (:data:`flask.config`)

   .. versionadded:: 0.6

   .. versionchanged:: 0.10
      This is now always available, even in imported templates.

.. data:: request
   :noindex:

   .. The current request object (:class:`flask.request`).  This variable is
      unavailable if the template was rendered without an active request
      context.

   現在のリクエストオブジェクト (:class:`flask.request`)

.. data:: session
   :noindex:

   .. The current session object (:class:`flask.session`).  This variable
      is unavailable if the template was rendered without an active request
      context.

   現在のセッションオブジェクト (:class:`flask.session`)

.. data:: g
   :noindex:

   .. The request-bound object for global variables (:data:`flask.g`).  This
      variable is unavailable if the template was rendered without an active
      request context.

   グローバルの値に対してリクエストにバインドされているオブジェクト (:data:`flask.g`)

.. function:: url_for
   :noindex:

   .. The :func:`flask.url_for` function.

   :func:`flask.url_for` 関数

.. function:: get_flashed_messages
   :noindex:

   .. The :func:`flask.get_flashed_messages` function.

   :func:`flask.get_flashed_messages` 関数

.. The Jinja Context Behavior

   These variables are added to the context of variables, they are not
   global variables.  The difference is that by default these will not
   show up in the context of imported templates.  This is partially caused
   by performance considerations, partially to keep things explicit.

   What does this mean for you?  If you have a macro you want to import,
   that needs to access the request object you have two possibilities:

   1.   you explicitly pass the request to the macro as parameter, or
        the attribute of the request object you are interested in.
   2.   you import the macro "with context".

   Importing with context looks like this:

.. admonition:: Jinjaのコンテキストの振る舞い

   これらの変数は変数のコンテキストに追加されるので、グローバル変数ではありません。
   違いとして、インポートしたテンプレートのコンテキストでこれらの変数デフォルトで参照できないことです。
   これはパフォーマンスに考慮する際に特に注意しなければいけません。

   What does this mean for you?  If you have a macro you want to import,
   that needs to access the request object you have two possibilities:

   1.   you explicitly pass the request to the macro as parameter, or
        the attribute of the request object you are interested in.
   2.   you import the macro "with context".

   以下のようにしてコンテキストをインポートして下さい。:

   .. sourcecode:: jinja

      {% from '_helpers.html' import my_macro with context %}

.. Standard Filters
   ----------------

標準のフィルター
-------------------

.. These filters are available in Jinja2 additionally to the filters provided
   by Jinja2 itself:

これらのフィルターはJinja2で有効になっています。
さらに、Jinja2自体で用意されているフィルターが利用できます。

.. function:: tojson
   :noindex:

   .. This function converts the given object into JSON representation.  This
      is for example very helpful if you try to generate JavaScript on the
      fly.

   この関数は、与えられたオブジェクトをJSONに整形して変換します。
   実行中にJavaScriptを生成しようとする場合、これはとても便利な例です。

   .. Note that inside `script` tags no escaping must take place, so make
      sure to disable escaping with ``|safe`` if you intend to use it inside
      `script` tags:

   `script` タグ

   .. sourcecode:: html+jinja

       <script type=text/javascript>
           doSomethingWith({{ user.username|tojson|safe }});
       </script>

   .. That the ``|tojson`` filter escapes forward slashes properly for you.

   その ``|tojson`` フィルターは適切にスラッシュをエスケープします。

.. Controlling Autoescaping
   ------------------------

自動エスケープのコントロール
---------------------------------

Autoescaping is the concept of automatically escaping special characters
of you.  Special characters in the sense of HTML (or XML, and thus XHTML)
are ``&``, ``>``, ``<``, ``"`` as well as ``'``.  Because these characters
carry specific meanings in documents on their own you have to replace them
by so called "entities" if you want to use them for text.  Not doing so
would not only cause user frustration by the inability to use these
characters in text, but can also lead to security problems.  (see
:ref:`xss`)

Sometimes however you will need to disable autoescaping in templates.
This can be the case if you want to explicitly inject HTML into pages, for
example if they come from a system that generate secure HTML like a
markdown to HTML converter.

There are three ways to accomplish that:

-   In the Python code, wrap the HTML string in a :class:`~flask.Markup`
    object before passing it to the template.  This is in general the
    recommended way.
-   Inside the template, use the ``|safe`` filter to explicitly mark a
    string as safe HTML (``{{ myvariable|safe }}``)
-   Temporarily disable the autoescape system altogether.

To disable the autoescape system in templates, you can use the ``{%
autoescape %}`` block:

.. sourcecode:: html+jinja

    {% autoescape false %}
        <p>autoescaping is disabled here
        <p>{{ will_not_be_escaped }}
    {% endautoescape %}

Whenever you do this, please be very cautious about the variables you are
using in this block.

.. _registering-filters:

フィルターの登録
----------------------

.. Registering Filters
   -------------------

.. If you want to register your own filters in Jinja2 you have two ways to do
   that.  You can either put them by hand into the
   :attr:`~flask.Flask.jinja_env` of the application or use the
   :meth:`~flask.Flask.template_filter` decorator.

Jinja2に独自のフィルターを登録したい場合は、2つの方法があります。
アプリケーションの :attr:`~flask.Flask.jinja_env` に手動で登録するか、
:meth:`~flask.Flask.template_filter` デコレーターを使うかのどちらかで可能です。

.. The two following examples work the same and both reverse an object::

以下の2つのサンプルは同じ事をしています。 ::

    @app.template_filter('reverse')
    def reverse_filter(s):
        return s[::-1]

    def reverse_filter(s):
        return s[::-1]
    app.jinja_env.filters['reverse'] = reverse_filter

.. In case of the decorator the argument is optional if you want to use the
   function name as name of the filter.  Once registered, you can use the filter
   in your templates in the same way as Jinja2's builtin filters, for example if
   you have a Python list in context called `mylist`::

デコレーターのケースではフィルターの名前として関数名使いたい場合は引数は任意です。
登録したなら、Jinja2の組み込みのフィルターと同じようにテンプレートでフィルターを使うことができます。
例として、以下のものは `mylist` というコンテキストにPythonのリストがある場合です。 ::

    {% for x in mylist | reverse %}
    {% endfor %}


.. Context Processors
   ------------------

コンテキストプロセッサー
---------------------------

To inject new variables automatically into the context of a template
context processors exist in Flask.  Context processors run before the
template is rendered and have the ability to inject new values into the
template context.  A context processor is a function that returns a
dictionary.  The keys and values of this dictionary are then merged with
the template context, for all templates in the app::

    @app.context_processor
    def inject_user():
        return dict(user=g.user)

The context processor above makes a variable called `user` available in
the template with the value of `g.user`.  This example is not very
interesting because `g` is available in templates anyways, but it gives an
idea how this works.

Variables are not limited to values; a context processor can also make
functions available to templates (since Python allows passing around
functions)::

    @app.context_processor
    def utility_processor():
        def format_price(amount, currency=u'€'):
            return u'{0:.2f}{1}.format(amount, currency)
        return dict(format_price=format_price)

.. The context processor above makes the `format_price` function available to all
   templates::

上のコンテキストプロセッサーは、 `format_price` 関数を全てのテンプレートで使えるようにしています。 ::

    {{ format_price(0.33) }}

.. You could also build `format_price` as a template filter (see
   :ref:`registering-filters`), but this demonstrates how to pass functions in a
   context processor.

テンプレートフィルター(:ref:`registering-filters` を見て下さい)として `format_price` を生成することもできますが、
このデモはコンテキストプロセッサーに関数がどのようにして渡されるのかを示すものです。
