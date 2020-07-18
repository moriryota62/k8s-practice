前： [postStart/preStop](Pod-lifecycle.md)  

---

# Namespace
NamespaceはK8sクラスタを論理的に分割した区画です。複数の顧客向けサービスの提供（マルチテナント）や複数チームでK8sクラスタを共有
する時に使います。デフォルトでは「kube-system」や「default」などのNamespaceが用意されています。（K8sサービスによっては他のNamespaceがあるかもしれません）　Namespace単位で使用できるHWリソースの上限を設定したり、Namespace間の通信制御を行うこともできます。ただし、それらを行うには別のK8sリソース（[ResourceQuota](ResourceQuota.md)、[NetworkPolicy](../../3.Advanced/docs/NetworkPolicy.md)）を使用します。デフォルトでは同じK8sクラスタ内のNamespace間は自由に通信可能です。実は今まで意識して来ませんでしたが、Namespaceを指定しない場合はdefaultのNamespaceを使用しています。

1. Namespaceリソースのオブジェクト一覧を表示してください。
2. Namespace:kube-systemに属するPodリソースのオブジェクト一覧を表示してください。
3. 以下を満たすマニフェストを作成しデプロイしてください。なお、Namespaceについては[公式ドキュメント](https://kubernetes.io/docs/tasks/configure-pod-container/)を参考にしてください。
   - Namespace
     - 名前は``3-b``
   - Deployment
     - 名前は``kinpachi``
     - Namespaceは``3-b``
     - replicas: ``1``
     - labelはすべて``app: kinpachi``
     - Pod
       - メインコンテナ1つめ
         - 名前は``nginx``
         - イメージは``nginx:1.12``
         - ボリューム``index-html``を``/usr/share/nginx/html``にマウント
       - メインコンテナ2つめ
         - 名前は``curl``
         - イメージは``appropriate/curl``
         - commandは``['/bin/sh','-c','sleep 3600']``
       - initContainer
         - 名前は``init``
         - イメージは``busybox``
         - commandは``['/bin/sh','-c','echo The teacher in this class is Kinpachi-sensei> /tmp/index.html']``
         - ボリューム``index-html``を``/tmp``にマウント
       - volume
         - 名前は``index-html``
         - ボリュームプラグインは``emptyDir``
   - Service
     - 名前は``sensei``
     - Namespaceは``3-b``
     - タイプは指定なし（ClusterIP） 
     - Portは``80``
     - selectorは``app: kinpachi``
4. 上記マニフェストをコピー＆修正し以下を満たすマニフェストを作成しデプロイしてください。(変更箇所のみ``ハイライト``にする)
   - Namespace
     - 名前は``3-e``
   - Deployment
     - 名前は``koro``
     - Namespaceは``3-e``
     - replicas: 1
     - labelはすべて``app: koro``
     - Pod
       - メインコンテナ1つめ
         - 名前はnginx
         - イメージはnginx:1.12
         - ボリュームindex-htmlを/usr/share/nginx/htmlにマウント
       - メインコンテナ2つめ
         - 名前はcurl
         - イメージはappropriate/curl
         - commandは``['/bin/sh','-c','sleep 3600']``
       - initContainer
         - 名前はinit
         - イメージはbusybox
         - commandは``['/bin/sh','-c','echo The teacher in this class is Koro-sensei> /tmp/index.html']``
         - ボリュームindex-htmlを/tmpにマウント
       - volume
         - 名前はindex-html
         - ボリュームプラグインはemptyDir
   - Service
     - 名前はsensei
     - Namepaceは``3-e``
     - タイプは指定なし（ClusterIP）
     - Portは80
     - selectorは``app: koro``
5. Namespaceリソースのオブジェクト一覧を表示し、``3-b``と``3-e``が作成されていることを確認してください。
6. Namespace:defaultのPodおよびServiceリソースのオブジェクト一覧を表示し、上記手順で作成したPodやServiceが``ない``ことを確認してください。
7. Namespace:3-bのPodおよびServiceリソースのオブジェクト一覧を表示し、``kinpachi``と名の付くPodとService:``sensei``があることを確認してください。
8. Namespace:3-eのPodおよびServiceリソースのオブジェクト一覧を表示し、``koro``と名の付くPodとService:``sensei``があることを確認してください。
9. Namespace:3-bのPodに含まれるcurlコンテナからService:senseiに対して「curl -s」を実行し、``The teacher in this class is Kinpachi-sensei``と表示されることを確認してください。
10. Namespace:3-bのPodに含まれるcurlコンテナからNamespace:3-eのService:senseiに対して「curl -s」を実行し、``The teacher in this class is Koro-sensei``と表示されることを確認してください。なお、別NamespaceのServiceを指定する場合は``<service名>.<namespace名>``と指定すればよいです。
11. Namespace:3-eのPodに含まれるcurlコンテナからService:senseiに対して「curl -s」を実行し、``The teacher in this class is Koro-sensei``と表示されることを確認してください。
12. Namespace:3-eのPodに含まれるcurlコンテナからNamespace:3-bのService:senseiに対して「curl -s」を実行し、``The teacher in this class is Kinpachi-sensei``と表示されることを確認してください。なお、別NamespaceのServiceを指定する場合は``<service名>.<namespace名>``と指定すればよいです。
13. Namespace:3-eおよび3-bを削除してください。その後、Namespace:3-eおよび3-bに属していたPodおよびServiceを確認し、``Namespaceと同時に削除された``ことを確認してください。
14. すべてのNamespaceのPodおよびServiceリソースのオブジェクト一覧を表示し、作成したオブジェクトがすべて消えていることを確認してください。

---

次： [DaemonSet](DaemonSet.md)  
