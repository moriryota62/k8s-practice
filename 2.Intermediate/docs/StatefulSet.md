前： [DaemonSet](DaemonSet.md)  

---

# StatefulSet
StatefulSetはDeployment（ReplicaSet）と同じく指定した数のPodを常に起動するようにするリソースです。違いとしては以下の3つです。わかりやすい1つ目と2つ目の動作をまず確認します。3つ目は長くなるので別データであつかいます。
- 起動するPod名に連番が付与される（Deploymentはランダム文字列）
- 名前の連番通りにPodを1つずつ起動し、前のPodが起動完了してから次のPodを起動する。（Deploymentは起動順を気にしない）
- 各PodごとにPVを割り当てるvolumeClaimTemplateが使える（Deploymentでは使えない）
StatefulSetはDBなどのワークロードを想定したリソースです。たとえば3ノードのDBだとMaster:1,Slave:2の構成などにすると思います。この際、Masterをまず起動し、次いでSlaveを起動します。このような順番を意識した起動はDeploymentではできないため、StatefulSetを使います。またHeadless Serviceを使った特定Podへのアクセスを行うこともあります。

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、StatefulSetについては[公式ドキュメント](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)を参考にしてください。
   - StatefulSet
     - 名前は``stateful``
     - labelはすべて``app: stateful``
     - serviceNameは``stateful-svc``
     - replicasは ``3``
     - Pod
       - メインコンテナ1
         - 名前は``nginx``
         - イメージは``nginx:1.12``
       - メインコンテナ2
         - 名前は``curl``
         - イメージは``appropriate/curl``
         - commandは``['/bin/sh','-c','sleep 3600']``
   - Service
     - 名前は``stateful-svc``
     - typeは``ClusterIP``（``明示的に指定する``）
     - clusterIPに``None``
     - Portは``80``
     - selectorは``app: stateful``
2. StatefulSetリソースのオブジェクト一覧を表示し、``stateful``があることを確認してください。
3. Podリソースのオブジェクト一覧を表示し、``stateful-0,1,2``の３つのPodがあることを確認してください。また起動時間から起動が順番に行われたことを確認してください。
4. Serviceリソースのオブジェクト一覧を表示し、``stateful-svc``があること。また、``Headless Service``であることを確認してください。（Headless ServiceはCluster-IPがNoneなServiceです。）
5. Pod:``stateful-0``に含まれるcurlコンテナに以下の追加コマンドを発行してください。
   ``` sh
   nslookup stateful-svc
   curl stateful-0.stateful-svc.default.svc.cluster.local
   curl stateful-1.stateful-svc.default.svc.cluster.local
   curl stateful-2.stateful-svc.default.svc.cluster.local
   ```
6. Pod:``stateful-1``のIPアドレスを確認してからPod:``stateful-1``を削除してください。
7. Pod:``stateful-1``がセルフ・ヒーリングされていることを確認してください。また、IPアドレスが変わっていることを確認してください。
8. Pod:``stateful-0``に含まれるcurlコンテナに以下の追加コマンドを発行してください。
   ``` sh
   nslookup stateful-svc
   curl stateful-1.stateful-svc.default.svc.cluster.local
   ```
9. StatefulSet:statefulおよびService:stateful-svcを削除してください。

上記の様に、Headless Serviceを使うことでPod IPアドレスが変わっても任意のPodにアクセスすることができます。単純に特定PodにアクセスするだけならPodのIPアドレスでも可能ですが、StatefulSetで展開しているPodでもセルフヒーリングで再作成されるとIPアドレスは変わってしまいます。Headless Serviceであればセルフヒーリング後も自動でPodを検出するため、宛先の指定を変える必要がありません。

とはいえ、Headless Serviceは少しイレギュラーなServiceです。replicas:2以上のStatefulSet以外で使うことは稀だと思います。replicas:1のStatefulSetやDeploymentの場合は通常のServiceを使うことがほとんどだと思います。

---

次： [StatefulSet-volumeClaimTemplate](StatefulSet-volumeClaimTemplate.md)  
