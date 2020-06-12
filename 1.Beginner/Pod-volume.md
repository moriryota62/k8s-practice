前： [Service-ClusterIP](Service-ClusterIP.md)  

---

# Pod設定 volume
Podのボリュームは一時的なものでありPodが消えると失われてしまいます。消えて困るデータはPod外のボリュームをマウントしてそこに格納します。

1. クラスタの参加ノードを確認しワーカーノードが1台だけの状態にしてください。
2. 以下を満たすDeploymentとServiceをデプロイしてください。
   - Deployment
     - Deploymentの名前は``volume``
     - replicas: ``1``
     - Pod
       - labelはすべて``app: volume``
       - containerのイメージは``nginx:1.12``
   - Service
     - Serviceの名前は``volume-svc``
     - typeは``指定なし``（type: ClusterIPでも可）
     - Portは80
     - 上記Deployment:volumeで展開したPodを対象とする
3. デプロイしたPod内のコンテナに対して追加コマンドを発行し/usr/share/nginx/html/index.htmlを以下内容に修正してください。
  ```
  zettai ni nakunatte ha ikenai data ga kokoni aru
  ```
4. curlを実行できるPodを展開しService:volume-svcに対してcurlを実行してください。さきほど修正したメッセージが表示されることを確認してください。
5. Podを削除（Deploymentは消さない！）しPodが再作成されたことを確認してください。
6. 再度Service:volume-svcに対してcurlを実行してください。さきほど修正したメッセージが``表示されない``ことを確認してください。
7. Deploymentを削除してください。（Serviceはそのままで良い）
8. 以下を満たす様にDeploymentのマニフェストを修正しapplyしてください。（次の[公式ドキュメント](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)を参考にするとよい）
   - volumes
     - ``hostPath``のボリュームプラグインを使用
     - ホスト (worker node) の/mntなど任意のディレクトリを対象とする
   - volumeMount
     - 上記で定義したhostPathのボリュームを指定する
     - コンテナのマウントパスは/usr/share/nginx/html/を指定する
9. デプロイしたPod内のコンテナに対して追加コマンドを発行し以下内容の/usr/share/nginx/html/index.htmlを作成してください。
  ```
  zettai ni nakunatte ha ikenai data ga kokoni aru
  ```
9. curlを実行できるPodを展開しService:volume-svcに対してcurlを実行してください。さきほど修正したメッセージが表示されることを確認してください。
10. Podを削除（Deploymentは消さない！）しPodが再作成されたことを確認してください。
11. 再度Service:volume-svcに対してcurlを実行してください。さきほど修正したメッセージが``表示される``ことを確認してください。
12. Deployment:volume、Deployment:curl、Service:volume-svcを削除してください。

---

次： [Pod-env](Pod-env.md)