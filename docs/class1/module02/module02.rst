LinuxでのNMSの利用
####

本手順ではいくつかの環境でNMS/NIMをご利用いただくにあたり、セットアップ手順を複数紹介します。
環境にあった手順を実施してください。

こちらの作業は `NGINX Management Suite Guide <https://docs.nginx.com/nginx-management-suite/>`__ の内容を参照し、実行しています

ラボ環境で動作を確認される場合、作業ホストは ``ubuntu-host1(10.1.1.5)`` となります

- `Installation Guide <https://docs.nginx.com/nginx-management-suite/admin-guides/installation/install-guide/>`__

1. Click HouseのInstall
----

Install手順はClick Houseのマニュアルを参照しています

- `Installing ClickHouse <https://clickhouse.com/docs/en/install/>`__

.. NOTE::

  こちらの手順は Click House 22.11.2 のInstall手順となります

Installに必要なコンポーネントの取得、Installを行います

.. code-block:: cmdin

  sudo apt-get install -y apt-transport-https ca-certificates dirmngr
  sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754
  
  echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee \
      /etc/apt/sources.list.d/clickhouse.list
  sudo apt-get update

Click HouseのInstallします

.. code-block:: cmdin

  sudo apt-get install -y clickhouse-server clickhouse-client

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

   ** 省略 **
   chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-odbc-bridge'
   chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-library-bridge'
  Enter password for default user: password << 左の文字列を入力
  Password for default user is saved in file /etc/clickhouse-server/users.d/default-password.xml.
  Setting capabilities for clickhouse binary. This is optional.
   chown -R clickhouse:clickhouse '/etc/clickhouse-server'
  
  ClickHouse has been successfully installed.

Click Houseのサービスを起動し、状態を確認します

.. code-block:: cmdin

  sudo service clickhouse-server start
  sudo service clickhouse-server status

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ● clickhouse-server.service - ClickHouse Server (analytic DBMS for big data)
       Loaded: loaded (/lib/systemd/system/clickhouse-server.service; enabled; vendor preset: enabled)
       Active: active (running) since Tue 2022-12-13 09:37:45 UTC; 3s ago
     Main PID: 2774 (clckhouse-watch)
        Tasks: 205 (limit: 4652)
       Memory: 65.0M
       CGroup: /system.slice/clickhouse-server.service
               ├─2774 clickhouse-watchdog        --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid
               └─2787 /usr/bin/clickhouse-server --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid
  
  Dec 13 09:37:45 ip-10-1-1-5 systemd[1]: Started ClickHouse Server (analytic DBMS for big data).
  Dec 13 09:37:45 ip-10-1-1-5 clickhouse-server[2774]: Processing configuration file '/etc/clickhouse-server/config.xml'.
  Dec 13 09:37:45 ip-10-1-1-5 clickhouse-server[2774]: Logging trace to /var/log/clickhouse-server/clickhouse-server.log
  Dec 13 09:37:45 ip-10-1-1-5 clickhouse-server[2774]: Logging errors to /var/log/clickhouse-server/clickhouse-server.err.log
  Dec 13 09:37:45 ip-10-1-1-5 clickhouse-server[2787]: Processing configuration file '/etc/clickhouse-server/config.xml'.
  Dec 13 09:37:45 ip-10-1-1-5 clickhouse-server[2787]: Saved preprocessed configuration to '/var/lib/clickhouse/preprocessed_configs/config.xml'.
  Dec 13 09:37:45 ip-10-1-1-5 clickhouse-server[2787]: Processing configuration file '/etc/clickhouse-server/users.xml'.
  Dec 13 09:37:45 ip-10-1-1-5 clickhouse-server[2787]: Merging configuration file '/etc/clickhouse-server/users.d/default-password.xml'.
  Dec 13 09:37:45 ip-10-1-1-5 clickhouse-server[2787]: Saved preprocessed configuration to '/var/lib/clickhouse/preprocessed_configs/users.xml'.

Click House Clientを実行し、接続できることを確認します

.. code-block:: cmdin

  clickhouse-client --password

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ClickHouse client version 22.11.2.30 (official build).
  Password for user (default): password << 先程設定したパスワードを入力してください
  Connecting to localhost:9000 as user default.
  Connected to ClickHouse server version 22.11.2 revision 54460.
  
  Warnings:
   * Maximum number of threads is lower than 30000. There could be problems with handling a lot of simultaneous queries.
  
  ip-10-1-1-5.xxx.internal :) q << "q" を入力し、クライアントを終了してください
  Bye.

