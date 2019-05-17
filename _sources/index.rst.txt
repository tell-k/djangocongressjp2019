==========================================================
Djangoで静的ファイルとうまくやる
==========================================================

| tell-k
| DjangoCongress JP 2019 (2019.05.18)


おまえ誰よ？
=====================================

.. image:: https://pbs.twimg.com/profile_images/1045138776224231425/3GD8eWeG_200x200.jpg

* tell-k
* BePROUD.inc
* 情弱プログラマー
* https://twitter.com/tell_k
* https://tell-k.github.io/djangocongressjp2019/

BePROUD - Pythonメインの受託開発
========================================

.. figure:: https://dl.dropboxusercontent.com/spa/ghyn87yb4ejn5yy/3ce742cb9bef4e4f925cb7385e494ff4.png
   :width: 70%

   https://www.beproud.jp/

connpass - エンジニアをつなぐIT勉強会支援プラットフォーム
===============================================================

.. figure:: https://dl.dropboxusercontent.com/spa/ghyn87yb4ejn5yy/c4f1511638f241daaaac9e29ed3151c1.png
   :width: 70%

   https://connpass.com/

PyQ - Pythonオンライン学習サービス
========================================

.. figure:: https://dl.dropboxusercontent.com/spa/ghyn87yb4ejn5yy/e9121e88c3b64179993a02198a7514f9.png
   :width: 70%

   https://pyq.jp/ ★ Djangoを使ったWeb開発も学習できます! ★

.. rst-class:: slide-green-white

目的/動機
=====================================

* 最近Q&Aサイトとかで「 **本番にあげたらCSS/JS/画像が表示されない!** 」というのをよく見かけました
* 私も最初の頃は良く分からなかくて辛かった
* 入門したてのころはPythonコードやDBやらを覚えることがいっぱい
* 何とく表示できたりするので静的ファイル扱いは後回しにされがち
* 結果、後から失敗して痛い目をみる辛い
* そういう経験も踏まえて、DjangoもといWebアプリ開発における静的ファイルの取り回しについてまとめてみようと思った次第

.. rst-class:: slide-green-white

対象
=====================================

* これから Django を学び始める人
* Webアプリを作ろうと勉強してる人
* Django は少し触ったことがあるけど、 ``collectstatic`` が意味わからなかった人
* ``settings.DEBUG = False`` にして静的ファイルが表示されなくて焦った経験がある **私みたいな人**

.. rst-class:: slide-green-white

今日の目標
=====================================

**「わかるっ！俺には静的ファイルがわかる！！！」**

.. image:: http://4.bp.blogspot.com/-zTvzECyWEsk/VwIjHWMdszI/AAAAAAAA5e4/W_kAnVythXoHGzGO3AkgrHImS3cpvMiuQ/s800/internet_kanki_man1.png

.. rst-class:: slide-green-white

前提
=====================================

* サンプルコードは全て Python 3.7,  Django 2.2.1
* クラウドサービスの話は AWS がメインです
* HTTPサーバーの話は nginx がメインです

.. rst-class:: slide-green-white

目次
=========================================

* 静的ファイルとは
* わたしの失敗談
* Djangoの静的ファイルの扱い
* 本番公開のパターン

  * 単一のサーバ
  * 複数台のサーバー
  * クラウドのストレージの利用
  * CDNを利用する

* 静的ファイル関連Tips
* 参考


.. rst-class:: slide-cyan
.. rst-class:: center-text

静的ファイルとは
=============================

静的ファイルとは

.. rst-class:: slide-cyan-white

静的ファイルとは
=============================

* そもそも静的ファイルとはなんでしょうか？
* よく **「静的ファイルとは CSS, JS、画像ファイル とかです」**
* みたいなざっくりな説明されますが Python スクリプトだってファイルじゃないですか？静的とは一体？
* みたいな疑問をもったりしませんでしたか？

.. rst-class:: slide-cyan-white

静的ファイルとは
=============================

