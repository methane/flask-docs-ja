.. _deploying-wsgi-standalone:

スタンドアロンのWSGIコンテナ
=============================

.. Standalone WSGI Containers
   ==========================

.. There are popular servers written in Python that contain WSGI applications and
   serve HTTP.  These servers stand alone when they run; you can proxy to them
   from your web server.  Note the section on :ref:`deploying-proxy-setups` if you
   run into issues.

Pythonで書かれているWSGIアプリケーションやHTTP機能を持つ人気のサーバーがあります。
これらのサーバーは、それだけで動かすことができます。
ウェブサーバーからそれらにプロキシさせて動かすこともできます。
起動するときに問題があった場合、 :ref:`deploying-proxy-setups` の章を確認して下さい。

Gunicorn
--------

.. `Gunicorn`_ 'Green Unicorn' is a WSGI HTTP Server for UNIX. It's a pre-fork
   worker model ported from Ruby's Unicorn project. It supports both `eventlet`_
   and `greenlet`_. Running a Flask application on this server is quite simple::

`Gunicorn`_ 、 'Green Unicorn' は、UNIX用のWSGIHTTPサーバーです。
RubyのUnicornプロジェクトから派生したpre-fork workerモデルを採用しています。
`eventlet`_ と `greenlet`_ をサポートしています。
このサーバーでFlaskアプリケーションを起動するのはとても簡単です。 ::

    gunicorn myproject:app

.. `Gunicorn`_ provides many command-line options -- see ``gunicorn -h``.
   For example, to run a Flask application with 4 worker processes (``-w
   ``) binding to localhost port 4000 (``-b 127.0.0.1:4000``)::

`Gunicorn`_ には、たくさんのコマンドラインオプションがあります。
``gunicorn -h`` で確認して下さい。
例えば、Flaskアプリケーションをlocalhostのポート番号4000(``-b 127.0.0.1:4000``)で、
4つのworkerプロセス(``-w``)で起動するには以下のようにします。 ::

    gunicorn -w 4 -b 127.0.0.1:4000 myproject:app

.. _Gunicorn: http://gunicorn.org/
.. _eventlet: http://eventlet.net/
.. _greenlet: http://codespeak.net/py/0.9.2/greenlet.html

Tornado
--------

.. `Tornado`_ is an open source version of the scalable, non-blocking web
   server and tools that power `FriendFeed`_.  Because it is non-blocking and
   uses epoll, it can handle thousands of simultaneous standing connections,
   which means it is ideal for real-time web services.  Integrating this
   service with Flask is straightforward::

`Tornado`_ は、 `FriendFeed`_　で使われているスケーラブルでノンブロッキングなオープンソースのウェブサーバーです。
epollを使ってノンブロッキングなので、数千のコネクションを同時に処理することができます。
これは、リアルタイムのウェブサービスにとって理想的なものであるということを示しています。
このサービスをFlaskと統合することは簡単です。 ::

    from tornado.wsgi import WSGIContainer
    from tornado.httpserver import HTTPServer
    from tornado.ioloop import IOLoop
    from yourapplication import app

    http_server = HTTPServer(WSGIContainer(app))
    http_server.listen(5000)
    IOLoop.instance().start()


.. _Tornado: http://www.tornadoweb.org/
.. _FriendFeed: http://friendfeed.com/

Gevent
-------

.. `Gevent`_ is a coroutine-based Python networking library that uses
   `greenlet`_ to provide a high-level synchronous API on top of `libevent`_
   event loop::

`Gevent`_ は、 `libevent`_ のイベントループのハイレベルな同期APIを提供するために
`greenlet`_ が使われているコルーチンベースのPythonネットワークライブラリです。

    from gevent.wsgi import WSGIServer
    from yourapplication import app

    http_server = WSGIServer(('', 5000), app)
    http_server.serve_forever()

.. _Gevent: http://www.gevent.org/
.. _greenlet: http://codespeak.net/py/0.9.2/greenlet.html
.. _libevent: http://monkey.org/~provos/libevent/

.. _deploying-proxy-setups:

プロキシの設定
---------------

.. Proxy Setups
   ------------

.. If you deploy your application using one of these servers behind an HTTP proxy
   you will need to rewrite a few headers in order for the application to work.
   The two problematic values in the WSGI environment usually are `REMOTE_ADDR`
   and `HTTP_HOST`.  You can configure your httpd to pass these headers, or you
   can fix them in middleware.  Werkzeug ships a fixer that will solve some common
   setups, but you might want to write your own WSGI middleware for specific
   setups.

HTTPプロキシの後ろで、これらのサーバーのいずれかを使ってアプリケーションをデプロイする場合、
動作するアプリケーションのいくつかのヘッダーを書き換える必要があります。
WSGI環境で2つの問題となりそうな値は、通常 `REMOTE_ADDR` と `HTTP_HOST` です。
これらのヘッダーに渡すhttpdを設定したり、ミドルウェアでそれらを修正することができます。
Werkzeugはいくつかの一般的なセットアップを解決する機能を持っていますが、
特定のセットアップ用にWSGIミドルウェアを書きたいかもしれません。

.. Here's a simple nginx configuration which proxies to an application served on
   localhost at port 8000, setting appropriate headers:

以下に、localhostのポートが8000で配信されるアプリケーションに対してnginxでプロキシを設定する簡単な方法を紹介します。 :

.. sourcecode:: nginx

    server {
        listen 80;

        server_name _;

        access_log  /var/log/nginx/access.log;
        error_log  /var/log/nginx/error.log;

        location / {
            proxy_pass         http://127.0.0.1:8000/;
            proxy_redirect     off;

            proxy_set_header   Host             $host;
            proxy_set_header   X-Real-IP        $remote_addr;
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
    }

.. If your httpd is not providing these headers, the most common setup invokes the
   host being set from `X-Forwarded-Host` and the remote address from
   `X-Forwarded-For`::

httpdでこれらのヘッダーが提供されていない場合、
最も一般的な設定は `X-Forwarded-Host` を設定するか、
`X-Forwarded-For` のリモートアドレス

    from werkzeug.contrib.fixers import ProxyFix
    app.wsgi_app = ProxyFix(app.wsgi_app)

.. Trusting Headers

   Please keep in mind that it is a security issue to use such a middleware in
   a non-proxy setup because it will blindly trust the incoming headers which
   might be forged by malicious clients.

.. admonition:: 信頼できるヘッダー

   non-proxyでセットアップするミドルウェアを使うと
   それは盲目的に悪意のあるクライアントが偽造されるかもしれない入って来るヘッダを信頼するので、
   非プロキシ設定でこのようなミドルウェアを使用するようにセキュリティの問題であることに注意してください。

.. If you want to rewrite the headers from another header, you might want to
   use a fixer like this::

他のヘッダーからヘッダーを書き換えたい場合、以下のようなfixerを使えるかもしれません。 ::

    class CustomProxyFix(object):

        def __init__(self, app):
            self.app = app

        def __call__(self, environ, start_response):
            host = environ.get('HTTP_X_FHOST', '')
            if host:
                environ['HTTP_HOST'] = host
            return self.app(environ, start_response)

    app.wsgi_app = CustomProxyFix(app.wsgi_app)
