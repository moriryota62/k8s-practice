前： [kubectl](kubectl.md)  

---

# Deployment
Deploymentはもっとも基本的なPod Controllerリソースです。通常、PodをデプロイするときはDeploymentを使用します。Deploymentを作成するとReplicaSetというPod Controllerも作成されます。ReplicaSetはPodを常に定義した状態に保とうとするリソースです。

1. 以下を満たすDeploymentのマニフェストを作成しapplyしてください。Deploymentのマニフェストは[公式ドキュメント](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment)を参考にしてください。
   - Deploymentの名前は``nginx``
   - replicaは``1``
   - Pod
     - コンテナは``test``という名前のもの1つ
     - imageは``nginx:1.12``を使用
     - ラベルは``app: test``を付与する
2. Deployment,ReplicaSet,Podそれぞれのオブジェクト一覧を表示してください。
3. 作成したnginx-XXXXXXという名前のPodを削除してください。(Deploymentは消しちゃだめ！)　再度Podを確認し、さきほどとは違う名前のPodが作成されていることを確認してください。
4. マニフェストを修正しDeploymentのreplicaを``2``にしてください。
5. 再度Podを確認しPodが増えていることを確認してください。
6. (``1-03.Service-Cluster``を実施しない場合はDeploymnet:nginxを削除してください。)

---

次： [Service-ClusterIP](Service-ClusterIP.md)