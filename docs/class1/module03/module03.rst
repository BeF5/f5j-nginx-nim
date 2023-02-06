KubernetesでのNMSの利用
####

本手順ではいくつかの環境でNMS/NIMをご利用いただくにあたり、セットアップ手順を複数紹介します。
環境にあった手順を実施してください。

こちらの作業は `NGINX Management Suite Guide <https://docs.nginx.com/nginx-management-suite/>`__ の内容を参照し、実行しています


ラボ環境で動作を確認される場合、作業ホストは ``ubuntu-master(10.1.1.8)`` となります

1. 事前作業
----

`1. 事前セットアップ、HELMのインストール <https://f5j-nginx-k8s-observability.readthedocs.io/en/latest/class1/module02/module02.html#helm>`__ より手順を抜粋し、対象ホストにHELMをインストールします

.. code-block:: cmdin

  curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
  sudo apt-get update
  sudo apt-get install helm

HELMのVersionを確認します

.. code-block:: cmdin

  helm version

`3. NICのセットアップ <https://f5j-nginx-k8s-observability.readthedocs.io/en/latest/class1/module02/module02.html#nic>`__ より手順を抜粋し、対象ホストにHELMをインストールします
(こちらの手順では、NSMとの連携を実施していない、 ``nic2`` を利用します)

.. code-block:: cmdin

  cd ~/
  git clone https://github.com/BeF5/f5j-nginx-nim-lab
  git clone https://github.com/BeF5/f5j-nginx-observability-lab.git --branch v1.1.0
  git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v2.4.1
  
  cd ~/kubernetes-ingress/
  cp ~/nginx-repo* .
  ls nginx-repo.*
  make debian-image-nap-dos-plus PREFIX=registry.example.com/root/nic/nginxplus-ingress-nap-dos TARGET=container TAG=2.4.1
  docker login registry.example.com
   Username: root       << 左の文字列を入力
   Password: password   << 左の文字列を入力
  docker push registry.example.com/root/nic/nginxplus-ingress-nap-dos:2.4.1

HELMでNICをデプロイします

.. code-block:: cmdin

  cd ~/kubernetes-ingress/deployments/helm-chart-dos-arbitrator
  helm upgrade --install appdos-arbitrator . \
   --namespace nginx-ingress \
   --create-namespace

  cd ~/kubernetes-ingress/deployments/helm-chart
  cp ~/f5j-nginx-observability-lab/prep/helm/nic2-addvalue.yaml .
  helm upgrade --install nic2 -f nic2-addvalue.yaml . -n nginx-ingress

インストールの結果を確認します

.. code-block:: cmdin

  helm list -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                                   APP VERSION
  appdos-arbitrator       nginx-ingress   1               2022-12-13 15:41:32.431534051 +0000 UTC deployed        nginx-appprotect-dos-arbitrator-0.1.0   1.1.0
  nic2                    nginx-ingress   1               2022-12-13 15:50:28.582793864 +0000 UTC deployed        nginx-ingress-0.15.1                    2.4.1


NICへ通信を転送するための設定を行います。

NodePortの情報を確認します。

.. code-block:: cmdin

  kubectl get svc -n nginx-ingress | grep nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  nic2-nginx-ingress       NodePort    10.110.91.42   <none>        80:31253/TCP,443:31851/TCP   43s

表示されているポート番号を確認してください。これらの情報を元に、NGINXの設定を作成します。

.. code-block:: cmdin

  vi ~/f5j-nginx-nim-lab/prep/nginx.conf

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  user  nginx;
  worker_processes  auto;
  
  error_log  /var/log/nginx/error.log notice;
  pid        /var/run/nginx.pid;
  
  events {
      worker_connections  1024;
  }
  
  
  # TCP/UDP load balancing
  #
  stream {
  
      ##  TCP/UDP LB for NIC2 nginx2 ingressclass
      server {
          listen 80;
          proxy_pass localhost:31253;  # nic2 http port of NodePort
      }
      server {
          listen 443;
          proxy_pass localhost:31851;  # nic2 https port of NodePort
      }
  
  }

設定をコピーし、反映します

.. code-block:: cmdin

  sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf-
  sudo cp ~/f5j-nginx-nim-lab/prep/nginx.conf /etc/nginx/nginx.conf
  sudo nginx -s reload

Storage Class, Persistent Volume を作成します。こちらの内容は環境に合わせて適宜変更ください

