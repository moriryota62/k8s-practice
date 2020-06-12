前： [Namespace](Namespace.md)  

---

# DaemonSet
DaemonSetは常にすべてのワーカーノードで同じPodを起動するPod Controllerです。クラスタ管理系機能（ログやメトリクスの収集）によく使われます。

1. クラスタの参加ノードを確認し、ワーカーノードが2台以上の状態にしてください。
2. 以下を満たすマニフェストを作成しデプロイしてください。なお、DaemonSetについては[公式ドキュメント](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)を参考にしてください。
   - DaemonSet
     - 名前は``daemon``
     - labelはすべて``app: daemon``
     - Pod
       - メインコンテナ
         - 名前は``nginx``
         - イメージは``nginx:1.12``
3. DaemonSetリソースのオブジェクト一覧を表示し、``daemon``があることを確認してください。
4. Podリソースのオブジェクト一覧を起動ノード名も一緒に表示し、すべてのワーカーノードで``daemon``Podが起動していることを確認してください。
5. ``daemon``Podを1つ削除してからPodリソースのオブジェクト一覧を表示し、セルフヒーリングでPodが再作成されていることを確認してください。
6. DaemonSet:daemonを削除してください。

---

次： [StatefulSet](StatefulSet.md)  
