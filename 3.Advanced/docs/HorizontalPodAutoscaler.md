前： [MetricsServer](MetricsServer.md)  

---

# Horizontal Pod Autoscaler
Horizontal Pod Autoscaler（以下、HPA）はPodのHWリソース使用量を基にPodコントローラのreplica数を自動で増減させるリソースです。HPAを動かすにはmetrics serverが必要です。（EKSの場合は利用者でデプロイ、AKS/GKEの場合はマネージドサービスでデプロイ済）　また、Pod（コンテナ）にresources.requestsを指定していることも条件になります。HPAは目標の負荷状態状態を宣言します。この目標値に近づくようにPodの数量を変化させます。ターゲットとなるPod群の実際に消費しているHWリソース量の平均値をmetrics serverを使って計算します。この計算した値を目標値で割り、どれくらいの数量のPodを追加/削除するか決定します。計算式で表すと以下の通りとなります。
```
desiredReplicas = ceil[currentReplicas * ( currentMetricValue / desiredMetricValue )]
```

1. 以下を満たすマニフェストを作成しデプロイしてください。HPAについては[公式ドキュメント](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)を参考にしてください。
   - Deployment
     - replica: 1
     - image:nginx: 1.12
     - resources.limits.cpu: 100m
   - Service
     - 上記Deploymentをtype:ClusterIPで公開
     - Portは80
   - HPA
     - 上記Deploymentを対象にする
     - Pod数を最小1 ~ 最大5の範囲とする
     - CPU使用率80%を目標値とする
2. 作成したオブジェクトを確認してください。Podが1つであること。
3. 以下を満たすマニフェストを作成しデプロイしてください。
   - Deployment
     - image: httpd
4. 上記作成したPodに以下の様な追加コマンドを発行してください。(宛先は作成したService名にする。最後の/を忘れずに。-nと-cの値はお好みで)
   ``` sh
   ab -n 1000000 -c 100 http://hpa/
   ```
5. 別ターミナルを開き、「kubectl top pod」と「kubectl get pod」をwatch等で監視する。しばらくするとPodの数が増えることを確認する。Podが5つに自動拡張されることを確認する。すべてのPodの負荷が80%(80m)を超えていてもPodが5つ以上増えないことも確認する。
6. 作成したオブジェクトをすべて削除する。
---

次： [ClusterAutoscaller](ClusterAutoscaller.md)  