* MDN には以下のように書いてあります

静的Webサーバ、またはスタックは、コンピューター (ハードウェア) と
HTTP サーバ (ソフトウェア) から構成されます
**サーバが保持しているファイルをブラウザーへ「そのまま」送るので、「静的」と呼ばれます**

via `Web サーバとは - Web 開発を学ぶ | MDN <https://developer.mozilla.org/ja/docs/Learn/Common_questions/What_is_a_web_server>`_

.. rst-class:: slide-cyan-white

静的Webサーバ
=============================

リクエストされたサーバー内のファイルをそのままHTTPレスポンスとして返すだけ

.. image:: _static/img/static-server.png
   :width: 100%


.. rst-class:: slide-cyan-white

動的コンテンツ
=============================

* 一方動的とは以下のように書いてあります

動的Webサーバは、静的Webサーバに加えて追加のソフトウェア、
一般的にはアプリケーションサーバおよびデータベースで構成されます
**アプリケーションサーバが、 HTTP サーバを通してブラウザーに送信する前に、保持しているファイルを更新するので「動的」と呼ばれます**

----

**「動的」はサーバがコンテンツを処理したり、データベースからその場で作成したりすることを意味します**
この方法はより柔軟性を提供できますが、
技術スタックがより扱いにくくなり、Webサイトを構築することは劇的に複雑になります

via `Web サーバとは - Web 開発を学ぶ | MDN <https://developer.mozilla.org/ja/docs/Learn/Common_questions/What_is_a_web_server>`_

.. rst-class:: slide-cyan-white

動的Webサーバー
=============================

いわゆる Webアプリーケション

.. image:: _static/img/dynamic-server.png
   :width: 90%

.. rst-class:: slide-cyan-white

Django は?
=============================================

* Webアプリをつくるためのフレームワークです
* **動的コンテンツを生成し配信する** のが主な役割です
* アプリケーションサーバーとしてサーバー内に配置されて稼働します
* 本番環境では ``gunicorn`` や ``uWSGI`` というライブラリを利用して ``WSGIアプリケーション`` として運用されるのが一般的です

.. rst-class:: slide-cyan-white

ここまでのまとめ
=============================================

* **静的ファイルとはHTTPサーバーがそのまま返すファイル群の事を指す**
* そのまま返すならだいたい全部静的ファイル
* 静的ファイルは静的コンテンツとも呼ばれたり ``static`` ファイルや ``assets`` などとも呼ばれる

呼び方の話

* MDNにならってHTTPサーバーと呼んでいますが、一般的にはこのソフトウェアのことをさしてWebサーバーと呼んだりもします
* この発表では **HTTPサーバー** で統一します


.. rst-class:: slide-red
.. rst-class:: center-text

dummy
=============================

私が失敗した話

.. rst-class:: slide-red-white

私が失敗した話
==========================================

* Django を触り始めた頃に失敗した話
* 公開前の本番サーバーにDjangoアプリをデプロイ
* アプリが動いたと思ったら

.. rst-class:: slide-red-white

失敗1. CSSが全く当たってない...
=========================================

* 画像もJSもブラウザで読み込めてないよ
* ローカル環境や、開発サーバーでは表示されてましたよ？
* 開発サーバーとの差異は **本番用に設定ファイルを変えたくらいなのに**

.. rst-class:: slide-red-white

nginx で静的ファイルを配信すればいいいのか!
==============================================

* **プロジェクト内のstaticディレクトリを配信元に設定した**
* そして ``nginx`` 再起動したぞー
* やったーサイトが見えたー

.. rst-class:: slide-red-white

失敗2. まだ Django Admin が真っ白
=========================================

* 分からない俺はなにも分からない
* **そもそも Django AdminのCSS/JSはサーバー内のどこにあるんや??**
* 良く分からないけど調べたら **site-packages以下** にあった！

::

 例) djangoのadminアプリの中に静的ファイルがある

 /site-packages/django/contrib/admin/static/
 └── admin
     ├── css
     ├── fonts
     ├── img
     └── js


