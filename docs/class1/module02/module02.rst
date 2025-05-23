LinuxのNMSデプロイ
####

本手順ではいくつかの環境でNMS/NIMをご利用いただくにあたり、セットアップ手順を複数紹介します。
環境にあった手順を実施してください。

こちらの作業は `NGINX Management Suite Guide <https://docs.nginx.com/nginx-management-suite/>`__ の内容を参照し、実行しています

ラボ環境で動作を確認される場合、作業ホストは ``ubuntu-host1(10.1.1.5)`` となります

- `Installation Guide (On-Premises) <https://docs.nginx.com/nginx-management-suite/installation/vm-bare-metal/>`__

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
  Set up the password for the default user: password << 左の文字列を入力
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

1. 事前準備
~~~~

インストールに利用する証明書・鍵をコピーします。なお、SSL証明書および鍵ファイルは以下に配置済みです

なお、NGINX Plus R33以降はNGINXを起動するためにJWTファイルが必要になります。そのため、本LabではR32を使用しております。

JumpBox：C:\\Users\\user\\Desktop\\Key

ubuntu-host1：/home/ubuntu/

.. code-block:: cmdin

  sudo mkdir -p /etc/ssl/nginx
  sudo cp ~/nginx-repo.* /etc/ssl/nginx

インストールに必要なコンポーネントの取得、Installを行います

.. code-block:: cmdin

  wget -qO - https://cs.nginx.com/static/keys/nginx_signing.key | gpg --dearmor | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
  printf "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://pkgs.nginx.com/plus/R32/ubuntu `lsb_release -cs` nginx-plus\n" | sudo tee /etc/apt/sources.list.d/nginx-plus.list
  printf "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://pkgs.nginx.com/nms/ubuntu `lsb_release -cs` nginx-plus\n" | sudo tee /etc/apt/sources.list.d/nms.list
  sudo wget -P /etc/apt/apt.conf.d https://cs.nginx.com/static/files/90pkgs-nginx
  
  

2. NGINX Management Suite(NMS) のインストール
~~~~

NMSのプラットフォームとなる ``NGINX Instance Manager(NIM)`` をインストールします。
その他のコンポーネント(ACMなど)を利用する場合にもこちらのコンポーネントがベースとなりますので、 こちらの手順を実施してください。

.. code-block:: cmdin

  sudo apt-get update
  sudo apt-get install -y nms-instance-manager

