前： [NetworkPolicy](NetworkPolicy.md)  

---

# StorageClass
以前、[DynamicVolumeProvisioning](../../2.Intermediate/docs/DynamicVolumeProvisioning.md)に触れた時にはデフォルトのStorageClassを使用してDVPを行いました。StorageClassはストレージの種類を定義したリソースです。EKSの場合はEBS(gp2)のStorageClassがデフォルトで定義されています。EBS(gp2)以外のディスクタイプを使用してDVPを行うには追加でStorageClassを定義し、PVCでそのStorageClassを指定すれば良いです。なお、StorageClassとして定義可能なストレージサービスには制限があります。Provisionerというストレージサービスと連携をとるための機能が提供されているもののみになります。K8sにはデフォルトでAWS EBSやAzure Diskなど代表的なストレージサービスのProvisionerが組み込まれています。デフォルトで組み込み済のProvisionerは[公式ドキュメント](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner)で確認できます。組み込まれていないストレージサービス（たとえばAWS EFSなど）を使用してDVPしたい場合は利用者でProvisonerを導入してからStorageClassを定義します。

1. 以下を満たすマニフェストを作成しデプロイしてください。StorageClassリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/storage/storage-classes/)を参考にしてください。なお、デプロイする際は必ずStorageClassを最初にデプロイすること。
   - Deployment
     - イメージは何でもよい
     - volumeMountsで2つのボリュームを別々の適当なpathにマウントしてください。
     - volumeで以下2つのpvcをそれぞれ指定してください。
       - normal-disk-pvc
       - fast-disk-pvc
   - PVC
     - normal-disk-pvc
       - storageClassNameはデフォルト
       - readwriteonce
       - 容量1G
     - fast-disk-pvc
       - storageClassNameはio1
       - readwriteonce
       - 容量4G
   - StorageClass
     - 名前はio1
     - ebsのprovisionerを使用
     - reclaimPolicyはDelete
2. 作成したオブジェクトを確認してください。なお、StorageClassのgp2はデフォルトで作成されているものです。
3. マネジメントコンソールからEBSを確認し、gp1とio1のボリュームがあることを確認してください。
4. デプロイしたオブジェクトをすべて削除してください。

---

次： [Kustomize](Kustomize.md)  
