前： [Pod-volume](Pod-volume.md)  

---

# Pod設定 env
コンテナ環境では同じコンテナイメージを各環境（開発/ステ/本番等）で使い回すことがあります。しかし、各環境ごとに設定すべき値が違うといったことは往々にしてあります。たとえば本番と開発とではドメイン名が違う、外部DBのIPアドレスが違う場合などです。コンテナ環境においてこのような環境差異を表現するには環境ごとで変わる設定値を環境変数にし、コンテナ起動時に環境変数の値を設定します。

1. 以下を満たすDeploymentをデプロイしてください。
   - Deployment
     - Deploymentの名前は``env``
     - replicas: ``1``
     - labelはすべて``app: test``
     - Pod
       - containerのイメージは``busybox``
       - containerのコマンドは``['sh', '-c', 'echo $ENV1 $HOSTNAME $ENV2 && sleep 3600']``を指定
2. デプロイしたPod（コンテナ）のログを表示し「<Pod名>」が表示されることを確認してください。
3. デプロイしたPod（コンテナ）に追加コマンドを発行し設定されている環境変数を確認してください。
4. Deployment:testを削除してください。
5. 以下を満たす様にDeploymentのマニフェストを修正しapplyしてください。（次の[公式ドキュメント](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/#define-an-environment-variable-for-a-container)を参考にするとよい）
   - ``ENV1``を定義し、valueは``"watashi no namae ha "``
   - ``ENV2``を定義し、valueは``" desuYO"``
6. デプロイしたPod（コンテナ）のログを確認し「watashi no namae ha <Pod名> desuYO」と表示されること
7. デプロイしたPod（コンテナ）に追加コマンドを発行し、設定されている環境変数を確認してください。
8. Deployment:envを削除してください。

---

次： [2.Intermediate](../../2.Intermediate)