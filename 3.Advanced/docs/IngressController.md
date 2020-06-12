前： [descheduler](descheduler.md)  

---

# Ingress Controller
IngressはServiceの外部公開を制御するリソースおよびアドオン機能です。L7ロードバランサの様なものです。Ingressはその動作をコントロールするIngress ControllerをPodとしてクラスタにアドオンします。さらに、Ingress Controllerの動作設定を行うIngressリソース（こちらはK8sのリソース）を組み合わせて使います。

Kubernetes外からアクセスを受ける方法として、[Service Type:LB](../../2.Intermediate/docs/Service-LB.md)を紹介しました。Service Type:LBはとても便利ですが問題もあります。それはServiceごとにクラウドのLBができてしまう点です。たくさんのServiceを外部に公開したい場合、その数だけLBができてしまうのは管理、コストの面で問題となりえます。また、他の外部公開の方法としてService Type:NodePortという手段もありますが、この公開方法だとServiceの公開ポートで同じ番号が使えず、よく使う80や443で公開できるServiceが限られてしまいます。

Ingressを使うとこれらの問題を解決する事ができます。外部への公開はIngress用のService Type:LBが受け、Ingress Controllerに集約させます。Ingress ControllerはリクエストのURLを見てしかるべきServiceにリクエストを流します。このURLと転送先Serviceの設定はIngressリソースで定義します。

Ingress Controllerにはいくつかの種類があります。その中でもNginx Ingress ControllerはK8sがサポートしておりどのクラウドでも動かせます。

1. Nginx Ingress Controllerをデプロイしてください。なお、[Nginx Ingress Controllerの公式](https://kubernetes.github.io/ingress-nginx/deploy/)を参考にしてください。なお、Serviceは「service-l7.yaml」および「patch-configmap-l7.yaml」を使用してください。
2. AWSマネジメントコンソール（コマンドでも可）でNginx IngressのELBが作成されていることを確認してください。ELBのタグを見ればNginx Ingressのものか判断できる。
3. Nginx Ingress用LBにアタッチされているセキュリティグループを確認してください。
4. ワーカーノードのセキュリティグループを確認し、Nginx Ingress用LBのセキュリティグループからのインバウンドが許可されていることを確認してください。

次のテーマである[Ingress](Ingress.md)で実際の動作を確認します。

---

次： [Ingress](Ingress.md)  
