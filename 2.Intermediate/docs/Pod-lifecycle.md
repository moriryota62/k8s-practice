前： [Pod-nodeSelector](Pod-nodeSelector.md)  

---

# Pod設定 postStart/preStop
Pod内のコンテナ起動後とコンテナ停止前に任意のコマンドを実行できます。その設定がlifecycleのpostStartとpreStopです。とくにpreStopはコンテナ(Pod)をいつでも安全な停止ができるようにするため重要です。Podを手動で停止(kubectl delete)すると、SIGTARMが送られ30秒の猶予期間後にSIGKILLでプロセス強制終了となります。SIGTARMやSIGKILLで終了したくないコンテナにはpreStopでプロセスの終了コマンドを実行しましょう。また、postStartはコンテナ起動時に実行されるコマンドですが、プロセス起動前の実行は保証されません。そのため、プロセス起動前に行いたい処理はinitContainerやコンテナ起動コマンドに埋め込むなどしてください。

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、lifecycleについては[公式ドキュメント](https://kubernetes.io/ja/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/)を参考にしてください。
  - Deployment
     - 名前は``lifecycle``
     - replicas: ``1``
     - labelはすべて``app: lifecycle``
     - Pod
       - メインコンテナ
         - 名前は``lifecycle-container``
         - イメージは``nginx:1.12``
         - volume:index-htmlを``/usr/share/nginx/html``にマウント
         - lifecycleのpostStartで``['/bin/sh','-c','date >> /usr/share/nginx/html/index.html']``を実行
       - volume
         - 名前は``index-html``
         - volumeプラグインはPVC:``test-pvc``を指定
   - PVC
     - 名前は``test-pvc``
     - storageClassNameは``指定しない``（デフォルトのStorageClassを使用する）
     - accessModesは``ReadWriteOnce``
     - ストレージ容量は``1Gi``
   - Service
     - 名前は``lifecycle-svc``
     - タイプは指定なし（ClusterIP） 
     - Port: ``80``
     - 上記Deployment:``lifecycle``で展開したPodを対象
   - Deployment(動作確認用)
     - 名前は``curl``
     - replicas: ``1``
     - labelはすべて``app: curl``
     - Pod
       - メインコンテナ
         - 名前は``curl``
         - イメージは``appropriate/curl``
         - commandは``['/bin/sh','-c','sleep 3600']``
2. Pod:``lifecycle``を何度か手動で削除しセルフ・ヒーリングさせる。（Deploymentは消さない）
3. Pod:``curl``から``lifecycle-svc``に対しcurlする。Podを作成した時刻が複数行出力されることを確認する。
4. Deployment:``lifecycle``のマニフェストを以下にように追記して再デプロイする。
   - Deployment
     - 名前はlifecycle
     - Pod
       - メインコンテナ
         - 名前:lifecycle
         - lifecycleの``preStop``で``['/bin/sh','-c','rm /usr/share/nginx/html/index.html']``を追加
5. Pod:``lifecycle``を何度か手動で削除しセルフ・ヒーリングさせる。（Deploymentは消さない）
6. Pod:``curl``から``lifecycle-svc``に対しcurlする。Podを作成した時刻が``一行のみ``出力されることを確認する。
7. 作成したDeployment,Service,PVCを削除する。

---

次： [Namespace](Namespace.md)  