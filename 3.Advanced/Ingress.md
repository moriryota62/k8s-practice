前： [IngressController](IngressController.md)  

---

# Ingress

## http
Ingressを使い複数のサービスをhttpで公開します。

※ [IngressController](IngressController.md)実施直後の状態を想定しています。

1. 以下を満たすマニフェストを作成しデプロイしてください。Ingressリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/services-networking/ingress/)を参考にしてください。
   - 1セット目
     - Deployment
       - コンテナイメージはnginx:1.12
       - /usr/share/nginx/html/index.htmlの内容を1つ目のアプリケーションであることがわかる内容に書き換える（お好きにどうぞ）
     - Service
       - 上記DeploymentをClusterIPのPort:80で公開
     - Ingress
       - ``test-1.k8s.practice``に対するアクセスのルール
       - バックエンドを上記SerivceのPort:80に指定
   - 2セット目
     - Deployment
       - コンテナイメージはnginx:1.12
       - /usr/share/nginx/html/index.htmlの内容を2つ目のアプリケーションであることがわかる内容に書き換える（お好きにどうぞ）
     - Service
       - 上記DeploymentをClusterIPのPort:80で公開
     - Ingress
       - ``test-2.k8s.practice``に対するアクセスのルール
       - バックエンドを上記SerivceのPort:80に指定
2. インターネットに接続可能でcurlが実行できる端末から以下コマンドを発行し、それぞれのServiceにIngressを経由してアクセスできていることを確認してください。（プロキシ経由のアクセスだとリクエストがキャッシュされて表示が変わらないかもしれない。その場合はプロキシを通らない経路で試してみるとうまくいくかもしれない。）
   ``` sh
   curl -HHost:test-1.k8s.practice http://<nginx ingress用LBのDNS名>
   curl -HHost:test-2.k8s.practice http://<nginx ingress用LBのDNS名>
   ```
3. Nginx Ingress Controllerのログを確認し、上記2つのリクエストがIngress Controllerを経由していることを確認する。
4. （余裕があればやる）上記手順ではヘッダにホスト名を入れて実行したが、本来ならホスト名である「test-1.k8s.practice」および「test-2.k8s.practice」をRoute53などのDNSにレコード追加して動作確認するのが良いです。VPC内部限定のドメインで「.k8s.practice」を作成し、「*.k8s.practice」の宛先をNginx Ingress用LBとして以下のコマンドをVPC内のEC2インスタンス等から実行してみてください。
   ``` sh
   curl http://test-1.k8s.practice
   curl http://test-2.k8s.practice
   ```

## https
続いてhttpsの場合も確認する。TLSの終端はIngress用のService Type:LBで行う。証明書はオレオレを作成する。

1. 以下コマンドなどでオレオレ証明書を作成してください。（CNがk8s.practiceならどんな方法でも良いです。）
   ``` sh
   openssl req -x509 -sha256 -nodes -newkey rsa:2048 -subj '/CN=k8s.practice' -keyout k8s.practice.key -out k8s.practice.crt
   ```
2. 作成した「k8s.practice.key」と「k8s.practice.crt」の内容を表示してください。（ACMに登録するのに使う）
3. AWS ACMにオレオレ証明書を登録してください。
4. Nginx Ingress用のServiceを改良し、TLSで使用する証明書を設定してください。（ヒント：証明書はACMのarnを指定する。）
5. 以下コマンドでhttpsで通信できることを確認してください。（``httpではない``）
   ``` sh
   curl -k -HHost:test-1.k8s.practice https://<nginx ingress用LBのDNS名>
   curl -k -HHost:test-2.k8s.practice https://<nginx ingress用LBのDNS名>
   ```
6. 作成したtest-1、test-2関連のオブジェクトとNginx Ingress Controller関連のオブジェクトをすべて削除してください。 

---

次： [NetworkPolicy](NetworkPolicy.md)  
