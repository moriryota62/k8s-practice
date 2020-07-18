前： [Pod-initContainer](Pod-initContainer.md)  

---

# Pod設定 nodeSelector
Podはマスターコンポーネントのkube-schedulerによって起動するワーカーノードが決められます。このPodをワーカーノードに割り当てることをK8sでは``スケジュール``といいます。Podにスケジュール関連のパラメータが設定されていない場合、kube-schedulerはノードの負荷状況などを考慮してスケジュールします。しかし、どうしても起動先ノードを指定したい場合もあります。（たとえば、GPUを使いたいPodの場合はGPU搭載ノードで起動したいなど）　その様な場合にPodのスケジュール先を指定するもっとも簡単な方法がnodeSelectorです。

1. クラスタの参加ノードを確認し、ワーカーノードが2台以上の状態にしてください。
2. ノードに付与されたラベルを表示してください。
3. どれか一台のノードに``node-spec:monster``のラベルを付与してください。
4. 他のノードには``node-spec:normal``のラベルを付与してください。
5. key:node-specをカラムに追加してノードの一覧で表示してください。一台のノードにmonsterのvalueがあることを確認してください。
6. 以下を満たすマニフェストを作成しデプロイしてください。なお、nodeSelectorについては[公式ドキュメント](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)を参考にしてください。
   - Deploymet
     - 名前は``node-selector``
     - replicas: ``1``
     - labelはすべて``app: node-selector``
     - Pod
       - メインコンテナ
         - 名前は``nginx``
         - イメージは``nginx:1.12``
       - nodeSelector
         - ラベル``node-spec``にvalue``monster``が付与されたノードを指定
7. Podリソースの一覧をwideで表示し、Podが``node-spec:monster``のラベルを付与したノードで起動していることを確認してください。
8. Deployment:node-selectorのPodレプリカ数を10に変更してください。新たに作成されたPodもすべて``node-spec:monster``のラベルを付与したノードで起動していることを確認してください。
9. Deployment:node-selectorのnodeSelectorをkey``node-spec``にvalue``normal``が付与されたノード指定に変更してください。
10. Podのローリングアップデートが実行され、すべてのPodが``node-spec:normal``のラベルを付与したノードで起動していることを確認してください。
11. Deployment:node-selectorをいったん削除してください。
12. Deployment:node-selectorのnodeSelectorをkey``node-spec``にvalue``super``が付与されたノード指定に変更しデプロイしてください。
13. すべてのPodのSTATUSが``Pending``となることを確認してください。
14. Deployment:node-selectorを削除してください。
15. ノードに付与されたkey``node-spec``のラベルを削除してください。

---

次： [postStart/preStop](Pod-lifecycle.md)