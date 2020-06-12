前： [Ingress](Ingress.md)  

---

# NetworkPolicy
NetworkPolicyはK8sクラスタ内で使えるセキュリティグループの様なものです。指定したラベルが付与されたPodやNamespaceに対するインバウンドおよびアウトバウンドの通信制限を設けることができます。デフォルトの状態では同一クラスタ内のPodはNamespaceが違っても制限なしに相互アクセス可能です。たとえばNamespaceで顧客ごとに分割をしていても、ネットワークアクセス可能だとあまり意味がありません。その様な場合にNetworkPolicyでアクセスを制限すればよりセキュアな構成にできます。

なお、NetworkPolicyはK8sの標準リソースですが、使用するにはクラスタのネットワークが対応したCNIプラグインで構成されている必要があります。たとえばEKSの場合、デフォルトでは「Amazon VPC CNI plugin for Kubernetes」を使用していますが、2020/03時点で「Amazon VPC CNI plugin for Kubernetes」はNetworkPolicyに対応していません。（たしかGKEとAKSは標準のCNIで対応しているはず。）　NetworkPolicyに対応したCNIプラグインとしてはCalicoなどがあります。

## Calicoのデプロイ

※ EKS等、NetworkPolicyに対応していないCNIを使用している場合に実施 

まずはCalicoをクラスタにデプロイする。[公式の手順](https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/calico.html)を参考に作業する。

1. 以下コマンドを実行してください。
   ``` sh
   kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/release-1.5/config/v1.5/calico.yaml
   ```
2. 以下コマンドを実行し、Calicoがデプロイされたことを確認してください。（calico-nodeとcalico-typhaがnodeの数だけデプロイされすべてRunningすればOK）
   ``` sh
   kubectl get daemonset calico-node --namespace kube-system
   kubectl get pod --namespace kube-system
   ```

## NetworkPolicy
続いてNamespace単位のNetworkPolicyについて確認する。

1. 以下を満たすマニフェストを作成しデプロイしてください。
   - 1セット目
     - Namespace
       - 1セット目とわかる名前にする。
       - 1セット目とわかるラベルをつける
     - Deployment
       - １つ目
         - namespaceは1セット目
         - 1つ目のDeployment固有のラベルをつける
         - nginx:1.12のコンテナ
         - /usr/share/nginx/html/index.htmlの内容を1つ目のアプリケーションであることがわかる内容に書き換える
       - 2つ目
         - namespaceは1セット目
         - 2つ目のDeployment固有のラベルをつける
         - curlが実行できるコンテナ
     - Service
       - namespaceは1セット目
       - 上記1つ目のDeploymentをClusterIPのPort:80で公開
   - 2セット目
     - 1セット目と基本同じでNamespaceの名前、ラベルとindex.htmlの内容を変える
2. 各NamespaceのcurlできるPodから各NamespaceのService:nginxに``通信できる``ことを確認してください。（計4回curlする。他Namespaceへcurlする場合、namespace名.service名でアクセスできる。）
3. 以下を満たすマニフェストを作成しデプロイしてください。NetworkPolicyリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/services-networking/network-policies/)を参考にしてください。
   - 1セット目
     - NetworkPolicy
       - 対象は同じnamespaceのPodすべて
       - 1セット目のNamespaceからのインバウンド通信を許可する。（言い方を変えると1セット目のNamespace以外からの通信を許可しない）
   - 2セット目
     - 1セット目と基本同じで2セット目のNamespaceからのインバウンド通信のみを許可する。
4. 各NamespaceのPod:curlから各NamespaceのService:nginxに``curl -s -m 10``で通信してください。（-mでタイムアウトを短くしている）　Namespaceを跨いだ通信は失敗することを確認してください。
5. デプロイしたオブジェクトをすべて削除してください。

上記のようにNetworkPolicyで通信制御をNamespaceにかけることができる。さらに、同じNamespace内でもPodごとに通信制御をかけることもできます。

6. 1セット目のマニフェストを以下のように修正しデプロイしてください。
   - Deployment
     - 3つ目
       - 3つ目のDeployment固有のラベルをつける
       - curlが実行できるコンテナ
   - NetworkPolicy
     - 対象は1つ目のDeployment
     - 3つ目のDeploymentのみ通信を許可する。
7. 2つ目と3つ目のDeploymentでデプロイしたPodからService:nginxに対して通信してください。3つ目でデプロイしたPodからのみアクセスできることを確認してください。（もしできてしまう場合、同一Namespaceからの通信を許可するNetworkPolicyが残っていないか確認し、残っていれば削除する）
8. デプロイしたすべてのオブジェクトを削除する。Calicoも削除する。

---

次： [StorageClass](StorageClass.md)  