* これを **自分のプロジェクトにコピーして表示できた！ヨシ！**

.. rst-class:: slide-red-white

全然「ヨシ！」 ではない
====================================

.. image:: https://4.bp.blogspot.com/-EBpxVigkCCY/V5Xc1CHSeEI/AAAAAAAA8u0/9XIAzDJaQNU3HIiXi4PCPK3aMip3aoGyACLcB/s400/pose_sugoi_okoru_man.png

.. rst-class:: slide-green
.. rst-class:: center-text

Djangoの静的ファイルの扱い
=================================

Djangoの静的ファイルの扱い

.. rst-class:: slide-green-white

Djangoの静的ファイルの扱い
=================================

* **茶番** はこれくらいにして Django の話に戻りましょう
* ここからは、Djangoでの静的ファイルを基本的な扱い方について見ていきましょう
* ついでにメディアファイルの話もします
* その過程でさっきの何が悪かったのかもを解説します

.. rst-class:: slide-green-white

開発モードで静的ファイルを配信
======================================

* 開発する時は、 ``python manage.py runserver``  でDjangoアプリを起動
* デフォルトで何もしなければ **静的ファイル** を配信してくれます
* **settings.DEBUG = True** の時だけ有効になります

なぜ？

* **開発時に静的ファイルが表示されないのは不便だから**
* だから開発モードの時だけは静的ファイルを配信する
* ``django.contrib.staticfiles`` が機能を提供している

.. rst-class:: slide-green-white

あくまで開発時の補助用
======================================

* あくまで開発の補助用なので **本番で利用することを想定していない**

::

  上で述べたたように django.contrib.staticfiles を利用する場合、
  DEBUG が True であれば runserver は自動的にこの処理を行います

  〜略〜

  この機能は本番環境で利用するのに適していません!
  一般的なデプロイ方法に関しては 静的ファイルのデプロイ を参照ください

via `静的ファイル (画像、JavaScript、CSS など) を管理する > 開発時の静的ファイルの取扱い <https://docs.djangoproject.com/ja//2.2/howto/static-files/#serving-static-files-during-development>`_

.. rst-class:: slide-green-white

失敗1の原因
======================================

* 先ほどの **失敗1. CSSが全く当たってない...** の理由がこれです
* 本番で ``settings.DEBUG = False`` に設定したまではよかったが...
* この結果、 **開発(DEBUG)モードがオフになった**  ので **静的ファイル配信機能も自動的にオフになった**
* 結果CSSが当たってなかった...

言われてみれば当たり前だけど、当時は良くわかってなかった

.. rst-class:: slide-red
.. rst-class:: center-text

DEBUG = True のまま本番公開してはダメ
=============================================

| DEBUG = True のまま
| 本番公開してはダメ

.. rst-class:: slide-red-white

何がまずいの？
======================================

* Django自体が静的ファイル配信に最適化されてない

  * なので ``python manage.py version runserver --insecure`` で公開もダメ

* デバッグ情報が画面上に出力される
* 例えばファイルパス、設定内容などが表示される
* 結果サイトの攻撃者に優位な情報を渡すことになります

それ以外にも

* SQLクエリも常に保存するのでメモリの消費も激しい
* パフォーマンスの観点でもよくない

via. https://docs.djangoproject.com/ja/2.2/ref/settings/#debug

.. rst-class:: slide-red
.. rst-class:: center-text

center2
=================================

Q．Django が静的ファイルを配信しないなら、**何が配信するの？**

.. rst-class:: slide-blue
.. rst-class:: center-text

center3
=================================

A．静的ファイルを配信するHTTPサーバーを用意します

.. rst-class:: slide-blue-white

DEBUG = Falseの時の静的ファイル配信方法
===========================================

* 例えば Djangoプロジェクト が 以下のような構成だったら

