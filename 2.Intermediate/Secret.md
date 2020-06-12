前： [ConfigMap(mount)](ConfigMap-mount.md)  

---

# Secret
SecretはConfigMapとほぼ一緒の使い方ができるリソースです。値を環境変数として読み込んだり、ファイルをボリュームマウントしたりできます。違いはdata部分がbase64エンコードされるか否かです。ConfigMapはbase64エンコードされていないためdataの内容がそのまま見えます。Secretはdata部分がbase64でエンコードされるためぱっと見はわかりません。（ただし、base64デコードしてしまえば誰でも見えます。）　ログイン情報など、Pod外で管理したいけど値が知られたくない情報を格納するのに使います。（ただし、ただのbase64エンコードなのでbase64デコードすれば誰でも見えてしまいます。）

1. 以下を満たすマニフェストを作成しデプロイしてください。Secretリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/configuration/secret/)を参考にしてください。
   - Secret
     - 以下のKey-Valueをdataとして持つ（以下はbase64デコードされた状態）
       - password: "zettai ni sirarete ha ikenai jyouhou"
   - Deployment
     - 上記Secretのpasswordの値を環境変数PASSWORDに格納
2. 上記Deploymentで展開したPodに「echo $PASSWORD」の追加コマンドを発行してください。Secretの内容が表示されること。
3. Secretの詳細を表示してください。dataのvalue部分が見えないこと。
4. Secretの内容をYAML形式で表示してください。passwordのvalueが表示されるのでコピーする。
5. どこでも良いので以下のコマンドを実行してください。valueの中身が見えてしまうこと。
   echo <前の手順でコピーしたvalue> | base64 --decode
6. SecretとDeploymentを削除

このように、SecretではK8sにアクセス可能な人ならだれでも値が知れてしまうので注意が必要です。また、Gitなどにマニフェストを置く時はSecretをさらに暗号化する仕組み(kubesec,sealedsecret,vault)を併用しましょう。

---

次： [Job](Job.md)  