.. code-block:: cmdin

  cd ~/f5j-nginx-nim-lab/prep
  kubectl apply -f local-sc.yaml
  kubectl apply -f local-pv-10-1-1-9.yaml

  kubectl get sc,pv

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                                        PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
  storageclass.storage.k8s.io/local-storage   kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   false                  24s
  
  NAME                    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    REASON   AGE
  persistentvolume/pv01   1Gi        RWO            Delete           Available           local-storage            12s
  persistentvolume/pv02   1Gi        RWO            Delete           Available           local-storage            12s
  persistentvolume/pv03   1Gi        RWO            Delete           Available           local-storage            12s
  persistentvolume/pv04   1Gi        RWO            Delete           Available           local-storage            12s
  persistentvolume/pv05   1Gi        RWO            Delete           Available           local-storage            12s
  persistentvolume/pv06   1Gi        RWO            Delete           Available           local-storage            12s


2. HELMによるNMSのinstall
----

.. NOTE::

  こちらの手順は NMS v2.6.0 のInstall手順となります


F5 Supportサイト `MyF5 <https://my.f5.com/>`__ にログインし、HELMに利用するパッケージをダウンロードすることでインストールが可能となります。

画面上部 ``RESOURCES`` > ``Downloads`` を開き、各プルダウンに以下の内容を選択しダウンロードします

+--------------------+-------------------------+
|Group               |NGINX                    |
+--------------------+-------------------------+
|Product Line        |NGINX Instance Manager   |
+--------------------+-------------------------+
|Product Version     |2.6.0                    |
+--------------------+-------------------------+
|Linux Distribution  |helmchart                |
+--------------------+-------------------------+
|Distribution Version|6.0                      |
+--------------------+-------------------------+
|Architecture        |k8                       |
+--------------------+-------------------------+

   .. image:: ./media/myf5-nsm-helm-download.png
      :width: 400

HELM Installに利用するDocker Imagesファイルが表示されます。ダウンロードし、Installを行う環境へ送付します
取得するファイルは以下のような名称となります。

.. code-block:: cmdin

  nms-helm-2.6.0.tar.gz

ダウンロードしたファイルをKubernetesへのデプロイを行うホストへ転送します

.. code-block:: cmdin

  cd ~/
  mkdir nim-install
  tar -xf nms-helm-2.6.0.tar.gz -C ./nim-install
  # gzip で圧縮されていない模様

展開した各Docker Imageをloadします

.. code-block:: cmdin

  cd ~/nim-install/
  ls | grep -v hybrid | awk '{ print  "docker load -i "$1 }' | sh

結果を確認します

.. code-block:: cmdin

  docker images | grep nginxdevops

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  nginxdevopssvcs.azurecr.io/indigo-tools-docker/platform/release-2-6-0/apigw          latest    585fd202532e   3 weeks ago     148MB
  nginxdevopssvcs.azurecr.io/indigo-tools-docker/platform/release-2-6-0/integrations   latest    5e4f407f4e1f   3 weeks ago     109MB
  nginxdevopssvcs.azurecr.io/indigo-tools-docker/platform/release-2-6-0/ingestion      latest    9c346bac76b4   3 weeks ago     115MB
  nginxdevopssvcs.azurecr.io/indigo-tools-docker/platform/release-2-6-0/dpm            latest    cb116746f789   3 weeks ago     125MB
  nginxdevopssvcs.azurecr.io/indigo-tools-docker/platform/release-2-6-0/core           latest    e6084032b6ee   3 weeks ago     117MB

Docker Imageのタグを変更します

.. code-block:: cmdin

  # 予め nms を registry.example.com に作成する
  docker images | grep nginxdevops | awk '{ print $1 }' |  awk -F"2-6-0" '{ print "docker tag "$1"2-6-0"$2" registry.example.com/root/nms"$2":2.6.0"  }' |sh

コンテナイメージをRegistryにPushします

.. code-block:: cmdin

  docker images | grep nms | awk '{ print "docker push "$1":"$2}' | sh

以下手順でNGINXが提供するHELMチャートの展開が可能です。

.. code-block:: cmdin

  ## cd ~/nim-install/
  tar -xf nms-hybrid-2.6.0.tgz

ラボ環境では予め作成したHELMチャートを利用します。