- 1行目にClient Version、4行目にClick HouseのVersionが表示されていることがわかります


2. NMSのinstall
----

インストールに利用する証明書・鍵をコピーします

.. code-block:: cmdin

  sudo mkdir -p /etc/ssl/nginx
  sudo cp ~/nginx-repo.* /etc/ssl/nginx

Installに必要なコンポーネントの取得、Installを行います

.. code-block:: cmdin

  printf "deb https://pkgs.nginx.com/nms/ubuntu `lsb_release -cs` nginx-plus\n" | sudo tee /etc/apt/sources.list.d/nms.list
  sudo wget -q -O /etc/apt/apt.conf.d/90pkgs-nginx https://cs.nginx.com/static/files/90pkgs-nginx
  wget -O /tmp/nginx_signing.key https://cs.nginx.com/static/keys/nginx_signing.key
  sudo apt-key add /tmp/nginx_signing.key

NGINX Management Suite を Install します

.. code-block:: cmdin

  sudo apt-get update
  sudo apt-get install -y nms-instance-manager=2.6.0-698150575~jammy

Install時に出力される結果を確認します

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2-3, 6,8, 56,58

  ** 省略 **
  WARNING: user 'nginx' does not exist. Installation will continue.
  Adding user www-data to group nms
  Adding user syslog to group nms
  Ensuring the log file exists, 'touch /var/log/nms/nms.log'
  Generating default password for 'admin' user account
  Using openssl version 1.1.1f
  Writing admin password to /etc/nms/nginx/.htpasswd
  Checking if clickhouse-server is installed, 'which clickhouse-server'.
  /usr/bin/clickhouse-server
  Restarting rsyslog process
  ----------------------------------------------------------------------
  NGINX Management Suite package has been successfully installed.
  
  Please follow the next steps to start the software:
      # Start the Clickhouse database server
      sudo systemctl start clickhouse-server
  
      # Start NGINX web server
      sudo systemctl start nginx
  
      # If NGINX is already running, reload it
      sudo service nginx reload
  
      # Optional: load the included SELinux policy
      sudo semodule -n -i /usr/share/selinux/packages/nms.pp
      sudo /usr/sbin/load_policy
      sudo restorecon -F -R /usr/bin/nms-core
      sudo restorecon -F -R /usr/bin/nms-dpm
      sudo restorecon -F -R /usr/bin/nms-ingestion
      sudo restorecon -F -R /usr/bin/nms-integrations
      sudo restorecon -F -R /usr/lib/systemd/system/nms.service
      sudo restorecon -F -R /usr/lib/systemd/system/nms-core.service
      sudo restorecon -F -R /usr/lib/systemd/system/nms-dpm.service
      sudo restorecon -F -R /usr/lib/systemd/system/nms-ingestion.service
      sudo restorecon -F -R /usr/lib/systemd/system/nms-integrations.service
      sudo restorecon -F -R /var/lib/nms/modules/manager.json
      sudo restorecon -F -R /var/lib/nms/modules.json
      sudo restorecon -F -R /var/lib/nms/streaming
      sudo restorecon -F -R /var/lib/nms
      sudo restorecon -F -R /var/lib/nms/dqlite
      sudo restorecon -F -R /var/run/nms
      sudo restorecon -F -R /var/lib/nms/modules
      sudo restorecon -F -R /var/log/nms
  
      # Start now and ensure the services also starts whenever the system boots
      sudo systemctl enable nms nms-core nms-dpm nms-ingestion nms-integrations --now
  
      # Optional: Start NGINX Management Suite services
      sudo systemctl start nms
      sudo systemctl start nms-core
      sudo systemctl start nms-dpm
      sudo systemctl start nms-ingestion
      sudo systemctl start nms-integrations
  
      Admin username: admin
  
      Admin password: O5oa1sZN9rmvGSo1gHi2BbjQzofSvE
  
  Please change this password with your own as soon as possible:
  https://docs.nginx.com/nginx-management-suite/admin-guides/access-control/configure-authentication/
  
  For UI access, point your browser to the HTTPS port of this machine.
  ----------------------------------------------------------------------
  Processing triggers for rsyslog (8.2001.0-1ubuntu1.1) ...
  Processing triggers for ufw (0.36-6) ...
  Processing triggers for systemd (245.4-4ubuntu3.6) ...
  Processing triggers for man-db (2.9.1-1) ...
  Processing triggers for libc-bin (2.31-0ubuntu9.2) ...

