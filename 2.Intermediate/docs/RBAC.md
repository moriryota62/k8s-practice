前： [ResourceQuota](ResourceQuota.md)  

---

# RBAC関連
K8sにはRBACの仕組みがあります。ユーザまたはK8s内のオブジェクトに対して権限を付与するServiceAccountがあります。ServiceAccountはK8sのリソースですが、ユーザはK8sのリソースではありません。ユーザはK8s外部の仕組みで認証され、K8sにアクセスします。ユーザやServiceAccountに許可する操作権限は4つのリソースで管理します。Namespaceに属するRole/RoleBinding、Namespaceに属さないClusterRole/ClusterRoleBindingです。これらのRBACはクラスタ操作権限を制限したい場合（たとえば、admin権限と参照権限だけ持つユーザを分ける、操作できるNamespaceの範囲を絞るなど）やK8sクラスタ内のPodからkube-apiserverへ通信（kubectl）したい場合などに用います。それ以外の場合としてはServiceAccountに対してデフォルトのimagePullSecretsを設定し、各Pod定義からimagePullSecretsの設定を省略したいときにServiceAccountを使います。ちなみに、Podは何かしらのServiceAccountで実行されますが、とくに指定しない場合はそのPodが起動するNamespace内にあるdefaultと言う名前のServiceAccountで実行されます。このServiceAccount:defaultはデフォルトの状態では何も権限が付与されていないため、kubectlによる操作は何も行えません。

1. 以下を満たすマニフェストを作成しデプロイしてください。
   - Deployment
     - イメージは``bitnami/kubectl``
     - replicas: 1
     - commandは["/bin/sh","-c","sleep 3600"]
2. デプロイしたPodに「kubectl get pod」の追加コマンドを発行し以下のようなメッセージで失敗すること
   ```
   Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:default" cannot list resource "pods" in API group "" in the namespace "default"
   ```
3. 以下を満たすマニフェストを作成しデプロイしてください。ServiceAccountリソースについては[公式ドキュメント](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)を参考にしてください。Roleリソースについては[公式ドキュメント](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole)を参考にしてください。RoleBindingリソースについては[公式ドキュメント](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)を参考にしてください。
   - 1セット目
     - ServiceAccount
       - 名前はget-pod
     - Role
       - ``CoreAPI``グループ内の``pods``リソースに対する``get``と``list``を許可する
     - RoleBinding
       - 上記、get-podとRoleを紐付ける
   - 2セット目  
     - ServiceAccount
       - 名前はget-deploy
     - Role
       - ``extensions``および``apps``グループ内の``deployments``リソースに対する``get``と``list``を許可する
     - RoleBinding
       - 上記、get-deployとRoleを紐付ける
4. 作成したServiceAccount,Role,RoleBindingを確認してください。
5. Deploymentのマニフェストを修正しServiceAccount:get-podでPodを起動するように修正しデプロイしなおしてください。
6. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが``実行できる``こと。
7. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行し以下のようなメッセージで失敗すること。
   ```
   Error from server (Forbidden): deployments.extensions is forbidden: User "system:serviceaccount:default:get-pod" cannot list resource "deployments" in API group "extensions" in the namespace "default"
   ```
8. Deploymentのマニフェストを修正しServiceAccount:get-deployでPodを起動するように修正しデプロイしなおしてください。
9. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが``実行できない``こと。
10. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行しコマンドが``実行できる``こと。
11. 1セット目のRoleBindingのマニフェストを修正し、紐付けるServiceAccountをget-podからget-deployに変更しデプロイしなおしてください。
12. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが``実行できる``こと。
13. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行しコマンドが``実行できる``こと。
14. 作成したDeployment、ServiceAccount、Role、RoleBindingをすべて削除する。

---

次： [3.Advanced](../../3.Advanced)  