Install時に出力される結果を確認します

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2, 89, 91

  ** 省略 **
  FQDN [nim.local]:    <-Enterを押してください。
  You have chosen: nim.local

  Further certificate generation steps will use this FQDN.
   * Creating certificates for NGINX Modules...
   *** Generating certificates for NGINX Services: agent-ingest, type - auth_server...
   *** Generating certificates for NGINX Services: dataplane-manager, type - auth_server...
   *** Generating certificates for NGINX Services: core, type - auth_server...
   *** Generating certificates for NGINX Services: integrations, type - auth_server...
   *** Generating certificates for NGINX Services: secmon, type - auth_server...
   *** Generating certificates for NGINX Services: agent-ingest, type - auth_client...
   *** Generating certificates for NGINX Services: dataplane-manager, type - auth_client...
   *** Generating certificates for NGINX Services: core, type - auth_client...
   *** Generating certificates for NGINX Services: devportal, type - auth_client...
   *** Generating certificates for NGINX Services: integrations, type - auth_client...
   *** Generating certificates for NGINX Services: secmon, type - auth_client...
   * Creating certificates for internal database components...
  Reloading systemd manager configuration
  Unmasking the service unit, 'systemctl unmask nms'
  Setting the preset flag for service unit, 'systemctl preset nms'
  Created symlink /etc/systemd/system/multi-user.target.wants/nms.service → /lib/systemd/system/nms.service.
  Unmasking the service unit, 'systemctl unmask nms-core'
  Setting the preset flag for service unit, 'systemctl preset nms-core'
  Created symlink /etc/systemd/system/nms.service.wants/nms-core.service → /lib/systemd/system/nms-core.service.
  Unmasking the service unit, 'systemctl unmask nms-dpm'
  Setting the preset flag for service unit, 'systemctl preset nms-dpm'
  Created symlink /etc/systemd/system/nms.service.wants/nms-dpm.service → /lib/systemd/system/nms-dpm.service.
  Unmasking the service unit, 'systemctl unmask nms-ingestion'
  Setting the preset flag for service unit, 'systemctl preset nms-ingestion'
  Created symlink /etc/systemd/system/nms.service.wants/nms-ingestion.service → /lib/systemd/system/nms-ingestion.service.
  Unmasking the service unit, 'systemctl unmask nms-integrations'
  Setting the preset flag for service unit, 'systemctl preset nms-integrations'
  Created symlink /etc/systemd/system/nms.service.wants/nms-integrations.service → /lib/systemd/system/nms-integrations.service.
  Unmasking the service unit, 'systemctl unmask nms-sm'
  Setting the preset flag for service unit, 'systemctl preset nms-sm'
  Created symlink /etc/systemd/system/multi-user.target.wants/nms-sm.service → /lib/systemd/system/nms-sm.service.
  Created symlink /etc/systemd/system/nms.service.wants/nms-sm.service → /lib/systemd/system/nms-sm.service.
  Adding user nginx to group nms
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

  Please follow the next steps to Start the software:

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
      sudo restorecon -F -R /usr/bin/nms-sm
      sudo restorecon -F -R /usr/lib/systemd/system/nms.service
      sudo restorecon -F -R /usr/lib/systemd/system/nms-core.service
      sudo restorecon -F -R /usr/lib/systemd/system/nms-dpm.service
      sudo restorecon -F -R /usr/lib/systemd/system/nms-ingestion.service
      sudo restorecon -F -R /usr/lib/systemd/system/nms-integrations.service
      sudo restorecon -F -R /var/lib/nms/modules/manager.json
      sudo restorecon -F -R /var/lib/nms/modules.json
      sudo restorecon -F -R /var/lib/nms/secrets
      sudo restorecon -F -R /var/lib/nms/streaming
      sudo restorecon -F -R /var/lib/nms
      sudo restorecon -F -R /var/lib/nms/dqlite
      sudo restorecon -F -R /var/run/nms
      sudo restorecon -F -R /var/lib/nms/modules
      sudo restorecon -F -R /var/log/nms

      # Enable NGINX Management Suite services
      sudo systemctl enable nms nms-core nms-dpm nms-ingestion nms-integrations nms-sm --now

      Admin username: admin

      Admin password: B8oTVUIK73cRQB11hZv6HZQGY5NUEh

  Please change this password with your own as soon as possible:
  https://docs.nginx.com/nginx-instance-manager/admin-guide/authentication/basic-auth/set-up-basic-authentication/

  For UI access, point your browser to the HTTPS port of this machine. 

  IMPORTANT: By default, NGINX Instance Manager may collect and send anonymized telemetry and interaction information for analysis by F5 NGINX.   This information is used to make improvements to our products and services. Administrators may disable this functionality for all users in the web portal.
  ----------------------------------------------------------------------
  Processing triggers for systemd (245.4-4ubuntu3.6) ...
  Processing triggers for man-db (2.9.1-1) ...
  Processing triggers for rsyslog (8.2001.0-1ubuntu1.1) ...

- NIMのAdmin情報は89,91行目の内容となりますので確認してください


設定ファイルの内容の確認します

.. code-block:: cmdin

  sudo cp /etc/nms/nms.conf /etc/nms/nms.conf-
  sudo vi /etc/nms/nms.conf