.. code-block:: python

 # settings.py  ---

 # 静的ファイルの探索はデフォルトでは アプリの staticディレクトリ以下
 # ファイルを探したりするのが面倒なので、プロジェクトの直下に
 # staticディレクトリをおきます

 STATICFILES_DIRS = (
     os.path.join(BASE_DIR, '..', 'static'),
 )

.. rst-class:: slide-blue-white

DEBUG = Falseの時の静的ファイル配信方法
===========================================

* 今回は プロジェクト直下の ``static`` に全ての静的ファイルがあるという前提

.. code-block:: bash

 myproject
 ├── .gitignore
 ├── apps
 │   ├── core
 │   ├── manage.py
 │   ├── polls
 │   ├── static  # <- デフォはここから探すが、今回はここに置かない
 │   └── apps
 ├── static       # <- 全部ここに置く
 │   ├── css
 │   ├── images
 │   └── js
 └── templates
     ├── base.html
     └── polls


.. rst-class:: slide-blue-white

HTTPサーバー で static 以下 を静的に配信
=========================================================

* HTTPサーバーとして Nginxを例にします

.. code-block:: nginx

 server {

     server_name example.com;

     location = /favicon.ico { access_log off; log_not_found off; }
     # http://example.com/static 以下は全てファイルを静的にそのまま返す
     location /static/ {
         root /path/to/myproject;
     }

     # それ以外はDjangoアプリにプロキシする
     location / {
         include proxy_params;
         proxy_pass http://unix:/run/uwsgi/myproject.sock;
     }
 }

.. rst-class:: slide-blue-white

HTTPサーバーが静的/動的を振り分ける
=========================================================

URLパスによって、静的ファイルを配信するか、Djangoが受けるかを振り分けます

.. image:: _static/img/http-server-1.png
   :width: 80%

.. rst-class:: slide-blue

静的ファイルを一つの場所に集める
=========================================================

.. rst-class:: slide-blue-white

静的ファイルを一つの場所に集める
=========================================================

* この状態はまだ **失敗2. まだ Django Admin が真っ白** の状態です
* Django系のアプリは、Pythonパッケージの中に **静的ファイルを一緒に持っています**
* それらも静的ファイルとして配信する必要があります
* ``django.cotnrib.admin`` だけでなく、そういうライブラリが他にもあります

.. rst-class:: slide-blue-white

静的ファイルを一つの場所に集める
=========================================================

* とはいえHTTPサーバーに一個一個設定するのは、現実的ではない
* コピペして持ってくるのも微妙

