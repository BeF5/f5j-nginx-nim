Docker ContainerのNMSデプロイ
####

本手順ではいくつかの環境でNMS/NIMをご利用いただくにあたり、セットアップ手順を複数紹介します。
環境にあった手順を実施してください。

こちらはラボでの利用を目的とした参考手順となります。
その他手順は `NGINX Management Suite Guide <https://docs.nginx.com/nginx-management-suite/>`__ をご確認ください。

ラボ環境で動作を確認される場合、作業ホストは ``ubuntu-host1(10.1.1.5)`` となります

1. Docker ImageのBuild
----

必要なファイルを取得します。

.. code-block:: cmdin

  cd ~/
  git clone https://github.com/BeF5/f5j-nms-docker-simple.git


.. NOTE::

  予め実行ホストにNGINXリポジトリにアクセスするための証明書・鍵を保存してください
  ``ubuntu-host1(10.1.1.5)`` では以下コマンドによりファイルの取得が可能です

  .. code-block:: bash

     scp 10.1.1.8:nginx-repo* .

以下コマンドを実行し、Docker Imageを作成します。以下はNIMのみの構成です

.. code-block:: cmdin

  cd ~/f5j-nms-docker-simple/docker-compose
  cp ~/nginx-repo* .
  sudo sh ./scripts/buildNIM.sh -C nginx-repo.crt -K nginx-repo.key -i -t nim


NIMに加え、ACM(API Connectivity Manasger)、SM(Security Monitoring)、WAF Compilerをデプロイする場合は以下コマンドとなります。

.. code-block:: cmdin

  sudo sh ./scripts/buildNIM.sh -A -W -P v4.100.1 -C nginx-repo.crt -K nginx-repo.key -i -t nim-acm-sp-waf

2. Docker Composeによるコンテナの起動
----

docker-compose でコンテナを実行します  

.. code-block:: cmdin

  ## cd ~/f5j-nms-docker-simple/docker-compose
  sudo docker-compose -f docker-compose.yaml up -d


NIMに加え、ACM(API Connectivity Manasger)、SM(Security Monitoring)、WAF Compilerを実行する場合は、 ``docker-compose-acm-sp-waf.yaml`` を指定してください

.. code-block:: cmdin

  ## cd ~/f5j-nms-docker-simple/docker-compose
  docker compose -f docker-compose-acm-sp-waf.yaml up -d


NIMが正しく動作した場合のサンプルのステータスを示します  
動作するDockerイメージの状態。clickhouseと前の手順でBuildしたnimのイメージが動作します

.. code-block:: cmdin

  sudo docker ps

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  CONTAINER ID   IMAGE                                    COMMAND                  CREATED          STATUS          PORTS                                                                                            NAMES
  90479d97b943   clickhouse/clickhouse-server:21.12.4.1   "/entrypoint.sh"         17 minutes ago   Up 17 minutes   0.0.0.0:8123->8123/tcp, :::8123->8123/tcp, 0.0.0.0:9000->9000/tcp, :::9000->9000/tcp, 9009/tcp   f5j-nms-docker-simple_clickhouse_1
  cbe21f598fb7   nim:latest                               "/bin/sh -c /deploym…"   17 minutes ago   Up 17 minutes   0.0.0.0:443->443/tcp, :::443->443/tcp, 0.0.0.0:5000->5000/tcp, :::5000->5000/tcp                 f5j-nms-docker-simple_nginx-nim2_1

正しく動作した場合、NIMのコンテナで以下のログが確認できます

.. code-block:: cmdin

  sudo docker logs $(sudo docker ps -a -f name=nim -q)

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Waiting for ClickHouse...
  Waiting for ClickHouse...
  Using openssl version 3.0.2 to update NGINX password for admin in file: /etc/nms/nginx/.htpasswd
   * Starting nginx nginx
     ...done.
  [91] [INF] Starting nats-server
  [91] [INF]   Version:  2.9.1
  [91] [INF]   Git:      [not set]
  [91] [INF]   Name:     NAKHTJIAR5EXUKXQO4ASOM427BVOPXY34UR2FE5L2O5255IA55NQTT4J
  [91] [INF]   ID:       NAKHTJIAR5EXUKXQO4ASOM427BVOPXY34UR2FE5L2O5255IA55NQTT4J
  [91] [INF] Listening for client connections on 127.0.0.1:4222
  [91] [INF] Server is ready
  [80] [INF] Starting nats-server
  [80] [INF]   Version:  2.9.1
  [80] [INF]   Git:      [not set]
  [80] [INF]   Name:     NDRB6PWV4DYBD4AUAKYRZJX4JWT6YX4SCZAR5VP44ONPIFFCISGRLEE4
  [80] [INF]   Node:     5e1qS4jS
  [80] [INF]   ID:       NDRB6PWV4DYBD4AUAKYRZJX4JWT6YX4SCZAR5VP44ONPIFFCISGRLEE4
  [80] [INF] Starting JetStream
  [80] [INF]     _ ___ _____ ___ _____ ___ ___   _   __  __
  [80] [INF]  _ | | __|_   _/ __|_   _| _ \ __| /_\ |  \/  |
  [80] [INF] | || | _|  | | \__ \ | | |   / _| / _ \| |\/| |
  [80] [INF]  \__/|___| |_| |___/ |_| |_|_\___/_/ \_\_|  |_|
  [80] [INF]
  [80] [INF]          https://docs.nats.io/jetstream
  [80] [INF]
  [80] [INF] ---------------- JETSTREAM ----------------
  [80] [INF]   Max Memory:      1.00 GB
  [80] [INF]   Max Storage:     10.00 GB
  [80] [INF]   Store Directory: "/var/lib/nms/streaming/jetstream"
  [80] [INF] -------------------------------------------
  [80] [INF] Listening for client connections on 127.0.0.1:9100
  [80] [INF] Server is ready
  ⇨ http server started on /var/run/nms/core.sock
  ⇨ http server started on /var/run/nms/dpm.sock

3. NMS への接続
----

対象となるホストのIPアドレスを確認し、 踏み台ホストにてChromeを開き、 ``https://<ホストのIPアドレス>/ui`` に接続してください。
ログイン情報は ``docker-compose.yaml`` の環境変数として指定している以下文字列となります。

+--------+---------------+
|username|admin          |
+--------+---------------+
|password|nimadmin       |
+--------+---------------+

以下の様にTop画面が表示されます

   .. image:: ../module02/media/nim-login.png
      :width: 400

``Sign In`` をクリックすると Basic認証によるポップアップが表示されます。Username ``admin`` 、 Password は ``Install時の出力で予め確認した文字列`` を入力してください
ログインが完了すると以下のような画面が表示されます

   .. image:: ../module02/media/nim-top.png
      :width: 400
      
