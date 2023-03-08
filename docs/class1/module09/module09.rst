NIM API操作
####


1. API操作画面
====

NIMは各種機能をAPIで操作することが可能です。

NIMの管理画面より、以下の手順でAPIドキュメントを参照することが可能です


- プルダウン(Select module) をクリックし、 ``API Documentation`` をクリック

   .. image:: ./media/nim-api-documentation.png
      :width: 400

- トップメニューを開き、 ``API Documentation`` をクリック

   .. image:: ./media/nim-api-documentation2.png
      :width: 400

2. GUIの操作・API動作確認
====

こちらのAPI操作画面では、実際にAPIの実行及び結果を確認することが出来ます。

以降、いくつか主要な項目の操作について解説します。

1. インスタンスの情報
----

インスタンス情報を取得する操作を確認します

``NGINX Instances`` の ``/instances`` を開きます。該当項目の画面右側 ``Try it out`` をクリックします

   .. image:: ./media/nim-api-instances.png
      :width: 400

API接続に必要なオプションパラメータが指定可能となります。 ``napOnly`` に ``false`` を指定し、下側 ``Execute`` をクリックします

   .. image:: ./media/nim-api-instances2.png
      :width: 400

API接続を行った結果が表示されます。curlコマンドは実際にリクエストを行う場合のコマンドを指します。実行結果がその下に表示されます

   .. image:: ./media/nim-api-instances3.png
      :width: 400

- ``count`` : NIMの管理対象として登録されているインスタンスの数が ``2`` であることがわかります
- ``items`` : 管理対象のNGINXインスタンスの情報が確認できます
- ``nginxAppProtectWAFCount`` : NGINX App Protect WAFがインストールされたインスタンスの数を示します
- ``nginxPlusCount`` : NGINX Plusインスタンスの数を示します


2. 設定情報の取得 
----

saved config の情報を取得します。saved config はNIM上で設定を変更しその後に保存を行った ``staged configs`` を指します。 

``Configurations`` の ``/configs`` を開きます。該当項目の画面右側 ``Try it out`` をクリックします

   .. image:: ./media/nim-api-configs.png
      :width: 400

下側 ``Execute`` をクリックします

   .. image:: ./media/nim-api-configs2.png
      :width: 400

API接続を行った結果が表示されます。curlコマンドは実際にリクエストを行う場合のコマンドを指します。実行結果がその下に表示されます

   .. image:: ./media/nim-api-configs3.png
      :width: 400

- ``items`` : 保存されているコンフィグの情報が出力されています


3. CLIの操作
====

1. インスタンスの情報
----

`GUI インスタンスの情報 <#id1>`__ で実施した内容をCLIで実行します

curl コマンドを実行します

.. code-block:: cmdin

  curl -sk -u "admin:nimadmin" 'https://10.1.1.5/api/platform/v1/instances?napOnly=false' |  jq .

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  {
    "count": 2,
    "items": [
      {
        "displayName": "ip-10-1-1-6",
        "systemUid": "c233d837-fdfc-3ca5-be2b-23a39c9b4873",
        "uid": "79453f64-4453-5eea-a9f6-a3c369e295d2"
      },
      {
        "displayName": "ip-10-1-1-7",
        "systemUid": "74d52621-eb60-35ea-84f2-8f754974d560",
        "uid": "754ddc18-898a-59ee-b5a3-fe22cf6b3da6"
      }
    ],
    "nginxAppProtectWAFCount": 0,
    "nginxPlusCount": 2
  }

特定のインスタンスを対象とした処理を後ほど行います。 ``ip-10-1-1-7`` の値が以下であることを確認してください

+-----------+---------------------------------------+
|displayName| ip-10-1-1-7                           |
+-----------+---------------------------------------+
|systemUid  | 74d52621-eb60-35ea-84f2-8f754974d560  |
+-----------+---------------------------------------+
|uid        | 754ddc18-898a-59ee-b5a3-fe22cf6b3da6  |
+-----------+---------------------------------------+


2. 設定情報の取得 
----

`GUI 設定情報の取得 <#id2>`__ で実施した内容をCLIで実行します

curl コマンドを実行します

.. code-block:: cmdin

  curl -sk -u "admin:nimadmin" https://10.1.1.5/api/platform/v1/configs | jq .

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  {
    "items": [
      {
        "configName": "ip-10-1-1-6-nim-test",
        "createTime": "2023-03-08T08:46:28.826Z",
        "nginxUid": "79453f64-4453-5eea-a9f6-a3c369e295d2",
        "uid": "d679d216-799e-49d3-8139-c9b8a7bb2512",
        "updateTime": "2023-03-08T08:46:28.826Z"
      }
    ]
  }

こちらの設定情報を利用し、処理を後ほど行います。 ``ip-10-1-1-6-nim-test`` の値が以下であることを確認してください

+-----------+---------------------------------------+
|configName | ip-10-1-1-6-nim-test                  |
+-----------+---------------------------------------+
|nginxUid   | 79453f64-4453-5eea-a9f6-a3c369e295d2  |
+-----------+---------------------------------------+
|uid        | d679d216-799e-49d3-8139-c9b8a7bb2512  |
+-----------+---------------------------------------+


3. 特定saved configの詳細 
----

`CLI 設定情報の取得 <#id4>`__ で確認した情報の詳細を確認します

API Documentationを確認すると、情報の取得を行うURLは ``/configs/{configUid}`` であることがわかります

   .. image:: ./media/nim-api-config-detail.png
      :width: 400

GUIで ``Try it out`` から ``configUid`` を指定し動作を確認いただくことも可能です。
こちらではCLIで実行する方法を以下の通り示します

``configUid`` に入力する情報は、 `CLI 設定情報の取得 <#id4>`__ の ``uid`` となります。
こちらの情報を指定したcurlコマンドが以下となります。こちらのコマンドを実行します