.. code-block:: nginx

 server {

     server_name example.com;

     location = /favicon.ico { access_log off; log_not_found off; }

     # /static/admin だけは site-package以下から配信する <- 流石に辛い
     location /static/admin/ {
         root /path/to/venv/lib/python3.7/site-packages/django/contrib/admin/;
     }

     location /static/ {
         root /path/to/myproject;
     }

     # 〜 省略 〜

.. rst-class:: slide-blue-white

collectstatic で集める
=========================================================

* なので必要な静的ファイルを一つのディレクトリに集約するコマンドがある
* それが ``collectstatic`` コマンドです集める場所の設定が ``STATIC_ROOT`` です


.. code-block:: python

 # settings.py ---

 STATIC_ROOT = os.path.join(BASE_DIR, '..', 'collected_static')

 # 追加の静的ファイル探索パス
 STATICFILES_DIRS =  (
    os.path.join(BASE_DIR, '..', 'static'),
 )

.. code-block:: bash

 $ python manage.py collectstatic

https://docs.djangoproject.com/ja/2.2/ref/contrib/staticfiles/#collectstatic

.. rst-class:: slide-blue-white

collectstatic で集める
=========================================================

.. code-block:: bash

 site-packages/django/contrib/admin/static/admin  # ... ①

 # ~ 省略 ~

 ├── apps
 ├── collected_static  # <- 全ての静的ファイルが集約される
 │   ├── admin         # ... ①
 │   ├── css           # ... ②
 │   ├── images
 │   └── js
 ├── static            # ... ② 設定で探索対象になっている
 │   ├── css
 │   ├── images
 │   └── js
 └── templates

.. code-block:: nginx

 # nginxの設定を変える

 # ディレクトリ名が変わったので root -> alias に変更
 location /static/ {
    alias /path/to/myproject/collcected_static;
 }

.. rst-class:: slide-blue-white

結局、公開するときはどうしたらいい？
==================================================

* 公開する時は ``DEBUG = False`` にしつつ ``collectstatc`` で静的ファイル集めましょう
* 集約ディレクトリを静的に配信するようにHTTPサーバーを設定しよう
* あとはデプロイツールの中で ``collectstatic`` を毎回実行させると良いです
* デプロイの時に ``Yes/No`` を毎回聞かれるのが面倒な場合は ``--no-input`` をつけると良いです

.. code-block:: bash

 # 毎回集めるかどうかを聞かれずに、実行されます
 $ python manage.py collectstatic --no-input

.. rst-class:: slide-blue-white

メディアファイル
==============================================

.. rst-class:: slide-blue-white

メディアファイル
==============================================

* Django では ``STAIC_`` から始まる設定と、 ``MEDIA_`` から始まる設定の2種類あります
* ``STATIC_`` は 静的ファイルの配信関連のものです
* ``MEDIA_`` は Djangoアプリが管理するファイルための設定です
* Djangoアプリが管理するファイル群を **メディアファイル** とDjangoは呼んでいます
* 例えば、アップロードファイルや、アップロード画像です

.. code-block:: python

  # settings.py ---
  # 例) メディアファイル関連の設定例

  MEDIA_URL = '/media/'  # メディアファイル配信URL
  MEDIA_ROOT = os.path.join(BASE_DIR, '..', 'media') # メディアファイルの保存先

.. rst-class:: slide-green-white

開発モードでメディアファイルを配信するには？
====================================================

* メディアファイルの場合は、自分で設定する必要があります
* 静的ファイルは ``DEBUG = True`` で自動で設定されます

.. code-block:: python

  from django.urls import path
  from django.conf import settings
  from django.conf.urls.static import static

  urlpatterns = [
      # 省略
  ]

  # 開発モードの時だけ、Djangoでメディアファイルを静的配信
  if settings.DEBUG:
      urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

.. rst-class:: slide-green-white

メディアファイルのアップロード時の注意点
====================================================

* HTTPサーバーのデフォルト設定だとリクエストボディ(ファイルサイズ)の上限が小さい
* 例えば Nginx は デフォルトで **1MB** が上限です
* なので上限に引っかかってエラーになったります
* 大きなファイルをアップロードする時は、Webサーバーの設定を変えておくと良いです

.. code-block:: nginx

  # 例) nginx で 10MBまで上限をあげる
  client_max_body_size 10M;

http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size

.. rst-class:: slide-green
.. rst-class:: center-text

本番環境で公開する
====================================================

本番環境で公開する

.. rst-class:: slide-green-white

本番環境で公開する
====================================================

* 本番環境と一口にいっても色々な環境があります

  * VPSなのか？
  * AWS使えるのか？
  * S3は使うのか？
  * Herokuなのか？
  * サーバは何台なのか？

* 採用するクラウドサービス や アーキテクチャによって静的ファイルの取り扱いも大きく異なります
* どういうパターンがあるのかみてみましょう

.. rst-class:: slide-green

単一のサーバーが配信する
=======================================================

.. rst-class:: slide-green-white

単一のサーバーが配信する
=======================================================

* サーバー1台だけ用意して配信するパターン
* 自分でサーバの設定をしてDjangoアプリを動かす
* 例えば以下のチュートリアルはこのパターンを想定してます

* `How To Set Up Django with Postgres, Nginx, and Gunicorn on Ubuntu 16.04 | DigitalOcean  <https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-16-04>`_
* `Setting up Django and your web server with uWSGI and nginx | uWSGI 2.0 documentation <https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html>`_

.. rst-class:: slide-green-white

Nginxの設定例
=======================================================

* Nginxの例

.. code-block:: nginx

 server {

     server_name example.com;

     location /media/ { # <- メディアファイル
        root /path/to/myproject;
     }
     location /static/ { # <- 静的ファイル
        alias /path/to/myproject/collcected_static;
     }
     location / { # <- それ以外はDjangoアプリ
         include proxy_params;
         proxy_pass http://unix:/run/uwsgi/myproject.sock;
     }

 # 〜 省略 〜

.. rst-class:: slide-green-white

単一のサーバーが配信する
=======================================================

.. image:: _static/img/http-server-2.png
   :width: 100%

.. rst-class:: slide-green

複数サーバー構成
=======================================================

.. rst-class:: slide-green-white

複数サーバー構成
=======================================================

* サイトの負荷が増大したら、DBサーバを別にしたり、サーバーを増やします
* サーバーの増えると前段に、ロードバランサー(負荷分散装置)が置かれてたりします
* サーバーを増やした場合は以下のようなことを考える必要があります

  * ``collecstatic`` はどこで実行するのか？
  * **片方のサーバーだけにアップロード** されたメディアファイルをどうするか？

.. rst-class:: slide-green-white

デプロイする時は
=======================================================

静的ファイル

* そろぞれのサーバーで ``collectstatic`` コマンドを叩く
* もしくは ``collectstatic`` で事前に集めたものを各サーバーに同期する

メディアファイル

* どのサーバーにアップロードされるかわからない
* アップロードされたら、各サーバーに同期する

どうやって同期するの？

* ``lsyncd`` とか ``rsync`` などのツールを使って同期
* 特定のディレクトリの変更を監視して、ファイルが更新された同期処理を行う
* サーバーが増えすぎて同期が追いつかなくなったら、静的ファイル専用のサーバーを用意する

.. rst-class:: slide-green-white

複数サーバー構成
=======================================================

.. image:: _static/img/multi-server.png
   :width: 90%

.. rst-class:: slide-green

クラウドストレージを利用する
=======================================================


.. rst-class:: slide-green-white

クラウドストレージを利用する
=======================================================

* 各社クラウドサービスには、静的ファイルを扱えるストレージサービスを持ってます

  * Amazon S3
  * Google Cloud Storage
  * Mirosoft Azure Storage

* これらのサービスを使うことで、先ほどの同期作業から解放されます
* 現代的には割とこちらの方が馴染みがありそうです
* ここでは AWS で S3 を使う例をみてみましょう

.. rst-class:: slide-green-white

クラウドストレージを利用する
=======================================================

.. image:: _static/img/cloud-multi-server.png
   :width: 100%

.. rst-class:: slide-green-white

Django で どうやるの？
============================================

* `django-storages <https://django-storages.readthedocs.io/en/latest/>`_  というライブラリを使うとできます
* S3 以外にも Azure Storage や Google Cloud Storage 等にも対応しています
* ほぼデフォクトスタンダードなライブラリな気がします

.. rst-class:: slide-green-white

django-storages の使い方例
============================================

* Djangoの設定は以下の通りです

.. code-block:: python

 # settings.py --

 AWS_ACCESS_KEY_ID = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
 AWS_SECRET_ACCESS_KEY = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxx'
 AWS_STORAGE_BUCKET_NAME = 'xxxxxxx'

 # メディアファイルをS3に保存
 DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'

 # collectstatic を実行すると静的ファイルをS3に保存
 STATICFILES_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'

https://django-storages.readthedocs.io/en/latest/backends/amazon-S3.html#settings

.. rst-class:: slide-green-white

django-storages の使い方例
============================================

* nginx 設定は S3 のバケットのURLに対してプロキシ設定します

.. code-block:: nginx

 location ~ ^/(static/|media/) {
   resolver         10.0.0.2;
   resolver_timeout 5s;
   set $s3_bucket "xxxxxxx.s3.amazonaws.com";

   proxy_pass http://$s3_bucket;
 }

.. rst-class:: slide-green-white

Nginx -> S3 にプロキシする時の注意点
============================================

* Nginx は 起動時に名前解決をして IPアドレスをキャッシュ
* しかし、S3のURLは一定時間でIPアドレスが動的に変更されてしまいます
* 定期的に ``resolver`` で名前解決を実行してIPアドレスを更新する必要があります
* `Nginx のDNS 名前解決とS3 やELB へのリバースプロキシ <https://yulii.github.io/nginx-dns-cache-20150815.html>`_

.. code-block:: nginx

  resolver         10.0.0.2;
  resolver_timeout 5s;

.. rst-class:: slide-green-white

Nginxでプロキシする必要があるの？
============================================

* S3はURLを吐き出すのでそれをそのまま利用しても良いです

  * 例) ``https://xxxxx.s3.amazonaws.com/media/hoge4.png``

* Nginxを介さない分、その分サーバーの負荷は減りそうです
* ただ、いざURLが変わるとなった時に取り回しがしにくいので、私はNginxでプロキシしてます

  * 例) S3のURLがテキストデータでどこかに保存されてるとか

