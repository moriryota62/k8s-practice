前： [LimitRange](LimitRange.md)  

---

# ResourceQuota
ResourcesQuotaはNamespaceに対しHWリソース上限や作成できるK8sのオブジェクト数に制限を設けるリソースです。Namespaceに属します。LimitRangeとは違い、Namespaceに対する制限です。

## HWリソース制限
まずはHWリソース量の制限について確認します。

※ ワーカーノードのHWリソースを十分な量(CPU:1コア、メモリ:1G)確保しておいてください。

1. 以下を満たすマニフェストを作成しデプロイしてください。ResourceQuotaリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/policy/resource-quotas/)を参考にしてください。
   - Deployment
     - イメージは何でもよい
     - replicas: 1
     - resources.limitsに以下を設定
       - cpu: 200m
       - memory: 100Mi
     - reqources.requestsに以下を設定
       - cpu: 100m
       - memory: 50Mi
   - ResourceQuota
     - cpuの制限を1core
     - memoryの制限を1G
2. デプロイしたResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示される。（利用量はrequestsで計算される）
3. Deploymentのreplica数を10に拡張してください。
4. ResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示される。（利用量はrequestsで計算される）
5. Deploymentのreplica数を11に拡張してください。
6. Podのオブジェクト一覧を表示してください。Podの数が10のままであること。
7. ReplicaSetの詳細を表示し、以下のようにCPUの利用量がResourceQuotaで指定した範囲に違反したメッセージが表示されること。
   ```
   Error creating: pods "quota-5c4f499fb8-nxfqb" is forbidden: exceeded quota: quota, requested: cpu=100m, used: cpu=1, limited: cpu=1
   ```
8. DeploymentとResourceQuotaを削除してください。

以上のようにNamespaceに対してHWリソース量の制限を設けることができます。ちなみに、ResourceQuotaで制限を設けている状態でrequestsを指定しないPodを起動しようとしても起動できません。

## オブジェクト数制限
次にK8sリソースのオブジェクト数の制限について確認します。

1. 以下を満たすマニフェストを作成しデプロイしてください。ResourceQuotaリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/policy/resource-quotas/)を参考にしてください。
   - Deployment
     - イメージは何でもよい
     - replicas: 1
   - ResourceQuota
     - Podの制限を5
2. デプロイしたResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示される。
3. Deploymentのreplica数を5に拡張してください。
4. ResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示される。（利用量はrequestsで計算される）
5. Deploymentのreplica数を6に拡張してください。
6. Podのオブジェクト一覧を表示してください。Podの数が5のままであること。
7. ReplicaSetの詳細を表示し、以下のようにCPUの利用量がResourceQuotaで指定した範囲に違反したメッセージが表示されること。
   ```
   replicaset-controller  Error creating: pods "quota-68847df66-bplgc" is forbidden: exceeded quota: quota, requested: count/pods=1, used: count/pods=5, limited: count/pods=5
   ```
8. DeploymentとResourceQuotaを削除してください。

以上のようにNamespace内のK8sオブジェクト数に制限を設けることができます。

Namespace単位で制限をかけられるのでLimitRangeよりも手っ取り早くリソース制限をかけることができます。しかし、各オブジェクト単位のリソース量制限はできため、その場合はLimitRangeを使います。このように、ResourceQuotaはたとえば1つのK8sクラスタを複数のチームで共有利用する場合などに使える機能です。

---

次： [RBAC](RBAC.md)  