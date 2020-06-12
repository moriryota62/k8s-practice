前： [Pod-Probe](Pod-Probe.md)  

---

# Pod設定 initContainer
Pod内のメインコンテナを起動する前に一時的なコンテナをPod内で起動できます。たとえばメインコンテナ起動前の準備を実装したりできます。ポイントとしてはinitContainerで起動するコンテナはあくまでもメインコンテナ起動前の一時的なものであることです。そのため、initContainerで指定するcommandはexit0で終わるものを指定しましょう。

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、initContainerについては[公式ドキュメント](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#init-containers-in-use)を参考にしてください。
   - Deployment
     - 名前は``initcontainer``
     - replicas: ``1``
     - labelはすべて``app: initcontainer``
     - Pod
       - メインコンテナ
         - 名前は``init-container``
         - イメージは``nginx:1.12``
         - volume:index-htmlを``/usr/share/nginx/html``にマウント
       - initContainer
         - 名前``init``
         - イメージは``busybox``
         - commandは``['/bin/sh','-c','echo initContainer de settei sita noda > /tmp/index.html']``
         - volume:index-htmlを``/tmp``にマウント
       - volume
         - 名前は``index-html``
         - volumeプラグインは``emptyDir``を指定
   - Service
     - 名前は``initcontainer-svc``
     - Type: ``LoadBalancer``
     - Port: ``80``
     - 上記Deployment:initContainerで展開したPodを対象
2. Podの詳細を確認し、init Containers の箇所を確認する。また、eventを確認しコンテナinitがメインコンテナの前に起動したことを確認する。
3. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。（デプロイしてから接続できるまで2分、最長でも5分ほど時間がかかる）　initContainerで作成したindex.htmlの内容が表示されることを確認する。
4. 作成したDeployment:initcontainerおよびService:initcontainer-svcを削除する。

---

次： [Pod-nodeSelector](Pod-nodeSelector.md)  
