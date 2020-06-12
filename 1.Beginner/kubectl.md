前： -  

---

# kubectl
まずは基本的なkubectlの扱い方を学びます。kubectlの使い方については[公式チートシート](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)や[kubectl公式サイト](https://kubectl.docs.kubernetes.io/)を参考にすると良いです。

また、PodやServiceなどK8sの中でも基礎的なリソースについても触れていきます。

1. k8sクラスタのノードを表示するkubectlコマンドを実行してください。
2. ノードに付与されたラベルを表示するkubectlコマンドを実行してください。
3. Podリソースのオブジェクト一覧を表示するkubectlコマンドを実行してください。（この時点では何も表示されないかもしれません）
4. Serviceリソースのオブジェクト一覧を表示するkubectlコマンドを実行してください。（この時点では何も表示されないかもしれません）
5. 以下のサンプルマニフェストを使用してクラスタにPodをデプロイしてください。
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
  labels:
    app: test
spec:
  containers:
    - image: nginx:1.12
      name: nginx
```
6. Podが起動しているワーカーノード名やPodのIPアドレスを確認するkubectlコマンドを実行してください。
7. Podに付与されたラベルを表示するkubectlコマンドを実行してください。
8. デプロイしたPodの詳細情報を表示するkubectlコマンドを実行してください。
9. デプロイしたPodに含まれるコンテナのログを確認するkubectlコマンドを実行してください。（今回のコンテナの場合、何も表示されないかもしれない）
10. デプロイしたPodに含まれるコンテナへログインするkubectlコマンドを実行してください。
11. デプロイしたPodに含まれるコンテナに対し「touch /mnt/test」でファイルを作成し「ls /mnt/test」で確認する追加コマンドを発行してください。
12. デプロイしたPodを削除して同じマニフェストを使ってPodを再作成してください。
13. 再作成したPodの/mnt/testを確認し、手順11で作成したファイルが``消えている``ことを確認してください。
14. サンプルマニフェストを使用してServiceをデプロイしてください。
``` yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: test
  name: test
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: test
```
15. Serviceを経由してPodにアクセスします。なお、接続確認のためにcurlコマンドが入ったコンテナを含むPodをデプロイしコマンドを実行します。
``` sh
# curlの入ったPod(Deploymnet)を作成
kubectl run curl --image=appropriate/curl -- /bin/sh -c "sleep 3600"
# 以下メッセージは無視して良い
# kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.

# 作成したPodの名前を確認
kubectl get pod

# pod:curl に「curl test」の追加コマンドを発行
kubectl exec curl-<乱数> -- curl -s test
# 以下のようなメッセージが出ればOK
# <html>
# <head>
# <title>Welcome to nginx!</title>
# <style>
# 〜略〜

# 確認完了後以下コマンドでPod(Deployment)とServiceを削除
kubectl delete deployment curl
kubectl delete srvice test
```
16.  以下サンプルマニフェストを使い、2つのコンテナを内包したPodをデプロイします。
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
  labels:
    app: test
spec:
  containers:
    - image: nginx:1.12
      name: nginx
      command: ['sh', '-c', 'echo ossu ora nginx && sleep 3600']
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'echo ossu ora busybox && sleep 3600']
```
17. デプロイしたPodに含まれる``各コンテナのログ``を確認するkubectlコマンドを実行してください。
18. デプロイしたPodに含まれる``各コンテナにログイン``するkubectlコマンドを実行してください。
19. Pod:testを削除してください。
20. 以下サンプルマニフェストを使い、別々のコンテナが同じemptyDirのボリュームをマウントするPodを作成します。作成したら一方のコンテナでマウントしたボリュームに書き込みを行い、もう一方のコンテナでボリュームを確認し``データが共有できている``ことを確認してください。
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
  labels:
    app: test
spec:
  containers:
    - image: nginx:1.12
      name: nginx
      volumeMounts:
      - mountPath: /cache
        name: cache-volume
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
      volumeMounts:
      - mountPath: /cache
        name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```
21.  Pod:testを削除してください。

---

次： [Deployment](Deployment.md)