.. code-block:: cmdin

  curl -sk -u "admin:nimadmin" https://10.1.1.5/api/platform/v1/configs/d679d216-799e-49d3-8139-c9b8a7bb2512 | jq .

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  {
    "configName": "ip-10-1-1-6-nim-test",
    "createTime": "2023-03-08T08:46:28.826Z",
    "nginxUid": "79453f64-4453-5eea-a9f6-a3c369e295d2",
    "uid": "d679d216-799e-49d3-8139-c9b8a7bb2512",
    "updateTime": "2023-03-08T08:46:28.826Z",
    "auxFiles": {
      "files": [],
      "rootDir": "/"
    },
    "configFiles": {
      "files": [
        {
          "contents": "CnVzZXIgIG5naW54Owp3b3JrZXJfcHJvY2Vzc2VzICBhdXRvOwoKZXJyb3JfbG9nICAvdmFyL2xvZy9uZ2lueC9lcnJvci5sb2cgbm90aWNlOwpwaWQgICAgICAgIC92YXIvcnVuL25naW54LnBpZDsKCgpldmVudHMgewogICAgd29ya2VyX2Nvbm5lY3Rpb25zICAxMDI0Owp9CgoKaHR0cCB7CiAgICBpbmNsdWRlICAgICAgIC9ldGMvbmdpbngvbWltZS50eXBlczsKICAgIGRlZmF1bHRfdHlwZSAgYXBwbGljYXRpb24vb2N0ZXQtc3RyZWFtOwoKICAgIGxvZ19mb3JtYXQgIG1haW4gICckcmVtb3RlX2FkZHIgLSAkcmVtb3RlX3VzZXIgWyR0aW1lX2xvY2FsXSAiJHJlcXVlc3QiICcKICAgICAgICAgICAgICAgICAgICAgICckc3RhdHVzICRib2R5X2J5dGVzX3NlbnQgIiRodHRwX3JlZmVyZXIiICcKICAgICAgICAgICAgICAgICAgICAgICciJGh0dHBfdXNlcl9hZ2VudCIgIiRodHRwX3hfZm9yd2FyZGVkX2ZvciInOwoKICAgIGFjY2Vzc19sb2cgIC92YXIvbG9nL25naW54L2FjY2Vzcy5sb2cgIG1haW47CgogICAgc2VuZGZpbGUgICAgICAgIG9uOwogICAgI3RjcF9ub3B1c2ggICAgIG9uOwoKICAgIGtlZXBhbGl2ZV90aW1lb3V0ICA2NTsKCiAgICAjZ3ppcCAgb247CgogICAgaW5jbHVkZSAvZXRjL25naW54L2NvbmYuZC8qLmNvbmY7Cn0KCgojIFRDUC9VRFAgcHJveHkgYW5kIGxvYWQgYmFsYW5jaW5nIGJsb2NrCiMKI3N0cmVhbSB7CiAgICAjIEV4YW1wbGUgY29uZmlndXJhdGlvbiBmb3IgVENQIGxvYWQgYmFsYW5jaW5nCgogICAgI3Vwc3RyZWFtIHN0cmVhbV9iYWNrZW5kIHsKICAgICMgICAgem9uZSB0Y3Bfc2VydmVycyA2NGs7CiAgICAjICAgIHNlcnZlciBiYWNrZW5kMS5leGFtcGxlLmNvbToxMjM0NTsKICAgICMgICAgc2VydmVyIGJhY2tlbmQyLmV4YW1wbGUuY29tOjEyMzQ1OwogICAgI30KCiAgICAjc2VydmVyIHsKICAgICMgICAgbGlzdGVuIDEyMzQ1OwogICAgIyAgICBzdGF0dXNfem9uZSB0Y3Bfc2VydmVyOwogICAgIyAgICBwcm94eV9wYXNzIHN0cmVhbV9iYWNrZW5kOwogICAgI30KI30K",
          "name": "/etc/nginx/nginx.conf"
        },
        {
          "contents": "c2VydmVyIHsKICAgIGxpc3RlbiAgICAgICA4MCBkZWZhdWx0X3NlcnZlcjsKICAgIHNlcnZlcl9uYW1lICBsb2NhbGhvc3Q7CgogICAgI2FjY2Vzc19sb2cgIC92YXIvbG9nL25naW54L2hvc3QuYWNjZXNzLmxvZyAgbWFpbjsKCiAgICBsb2NhdGlvbiAvIHsKICAgICAgICByb290ICAgL3Vzci9zaGFyZS9uZ2lueC9odG1sOwogICAgICAgIGluZGV4ICBpbmRleC5odG1sIGluZGV4Lmh0bTsKICAgIH0KCiAgICAjZXJyb3JfcGFnZSAgNDA0ICAgICAgICAgICAgICAvNDA0Lmh0bWw7CgogICAgIyByZWRpcmVjdCBzZXJ2ZXIgZXJyb3IgcGFnZXMgdG8gdGhlIHN0YXRpYyBwYWdlIC81MHguaHRtbAogICAgIwogICAgZXJyb3JfcGFnZSAgIDUwMCA1MDIgNTAzIDUwNCAgLzUweC5odG1sOwogICAgbG9jYXRpb24gPSAvNTB4Lmh0bWwgewogICAgICAgIHJvb3QgICAvdXNyL3NoYXJlL25naW54L2h0bWw7CiAgICB9CgogICAgIyBwcm94eSB0aGUgUEhQIHNjcmlwdHMgdG8gQXBhY2hlIGxpc3RlbmluZyBvbiAxMjcuMC4wLjE6ODAKICAgICMKICAgICNsb2NhdGlvbiB+IFwucGhwJCB7CiAgICAjICAgIHByb3h5X3Bhc3MgICBodHRwOi8vMTI3LjAuMC4xOwogICAgI30KCiAgICAjIHBhc3MgdGhlIFBIUCBzY3JpcHRzIHRvIEZhc3RDR0kgc2VydmVyIGxpc3RlbmluZyBvbiAxMjcuMC4wLjE6OTAwMAogICAgIwogICAgI2xvY2F0aW9uIH4gXC5waHAkIHsKICAgICMgICAgcm9vdCAgICAgICAgICAgaHRtbDsKICAgICMgICAgZmFzdGNnaV9wYXNzICAgMTI3LjAuMC4xOjkwMDA7CiAgICAjICAgIGZhc3RjZ2lfaW5kZXggIGluZGV4LnBocDsKICAgICMgICAgZmFzdGNnaV9wYXJhbSAgU0NSSVBUX0ZJTEVOQU1FICAvc2NyaXB0cyRmYXN0Y2dpX3NjcmlwdF9uYW1lOwogICAgIyAgICBpbmNsdWRlICAgICAgICBmYXN0Y2dpX3BhcmFtczsKICAgICN9CgogICAgIyBkZW55IGFjY2VzcyB0byAuaHRhY2Nlc3MgZmlsZXMsIGlmIEFwYWNoZSdzIGRvY3VtZW50IHJvb3QKICAgICMgY29uY3VycyB3aXRoIG5naW54J3Mgb25lCiAgICAjCiAgICAjbG9jYXRpb24gfiAvXC5odCB7CiAgICAjICAgIGRlbnkgIGFsbDsKICAgICN9CgogICAgIyBlbmFibGUgL2FwaS8gbG9jYXRpb24gd2l0aCBhcHByb3ByaWF0ZSBhY2Nlc3MgY29udHJvbCBpbiBvcmRlcgogICAgIyB0byBtYWtlIHVzZSBvZiBOR0lOWCBQbHVzIEFQSQogICAgIwogICAgI2xvY2F0aW9uIC9hcGkvIHsKICAgICMgICAgYXBpIHdyaXRlPW9uOwogICAgIyAgICBhbGxvdyAxMjcuMC4wLjE7CiAgICAjICAgIGRlbnkgYWxsOwogICAgI30KCiAgICAjIGVuYWJsZSBOR0lOWCBQbHVzIERhc2hib2FyZDsgcmVxdWlyZXMgL2FwaS8gbG9jYXRpb24gdG8gYmUKICAgICMgZW5hYmxlZCBhbmQgYXBwcm9wcmlhdGUgYWNjZXNzIGNvbnRyb2wgZm9yIHJlbW90ZSBhY2Nlc3MKICAgICMKICAgICNsb2NhdGlvbiA9IC9kYXNoYm9hcmQuaHRtbCB7CiAgICAjICAgIHJvb3QgL3Vzci9zaGFyZS9uZ2lueC9odG1sOwogICAgI30KfQo=",
          "name": "/etc/nginx/conf.d/default.conf"
        },
        {
          "contents": "c2VydmVyIHsNCiAgbGlzdGVuIDgxOw0KICByZXR1cm4gMjAwICJ0ZXN0IjsNCn0=",
          "name": "/etc/nginx/conf.d/nim-test.conf"
        },
        {
          "contents": "CnR5cGVzIHsKICAgIHRleHQvaHRtbCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBodG1sIGh0bSBzaHRtbDsKICAgIHRleHQvY3NzICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBjc3M7CiAgICB0ZXh0L3htbCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgeG1sOwogICAgaW1hZ2UvZ2lmICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGdpZjsKICAgIGltYWdlL2pwZWcgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBqcGVnIGpwZzsKICAgIGFwcGxpY2F0aW9uL2phdmFzY3JpcHQgICAgICAgICAgICAgICAgICAgICAgICAgICBqczsKICAgIGFwcGxpY2F0aW9uL2F0b20reG1sICAgICAgICAgICAgICAgICAgICAgICAgICAgICBhdG9tOwogICAgYXBwbGljYXRpb24vcnNzK3htbCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHJzczsKCiAgICB0ZXh0L21hdGhtbCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbW1sOwogICAgdGV4dC9wbGFpbiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHR4dDsKICAgIHRleHQvdm5kLnN1bi5qMm1lLmFwcC1kZXNjcmlwdG9yICAgICAgICAgICAgICAgICBqYWQ7CiAgICB0ZXh0L3ZuZC53YXAud21sICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgd21sOwogICAgdGV4dC94LWNvbXBvbmVudCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGh0YzsKCiAgICBpbWFnZS9hdmlmICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgYXZpZjsKICAgIGltYWdlL3BuZyAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBwbmc7CiAgICBpbWFnZS9zdmcreG1sICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgc3ZnIHN2Z3o7CiAgICBpbWFnZS90aWZmICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgdGlmIHRpZmY7CiAgICBpbWFnZS92bmQud2FwLndibXAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgd2JtcDsKICAgIGltYWdlL3dlYnAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB3ZWJwOwogICAgaW1hZ2UveC1pY29uICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGljbzsKICAgIGltYWdlL3gtam5nICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBqbmc7CiAgICBpbWFnZS94LW1zLWJtcCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgYm1wOwoKICAgIGZvbnQvd29mZiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB3b2ZmOwogICAgZm9udC93b2ZmMiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdvZmYyOwoKICAgIGFwcGxpY2F0aW9uL2phdmEtYXJjaGl2ZSAgICAgICAgICAgICAgICAgICAgICAgICBqYXIgd2FyIGVhcjsKICAgIGFwcGxpY2F0aW9uL2pzb24gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBqc29uOwogICAgYXBwbGljYXRpb24vbWFjLWJpbmhleDQwICAgICAgICAgICAgICAgICAgICAgICAgIGhxeDsKICAgIGFwcGxpY2F0aW9uL21zd29yZCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBkb2M7CiAgICBhcHBsaWNhdGlvbi9wZGYgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgcGRmOwogICAgYXBwbGljYXRpb24vcG9zdHNjcmlwdCAgICAgICAgICAgICAgICAgICAgICAgICAgIHBzIGVwcyBhaTsKICAgIGFwcGxpY2F0aW9uL3J0ZiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBydGY7CiAgICBhcHBsaWNhdGlvbi92bmQuYXBwbGUubXBlZ3VybCAgICAgICAgICAgICAgICAgICAgbTN1ODsKICAgIGFwcGxpY2F0aW9uL3ZuZC5nb29nbGUtZWFydGgua21sK3htbCAgICAgICAgICAgICBrbWw7CiAgICBhcHBsaWNhdGlvbi92bmQuZ29vZ2xlLWVhcnRoLmtteiAgICAgICAgICAgICAgICAga216OwogICAgYXBwbGljYXRpb24vdm5kLm1zLWV4Y2VsICAgICAgICAgICAgICAgICAgICAgICAgIHhsczsKICAgIGFwcGxpY2F0aW9uL3ZuZC5tcy1mb250b2JqZWN0ICAgICAgICAgICAgICAgICAgICBlb3Q7CiAgICBhcHBsaWNhdGlvbi92bmQubXMtcG93ZXJwb2ludCAgICAgICAgICAgICAgICAgICAgcHB0OwogICAgYXBwbGljYXRpb24vdm5kLm9hc2lzLm9wZW5kb2N1bWVudC5ncmFwaGljcyAgICAgIG9kZzsKICAgIGFwcGxpY2F0aW9uL3ZuZC5vYXNpcy5vcGVuZG9jdW1lbnQucHJlc2VudGF0aW9uICBvZHA7CiAgICBhcHBsaWNhdGlvbi92bmQub2FzaXMub3BlbmRvY3VtZW50LnNwcmVhZHNoZWV0ICAgb2RzOwogICAgYXBwbGljYXRpb24vdm5kLm9hc2lzLm9wZW5kb2N1bWVudC50ZXh0ICAgICAgICAgIG9kdDsKICAgIGFwcGxpY2F0aW9uL3ZuZC5vcGVueG1sZm9ybWF0cy1vZmZpY2Vkb2N1bWVudC5wcmVzZW50YXRpb25tbC5wcmVzZW50YXRpb24KICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBwcHR4OwogICAgYXBwbGljYXRpb24vdm5kLm9wZW54bWxmb3JtYXRzLW9mZmljZWRvY3VtZW50LnNwcmVhZHNoZWV0bWwuc2hlZXQKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB4bHN4OwogICAgYXBwbGljYXRpb24vdm5kLm9wZW54bWxmb3JtYXRzLW9mZmljZWRvY3VtZW50LndvcmRwcm9jZXNzaW5nbWwuZG9jdW1lbnQKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBkb2N4OwogICAgYXBwbGljYXRpb24vdm5kLndhcC53bWxjICAgICAgICAgICAgICAgICAgICAgICAgIHdtbGM7CiAgICBhcHBsaWNhdGlvbi93YXNtICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgd2FzbTsKICAgIGFwcGxpY2F0aW9uL3gtN3otY29tcHJlc3NlZCAgICAgICAgICAgICAgICAgICAgICA3ejsKICAgIGFwcGxpY2F0aW9uL3gtY29jb2EgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBjY287CiAgICBhcHBsaWNhdGlvbi94LWphdmEtYXJjaGl2ZS1kaWZmICAgICAgICAgICAgICAgICAgamFyZGlmZjsKICAgIGFwcGxpY2F0aW9uL3gtamF2YS1qbmxwLWZpbGUgICAgICAgICAgICAgICAgICAgICBqbmxwOwogICAgYXBwbGljYXRpb24veC1tYWtlc2VsZiAgICAgICAgICAgICAgICAgICAgICAgICAgIHJ1bjsKICAgIGFwcGxpY2F0aW9uL3gtcGVybCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBwbCBwbTsKICAgIGFwcGxpY2F0aW9uL3gtcGlsb3QgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBwcmMgcGRiOwogICAgYXBwbGljYXRpb24veC1yYXItY29tcHJlc3NlZCAgICAgICAgICAgICAgICAgICAgIHJhcjsKICAgIGFwcGxpY2F0aW9uL3gtcmVkaGF0LXBhY2thZ2UtbWFuYWdlciAgICAgICAgICAgICBycG07CiAgICBhcHBsaWNhdGlvbi94LXNlYSAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgc2VhOwogICAgYXBwbGljYXRpb24veC1zaG9ja3dhdmUtZmxhc2ggICAgICAgICAgICAgICAgICAgIHN3ZjsKICAgIGFwcGxpY2F0aW9uL3gtc3R1ZmZpdCAgICAgICAgICAgICAgICAgICAgICAgICAgICBzaXQ7CiAgICBhcHBsaWNhdGlvbi94LXRjbCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgdGNsIHRrOwogICAgYXBwbGljYXRpb24veC14NTA5LWNhLWNlcnQgICAgICAgICAgICAgICAgICAgICAgIGRlciBwZW0gY3J0OwogICAgYXBwbGljYXRpb24veC14cGluc3RhbGwgICAgICAgICAgICAgICAgICAgICAgICAgIHhwaTsKICAgIGFwcGxpY2F0aW9uL3hodG1sK3htbCAgICAgICAgICAgICAgICAgICAgICAgICAgICB4aHRtbDsKICAgIGFwcGxpY2F0aW9uL3hzcGYreG1sICAgICAgICAgICAgICAgICAgICAgICAgICAgICB4c3BmOwogICAgYXBwbGljYXRpb24vemlwICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHppcDsKCiAgICBhcHBsaWNhdGlvbi9vY3RldC1zdHJlYW0gICAgICAgICAgICAgICAgICAgICAgICAgYmluIGV4ZSBkbGw7CiAgICBhcHBsaWNhdGlvbi9vY3RldC1zdHJlYW0gICAgICAgICAgICAgICAgICAgICAgICAgZGViOwogICAgYXBwbGljYXRpb24vb2N0ZXQtc3RyZWFtICAgICAgICAgICAgICAgICAgICAgICAgIGRtZzsKICAgIGFwcGxpY2F0aW9uL29jdGV0LXN0cmVhbSAgICAgICAgICAgICAgICAgICAgICAgICBpc28gaW1nOwogICAgYXBwbGljYXRpb24vb2N0ZXQtc3RyZWFtICAgICAgICAgICAgICAgICAgICAgICAgIG1zaSBtc3AgbXNtOwoKICAgIGF1ZGlvL21pZGkgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtaWQgbWlkaSBrYXI7CiAgICBhdWRpby9tcGVnICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbXAzOwogICAgYXVkaW8vb2dnICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG9nZzsKICAgIGF1ZGlvL3gtbTRhICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtNGE7CiAgICBhdWRpby94LXJlYWxhdWRpbyAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgcmE7CgogICAgdmlkZW8vM2dwcCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIDNncHAgM2dwOwogICAgdmlkZW8vbXAydCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHRzOwogICAgdmlkZW8vbXA0ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1wNDsKICAgIHZpZGVvL21wZWcgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtcGVnIG1wZzsKICAgIHZpZGVvL3F1aWNrdGltZSAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtb3Y7CiAgICB2aWRlby93ZWJtICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgd2VibTsKICAgIHZpZGVvL3gtZmx2ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBmbHY7CiAgICB2aWRlby94LW00diAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbTR2OwogICAgdmlkZW8veC1tbmcgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1uZzsKICAgIHZpZGVvL3gtbXMtYXNmICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBhc3ggYXNmOwogICAgdmlkZW8veC1tcy13bXYgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdtdjsKICAgIHZpZGVvL3gtbXN2aWRlbyAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBhdmk7Cn0K",
          "name": "/etc/nginx/mime.types"
        }
      ],
      "rootDir": "/etc/nginx"
    }
  }

