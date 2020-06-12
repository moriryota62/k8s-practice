前： -

---

# Dynamic Volume Provisioning
通常のオンプレやVMのインフラ基盤で外部ボリュームをマウントしたい場合、あらかじめボリュームを作成しマウント対象のサーバに割り当てる必要がありました。K8sの場合、これらの作業をK8sに任せることができます。このPod作成時にPodが必要となるボリュームを作成しPodに割り当てる機能をDynamic Volume Provisioning（以下、DVP）と言います。なお、DVPが使えるのはAWSなど対応したクラウドプロバイダでK8sが動いている場合のみです。

1. 以下を満たすPersistentVolumeClaim（以下、PVC）およびDeploymentをデプロイしてください。PVCは[公式ドキュメント](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)を参考にしてください。volumeプラグインは[公式ドキュメント](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)を参考にしてください。
   - PVC
     - 名前は``test-pvc``
     - storageClassNameは``指定しない``（デフォルトのStorageClassを使用する）
     - accessModesは``ReadWriteOnce``
     - ストレージ容量は``1Gi``
   - Deployment
     - 名前は``dvp``
     - replicas: ``1``
     - labelはすべて``app: dvp``
     - Pod
       - containerは一つでイメージは``nginx:1.12``
       - volumeプラグインで上記PVC:``test-pvc``を指定
       - 上記で定義したボリュームをコンテナの/usr/share/nginx/html/にマウント
2. PVCリソースのオブジェクト一覧を確認してください。
3. PVリソースのオブジェクト一覧を確認してください。
4. AWSのマネジメントコンソールなどでEBSボリュームを確認し「kubernetes-dynamic-pvc-」から始まる名前のボリュームが作成されていることを確認してください。
5. デプロイしたPod内のコンテナに対して追加コマンドを発行し以下内容の/usr/share/nginx/html/index.htmlを作成してください。
  ```
  Dynamic Volume Provisoning ha tottemo benri na kinou desu
  ```
6. Podを削除してください。
7. 再作成されたPodに含まれるコンテナ内を確認し、/usr/share/nginx/html/index.htmlの内容が残っていることを確認してください。
8. Deployment:dvpを削除してください。（PVCは削除しない！）
9. PVC、PVおよびAWSのEBSを確認し、オブジェクトが``消えていない``ことを確認してください。
10. PVCを削除してください。
11. PVC、PVおよびAWSのEBSを確認し、オブジェクトが``消えている``ことを確認してください。

---

次： [Service-LB](Service-LB.md)  
