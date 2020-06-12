前： -

---

# metrics server
metrics serverはPodやワーカーノードのHWリソース量(CPU/メモリ)を取得するアドオン機能です。metrics serverはK8s内にPodとして起動させます。metrics serverを導入すると「kubectl top」コマンドが使用できるようになります。Pod数を自動で増減させるHorizontal Pod Autoscalerを使用するにはmetrics serverが必要となってきます。（なお、EKSの場合は利用者でデプロイが必要だがAKSやGKEではマネージド・サービスの中でデプロイされている）

1. metrics serverをデプロイしてください。[公式ドキュメント](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/#metrics-server)を参考にしてください。
2. metrics serverが起動してから情報収集まで1分ほど時間がかかる。時間が経ってもkubectl topで結果がうまく出力されず、metrics serverのログに``unable to fetch node metrics for node ~``や``"unable to fetch node metrics for pod ~"``が出力されている場合はデバッグが必要になる。インターネットを調べてデバッグしてください。
3. 以下コマンドを実行しmetrics serverでメトリクスの取得ができていることを確認してください。
   ``` sh
   kubectl top node
   kubectl top pod -n kube-system
   ```

---

次： [HorizontalPodAutoscaler](HorizontalPodAutoscaler.md)  