- ``configFiles`` : 各種設定ファイルの情報が格納されています
- ``files`` : 設定ファイルの情報が格納されています。 ``contents`` が設定情報を BASE64 Encodeしたもの、 ``name`` がコンフィグファイルのFull Pathとなります

この結果より、3つの設定ファイルがこの staged config に含まれていることがわかります

4. 設定の反映 
----

`CLI インスタンスの情報 <#id3>`__ 、 `CLI 特定saved configの詳細  <#saved-config>`__ で確認した情報を元に設定を反映します

API Documentationを確認すると、設定を反映するURLは ``/systems/{systemUid}/instances/{nginxUid}/config`` であることがわかります
また、設定の反映は ``POST`` Method で、JSON形式で設定情報の送付が必要となります。

   .. image:: ./media/nim-api-config-apply.png
      :width: 400

送付するファイルの内容を確認します

.. code-block:: cmdin

  cat file1.txt

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  {
    "auxFiles": {
      "files": [],
      "rootDir": "/"
    },
    "configFiles": {
      "files": [
        {
          "contents": "CnVzZXIgIG5naW54Owp3b3JrZXJfcHJvY2Vzc2VzICBhdXRvOwoKZXJyb3JfbG9nICAvdmFyL2xvZy9uZ2lueC9lcnJvci5sb2cgbm90aWNlOwpwaWQgICAgICAgIC92YXIvcnVuL25naW54LnBpZDsKCgpldmVudHMgewogICAgd29ya2VyX2Nvbm5lY3Rpb25zICAxMDI0Owp9CgoKaHR0cCB7CiAgICBpbmNsdWRlICAgICAgIC9ldGMvbmdpbngvbWltZS50eXBlczsKICAgIGRlZmF1bHRfdHlwZSAgYXBwbGljYXRpb24vb2N0ZXQtc3RyZWFtOwoKICAgIGxvZ19mb3JtYXQgIG1haW4gICckcmVtb3RlX2FkZHIgLSAkcmVtb3RlX3VzZXIgWyR0aW1lX2xvY2FsXSAiJHJlcXVlc3QiICcKICAgICAgICAgICAgICAgICAgICAgICckc3RhdHVzICRib2R5X2J5dGVzX3NlbnQgIiRodHRwX3JlZmVyZXIiICcKICAgICAgICAgICAgICAgICAgICAgICciJGh0dHBfdXNlcl9hZ2VudCIgIiRodHRwX3hfZm9yd2FyZGVkX2ZvciInOwoKICAgIGFjY2Vzc19sb2cgIC92YXIvbG9nL25naW54L2FjY2Vzcy5sb2cgIG1haW47CgogICAgc2VuZGZpbGUgICAgICAgIG9uOwogICAgI3RjcF9ub3B1c2ggICAgIG9uOwoKICAgIGtlZXBhbGl2ZV90aW1lb3V0ICA2NTsKCiAgICAjZ3ppcCAgb247CgogICAgaW5jbHVkZSAvZXRjL25naW54L2NvbmYuZC8qLmNvbmY7Cn0KCgojIFRDUC9VRFAgcHJveHkgYW5kIGxvYWQgYmFsYW5jaW5nIGJsb2NrCiMKI3N0cmVhbSB7CiAgICAjIEV4YW1wbGUgY29uZmlndXJhdGlvbiBmb3IgVENQIGxvYWQgYmFsYW5jaW5nCgogICAgI3Vwc3RyZWFtIHN0cmVhbV9iYWNrZW5kIHsKICAgICMgICAgem9uZSB0Y3Bfc2VydmVycyA2NGs7CiAgICAjICAgIHNlcnZlciBiYWNrZW5kMS5leGFtcGxlLmNvbToxMjM0NTsKICAgICMgICAgc2VydmVyIGJhY2tlbmQyLmV4YW1wbGUuY29tOjEyMzQ1OwogICAgI30KCiAgICAjc2VydmVyIHsKICAgICMgICAgbGlzdGVuIDEyMzQ1OwogICAgIyAgICBzdGF0dXNfem9uZSB0Y3Bfc2VydmVyOwogICAgIyAgICBwcm94eV9wYXNzIHN0cmVhbV9iYWNrZW5kOwogICAgI30KI30K",
          "name": "/etc/nginx/nginx.conf"
        },
        {
          "contents": "c2VydmVyIHsKICAgIGxpc3RlbiAgICAgICA4MCBkZWZhdWx0X3NlcnZlcjsKICAgIHNlcnZlcl9uYW1lICBsb2NhbGhvc3Q7CgogICAgI2FjY2Vzc19sb2cgIC92YXIvbG9nL25naW54L2hvc3QuYWNjZXNzLmxvZyAgbWFpbjsKCiAgICBsb2NhdGlvbiAvIHsKICAgICAgICByb290ICAgL3Vzci9zaGFyZS9uZ2lueC9odG1sOwogICAgICAgIGluZGV4ICBpbmRleC5odG1sIGluZGV4Lmh0bTsKICAgIH0KCiAgICAjZXJyb3JfcGFnZSAgNDA0ICAgICAgICAgICAgICAvNDA0Lmh0bWw7CgogICAgIyByZWRpcmVjdCBzZXJ2ZXIgZXJyb3IgcGFnZXMgdG8gdGhlIHN0YXRpYyBwYWdlIC81MHguaHRtbAogICAgIwogICAgZXJyb3JfcGFnZSAgIDUwMCA1MDIgNTAzIDUwNCAgLzUweC5odG1sOwogICAgbG9jYXRpb24gPSAvNTB4Lmh0bWwgewogICAgICAgIHJvb3QgICAvdXNyL3NoYXJlL25naW54L2h0bWw7CiAgICB9CgogICAgIyBwcm94eSB0aGUgUEhQIHNjcmlwdHMgdG8gQXBhY2hlIGxpc3RlbmluZyBvbiAxMjcuMC4wLjE6ODAKICAgICMKICAgICNsb2NhdGlvbiB+IFwucGhwJCB7CiAgICAjICAgIHByb3h5X3Bhc3MgICBodHRwOi8vMTI3LjAuMC4xOwogICAgI30KCiAgICAjIHBhc3MgdGhlIFBIUCBzY3JpcHRzIHRvIEZhc3RDR0kgc2VydmVyIGxpc3RlbmluZyBvbiAxMjcuMC4wLjE6OTAwMAogICAgIwogICAgI2xvY2F0aW9uIH4gXC5waHAkIHsKICAgICMgICAgcm9vdCAgICAgICAgICAgaHRtbDsKICAgICMgICAgZmFzdGNnaV9wYXNzICAgMTI3LjAuMC4xOjkwMDA7CiAgICAjICAgIGZhc3RjZ2lfaW5kZXggIGluZGV4LnBocDsKICAgICMgICAgZmFzdGNnaV9wYXJhbSAgU0NSSVBUX0ZJTEVOQU1FICAvc2NyaXB0cyRmYXN0Y2dpX3NjcmlwdF9uYW1lOwogICAgIyAgICBpbmNsdWRlICAgICAgICBmYXN0Y2dpX3BhcmFtczsKICAgICN9CgogICAgIyBkZW55IGFjY2VzcyB0byAuaHRhY2Nlc3MgZmlsZXMsIGlmIEFwYWNoZSdzIGRvY3VtZW50IHJvb3QKICAgICMgY29uY3VycyB3aXRoIG5naW54J3Mgb25lCiAgICAjCiAgICAjbG9jYXRpb24gfiAvXC5odCB7CiAgICAjICAgIGRlbnkgIGFsbDsKICAgICN9CgogICAgIyBlbmFibGUgL2FwaS8gbG9jYXRpb24gd2l0aCBhcHByb3ByaWF0ZSBhY2Nlc3MgY29udHJvbCBpbiBvcmRlcgogICAgIyB0byBtYWtlIHVzZSBvZiBOR0lOWCBQbHVzIEFQSQogICAgIwogICAgI2xvY2F0aW9uIC9hcGkvIHsKICAgICMgICAgYXBpIHdyaXRlPW9uOwogICAgIyAgICBhbGxvdyAxMjcuMC4wLjE7CiAgICAjICAgIGRlbnkgYWxsOwogICAgI30KCiAgICAjIGVuYWJsZSBOR0lOWCBQbHVzIERhc2hib2FyZDsgcmVxdWlyZXMgL2FwaS8gbG9jYXRpb24gdG8gYmUKICAgICMgZW5hYmxlZCBhbmQgYXBwcm9wcmlhdGUgYWNjZXNzIGNvbnRyb2wgZm9yIHJlbW90ZSBhY2Nlc3MKICAgICMKICAgICNsb2NhdGlvbiA9IC9kYXNoYm9hcmQuaHRtbCB7CiAgICAjICAgIHJvb3QgL3Vzci9zaGFyZS9uZ2lueC9odG1sOwogICAgI30KfQo=",
          "name": "/etc/nginx/conf.d/default.conf"
        },
        {
          "contents": "c2VydmVyIHsNCiAgbGlzdGVuIDgxOw0KICByZXR1cm4gMjAwICJ0ZXN0IjsNCn0=",
          "name": "/etc/nginx/conf.d/nim-test.conf"
        },
        {
          "contents": "CnR5cGVzIHsKICAgIHRleHQvaHRtbCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBodG1sIGh0bSBzaHRtbDsKICAgIHRleHQvY3NzICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBjc3M7CiAgICB0ZXh0L3htbCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgeG1sOwogICAgaW1hZ2UvZ2lmICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGdpZjsKICAgIGltYWdlL2pwZWcgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBqcGVnIGpwZzsKICAgIGFwcGxpY2F0aW9uL2phdmFzY3JpcHQgICAgICAgICAgICAgICAgICAgICAgICAgICBqczsKICAgIGFwcGxpY2F0aW9uL2F0b20reG1sICAgICAgICAgICAgICAgICAgICAgICAgICAgICBhdG9tOwogICAgYXBwbGljYXRpb24vcnNzK3htbCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHJzczsKCiAgICB0ZXh0L21hdGhtbCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbW1sOwogICAgdGV4dC9wbGFpbiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHR4dDsKICAgIHRleHQvdm5kLnN1bi5qMm1lLmFwcC1kZXNjcmlwdG9yICAgICAgICAgICAgICAgICBqYWQ7CiAgICB0ZXh0L3ZuZC53YXAud21sICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgd21sOwogICAgdGV4dC94LWNvbXBvbmVudCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGh0YzsKCiAgICBpbWFnZS9hdmlmICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgYXZpZjsKICAgIGltYWdlL3BuZyAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBwbmc7CiAgICBpbWFnZS9zdmcreG1sICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgc3ZnIHN2Z3o7CiAgICBpbWFnZS90aWZmICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgdGlmIHRpZmY7CiAgICBpbWFnZS92bmQud2FwLndibXAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgd2JtcDsKICAgIGltYWdlL3dlYnAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB3ZWJwOwogICAgaW1hZ2UveC1pY29uICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGljbzsKICAgIGltYWdlL3gtam5nICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBqbmc7CiAgICBpbWFnZS94LW1zLWJtcCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgYm1wOwoKICAgIGZvbnQvd29mZiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB3b2ZmOwogICAgZm9udC93b2ZmMiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdvZmYyOwoKICAgIGFwcGxpY2F0aW9uL2phdmEtYXJjaGl2ZSAgICAgICAgICAgICAgICAgICAgICAgICBqYXIgd2FyIGVhcjsKICAgIGFwcGxpY2F0aW9uL2pzb24gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBqc29uOwogICAgYXBwbGljYXRpb24vbWFjLWJpbmhleDQwICAgICAgICAgICAgICAgICAgICAgICAgIGhxeDsKICAgIGFwcGxpY2F0aW9uL21zd29yZCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBkb2M7CiAgICBhcHBsaWNhdGlvbi9wZGYgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgcGRmOwogICAgYXBwbGljYXRpb24vcG9zdHNjcmlwdCAgICAgICAgICAgICAgICAgICAgICAgICAgIHBzIGVwcyBhaTsKICAgIGFwcGxpY2F0aW9uL3J0ZiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBydGY7CiAgICBhcHBsaWNhdGlvbi92bmQuYXBwbGUubXBlZ3VybCAgICAgICAgICAgICAgICAgICAgbTN1ODsKICAgIGFwcGxpY2F0aW9uL3ZuZC5nb29nbGUtZWFydGgua21sK3htbCAgICAgICAgICAgICBrbWw7CiAgICBhcHBsaWNhdGlvbi92bmQuZ29vZ2xlLWVhcnRoLmtteiAgICAgICAgICAgICAgICAga216OwogICAgYXBwbGljYXRpb24vdm5kLm1zLWV4Y2VsICAgICAgICAgICAgICAgICAgICAgICAgIHhsczsKICAgIGFwcGxpY2F0aW9uL3ZuZC5tcy1mb250b2JqZWN0ICAgICAgICAgICAgICAgICAgICBlb3Q7CiAgICBhcHBsaWNhdGlvbi92bmQubXMtcG93ZXJwb2ludCAgICAgICAgICAgICAgICAgICAgcHB0OwogICAgYXBwbGljYXRpb24vdm5kLm9hc2lzLm9wZW5kb2N1bWVudC5ncmFwaGljcyAgICAgIG9kZzsKICAgIGFwcGxpY2F0aW9uL3ZuZC5vYXNpcy5vcGVuZG9jdW1lbnQucHJlc2VudGF0aW9uICBvZHA7CiAgICBhcHBsaWNhdGlvbi92bmQub2FzaXMub3BlbmRvY3VtZW50LnNwcmVhZHNoZWV0ICAgb2RzOwogICAgYXBwbGljYXRpb24vdm5kLm9hc2lzLm9wZW5kb2N1bWVudC50ZXh0ICAgICAgICAgIG9kdDsKICAgIGFwcGxpY2F0aW9uL3ZuZC5vcGVueG1sZm9ybWF0cy1vZmZpY2Vkb2N1bWVudC5wcmVzZW50YXRpb25tbC5wcmVzZW50YXRpb24KICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBwcHR4OwogICAgYXBwbGljYXRpb24vdm5kLm9wZW54bWxmb3JtYXRzLW9mZmljZWRvY3VtZW50LnNwcmVhZHNoZWV0bWwuc2hlZXQKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB4bHN4OwogICAgYXBwbGljYXRpb24vdm5kLm9wZW54bWxmb3JtYXRzLW9mZmljZWRvY3VtZW50LndvcmRwcm9jZXNzaW5nbWwuZG9jdW1lbnQKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBkb2N4OwogICAgYXBwbGljYXRpb24vdm5kLndhcC53bWxjICAgICAgICAgICAgICAgICAgICAgICAgIHdtbGM7CiAgICBhcHBsaWNhdGlvbi93YXNtICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgd2FzbTsKICAgIGFwcGxpY2F0aW9uL3gtN3otY29tcHJlc3NlZCAgICAgICAgICAgICAgICAgICAgICA3ejsKICAgIGFwcGxpY2F0aW9uL3gtY29jb2EgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBjY287CiAgICBhcHBsaWNhdGlvbi94LWphdmEtYXJjaGl2ZS1kaWZmICAgICAgICAgICAgICAgICAgamFyZGlmZjsKICAgIGFwcGxpY2F0aW9uL3gtamF2YS1qbmxwLWZpbGUgICAgICAgICAgICAgICAgICAgICBqbmxwOwogICAgYXBwbGljYXRpb24veC1tYWtlc2VsZiAgICAgICAgICAgICAgICAgICAgICAgICAgIHJ1bjsKICAgIGFwcGxpY2F0aW9uL3gtcGVybCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBwbCBwbTsKICAgIGFwcGxpY2F0aW9uL3gtcGlsb3QgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBwcmMgcGRiOwogICAgYXBwbGljYXRpb24veC1yYXItY29tcHJlc3NlZCAgICAgICAgICAgICAgICAgICAgIHJhcjsKICAgIGFwcGxpY2F0aW9uL3gtcmVkaGF0LXBhY2thZ2UtbWFuYWdlciAgICAgICAgICAgICBycG07CiAgICBhcHBsaWNhdGlvbi94LXNlYSAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgc2VhOwogICAgYXBwbGljYXRpb24veC1zaG9ja3dhdmUtZmxhc2ggICAgICAgICAgICAgICAgICAgIHN3ZjsKICAgIGFwcGxpY2F0aW9uL3gtc3R1ZmZpdCAgICAgICAgICAgICAgICAgICAgICAgICAgICBzaXQ7CiAgICBhcHBsaWNhdGlvbi94LXRjbCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgdGNsIHRrOwogICAgYXBwbGljYXRpb24veC14NTA5LWNhLWNlcnQgICAgICAgICAgICAgICAgICAgICAgIGRlciBwZW0gY3J0OwogICAgYXBwbGljYXRpb24veC14cGluc3RhbGwgICAgICAgICAgICAgICAgICAgICAgICAgIHhwaTsKICAgIGFwcGxpY2F0aW9uL3hodG1sK3htbCAgICAgICAgICAgICAgICAgICAgICAgICAgICB4aHRtbDsKICAgIGFwcGxpY2F0aW9uL3hzcGYreG1sICAgICAgICAgICAgICAgICAgICAgICAgICAgICB4c3BmOwogICAgYXBwbGljYXRpb24vemlwICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHppcDsKCiAgICBhcHBsaWNhdGlvbi9vY3RldC1zdHJlYW0gICAgICAgICAgICAgICAgICAgICAgICAgYmluIGV4ZSBkbGw7CiAgICBhcHBsaWNhdGlvbi9vY3RldC1zdHJlYW0gICAgICAgICAgICAgICAgICAgICAgICAgZGViOwogICAgYXBwbGljYXRpb24vb2N0ZXQtc3RyZWFtICAgICAgICAgICAgICAgICAgICAgICAgIGRtZzsKICAgIGFwcGxpY2F0aW9uL29jdGV0LXN0cmVhbSAgICAgICAgICAgICAgICAgICAgICAgICBpc28gaW1nOwogICAgYXBwbGljYXRpb24vb2N0ZXQtc3RyZWFtICAgICAgICAgICAgICAgICAgICAgICAgIG1zaSBtc3AgbXNtOwoKICAgIGF1ZGlvL21pZGkgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtaWQgbWlkaSBrYXI7CiAgICBhdWRpby9tcGVnICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbXAzOwogICAgYXVkaW8vb2dnICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG9nZzsKICAgIGF1ZGlvL3gtbTRhICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtNGE7CiAgICBhdWRpby94LXJlYWxhdWRpbyAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgcmE7CgogICAgdmlkZW8vM2dwcCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIDNncHAgM2dwOwogICAgdmlkZW8vbXAydCAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHRzOwogICAgdmlkZW8vbXA0ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1wNDsKICAgIHZpZGVvL21wZWcgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtcGVnIG1wZzsKICAgIHZpZGVvL3F1aWNrdGltZSAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtb3Y7CiAgICB2aWRlby93ZWJtICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgd2VibTsKICAgIHZpZGVvL3gtZmx2ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBmbHY7CiAgICB2aWRlby94LW00diAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbTR2OwogICAgdmlkZW8veC1tbmcgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1uZzsKICAgIHZpZGVvL3gtbXMtYXNmICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBhc3ggYXNmOwogICAgdmlkZW8veC1tcy13bXYgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdtdjsKICAgIHZpZGVvL3gtbXN2aWRlbyAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBhdmk7Cn0K",
          "name": "/etc/nginx/mime.types"
        }
      ],
      "rootDir": "/etc/nginx"
    },
    "configUID": "d679d216-799e-49d3-8139-c9b8a7bb2512",
    "ignoreConflict": false,
    "validateConfig": true
  }