- 2-3行目 で NGINXが存在しないためインストールしていることがわかります。NISのSubscriptionではNGINX Plusを利用することが可能で、RBACを利用する場合にはNGINX Plusが必要となります。その場合、NMSInstallの前にNGINX PlusのInstallが必要となります
- 6,8行目 で NIMの初期ユーザ ``admin`` を作成し、パスワード情報をセットしていることがわかります。その結果が 56,58行目の内容となりますので情報を確認してください


設定ファイルの内容の確認します

.. code-block:: cmdin

  sudo cp /etc/nms/nms.conf /etc/nms/nms.conf-
  sudo vi /etc/nms/nms.conf

.. NOTE::

  こちらに示す設定ファイルはNIM v2.6.0 の内容となります

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 61-62

  # This is default /etc/nms/nms.conf file which is distributed with Linux packages.

  user = nms
  access_log = stdout
  error_log = stderr
  log_encoding = console
  log_level = error
  # enable this for core on tcp
  # core_address = 127.0.0.1:8033
  core_address = unix:/var/run/nms/core.sock
  core_grpc_address = unix:/var/run/nms/coregrpc.sock
  core_secrets_dir = /var/lib/nms/secrets/
  # enable this for dpm on tcp
  # dpm_address = 127.0.0.1:8034
  dpm_address = unix:/var/run/nms/dpm.sock
  # enable this for dpm grpc server on tcp
  # dpm_grpc_addr = 127.0.0.1:8036
  dpm_grpc_addr = unix:/var/run/nms/am.sock
  # enable this for integrations on tcp
  # integrations_address = 127.0.0.1:8037
  integrations_address = unix:/var/run/nms/integrations.sock
  daemon = 1
  
  # Catalogs config
  metrics_data_dir = /usr/share/nms/catalogs/metrics
  events_data_dir = /usr/share/nms/catalogs/events
  dimensions_data_dir = /usr/share/nms/catalogs/dimensions
  
  # enable this for ingestion grpc server on tcp
  # ingest_grpc_addr = 127.0.0.1:8035
  ingest_grpc_addr = unix:/var/run/nms/ingestion.sock
  
  # enable this for integrations on tcp
  # integrations_http_addr = 127.0.0.1:8037
  integrations_http_addr = unix:/var/run/nms/integrations.sock
  
  # Root dqlite db directory
  ctr_db_root_dir = /var/lib/nms/dqlite # each sub directory here is dedicated to the process
  
  # Dqlite config
  dpm_dqlite_db_addr = 127.0.0.1:7890
  core_dqlite_db_addr = 127.0.0.1:7891
  integrations_dqlite_db_addr = 127.0.0.1:7892
  
  # NATS config
  nats_address = nats://127.0.0.1:9100
  # nats streaming
  nats_store_root_dir = /var/lib/nms/streaming
  # 10GB
  nats_max_store_bytes = 10737418240
  # 1GB
  nats_max_memory_bytes = 1073741824
  # https://docs.nats.io/reference/faq#is-there-a-message-size-limitation-in-nats
  # 8MB
  nats_max_message_bytes = 8388608
  
  modules_prefix = /var/lib/nms
  
  # ClickHouse config for establishing a ClickHouse connection
  # Below address not used if TLS mode is enabled
  # clickhouse_address = 127.0.0.1:9000
  # Ensure username and password are wrapped in quotes
  clickhouse_username = 'default' << 適切に接続できるようにパラメータを指定してください
  clickhouse_password = 'password' << 適切に接続できるようにパラメータを指定してください
  
  ### TLS configurations for ClickHouse connections
  # TLS is turned off by default
  # clickhouse_tls_mode = true
  # Address pointing to <tcp_port_secure> of ClickHouse
  # Below CH address is used when TLS mode is active
  # clickhouse_tls_address = 127.0.0.1:9440
  # Verification should be skipped for self-signed certificates
  # clickhouse_tls_skip_verify = true
  # clickhouse_tls_key_path = /path/to/client-key.pem
  # clickhouse_tls_cert_path = /path/to/client-cert.pem
  # clickhouse_tls_ca_path = /etc/ssl/certs/ca-certificates.crt

Clickhouse で指定した適切な ``username`` 、 ``password`` を記述します

NMSを有効にします

