前： [StorageClass](StorageClass.md)  

---

# Kustomize
Kustomizeはマニフェスト管理に役立つコマンドラインツールです。v1.14のkubectlからは標準で使用できます。K8sを複数の環境（開発/ステ/本番等）で使用する場合、環境間では差異が生じると思います。（環境変数のドメイン名が違う。HWリソース量が違う。など）　差異がある場合は各環境ごとにマニフェストを用意することになりますが、共通箇所に修正が発生した場合にすべての環境のマニフェストをいちいち修正するのは大変です。そこでKustomizeを使用すると、ベースとなるマニフェストを作成しておき、環境差異が生じる部分だけ各環境で上書きしてマニフェストを生成するということができます。

1. 以下構造のkustomizeディレクトリを作成してください。
   ```
   kustomize
   ├── base
   └── overlay
       ├── prod
       └── dev
   ```
2. kustomize/base配下に以下を満たすマニフェストを作成してください。（applyはしなくてよい）
   - Deployment
     - replica:1
     - コンテナイメージはnginx:1.12
     - ConfigMap:env-configのENVを環境変数ENVに格納
   - Service
     - 上記DeploymentをClusterIP:80で公開
   - ConfigMap
     - 名前はenv-config
     - dataはENV: base
3. 上記マニフェストをresourcesとして指定したkustomize/base/kustomization.yamlを書いてください。kustomizeについては[kustomize公式ドキュメント](https://github.com/kubernetes-sigs/kustomize)や[k8s公式ドキュメント](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)を参考にしてください。（公式はあまりわかりやすくないのでインターネットを検索しても良い）
4. 以下コマンドでkustomizeでapplyしてください。（以下コマンドはkustomizeディレクトリで実行した場合）
   ``` sh
   kubectl apply -k base/
   ```
5. 作成したオブジェクトを確認し、以下コマンドで削除してください。（以下コマンドはkustomizeディレクトリで実行した場合）
   ``` sh
   kubectl delete -k base/
   ```
6. overlay/dev/およびoverlay/prod/に以下を満たすマニフェストおよびkustomization.yamlを作成してください。
   - prod
     - baseは../../base
     - 作成するオブジェクトのプレフィックスに``prod-``をつける
     - 作成するオブジェクトに``env: prod``のラベルを追加
     - ConfigMap:env-configを上書き（patches）し``ENV: prod``に変更してください。
   - dev
     - baseは../../base
     - 作成するオブジェクトのプレフィックスに``dev-``をつける
     - 作成するオブジェクトに``env: dev``のラベルを追加
     - ConfigMap:env-configを上書き（patches）し``ENV: dev``に変更してください。
7. 以下コマンドでprodとdevをデプロイしてください。.（以下コマンドはkustomizeディレクトリで実行した場合）
   ``` sh
   kubectl apply -k overlay/prod
   kubectl apply -k overlay/dev
   ```
8. 作成したオブジェクトを確認。pordおよびdevのPodに対して以下追加コマンドを発行しオーバーライドできていることを確認してください。
   ``` sh
   echo $ENV
   ```
9.  deploymentを以下の様にオーバーライドして再デプロイしてください。なお、オーバーライドする時のマニフェストは必要最小限の記載にすること
    - prod
      - replicas: 3
      - resources.requests.memory: 100Mi
    - dev
      - replicas: 2
      - resources.requests.memory: 50Mi
10. 上記オーバーライドした内容が反映されていることを確認してください。
11. Podの/usr/share/nginx/html/index.htmlの内容を環境変数ENVの値とし、各環境で表示を変えたい。kustomize/base配下のみを修正し実装してください。
12. curlが実行可能なPodを展開し、prodとdevそれぞれにcurlしてください。各環境名が表示されること
13. curl、dev、prodをすべて削除してください。

---

次： -  
