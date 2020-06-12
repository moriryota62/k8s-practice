前： [StatefulSet](StatefulSet.md)  

---

# StatefulSet volumeClaimTemplate

## PVC指定の場合
volumeClaimTemplateはPodごとにPVCおよびPVを作成する、StatefulSetで使える機能です。この機能を理解するため、Deploymentでのボリュームプロビジョニングの特徴をおさらいします。

※ ワーカーノードを2台以上にしてください。

1. 以下を満たすマニフェストを作成しデプロイしてください。
   - Deployment
     - 名前は``dep-dvp``
     - replicas: ``2``
     - labelはすべて``app: dep-dvp``
     - Pod
       - containerは一つでイメージは``nginx:1.12``
       - volumeプラグインでPVC:``dep-dvp-pvc``を指定
       - 上記で定義したボリュームをコンテナの/usr/share/nginx/html/にマウント
       - postStartで次のコマンドを実行``['/bin/sh','-c','echo $HOSTNAME >> /usr/share/nginx/html/index.html']``
   - PVC
     - 名前は``dep-dvp-pvc``
     - storageClassNameは``指定しない``（デフォルトのStorageClassを使用する）
     - accessModesは``ReadWriteOnce``
     - ストレージ容量は``1Gi``
   - Service
     - 名前は``dep-dvp-svc``
     - タイプは指定なし（ClusterIP）
     - Portは80
     - selectorは``app: dep-dvp``
2. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ``1つ``作成されていることを確認してください。
3. Podリソースのオブジェクト一覧を表示し2つのPodの名前を確認してください。
4. curlが実行できるコンテナを含むPodをデプロイし、Service``dep-dvp-svc``にcurlしてください。``2つ``のPod名が表示されることを確認してください。
5. Deployment:dep-dvpのreplicaを``5``に拡張してください。
6. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ``1つのまま``であることを確認してください。
7. Podリソースのオブジェクト一覧を表示し5つのPodの名前を確認してください。また、すべてのPodが同じノードで起動していることも確認してください。
8. curlが実行できるコンテナを含むPodからService``dep-dvp-svc``にcurlしてください。``5つ``のPod名が表示されることを確認してください。
9.  Deploymet:dep-dvp、PVC:dep-dvp-pvc、Service:dev-dvp-svcを削除してください。

ここまでがDeploymentでのボリュームプロビジョニングのおさらいです。注目するポイントとしては展開したPodすべてで同じボリュームを共有する点です。そのため、PVCおよびPVは1つしか作られません。ボリュームの中身もすべてのPodで共有します。なので、たとえばWEBサーバなどのワークロードでセッション情報をPod間で共有したいなどと言った場合にはこの構成が有効です。一方で、DBなどのワークロードで各Podが専用のボリュームを確保したい場合には適しません。また、今回はPVCでstorageClassNameを指定しなかったためデフォルトのStorageClassであるEBSのtype:gp2(EKSの場合)で実際のボリュームは作られています。EBSは単一のEC2インスタンスにしかボリュームを提供できません。そのため、replica数が増えても単一のワーカーノード（EC2）にしかPodがスケジュールできません。replica数を2以上にするDeploymentでDVPするときは、EBSではなくEFSなどRead/Write Anyできるボリュームが使えるStorageClassを使用した方が良いです。  

## volumeClaimTemplateの場合
つぎに、StatefulSetでvolumeClaimTemplateを使用した場合の挙動を確認します。

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、volumeClaimTemplateについては[公式ドキュメント](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#components)を参考にしてください。
   - StatefulSet
     - 名前は``ss-vct``
     - labelはすべて``app: ss-vct``
     - serviceNameは``ss-vct-svc``
     - replicasは``2``
     - Pod
       - containerは一つでイメージは``nginx:1.12``
       - ``index``ボリュームをコンテナの/usr/share/nginx/html/にマウント
       - postStartで次のコマンドを実行``['/bin/sh','-c','echo $HOSTNAME >> /usr/share/nginx/html/index.html']``
     - volumeClaimTemplates
       - 名前は``index``
       - storageClassNameは``指定しない``（デフォルトのStorageClassを使用する）
       - accessModesは``ReadWriteOnce``
       - ストレージ容量は``1Gi``
   - Service
     - 名前は``ss-vct-svc``
     - typeは``ClusterIP``（``明示的に指定する``）
     - clusterIPに``None``
     - Portは``80``
     - selectorは``app: ss-vct``
2. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ``2つ``作成されていることを確認してください。
3. Podリソースのオブジェクト一覧を表示し2つのPodの名前を確認してください。
4. curlが実行できるコンテナを含むPodをデプロイし、以下にcurlし``それぞれのPod名``が表示されることを確認してください。
   - ss-vct-0.ss-vct-svc.default.svc.cluster.local
   - ss-vct-1.ss-vct-svc.default.svc.cluster.local
5. StatefulSet:ss-vctのreplicaを``5``に拡張してください。
6. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ``5つ``作成されていることを確認してください。
7. Podリソースのオブジェクト一覧を表示し5つのPodの名前を確認してください。また、Podの起動ノードが分散していることも確認してください。
8. curlが実行できるコンテナを含むPodから以下にcurlし``それぞれのPod名``が表示されることを確認してください。
   - ss-vct-0.ss-vct-svc.default.svc.cluster.local
   - ss-vct-1.ss-vct-svc.default.svc.cluster.local
   - ss-vct-2.ss-vct-svc.default.svc.cluster.local
   - ss-vct-3.ss-vct-svc.default.svc.cluster.local
   - ss-vct-4.ss-vct-svc.default.svc.cluster.local
9. StatefulSet:ss-vctおよびService:ss-vct-svcを削除してください。
10. PVCおよびPVリソースのオブジェクト一覧を表示し、5つ``まだ残っている``ことを確認してください。
11. PVC:index-ss-vct-0~4を削除してください。

以上がvolumeClaimTemplateを使用したボリュームプロビジョニングです。Deploymentの時とは違い、各Pod用にPVCおよびPVが作成されました。また、StorageClassはデフォルトのEBSですが、Podのスケジュール先も分散されました。この様にPodごとに専用のボリュームを確保したい時にvolumeClaimTemplateは有効です。また、Podがセルフ・ヒーリングで再作成された場合はそのPodに対応したボリュームが引き続き使用されるます。なお、volumeClaimTemplateで作成されたPVCおよびPVはStatefulSetを削除しても``消えません``。（データを残すためにあえてこういう仕様になっていると思われます。）　この特性を利用してreplica1でもあえてStatefluSetを使用することもありますが、ボリュームが不要になった時は手動で消すのを忘れないようにしましょう。

なお、StatefulSetでもDeploymetと同じようにPVCを指定すれば1つのボリュームを複数Podで共有することもできます。

---

次： [ConfigMap(env)](ConfigMap-env.md)  
