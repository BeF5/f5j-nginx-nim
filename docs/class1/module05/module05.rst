AnsibleのNMSデプロイ 
####

本手順ではいくつかの環境でNMS/NIMをご利用いただくにあたり、セットアップ手順を複数紹介します。
環境にあった手順を実施してください。

こちらはラボでの利用を目的とした参考手順となります。
その他手順は `NGINX Management Suite Guide <https://docs.nginx.com/nginx-management-suite/>`__ をご確認ください。

ラボ環境で動作を確認される場合、作業ホストは ``ubuntu-host1(10.1.1.5)`` となります

1. Ansible Playbook の実行
----

必要なファイルを取得します。

.. code-block:: cmdin

  cd ~/
  git clone https://github.com/BeF5/f5j-nms-docker-simple.git
  
以下コマンドを実行し、Ansibleを利用してNMSをデプロイします。

Ansibleのセットアップなど参考情報は以下のページを参照してください

- `Ansible実行環境のセットアップ <https://f5j-nginx-ansible.readthedocs.io/en/latest/class1/module2/module2.html>`__

.. NOTE::

  予め実行ホストにNGINXリポジトリにアクセスするための証明書・鍵を保存してください
  ``ubuntu-host1(10.1.1.5)`` では以下コマンドによりファイルの取得が可能です

  .. code-block:: bash

     scp 10.1.1.8:nginx-repo* .

.. code-block:: cmdin

  cd ~/f5j-nms-docker-simple/ansible
  cp ~/nginx-repo* .
  ansible-playbook -i inventory/hosts -l host1 nms/nms-setup.yaml

Playbookを実行すると、以下のようにPromptが表示されます。
NMSにインストールするNAP WAF compilerのVersionを指定してください。

WAF compilerがSupportするNAP WAFのVersionは以下のページを参照してください。

- `WAF Compiler and Supported App Protect Versions <https://docs.nginx.com/nginx-management-suite/nim/how-to/app-protect/setup-waf-config-management/#waf-compiler-and-supported-app-protect-versions>`__

.. code-block:: bash
  :linenos:
  :caption: Playbook実行後の表示Version

  “Set NAP WAF Compile Ver. v4.100.1 (default) (NAP WAF 4.1), v4.2.0 (NAP WAF 4.0), v3.1088.2 (NAP WAF 3.12.2)”: <<Verを入力>>

  **省略**

  TASK [debug] **************************************************************************************************************
  ok: [10.1.1.5] => {
      "msg": "‘NAP WAF Compiler Version v4.100.1’"
  }

  **上記の通り入力したVersionが表示されます。空白の場合Default値が反映されます**


コマンドの実行が完了すると以下のような出力結果となります

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  PLAY RECAP ****************************************************************************************************
  10.1.1.5                   : ok=37   changed=27   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

インストールしたパッケージの情報を参考に示します

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  $ dpkg -l | grep nginx
  ii  libnginx-mod-http-image-filter     1.18.0-0ubuntu1.4                 amd64        HTTP image filter module for Nginx
  ii  libnginx-mod-http-xslt-filter      1.18.0-0ubuntu1.4                 amd64        XSLT Transformation module for Nginx
  ii  libnginx-mod-mail                  1.18.0-0ubuntu1.4                 amd64        Mail module for Nginx
  ii  libnginx-mod-stream                1.18.0-0ubuntu1.4                 amd64        Stream module for Nginx
  ii  nginx                              1.18.0-0ubuntu1.4                 all          small, powerful, scalable web/proxy server
  ii  nginx-common                       1.18.0-0ubuntu1.4                 all          small, powerful, scalable web/proxy server - common files
  ii  nginx-core                         1.18.0-0ubuntu1.4                 amd64        nginx web/proxy server (standard version)
  
  $ dpkg -l | grep nms
  ii  nms-api-connectivity-manager       1.4.1-762997411~focal             amd64        NGINX Management Suite ACM Module.
  ii  nms-instance-manager               2.8.0-759861272~focal             amd64        NGINX Management Suite - Instance Manager (core system)
  ii  nms-nap-compiler-v4.100.1          4.100.1-1~focal                   amd64        NGINX App Protect repackaged compiler for compatability with NGINX Instance Manager
  ii  nms-sm                             1.2.0-751410248~focal             amd64        NGINX Security Monitoring Dashboard Module


2. NMS への接続
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
      
