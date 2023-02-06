AnsibleでのNMSのデプロイ 
####

本手順ではいくつかの環境でNMS/NIMをご利用いただくにあたり、セットアップ手順を複数紹介します。
環境にあった手順を実施してください。

こちらはラボでの利用を目的とした参考手順となります。
その他手順は `NGINX Management Suite Guide <https://docs.nginx.com/nginx-management-suite/>`__ をご確認ください。


ラボ環境で動作を確認される場合、作業ホストは ``ubuntu-host1(10.1.1.5)`` となります

ラボ環境で動作を確認される場合、作業ホストは ``ubuntu-host1(10.1.1.5)`` となります

1. Ansible Playbook の実行
----

必要なファイルを取得します。

.. code-block:: cmdin

  cd ~/
  git clone https://github.com/BeF5/f5j-nms-docker-simple.git --branch v1.1.0
  
以下コマンドを実行し、Ansibleを利用してNMSをデプロイします。

Ansibleのセットアップ、NGINX Galaxy Moduleの取得の詳細は以下のページを参照してください
- `Ansible実行環境のセットアップ <https://f5j-nginx-ansible.readthedocs.io/en/latest/class1/module2/module2.html>`__

.. code-block:: cmdin

  cd ~/
  ansible-galaxy collection install nginxinc.nginx_core
  cd ~/f5j-nms-docker-simple/ansible
  cp ~/nginx-repo* .
  ansible-playbook -i inventory/hosts -l ubuntu-host1 web-servers/nms-setup.yaml

コマンドの実行が完了すると以下のような出力結果となります

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  PLAY RECAP ***************************************************************************************
  10.1.1.5                  : ok=91   changed=44   unreachable=0    failed=0    skipped=50   rescued=0    ignored=0
  

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
      
