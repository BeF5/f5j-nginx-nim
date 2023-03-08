NGINX 設定管理の利用
####


1. NGINXの設定管理画面
====

AgentをインストールしたNGINXは設定をNIM経由で変更・管理することが可能となります

一覧に表示の対象インスタンス右側に表示される ``…`` をクリックし ``Edit Config`` を開く、
または、対象のNGINXインスタンスをクリックし画面右側の ``Edit Config`` をクリックし、
インスタンスのNGINX設定ファイルを開きます

- Config

   .. image:: ./media/nim-editconfig.png
      :width: 400

   .. image:: ./media/nim-setting.png
      :width: 400

ファイルの新規作成、削除、そして設定内容を修正し反映することが可能です

2. ショートカットキーの確認
====

コンフィグ管理画面ではショートカットキーが利用できます

検索: ``Ctrl-F``
----

適当なコンフィグファイルを開き、 ``Ctrl-F`` を入力してください。
画面右上に検索ウィンドウが表示されます。

   .. image:: ./media/nim-setting-search.png
      :width: 400


検索ウィンドウの左側 ``>`` をクリックすると ``置換`` の機能が利用できます。詳細は次の項目を参照してください

置換: ``Ctrl-H``
----

適当なコンフィグファイルを開き、 ``Ctrl-H`` を入力してください。
画面右上に文字列置換のウィンドウが表示されます。

   .. image:: ./media/nim-setting-replace.png
      :width: 400

3. 設定候補の出力
====

設定ファイル内にディレクティブを記述すると、入力内容に従って候補が出力されます。
プルダウンより利用されたい内容を選択してください

   .. image:: ./media/nim-setting-candidate.png
      :width: 400

4. ディレクティブの解説
====

設定ファイル内のディレクティブにマウスカーソルを合わせると、それらの簡単な解説が英語表記で表示されます。

以下は ``nginx.conf`` 内、 ``log_format`` 、 ``include`` にカーソルをあわせた場合の表示です

   .. image:: ./media/nim-setting-commandsyntax.png
      :width: 400

5. 設定記述内容のミス
====

設定ファイルに誤った内容を記述するとエラーが出力されます。
以下は ``nginx.conf`` 内、 http block の外に ``server block`` を記述した場合の表示です

   .. image:: ./media/nim-setting-error.png
      :width: 400

6. 設定を元に戻す
====

変更内容を破棄し、元の状態に戻す(Revert)が可能です。
先程誤った場所に ``server block`` を記述した状態で、画面右上の ``Revert`` をクリックします。その後設定が元の状態に戻ります


7. 新たな設定ファイルの作成
====

画面左上の ``+ Add File`` よりファイルを追加することが可能です。

``+ Add File`` をクリックします。

   .. image:: ./media/nim-setting-addfile.png
      :width: 400

ポップアップが表示されますので、 ``Full Pathで新規作成するファイル名`` を指定し、 ``Create`` をクリックします

   .. image:: ./media/nim-setting-addfile2.png
      :width: 400

以下のように新たにファイルが生成されます

   .. image:: ./media/nim-setting-addfile3.png
      :width: 400

8. 設定内容の反映
====

`新たな設定ファイルの作成 <>`__ で作成したファイルに対し、設定内容を記述し、NGINXに反映します。

以下の内容をコピー＆ペーストしてください

.. code-block:: bash
  :linenos:
  :caption: 設定内容サンプル

  server {
    listen 81;
    return 200 "nim test";
  }

画面右上の ``Save as...`` をクリックします

   .. image:: ./media/nim-setting-save.png
      :width: 400

ポップアップメニューが表示されます。Satged Config として保存する際の名称を入力します

   .. image:: ./media/nim-setting-save2.png
      :width: 400

正しく設定が保存できた旨、メッセージが表示されます。

   .. image:: ./media/nim-setting-save3.png
      :width: 400

画面右上の ``Publish`` をクリックします

   .. image:: ./media/nim-setting-publish.png
      :width: 400

ポップアップメニューが表示されます。 ``Publish`` をクリックします

   .. image:: ./media/nim-setting-publish2.png
      :width: 400

正しく設定が反映できた旨、メッセージが表示されます。

   .. image:: ./media/nim-setting-publish3.png
      :width: 400

Instanceにログインし、状態を確認します

正しく設定ファイルが生成されていることが確認できます

.. code-block:: bash
  :linenos:
  :caption: 生成されたファイルの情報

  $ ls -l /etc/nginx/conf.d
  total 8
  -rw-r--r-- 1 root root        1508 Mar  7 15:58 default.conf
  -rw-r--r-- 1 root nginx-agent   51 Mar  7 15:58 nim-test.conf

.. code-block:: bash
  :linenos:
  :caption: 生成されたファイルの内容

  $ cat /etc/nginx/conf.d/nim-test.conf
  server {
    listen 81;
    return 200 "nim test";
  }

.. code-block:: bash
  :linenos:
  :caption: 反映された内容の確認

  $ curl localhost:81
  nim test


9. 設定ファイルの削除
====

`設定内容の反映 <>`__ で作成したファイルを削除します

メニューから対象のファイルを選択します

画面右上の ``ゴミ箱のマーク`` をクリックします

   .. image:: ./media/nim-setting-delete.png
      :width: 400

ポップアップメニューが表示されます。 ``Delete`` をクリックします

   .. image:: ./media/nim-setting-delete.png
      :width: 400

設定の反映と同様に、画面右上の ``Publish`` をクリックし、状態を反映します。

Instanceにログインし、状態を確認します

正しく設定ファイルが削除されていることが確認できます

.. code-block:: bash
  :linenos:
  :caption: 生成されたファイルの情報

  $ ls -l /etc/nginx/conf.d
  total 8
  -rw-r--r-- 1 root root 1508 Mar  7 16:08 default.conf


.. code-block:: bash
  :linenos:
  :caption: 生成されたファイルの内容

  $ cat /etc/nginx/conf.d/nim-test.conf
  server {
    listen 81;
    return 200 "nim test";
  }

.. code-block:: bash
  :linenos:
  :caption: 反映された内容の確認

  $ curl localhost:81
  curl: (7) Failed to connect to localhost port 81: Connection refused

10. 保存したコンフィグを別のインスタンスへ反映
====

`設定内容の反映 <>`__ でStaged Configとして保存した内容を反映します

画面左メニューの ``Staged Configs`` をクリックします

   .. image:: ./media/nim-setting-stagedconfig.png
      :width: 400

先程保存したコンフィグが表示されています。該当のコンフィグをクリックします

   .. image:: ./media/nim-setting-stagedconfig2.png
      :width: 400

こちらには先程保存した内容が表示されています。

   .. image:: ./media/nim-setting-stagedconfig3.png
      :width: 400

画面右上の ``Publish to...`` をクリックします。

   .. image:: ./media/nim-setting-stagedconfig4.png
      :width: 400

Staged Configを作成したインスタンスと別のインスタンスを選択し、 ``Publish`` をクリックします

   .. image:: ./media/nim-setting-stagedconfig5.png
      :width: 400

設定が正しく反映されたことが確認できます

   .. image:: ./media/nim-setting-stagedconfig6.png
      :width: 400