- `CLI 特定saved configの詳細  <#saved-config>`__ で取得した結果の ``auxFiles`` 、 ``configFiles`` をそのまま記載しています。
- ``configUid`` に入力する情報は、 `CLI 設定情報の取得 <#id4>`__ の ``uid`` と同じ値となります。
- ``ignoreConflict`` は設定の反映に関する条件を指定します。 ``false`` の場合、実機より新しいstaged configが反映されます。 ``ture`` の場合にはファイルの更新日時を無視し設定を反映します

こちらの設定情報を ``ip-10-1-1-7`` に以下curlコマンドで設定を反映します。
URLに指定する ``systemUid`` 、 ``nginxUid`` は以下の内容となります

+-----------+-----------+---------------------------------------+
|URLの値    |           |                                       |
+-----------+-----------+---------------------------------------+
|systemUid  |systemUid  | 74d52621-eb60-35ea-84f2-8f754974d560  |
+-----------+-----------+---------------------------------------+
|nginxUid   |uid        | 754ddc18-898a-59ee-b5a3-fe22cf6b3da6  |
+-----------+-----------+---------------------------------------+

.. code-block:: cmdin

  curl -sk -u "admin:nimadmin" -H "Content-Type: application/json"  https://10.1.1.5/api/platform/v1/systems/74d52621-eb60-35ea-84f2-8f754974d560/instances/754ddc18-898a-59ee-b5a3-fe22cf6b3da6/config -X POST -d @file1.txt | jq .

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  {
    "deploymentUID": "617db954-f888-46ae-80db-fc907d393ebc",
    "links": {
      "rel": "/api/platform/v1/systems/instances/deployments/617db954-f888-46ae-80db-fc907d393ebc"
    },
    "result": "Publish configuration request Accepted"
  }

``result`` に反映の結果が表示されており、正しく ``Publish configuration request Accepted`` となっていることがわかります