.. NOTE::

  こちらに示す設定ファイルはNIM v2.7.0以上 の内容となります

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 81-86


  # This is default /etc/nms/nms.conf file which is distributed with Linux packages.
  
  user: nms
  daemon: true
  # Root dqlite db directory. Each sub directory here is dedicated to the process
  db_root_dir: /var/lib/nms/dqlite
  
  # default log level for all processes. Each process can override this level.
  log:
    encoding: console
    level: error
  
  modules:
    prefix: /var/lib/nms
    # NMS modules config are available here to be read if installed
    conf_dir: /etc/nms/modules
  
  core:
    # enable this for core on tcp
    # address: 127.0.0.1:8033
    address: unix:/var/run/nms/core.sock
    grpc_addr: unix:/var/run/nms/coregrpc.sock
    analytics:
      # Catalogs config
      catalogs:
        metrics_data_dir: /usr/share/nms/catalogs/metrics
        events_data_dir: /usr/share/nms/catalogs/events
        dimensions_data_dir: /usr/share/nms/catalogs/dimensions
    # Dqlite config
    dqlite:
      addr: 127.0.0.1:7891
    # disable this to prevent automatic cleanup on a module removal of it's RBAC features and permissions
    disable_rbac_cleanup: false
  
  dpm:
    # enable this for dpm on tcp
    # address: 127.0.0.1:8034
    address: unix:/var/run/nms/dpm.sock
    # enable this for dpm grpc server on tcp
    # grpc_addr: 127.0.0.1:8036
    grpc_addr: unix:/var/run/nms/am.sock
    # Dqlite config
    dqlite:
      addr: 127.0.0.1:7890
    # NATS config
    nats:
      address: nats://127.0.0.1:9100
      # nats streaming
      store_root_dir: /var/lib/nms/streaming
      # 10GB
      max_store_bytes: 10737418240
      # 1GB
      max_memory_bytes: 1073741824
      # https://docs.nats.io/reference/faq#is-there-a-message-size-limitation-in-nats
      # 8MB
      max_message_bytes: 8388608
  
  integrations:
    # enable this for integrations on tcp
    # address: 127.0.0.1:8037
    address: unix:/var/run/nms/integrations.sock
    # Dqlite config
    dqlite:
      addr: 127.0.0.1:7892
    app_protect_security_update:
      # Enable this setting to automatically retrieve the latest Attack Signatures and Threat Campaigns.
      # enable: true
      # Enable this setting to specify how often, in hours, the latest Attack Signatures and Threat Campaigns are retrieved.
      # The default interval is 6 hours, the maximum interval is 48 hours, and the minimum is 1 hour.
      # interval: 6
      # Enable this setting to specify how many updates to download for the latest Attack Signatures and Threat Campaigns.
      # By default, the 10 latest updates are downloaded. The maximum value is 20, and the minimum value is 1.
      # number_of_updates: 10
  
  ingestion:
    # enable this for ingestion grpc server on tcp
    # grpc_addr: 127.0.0.1:8035
    grpc_addr: unix:/var/run/nms/ingestion.sock
  
  # ClickHouse config for establishing a ClickHouse connection
  clickhouse:
  #   # Below address not used if TLS mode is enabled
    address: 127.0.0.1:9000
  #   # Ensure username and password are wrapped in quotes
    username: 'default'
    password: 'password'
  #   # Enable TLS configurations for ClickHouse connections
  #   tls:
  #     # Address pointing to <tcp_port_secure> of ClickHouse
  #     # Below CH address is used when TLS mode is active
  #     tls_address: 127.0.0.1:9440
  #     # Verification should be skipped for self-signed certificates
  #     skip_verify: true
  #     key_path: /path/to/client-key.pem
  #     cert_path: /path/to/client-cert.pem
  #     ca_path: /etc/ssl/certs/ca-certificates.crt


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
  
  root       12619  0.0  0.0   8160   672 pts/1    S+   16:50   0:00              \_ grep --color=auto nms
  nms        12469  0.1  0.4 1267192 36336 ?       Ssl  16:47   0:00 /usr/bin/nms-ingestion
  nms        12494  0.0  0.5 1257480 42436 ?       Ssl  16:47   0:00 /usr/bin/nms-sm start
  nms        12511  0.2  0.5 1304640 45328 ?       Ssl  16:47   0:00 /usr/bin/nms-integrations
  nms        12537  0.4  0.6 1302140 52588 ?       Ssl  16:47   0:00 /usr/bin/nms-core
  nms        12545  0.7  0.9 1318664 75736 ?       Ssl  16:47   0:01 /usr/bin/nms-dpm

NGINX Web Serverを再起動します

.. code-block:: cmdin

  sudo systemctl restart nginx

NGINXが正しく起動していることを確認します

.. code-block:: cmdin

  sudo systemctl status nginx

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

3. API Connectivity Manager(ACM)のインストール
~~~~

こちらの手順は `Install NGINX Management Suite Modules <https://docs.nginx.com/nginx-management-suite/installation/vm-bare-metal/install-acm/>`__ の ``API CONNECTIVITY MANAGER`` のタブを参考にしています

なお、API Connectivity Managerは2024年1月1日にEoSのため参考として掲載しております。

ACMをインストールします

.. code-block:: cmdin

  # sudo apt-get update
  sudo apt-get install -y nms-api-connectivity-manager

NMSを起動します

.. code-block:: cmdin

  sudo systemctl enable nms-acm

  sudo systemctl restart nms
  sudo systemctl restart nms-core
  sudo systemctl restart nms-dpm
  sudo systemctl restart nms-ingestion
  sudo systemctl restart nms-integrations
  sudo systemctl restart nginx
  sudo systemctl start nms-acm