.. code-block:: cmdin

  ## cd ~/nim-install/
  mv nms-hybrid/values.yaml nms-hybrid/values.yaml-
  cp ~/f5j-nginx-nim-lab/prep/nms-values.yaml nms-hybrid/values.yaml

HELMを利用しデプロイします。この例ではオプションパラメータを指定し、参照する各Imageを指定します

.. code-block:: cmdin

  ## cd ~/nim-install/
  helm upgrade --install \
  --set adminPasswordHash=$(openssl passwd -1 "NIMPassword1234") \
  --set apigw.image.repository=registry.example.com/root/nms/apigw \
  --set apigw.image.tag=2.6.0 \
  --set core.image.repository=registry.example.com/root/nms/core \
  --set core.image.tag=2.6.0 \
  --set dpm.image.repository=registry.example.com/root/nms/dpm \
  --set dpm.image.tag=2.6.0 \
  --set ingestion.image.repository=registry.example.com/root/nms/ingestion \
  --set ingestion.image.tag=2.6.0 \
  --set integrations.image.repository=registry.example.com/root/nms/integrations \
  --set integrations.image.tag=2.6.0 \
  --set persistence.enable=false \
  nim ./nms-hybrid
  ## Persistent Volume の作成が必要

正しくデプロイされたことを確認します

.. code-block:: cmdin

  helm list

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
  nim     default         1               2022-12-13 15:32:57.809164688 +0000 UTC deployed        nms-hybrid-2.6.0        2.6.0

Persistent Volumeの状態を確認します。デプロイする各Podに割り当てられていることが確認できます

.. code-block:: cmdin

  kubectl get sc,pv

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                                        PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
  storageclass.storage.k8s.io/local-storage   kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   false                  13m
  
  NAME                    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS    REASON   AGE
  persistentvolume/pv01   1Gi        RWO            Delete           Bound    default/dpm-dqlite            local-storage            13m
  persistentvolume/pv02   1Gi        RWO            Delete           Bound    default/dpm-nats-streaming    local-storage            13m
  persistentvolume/pv03   1Gi        RWO            Delete           Bound    default/core-secrets          local-storage            13m
  persistentvolume/pv04   1Gi        RWO            Delete           Bound    default/core-dqlite           local-storage            13m
  persistentvolume/pv05   1Gi        RWO            Delete           Bound    default/integrations-dqlite   local-storage            13m
  persistentvolume/pv06   1Gi        RWO            Delete           Bound    default/clickhouse            local-storage            13m

各PodがRunningであることを確認します

.. code-block:: cmdin

  kubectl get pod

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                           READY   STATUS    RESTARTS   AGE
  apigw-749449768c-hnl2l         1/1     Running   0          30s
  clickhouse-86f5dd868b-ptdh5    1/1     Running   0          31s
  core-6d4c9b8ddb-r9xp2          1/1     Running   0          31s
  dpm-6ffb9c9ff-c7cmx            1/1     Running   0          31s
  ingestion-696445c77d-br9wr     1/1     Running   0          31s
  integrations-db4c7c66c-gtwhd   1/1     Running   0          31s

3. 外部から接続するためNICのセットアップ
----

.. code-block:: cmdin

  cat ~/f5j-nginx-nim-lab/prep/nms-apigw-vs.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: nms-vs
  spec:
    ingressClassName: nginx2
    host: nms.example.com
    upstreams:
    - name: nms
      service: apigw
      port: 443
      tls:
        enable: true
    routes:
    - path: /
      action:
        pass: nms

設定を反映します

.. code-block:: cmdin

  kubectl apply -f ~/f5j-nginx-nim-lab/prep/nms-apigw-vs.yaml


4. NMS への接続
----

踏み台ホストにてChromeを開き、 `http://nms.example.com/ui <http://nms.example.com/ui>`__ に接続してください
ログイン情報は以下です。

+--------+---------------+---------------------+
|username|admin          |                     |
+--------+---------------+---------------------+
|password|NIMPassword1234|HELMで指定した文字列 |
+--------+---------------+---------------------+

以下の様にTop画面が表示されます

   .. image:: ../module02/media/nim-login.png
      :width: 400

``Sign In`` をクリックすると Basic認証によるポップアップが表示されます。Username ``admin`` 、 Password は ``Install時の出力で予め確認した文字列`` を入力してください
ログインが完了すると以下のような画面が表示されます

   .. image:: ../module02/media/nim-top.png
      :width: 400
