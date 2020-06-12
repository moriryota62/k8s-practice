前： [Service-LB](Service-LB.md)  

---

# Pod設定 resources
Podはとくに指定がない場合、ワーカーノードのHWリソース（CPU/メモリ）を使いたいだけ使います。これだと、各Podが際限なくHWリソースを要求した結果、ワーカーノードのHWリソースが足りなくなる場合もあります。このような事態を防ぐため、PodにHWリソースの``要求容量（requests）``と``上限容量（limits）``を設定できます。この設定がresourcesです。resourcesはPodに含まれる各コンテナ単位で設定できます。Podのresourcesは含まれるコンテナのresourcesを合算した値です。  

要求容量（requests）はPod起動時に最低限確保が保証されるHWリソース容量です。Podはこの要求容量が確保できるワーカーノードで起動します。もし、どのワーカーノードでも要求容量が確保できなかった場合、Podは実行されず実行待ち状態（Pending）となります。  

上限容量（limits）はPodが使用できるHWリソース容量の上限です。上限までHWリソースを使用できるが確保は保証されません。つまり、上限容量はワーカーノード上でオーバーコミットされる可能性があります。オーバーコミットを許容しない場合は要求容量と上限容量を同じにすればよいです。（もしくは上限容量だけ設定すると自動で要求容量も同じ値で設定されます。）

1. 以下を満たすDeploymentのマニフェストを作成しデプロイしてください。なお、resourcesの設定については[公式ドキュメント](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)を参考にしてください。
   - Deployment
     - 名前は``resources``
     - replicas: ``1``
     - labelはすべて``app: resources``
     - Pod
       - コンテナの名前は``cpu-tukau-man``
       - イメージは``busybox``
       - commandは``['/bin/sh', '-c', 'sleep 3600']``
       - 以下のHWリソース容量を指定
         - CPU要求``500m``
         - CPU上限``600m``
2. デプロイしたPodの詳細を表示し、コンテナにrequestsとlimitsが設定されていることを確認してください。
3. 作成したDeploymentを削除してください。
4. 以下を満たすようにマニフェストを改良してデプロイしてください。
   - Podに以下containerを追加
     - 名前は``memory-tukau-man``
     - イメージは``busybox``
     - commandは``['/bin/sh', '-c', 'sleep 3600']``
     - 以下のHWリソース容量を指定
       - memory上限``100Mi``
5. デプロイしたPodの詳細を表示し、各コンテナにresourcesが設定さていることを確認してください。また、memory-tukau-manにはmemoryの``requetsとlimitsが同じ値``で設定されていることを確認してください。
6. Deploymentのreplica数を10に拡張してください。
7. Podリソースのオブジェクト一覧を表示し、STATUS:PendingのPodがあること。（ない場合、Deploymentのreplica数をより大きな値にする）
8. STATUS:PendingのPodの詳細を表示しeventを確認してください。以下の様なメッセージが出力されているはずです。（これはワーカー2台の環境で要求した量のCPUを確保できるノードが見つからなかった場合のメッセージ）
   ```
   0/2 nodes are available: 2 Insufficient cpu.
   ```
9. Deployment:resourcesを削除してください。

---

次： [Pod-Probe](Pod-Probe.md)  