.. code-block:: cmdin

  sudo systemctl enable nms
  sudo systemctl enable nms-core
  sudo systemctl enable nms-dpm
  sudo systemctl enable nms-ingestion
  sudo systemctl enable nms-integrations

NMSを起動します

.. code-block:: cmdin

  sudo systemctl start nms
  sudo systemctl start nms-core
  sudo systemctl start nms-dpm
  sudo systemctl start nms-ingestion
  sudo systemctl start nms-integrations

NMSが起動していることを確認します

.. code-block:: cmdin

  ps aufx | grep nms

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ubuntu     18756  0.0  0.0   8160   724 pts/0    S+   10:49   0:00  |           \_ grep --color=auto nms
  nms        18667  1.3  0.9 1261428 37752 ?       Ssl  10:49   0:00 /usr/bin/nms-ingestion
  nms        18687  1.5  0.6 1357064 27024 ?       Ssl  10:49   0:00 /usr/bin/nms-integrations
  nms        18709  4.1  1.3 1374596 52064 ?       Ssl  10:49   0:00 /usr/bin/nms-dpm
  nms        18710  3.9  1.0 1364580 42960 ?       Ssl  10:49   0:00 /usr/bin/nms-core

NGINX Web Serverを再起動します

.. code-block:: cmdin

  sudo systemctl restart nginx

NGINXが正しく起動していることを確認します

.. code-block:: cmdin

  service nginx status

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ● nginx.service - A high performance web server and a reverse proxy server
       Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
       Active: active (running) since Tue 2022-12-13 10:50:05 UTC; 12s ago
         Docs: man:nginx(8)
      Process: 18761 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
      Process: 18775 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
     Main PID: 18776 (nginx)
        Tasks: 3 (limit: 4652)
       Memory: 4.2M
       CGroup: /system.slice/nginx.service
               ├─18776 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
               ├─18777 nginx: worker process
               └─18778 nginx: worker process
  
  Dec 13 10:50:05 ip-10-1-1-5 systemd[1]: nginx.service: Succeeded.
  Dec 13 10:50:05 ip-10-1-1-5 systemd[1]: Stopped A high performance web server and a reverse proxy server.
  Dec 13 10:50:05 ip-10-1-1-5 systemd[1]: Starting A high performance web server and a reverse proxy server...
  Dec 13 10:50:05 ip-10-1-1-5 systemd[1]: Started A high performance web server and a reverse proxy server.

3. NMS への接続
----

対象となるホストのIPアドレスを確認し、 踏み台ホストにてChromeを開き、 ``https://<ホストのIPアドレス>/ui`` に接続してください

以下の様にTop画面が表示されます

   .. image:: ./media/nim-login.png
      :width: 400

``Sign In`` をクリックすると Basic認証によるポップアップが表示されます。Username ``admin`` 、 Password は ``Install時の出力で予め確認した文字列`` を入力してください
ログインが完了すると以下のような画面が表示されます

   .. image:: ./media/nim-top.png
      :width: 400

(Option) NMS の Version確認
----

以下コマンドを使って動作するNIMのVersionを確認いただけます

.. code-block:: cmdin

  dpkg -s nms-instance-manager

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Package: nms-instance-manager
  Status: install ok installed
  Priority: optional
  Installed-Size: 188463
  Maintainer: NGINX Packaging <nginx-packaging@f5.com>
  Architecture: amd64
  Version: 2.6.0-698150575~focal
  Depends: adduser, gawk, lsb-release, nginx (>= 1.18.0) | nginx-plus (>= 22), openssl, rsyslog, systemd, tar
  Recommends: clickhouse-server (>= 21.3.19.1), openssl (>= 1.1.1)
  Conffiles:
   /etc/logrotate.d/nms.conf 9c4dc2b56a4496bb35547f205a81d750
   /etc/nginx/conf.d/nms-http.conf a4fa61b58ad35d03e1e3d7c6970797ee
   /etc/nms/nginx/.htpasswd d41d8cd98f00b204e9800998ecf8427e
   /etc/nms/nginx/errors-grpc.loc_conf 602e26ca21e12a11262c170f88e90c38
   /etc/nms/nginx/errors-grpc.server_conf 73f48a717d8e7cb6ce73cdc22efc67b3
   /etc/nms/nginx/errors.http_conf 73f1d2692f94440ad35c1c4934dc08cd
   /etc/nms/nginx/oidc/openid_configuration.conf 42b3c5cb96e5b8a0df87d8c882e59077
   /etc/nms/nginx/upstreams/README.md f29b0fe2b4d6856f26f7286f3c9e0579
   /etc/nms/nginx/upstreams/mapped_apis/README.md c287571d3c9cddf6a85d2cdd6fc14dae
   /etc/nms/nms.conf f63c6974768ec18a39977667b3bd820a
   /etc/rsyslog.d/nms.conf 3fdc4c5ef473f05d85251266b30d8521
   /usr/lib/systemd/system/nms-core.service 3bb5bb05e05e9dd1ff62d6f9ea650e3b
   /usr/lib/systemd/system/nms-dpm.service 9ee5e027e6694ee988c78eff4e043a26
   /usr/lib/systemd/system/nms-ingestion.service 69c2bf77c707f59b2f58f9bae0525d66
   /usr/lib/systemd/system/nms-integrations.service 23012c3c61c0df2046e65131cbab1fc7
   /usr/lib/systemd/system/nms.service 99ce4153417884beb7dac8556544c75c
   /var/lib/nms/modules.json 58e0494c51d30eb3494f7c9198986bb9
  Description: NGINX Management Suite - Instance Manager (core system)
  Homepage: https://www.nginx.com/products/nginx-instance-manager/