ACMが正しく起動していることを確認します

.. code-block:: cmdin

  sudo systemctl status nms-acm

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ● nms-acm.service - NGINX Management Suite - API Connectivity Manager
       Loaded: loaded (/lib/systemd/system/nms-acm.service; enabled; vendor preset: enabled)
       Active: active (running) since Fri 2023-02-10 02:43:05 UTC; 27s ago
         Docs: https://www.nginx.com/products/api-connectivity-manager
     Main PID: 12451 (nms-acm)
        Tasks: 13 (limit: 9445)
       Memory: 18.2M
       CGroup: /system.slice/nms-acm.service
               └─12451 /usr/bin/nms-acm server
  
  Feb 10 02:43:08 ip-10-1-1-6 acm[12451]: [INFO]         acm                                          templates/service.go:119     >
  Feb 10 02:43:08 ip-10-1-1-6 acm[12451]: [INFO]         acm                                          templates/service.go:119     >
  Feb 10 02:43:08 ip-10-1-1-6 acm[12451]: [INFO]         acm                                          templates/service.go:119     >
  Feb 10 02:43:08 ip-10-1-1-6 acm[12451]: [INFO]         acm                                          templates/service.go:119     >
  Feb 10 02:43:08 ip-10-1-1-6 acm[12451]: [INFO]         acm                                          templates/service.go:119     >
  Feb 10 02:43:08 ip-10-1-1-6 acm[12451]: [INFO]         acm                                          templates/service.go:119     >
  Feb 10 02:43:08 ip-10-1-1-6 acm[12451]: [INFO]         acm                                          templates/service.go:119     >
  Feb 10 02:43:08 ip-10-1-1-6 acm[12451]: [INFO]         acm                                          templates/service.go:119     >
  Feb 10 02:43:08 ip-10-1-1-6 acm[12451]: [INFO]         acm                                          templates/service.go:37      >
  Feb 10 02:43:09 ip-10-1-1-6 acm[12451]: [INFO]         acm                                          templates/service.go:61      >

プロセスの動作状況の結果を参考に示します

.. code-block:: cmdin

  ps aufx | grep nms

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ubuntu     12607  0.0  0.0   8160   672 pts/0    S+   02:55   0:00              \_ grep --color=auto nms
  nms        12385  0.2  0.7 1376852 62380 ?       Ssl  02:43   0:01 /usr/bin/nms-core
  nms        12435  0.3  0.7 1379940 63544 ?       Ssl  02:43   0:02 /usr/bin/nms-dpm
  nms        12479  0.1  0.3 1265868 31216 ?       Ssl  02:43   0:01 /usr/bin/nms-ingestion
  nms        12515  0.0  0.5 1334052 42072 ?       Ssl  02:43   0:00 /usr/bin/nms-integrations
  nms        12595  1.1  0.7 1268892 63196 ?       Ssl  02:53   0:01 /usr/bin/nms-acm server


4. WAF Compilerのインストール
~~~~

こちらの手順は `Set Up App Protect WAF Configuration Management <https://docs.nginx.com/nginx-management-suite/nim/how-to/app-protect/setup-waf-config-management/>`__ を参考にしています


WAF Compilerをインストールします

.. code-block:: cmdin

  # sudo apt-get update
  sudo apt-get install -y nms-nap-compiler-v5.144.0

NMSを起動します

.. code-block:: cmdin

  sudo systemctl restart nms-integrations

プロセスの動作状況の結果を参考に示します。 ``Compilerの名称のプロセスは動作しません。``

.. code-block:: cmdin

  ps aufx | grep nms

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ubuntu     18301  0.0  0.0   8160   672 pts/0    S+   03:09   0:00              \_ grep --color=auto nms
  nms        12988  0.2  0.7 1378084 59972 ?       Ssl  03:00   0:01 /usr/bin/nms-core
  nms        13046  0.4  0.7 1380308 59392 ?       Ssl  03:00   0:02 /usr/bin/nms-dpm
  nms        13089  0.1  0.4 1265868 32516 ?       Ssl  03:00   0:00 /usr/bin/nms-ingestion
  nms        13180  0.2  0.5 1334620 42576 ?       Ssl  03:01   0:01 /usr/bin/nms-acm server
  nms        18269  1.2  0.3 1284656 29796 ?       Ssl  03:09   0:00 /usr/bin/nms-integrations