.. rst-class:: slide-green-white

HTTPサーバーがない構成
============================================

.. rst-class:: slide-green-white

HTTPサーバーがない構成
============================================

* `Heroku <https://jp.heroku.com/>`_ ような PasS の場合
* そもそも HTTPサーバー が **置けないです**
* なので、Django で静的ファイルを配信する必要があります
* Herokuの場合、ファイル書き込みができないのでアップロードもできない

以下のチュートリアルは Heroku を想定したものです

* `Working with Django  | Heroku Dev Center <https://devcenter.heroku.com/categories/working-with-django>`_
* `Deploy your website on Heroku |  Django Girls Tutorial: Extensions <https://tutorial-extensions.djangogirls.org/ja/heroku/>`_
* `Django Tutorial Part 11: Deploying Django to production - Web 開発を学ぶ | MDN <https://developer.mozilla.org/ja/docs/Learn/Server-side/Django/Deployment>`_

.. rst-class:: slide-green-white

どうすればいいか？
===========================

静的ファイル

* `WhiteNoise <http://whitenoise.evans.io/en/stable/>`_  というライラリを使って配信します
* 開発用の静的ファイル配信とは違い、パフォーマンスを意識した静的ファイルが可能になります
* Django専用ではなく、WSGIアプリケーション全般に利用可能

