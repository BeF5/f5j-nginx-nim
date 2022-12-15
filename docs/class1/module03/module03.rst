NGINX Agentのデプロイ
####

管理対象となるNGINX OSS/NGINX PlusにNGINX Agentをインストールすることにより様々な操作が可能となります。


1. NGINX Agent のインストール
====

1. NIMより取得したNGINX AgentをLinuxへInstall
----

画面に表示された内容を参考に、NGINX Agent をInstallします

   .. image:: ./media/nim-instances.png
      :width: 400

.. code-block:: cmdin

  curl -k https://10.1.1.5/install/nginx-agent | sudo sh


NGINX Agentを起動します

.. code-block:: cmdin

  sudo systemctl enable nginx-agent
  sudo systemctl start nginx-agent
  
  sudo systemctl status nginx-agent
  ● nginx-agent.service - NGINX Agent
       Loaded: loaded (/etc/systemd/system/nginx-agent.service; enabled; vendor preset: enabled)
       Active: active (running) since Tue 2022-12-13 13:59:39 UTC; 24s ago
         Docs: https://www.nginx.com/products/nginx-agent/
     Main PID: 21479 (nginx-agent)
        Tasks: 9 (limit: 4652)
       Memory: 9.7M
       CGroup: /system.slice/nginx-agent.service
               └─21479 /usr/bin/nginx-agent
  
  Dec 13 13:59:40 ip-10-1-1-5 nginx-agent[21479]: time="2022-12-13T13:59:40Z" level=warning msg="The NGINX API is not configured. Please configure it to co>
  Dec 13 13:59:40 ip-10-1-1-5 nginx-agent[21479]: time="2022-12-13T13:59:40Z" level=info msg="OneTimeRegistration completed"
  Dec 13 13:59:40 ip-10-1-1-5 nginx-agent[21479]: time="2022-12-13T13:59:40Z" level=info msg="Commander received meta:<timestamp:<seconds:1670939980 nanos:>
  Dec 13 13:59:40 ip-10-1-1-5 nginx-agent[21479]: time="2022-12-13T13:59:40Z" level=info msg="config command &{agent_config:<details:<features:\"features_r>
  Dec 13 13:59:40 ip-10-1-1-5 nginx-agent[21479]: time="2022-12-13T13:59:40Z" level=info msg="Upload: Sending data chunk data 0 (messageId=02d98e5d-d09c-42>
  Dec 13 13:59:40 ip-10-1-1-5 nginx-agent[21479]: time="2022-12-13T13:59:40Z" level=info msg="Upload: Sending data chunk data 1 (messageId=02d98e5d-d09c-42>
  Dec 13 13:59:40 ip-10-1-1-5 nginx-agent[21479]: time="2022-12-13T13:59:40Z" level=info msg="Upload: Sending data chunk data 2 (messageId=02d98e5d-d09c-42>
  Dec 13 13:59:40 ip-10-1-1-5 nginx-agent[21479]: time="2022-12-13T13:59:40Z" level=info msg="Upload: Sending data chunk data 3 (messageId=02d98e5d-d09c-42>
  Dec 13 13:59:40 ip-10-1-1-5 nginx-agent[21479]: time="2022-12-13T13:59:40Z" level=info msg="Upload sending done 02d98e5d-d09c-42fb-b3dc-f94aec4722ef (chu>
  Dec 13 13:59:54 ip-10-1-1-5 systemd[1]: /etc/systemd/system/nginx-agent.service:23: PIDFile= references a path below legacy directory /var/run/, updating>

NIMの ``Instances`` を再度開くと、追加したインスタンスが表示されます

   .. image:: ./media/nim-instances2.png
      :width: 400
