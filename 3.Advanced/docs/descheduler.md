前： [ClusterAutoscaller](ClusterAutoscaller.md)  

---

# descheduler
deschedulerはクラスタを監視し、ルールに基づきPodを削除する機能です。この機能は削除されたPodがセルフ・ヒーリングで再配置されることを利用した機能です。deschedulerはK8s内にコントローラの役割を担うPodをデプロイして機能します。  
Podの起動先ノードを決めることをK8sではスケジュールと言います。このスケジュールはマスターコンポーネントであるkube-schedulerにより行われます。スケジュールはPodが起動するタイミングでしか行われません。たとえばノード障害などが発生し、あるノードで動いていたPodが別のノードに退避されたとします。その後、ノードを再度クラスタに参加させてもそのノードには何もPodが起動されません。（もし、ノードが参加した時点でPending状態のPodがあれば起動されます。）　PodのスケジュールはPod起動時にしか行われないため、このようにあとからノードを追加しても思うように負荷が分散されていない状況が起こります。これを防止する機能がdeschedulerです。deschedulerは定期的にクラスタを監視し、ルールに基づいてPodを削除します。Podを削除するとセルフ・ヒーリングされます。セルフヒーリング時にマスターコンポーネントのkube-shedulerによりノードの負荷状況などを考慮して再スケジュールが行われます。

1. ワーカーノード2台の状態から作業する。
2. 以下を満たすマニフェストを作成しデプロイしてください。
   - Deployment
     - イメージは何でもよい
     - replicas:10
3. 各Podが起動しているノードを一覧で表示し、2つのノードに分散配置されていることを確認してください。
4. どちらかのノードを停止してください。（停止の方法は何でも良いです）
5. ノード停止から1分ほどしてからPodの起動ノードを確認してください。1つのノードに片寄って配置されていることを確認してください。
6. ワーカーノードを1台クラスタに再参加さしてください。（ワーカーをAutoScalingGroupで構成している場合しばらく待っていれば自動で追加してくれると思います。）
7. ワーカー2台の状態に戻ってもPodの起動ノードが片寄った状態のままで有ることを確認してください。
8. 以下コマンドで必要なマニフェストが含まれるgitリポジトリをクローンしてください。
   ``` sh
   git clone https://github.com/kubernetes-sigs/descheduler.git
   ```
9.  以下コマンドで必要なマニフェストをapply
   ``` sh
   kubectl create -f kubernetes/rbac.yaml
   kubectl create -f kubernetes/configmap.yaml
   kubectl create -f kubernetes/cronjob.yaml
   ```
11. 2分ほどたってからPodの起動ノードを確認し、2ノードに分散されていることを確認してください。
12. デプロイしたDeploymentやdescheduler関連のオブジェクトを削除してください。

---

次： [IngressController](IngressController.md)  