(Option) Vault の Install (作成中)
----

NGINX Management Suite は Secret のストアとしてVaultを利用することが可能です。
Install手順はVaultのマニュアルを参照しています

- `Install Vault <https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install>`__

本手順は参考のInstall手順のみを示しております。利用方法は以下を参照してください。

- `Configure Vault for Storing Secrets <https://docs.nginx.com/nginx-management-suite/admin-guides/getting-started/configure-vault/>`__

.. NOTE::

  こちらの手順は Vault v1.12.2 のInstall手順となります

Installに必要なコンポーネントの取得、Installを行います

.. code-block:: cmdin

  sudo apt update && sudo apt install gpg

  wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg >/dev/null
  gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint
  echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

Vault を Install します

.. code-block:: cmdin

  sudo apt update && sudo apt install vault

Click Houseのサービスを起動し、状態を確認します

.. code-block:: cmdin

  service vault start
  service vault status

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ● vault.service - "HashiCorp Vault - A tool for managing secrets"
       Loaded: loaded (/lib/systemd/system/vault.service; disabled; vendor preset: enabled)
       Active: active (running) since Tue 2022-12-13 09:53:30 UTC; 3s ago
         Docs: https://www.vaultproject.io/docs/
     Main PID: 15746 (vault)
        Tasks: 8 (limit: 4652)
       Memory: 62.4M
       CGroup: /system.slice/vault.service
               └─15746 /usr/bin/vault server -config=/etc/vault.d/vault.hcl
  
  Dec 13 09:53:30 ip-10-1-1-5 vault[15746]:                    Mlock: supported: true, enabled: true
  Dec 13 09:53:30 ip-10-1-1-5 vault[15746]:            Recovery Mode: false
  Dec 13 09:53:30 ip-10-1-1-5 vault[15746]:                  Storage: file
  Dec 13 09:53:30 ip-10-1-1-5 vault[15746]:                  Version: Vault v1.12.2, built 2022-11-23T12:53:46Z
  Dec 13 09:53:30 ip-10-1-1-5 vault[15746]:              Version Sha: 415e1fe3118eebd5df6cb60d13defdc01aa17b03
  Dec 13 09:53:30 ip-10-1-1-5 vault[15746]: ==> Vault server started! Log data will stream in below:
  Dec 13 09:53:30 ip-10-1-1-5 vault[15746]: 2022-12-13T09:53:30.240Z [INFO]  proxy environment: http_proxy="" https_proxy="" no_proxy=""
  Dec 13 09:53:30 ip-10-1-1-5 vault[15746]: 2022-12-13T09:53:30.240Z [WARN]  no `api_addr` value specified in config or in VAULT_API_ADDR; falling back to >
  Dec 13 09:53:30 ip-10-1-1-5 vault[15746]: 2022-12-13T09:53:30.267Z [INFO]  core: Initializing version history cache for core
  Dec 13 09:53:30 ip-10-1-1-5 systemd[1]: Started "HashiCorp Vault - A tool for managing secrets".

Vault の Version を確認します

.. code-block:: cmdin

  vault version

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Vault v1.12.2 (415e1fe3118eebd5df6cb60d13defdc01aa17b03), built 2022-11-23T12:53:46Z