3. NMS への接続
----

対象となるホストのIPアドレスを確認し、 踏み台ホストにてChromeを開き、 ``https://<ホストのIPアドレス>/ui`` に接続してください

なお、すでにライセンス適用済みのNIMインスタンスがあり、``NIM UI``　からアクセス可能です。Username ``admin`` 、 Password は ``password``　でログインしてください。

以下の様にTop画面が表示されます

   .. image:: ./media/nim-login.png
      :width: 400

``Sign In`` をクリックすると Basic認証によるポップアップが表示されます。Username ``admin`` 、 Password は ``Install時の出力で予め確認した文字列`` を入力してください
ログインが完了すると以下のような画面が表示されます

   .. image:: ./media/nim-top.png
      :width: 400

(Option) NMS の Version確認
----

正しく意図したバージョンがインストールされていることを確認してください。


.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  $ dpkg -l | grep nms
  ii  nms-api-connectivity-manager     1.4.1-762997411~focal              amd64        NGINX Management Suite ACM Module.
  ii  nms-instance-manager             2.8.0-759861272~focal              amd64        NGINX Management Suite - Instance Manager (core system)
  ii  nms-nap-compiler-v4.2.0          4.2.0-1~focal                      amd64        NGINX App Protect repackaged compiler for compatability with NGINX Instance Manager
  ii  nms-sm                           1.2.0-751410248~focal              amd64        NGINX Security Monitoring Dashboard Module


以下コマンドを使ってインストールしたNIMの詳細情報を確認いただけます

.. code-block:: cmdin

  dpkg -s nms-instance-manager

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Package: nms-instance-manager
  Status: install ok installed
  Priority: optional
  Installed-Size: 208328
  Maintainer: NGINX Packaging <nginx-packaging@f5.com>
  Architecture: amd64
  Version: 2.8.0-759861272~focal
  Depends: adduser, gawk, lsb-release, nginx-plus (>= 22) | nginx (>= 1.18.0), openssl, rsyslog, systemd, tar
  Recommends: clickhouse-server (>= 21.3.19.1), openssl (>= 1.1.1)
  Conffiles:
   /etc/logrotate.d/nms.conf 9c4dc2b56a4496bb35547f205a81d750
   /etc/nginx/conf.d/nms-http.conf e9f45890256ca87cc64737de6aeb998f
   /etc/nms/nginx/.htpasswd d41d8cd98f00b204e9800998ecf8427e
   /etc/nms/nginx/errors-grpc.loc_conf 602e26ca21e12a11262c170f88e90c38
   /etc/nms/nginx/errors-grpc.server_conf 73f48a717d8e7cb6ce73cdc22efc67b3
   /etc/nms/nginx/errors.http_conf 73f1d2692f94440ad35c1c4934dc08cd
   /etc/nms/nginx/oidc/openid_configuration.conf 42b3c5cb96e5b8a0df87d8c882e59077
   /etc/nms/nginx/upstreams/README.md f29b0fe2b4d6856f26f7286f3c9e0579
   /etc/nms/nginx/upstreams/mapped_apis/README.md c287571d3c9cddf6a85d2cdd6fc14dae
   /etc/nms/nms.conf 88e66e7f0f891bb3c4d8dc0ac7871f6e
   /etc/rsyslog.d/nms.conf 3fdc4c5ef473f05d85251266b30d8521
   /usr/lib/systemd/system/nms-core.service 3bb5bb05e05e9dd1ff62d6f9ea650e3b
   /usr/lib/systemd/system/nms-dpm.service 9ee5e027e6694ee988c78eff4e043a26
   /usr/lib/systemd/system/nms-ingestion.service 69c2bf77c707f59b2f58f9bae0525d66
   /usr/lib/systemd/system/nms-integrations.service 23012c3c61c0df2046e65131cbab1fc7
   /usr/lib/systemd/system/nms.service 99ce4153417884beb7dac8556544c75c
   /var/lib/nms/modules.json 58e0494c51d30eb3494f7c9198986bb9
  Description: NGINX Management Suite - Instance Manager (core system)
  Homepage: https://www.nginx.com/products/nginx-instance-manager/


(Option) SMへSignatureのinstall
----

 - `Set Up Attack Signatures and Threat Campaigns <https://docs.nginx.com/nginx-management-suite/nim/how-to/app-protect/setup-waf-config-management/#set-up-attack-signatures-and-threat-campaigns>`__
