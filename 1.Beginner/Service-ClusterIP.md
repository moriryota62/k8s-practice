前： [Deployment](Deployment.md)  

---

# Service ClusterIP
ServiceはPodへのアクセスを中継するL4ロードバランサのようなものです。PodのIPアドレスは一時的なもので再作成すると変わってしまいます。Serviceを指定してアクセスすれば、Podを再作成してもアクセス先を変えずに済みます。なお、Serviceにはいくつかタイプがあります。今回はもっとも基本的でK8sクラスタ内のアクセスにのみ使用できるtype:ClusterIPについて学びます。

※ ``1-02.Deploymnet``実施直後の状態から開始してください。

1. ``1-02.Deployment``で作成したPodに対するServiceのマニフェスト作成しapplyしてください。Serviceは以下要件を満たしてください。Serviceのマニフェストは[公式ドキュメント](https://kubernetes.io/docs/concepts/services-networking/service/#defining-a-service)を参考にしてください。
   - Serviceの名前は``nginx-svc``
   - Serviceのtypeは``指定なし``（明示的にtype: ClusterIPを指定しても良い）
   - portは``80``を指定
   - プロトコルは``TCP``
   - labelSelectorで``app: test``を指定
2. 作成したService:nginx-svcのCluster-IPを確認してください。
3. Serviceを経由してPodにアクセスしてください。なお、接続確認のためにcurlコマンドが入ったコンテナを含むPodをデプロイしcurlコンテナにログインしてcurlコマンドを使用してください。curlコマンドの宛先は確認した``Service:nginx-svcのCluster-IP``を指定してください。
``` sh
# curl の起動
kubectl run curl --image=appropriate/curl -- /bin/sh -c "sleep 3600"
# 以下メッセージは無視して良い
# kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run nerator=run-pod/v1 or kubectl create instead.

# 作成したPodの名前を確認
kubectl get pod

# pod:curl に「curl test」の追加コマンドを発行
kubectl exec curl-<乱数> -- curl -s <Service:nginx-svcのCluster-IP>
# 以下のようなメッセージが出ればOK
# <html>
# <head>
# <title>Welcome to nginx!</title>
# <style>
# 〜略〜
```
4. Serviceを経由してPodにアクセスしてください。今度はcurlコマンドの宛先を``nginx-svc``にして実行してください
``` sh
# pod:curl に「curl test」の追加コマンドを発行
kubectl exec curl-<乱数> -- curl -s nginx-svc
# 以下のようなメッセージが出ればOK
# <html>
# <head>
# <title>Welcome to nginx!</title>
# <style>
# 〜略〜
```
5. 2つあるPod:nginx-XXXXXそれぞれに含まれるtestコンテナに対して追加コマンドを発行し、コンテナ内の/usr/share/nginx/html/index.htmlを以下内容に修正してください。
   - どちらかのコンテナ
     ```
     watashi no sentouryoku ha
     ```
   - もう一方のコンテナ
     ```
     530000 desu
     ```
6. curlコマンドの宛先を``nginx-svc``とし、Serviceを経由してPodに複数回アクセスしてください。表示される結果が``ランダムで変わる``ことを確認してください。
7. Service:nginx-svcのlabelSelectorを「app: test」から「``app: test2``」に修正しapplyしてください。
8. curlコマンドの宛先を``nginx-svc``とし、Serviceを経由してPodに複数回アクセスしてください。``アクセスできない``ことを確認してください。
9. Deployment:nginxを削除してください。
10. Deployment:nginxのマニフェストを修正し、ラベルを「app: test」から「``app: test2``」に変えてapplyしてください。
11. curlコマンドの宛先を``nginx-svc``とし、Serviceを経由してPodに複数回アクセスしてください。アクセスできるようにはなるが、/usr/share/nginx/html/index.htmlを修正する前の状態であることを確認してください。
12. Deployment:nginx、Deployment:curlおよびService:nginx-svcを削除してください。

---

次： [Pod-volume](Pod-volume.md)