メディアファイル

* S3 にアップロードするのが良いそうです
* `Direct to S3 File Uploads in Python <https://devcenter.heroku.com/articles/s3-upload-python>`_
* `CDP:Direct Object Uploadパターン <http://aws.clouddesignpattern.org/index.php/CDP:Direct_Object_Upload%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3>`_

.. rst-class:: slide-green-white

HTTPサーバーがない構成
============================

.. figure:: _static/img/heroku-arch.png
   :width: 100%

.. rst-class:: slide-green-white

WhiteNoise は大丈夫なの？
============================

* http://whitenoise.evans.io/en/stable/
* Djangoの開発用の静的ファイル配信とは違って
* Herokkuのような環境で動かすこと念頭に開発されている
* 静的ファイルの圧縮/キャッシングにも対応している
* Django単体で頑張るのでなく、CDNとの連携をしやすいようにしている

.. rst-class:: slide-green-white

ちなみにGAEは？
============================

* 同じ PaaS でも 静的ファイルを配信できる設定ができる

.. code-block:: yaml

 runtime: python37

 handlers:
 - url: /static
   static_dir: static/  # <- 静的配信してくれる

 - url: /.*
   script: auto

https://cloud.google.com/python/django/appengine?hl=ja

.. rst-class:: slide-green-white

CDNを利用する
===================

.. rst-class:: slide-green-white

CDNを利用する
===================

