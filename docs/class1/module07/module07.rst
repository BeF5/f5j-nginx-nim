NGINX Instance Managerの利用
####

1. NGINX Scan 
====

Scan機能は、NIMから対象のネットワークに対し、NGINXやその他Webサーバを検知する機能です。
``この機能はInsntance Managerのライセンスをインストールしていないホスト`` でも実行することが可能です。

Dockerが動作するホストで以下コマンドを参考に古いVersionのNGINX OSS Container Imageを実行します。

作業ホストは ``ubuntu-host1(10.1.1.5)`` となります。

.. code-block:: cmdin

  sudo docker run --name nginx-1.9.5 -d -p 8080:80  nginx:1.9.5

NIMログインし、Modules欄に表示された ``Instance Manager`` をクリックします

   .. image:: ./media/nim-launchpad-nim.png
      :width: 400

左メニューの ``Scan`` をクリックします。


画面上部のメニューを以下の内容を入力し 右側に表示される ``Scan`` ボタンをクリックします

+-----------+------------+
|CIDR       |10.1.1.0/24 |
+-----------+------------+
|Port Ranges|80,8080,443 |
+-----------+------------+

   .. image:: ./media/nim-scan-start.png
      :width: 400

``Learn more about NGINX CVEs`` をクリックすると、 `nginx security advisories <https://nginx.org/en/security_advisories.html>`__ のページが開きます。こちらからすべてのCVEに関する詳細を確認いただけます。

   .. image:: ./media/nim-scan-cves.png
      :width: 400

``NGINX 1.9.5`` のインスタンスは、 ``19`` の ``CVE`` があることがわかります。
このインスタンスが表示されている行をクリックします。

   .. image:: ./media/nim-scan-instance-cve.png
      :width: 400

該当するCVEの情報が表示されることが確認できます


``Certificate`` の欄が ``1 (0以外)`` のインスタンスをクリックします

   .. image:: ./media/nim-scan-instance-cert.png
      :width: 400

証明書の情報が表示されます

2. NGINXのステータス画面
====

AgentをインストールしたNGINXは各種ステータスの閲覧が可能となります

一覧に表示される、AgentをインストールしたNGINXインスタンスをクリックしてください

- Details

   .. image:: ./media/nim-monitor.png
      :width: 400

- Metrics Summary

   .. image:: ./media/nim-monitor2.png
      :width: 400

- Metrics : Histrical Data & Graph

   .. image:: ./media/nim-monitor3.png
      :width: 400
