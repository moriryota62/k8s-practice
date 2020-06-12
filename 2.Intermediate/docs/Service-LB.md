前： [DynamicVolumeProvisioning](DynamicVolumeProvisioning.md)    

---

# Service LoadBalancer
今までの内容ではPodへのアクセスをK8sクラスタ内部でおこなっていました。次はK8sクラスタ外部からアクセスしたいと思います。K8sクラスタ外部からアクセスを受けられるようにするにはLoadBalancerタイプのServiceを作成するのがもっとも手っ取り早いです。ただし、LoadBalancerタイプのServiceはAWSなどの対応したクラウドプロバイダで動いている場合のみです。また、LoadBalancerタイプのServiceは1つのServiceに1つのLoadBalancer(AWSだとELB)を作成するため、たくさん外部公開したいときは[Ingress](../3.Advanced/3-06.Ingress.md)などの利用を検討しましょう。

1. 以下を満たすServiceおよびDeploymentをデプロイしてください。Service Type:LoadBalancerについては[公式ドキュメント](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)を参考にしてください。
   - Service
     - 名前は``lb-svc``
     - 対象のlabelは``app: lb``
     - プロトコルは``TCP``
     - Portは``80``
     - clusterIPは``指定なし``で良い
     - typeは``LoadBalancer``
   - Deployment
     - 名前は``lb``
     - replicas: ``1``
     - labelはすべて``app: lb``
     - Pod
       - containerは一つでイメージは``nginx:1.12``
2. Serviceリソースの一覧を表示しデプロイしたlb-svcの``EXTERNAL-IP``を確認してください。（あとで使うのでメモしておく）
3. デプロイしたPod内のコンテナに対して追加コマンドを発行し、以下内容の/usr/share/nginx/html/index.htmlを作成してください。
  ```
  Type:LoadBalancer ha tottemo benri na kinou desu
  ```
4. インターネット接続可能な端末のwebブラウザからさきほど確認したlb-svcのEXTERNAL-IPにアクセスしてください。上記、修正した内容が表示されることを確認してください。（なお、AWSだとLBが使用可能になるまで2分くらいかかるので何度かアクセスしてみる。最長でも5分くらいすればアクセスできると思う。）
5. AWSマネジメントコンソールなどでELBを確認し、EXTERNAL-IPと同じDNS名を持つclassicのLBがデプロイされていることを確認してください。また、LBにアタッチされたsecurity group名を確認してください。
6. AWSマネジメントコンソールなどでK8sのワーカーにアタッチされているsecurity groupを確認し、LBにアタッチされたsecurity groupからのインバウンドが許可されていることを確認してください。
7. curlを実行できるPodを展開し、Service:lb-svcに対してcurlを実行してください。クラスタ内部からはtype:ClusterIPと同じようにアクセスできることを確認してください。
8. Service:lb-svcおよびDeployment:lbを削除してください。
9. AWSマネジメントコンソールなどでELBおよびワーカーノードのsecurity groupを確認し、Type:LoadBalancerの設定が消えていることを確認してください。

---

次： [Pod-resources](Pod-resources.md)  
