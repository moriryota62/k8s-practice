前： [Pod-resources](Pod-resources.md)  


---

# Pod設定 livenessProbe/readinessProbe
K8sが意識するPodの状態はコンテナのメインプロセスがあるか/ないかだけです。（Dockerの場合の話、コンテナランタイムによって違うかも）　これだとたとえばメインプロセスがハングしても異常として検知されず、セルフ・ヒーリングが行われません。また、プロセスの起動後に初期化処理が必要なコンテナであっても、Pod起動（プロセス起動）と同時に通信を受けることとなり、通信が失敗する可能性もあります。これらを回避するため、コンテナのヘルスチェックを追加で設定することができます。ヘルスチェックはコンテナの``正常性確認（livenessProbe）``と``待受状態確認（readinessProbe）``の2種類があります。（K8s1.16以上ではさらに追加でstartupProbeもあるがこのpracticeでは触れない）　ヘルスチェックはPodに含まれるコンテナ単位で設定できます。

正常性確認（livenessProbe）はコンテナの稼働状態を確認するヘルスチェックです。このヘルスチェックに失敗した場合、コンテナの再作成が行われます。Podデプロイ後に威力を発揮するヘルスチェックです。  

待受状態確認（readinessProbe）もコンテナの稼働状態を確認するヘルスチェックです。ただし、こちらのヘルスチェックに失敗してもコンテナの再作成は行われません。代わりに、ヘルスチェックに失敗したコンテナを含むPodをServiceのバランシング対象Podから一時的に除外します。ヘルスチェックに成功するとServiceのバランシング対象Podにまた追加される。Podデプロイ時に威力を発揮するヘルスチェックです。

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、probeについては[公式ドキュメント](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)を参考にしてください。
   - Deploymet
     - 名前は``probe``
     - replicas: ``1``
     - labelはすべて``app: probe``
     - Pod
       - メインコンテナ
         - 名前は``probe``
         - イメージは``nginx:1.12``
         - readinessProbe
           - ``httpGet``のヘルスチェックを行う
           - pathは``/``
           - portは``80``
           - ヘルスチェックの詳細設定はとくに行わない
   - Service
     - 名前は``probe-svc``
     - Type: ``LoadBalancer``
     - Port: ``80``
     - 上記Deployment:probeで展開したPodを対象
2. Podの詳細を表示し、readinessProbeが設定されていることを確認してください。
3. Podリソースのオブジェクト一覧を表示し、上記マニフェストでデプロイしたPodの``READYが1/1``になっていることを確認する。また、PodのIPアドレスも確認しておく。
4. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。（デプロイしてから接続できるまで2分、最長でも5分ほど時間がかかる）　「Welcome to nginx!」と``表示される``こと
5. Service:probe-svcの詳細を表示し、EndpointsにPodのIPアドレスが``設定されている``ことを確認。
6. Podに以下の追加コマンドを発行する
   ``` sh
   mv /usr/share/nginx/html/index.html /usr/share/nginx/html/hoge.html
   ```
7. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの``READYが0/1``になっていることを確認する。
8. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。「Welcome to nginx!」が``表示されない``こと。(もし表示されてしまう場合、どこかにキャッシュされているかもしれない。別の端末など異なる経路で接続してみる。)
9.  Service:probe-svcの詳細を表示し、EndpointsにPodのIPアドレスが``設定されていない``ことを確認。
10. Podに以下の追加コマンドを発行する
   ``` sh
   mv /usr/share/nginx/html/hoge.html /usr/share/nginx/html/index.html
   ```
11. しばらく（10秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの``READYが1/1``になっていることを確認する。
12. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。「Welcome to nginx!」が``表示される``こと。
13. Service:probe-svcの詳細を表示し、EndpointsにPodのIPアドレスが``設定されている``ことを確認。
14. Deployment:probeを削除（Serviceも消して良いけどこのあとデプロイ待ちとかEXTERNAL-IP確認面倒なので残しておいた方がよい）
15. Deployment:probeのマニフェストを以下のように編集してデプロイする。
    - readinessProbeのブロックを丸コピペし、readinessProbeを``livenessProbe``に書き換える
    - readinessProbeのブロックを``コメントアウト``する
16. Podの詳細を表示し、livenessProbeが設定されていることを確認する。
17. Podリソースのオブジェクト一覧を確認し、Podの``RESTARTS``の値を確認する。（たぶん0）
18. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。「Welcome to nginx!」が``表示される``こと。
19. Podに以下の追加コマンドを発行する。
   ``` sh
   rm /usr/share/nginx/html/index.html
   ```
18. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの``RESTARTSが増えている``ことを確認する。
19. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。「Welcome to nginx!」が``表示される``こと。
20. Deployment:probeを削除（ここではServiceは絶対消さない。あとの操作の切り分けが面倒になる）
21. Deployment:probeのマニフェストを以下のように編集してデプロイする。
    - メインコンテナのcommandを``['/bin/sh','-c','sleep 60;nginx -g "daemon off;"']``にする（nginxのプロセスがPod起動してから60秒後に起動する）
    - コメントアウトしていたreadinessProbeのブロックの``コメントアウトを解除``する
    - livenessProbeのヘルスチェックの詳細設定に``initialDelaySeconds: 120``を設定
22. （デプロイしてから60秒以内に実施）Podリソースのオブジェクト一覧を確認し、上記マニフェストでデプロイしたPodの``READYが0/1``になっていることを確認する。
23. （デプロイしてから60秒以内に実施）作業端末のwebブラウザからService経由で接続する。「Welcome to nginx!」が``表示されない``こと。
24. （デプロイしてから60秒以降に実施）Podリソースのオブジェクト一覧を確認し、上記マニフェストでデプロイしたPodの``READYが1/1``になっていることを確認する。
25. （デプロイしてから60秒以降に実施）作業端末のwebブラウザからService経由で接続する。「Welcome to nginx!」が``表示される``こと。
26. Podに以下の追加コマンドを発行する。
   ``` sh
   rm /usr/share/nginx/html/index.html
   ```
27. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの``RESTARTSが増えている``ことを確認する。
28. セルフ・ヒーリングされてから60秒ほどたったあとにPodの``READYが1/1``となり作業端末のwebブラウザからService経由で接続すると「Welcome to nginx!」が``表示される``こと。
29. Deployment:probeおよびService:probe-svcを削除

---

次： [Pod-initContainer](Pod-initContainer.md)  