* Contents Delivery Networkの略
* 静的ファイルのロード時間はユーザー体験の良し悪しに直結します
* なるべく速くユーザーに静的ファイルを届けるための仕組みです

簡単に説明すると

* 世界中に配信サーバーを持ち、ユーザーに一番近い場所からコンテンツを配信可能
* コンテンツをキャッシュするので、オリジン(配信元)の負荷が減る
* `CDP:Cache Distributionパターン <http://aws.clouddesignpattern.org/index.php/CDP:Cache_Distribution%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3>`_

代表的なサービスは

* Amazon CloudFront
* Akamai
* Fastly

.. rst-class:: slide-green-white

CDNを利用する(CloudFront)
============================

.. figure:: _static/img/cloundfront-pattern.png
   :width: 85%

.. rst-class:: slide-green-white

CDN は Heroku で 使うのも良いです。
========================================

* `Using Amazon CloudFront CDN | Heroku Dev Center <https://devcenter.heroku.com/articles/using-amazon-cloudfront-cdn>`_
* キャッッシュが利用されることでDjangoへの負担が減る


.. rst-class:: slide-purple
.. rst-class:: center-text


その他Tips
=============================

その他Tips

.. rst-class:: slide-purple-white

Django Compressor
=============================

* CSS や JS を結合・圧縮してくれるライブラリ
* ファイルサイズを小さくして、リクエスト数が減らせる
* https://django-compressor.readthedocs.io/en/stable/usage/

.. code-block:: text

  {% load compress %}

  {% compress css %}
  <link rel="stylesheet" href="/static/css/one.css" type="text/css" charset="utf-8">
  <style type="text/css">p { border:5px solid green;}</style>
  <link rel="stylesheet" href="/static/css/two.css" type="text/css" charset="utf-8">
  {% endcompress %}

↓ 一つのCSSファイルに圧縮される

.. code-block:: text

  <link rel="stylesheet" href="/static/CACHE/css/f7c661b7a124.css" type="text/css" charset="utf-8">

.. rst-class:: slide-purple-white

認証付きの静的ファイル
=============================

* アップロードしたファイルとかは、その認証済みのユーザーしか見えないようにする
* Djangoで受けて静的配信しなくて良い方法がある
* アプリで閲覧権限をチェックしつつ、配信はHTTPサーバーでできるようにする設定

  * Nginx なら X-Accel-Redirect
  * Appache なら X-SendFile

* `X-SendFile、X-Accel-Redirectの使い方 <https://jyn.jp/x-sendfile-accel-redirect/>_`

.. rst-class:: slide-purple-white

認証付きの静的ファイル
===============================

.. image:: _static/img/x-accell-redirect.png
   :width: 80%

.. rst-class:: slide-green-white

参考
===============================

* 現場で使える Django の教科書《基礎編》

 * 10.4 静的ファイル関連の設定
 * 10.5 メディアファイル関連の設定

* 現場で使える Django の教科書《実践編》

  * 第5章開発のヒント(ファイルアップロード)

* Webを支える技術
* Real World HTTP

.. rst-class:: slide-green-white

参考
===============================

* ハイパフォーマンスWebサイト
* ハイパフォーマンスブラウザネットワーキング
* Nginx実践入門
* 超速! Webページ速度改善ガイド

Djangoのドキュメントも充実しています

* `静的ファイルのデプロイ <https://docs.djangoproject.com/ja/2.2/howto/static-files/deployment/>`_

.. rst-class:: slide-green-white

参考
===============================

* Webページ や 書籍 の著者の皆さん 本当に ありがとうございます。m(_ _)m

.. rst-class:: slide-green-white

まとめ
===============================

下記のような話をいたしました。

* 静的ファイルとは
* わたしの失敗談
* Djangoの静的ファイルの扱い
* 本番公開のパターン
* 静的ファイル関連Tips

静的ファイル配信というものについて何か参考になれば幸いです。


.. rst-class:: slide-green-white
.. rst-class:: center-text

ご静聴ありがとうございました
==================================

ご静聴ありがとうございました
