前： [StatefulSet-volumeClaimTemplate](StatefulSet-volumeClaimTemplate.md)  

---

``ここまででだいぶK8sにも慣れてきたと思うのでここからは少し手順を簡略化して書いていきます。``

# Configmap 環境変数

ConfigMapはPodに設定する環境変数やファイルをPodとは切り離して扱うようにするリソースです。まずは環境変数としてのConfigMapの利用を確認します。

Podにパラメータを渡す方法として、Podのenvに指定する方法を紹介しましたが、1つ2つならまだしも、大量の環境変数が必要な場合に1つずつenvを定義するのは大変です。この様な場合、ConfigMapにパラメータを定義しておき、PodからConfigMapを読み込んで環境変数を設定することができます。また、ConfigMapは複数のPodから扱うことができるため、複数Podで使用する環境変数を集中管理もできます。

1. 以下を満たすマニフェストを作成しデプロイしてください。ConfigMapについては[公式ドキュメント](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)を参考にしてください。PodでConfigMapを環境変数として読み込む方法は[公式ドキュメント](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables)を参考にしてください。
   - ConfigMap
     - 以下のKey-Valueをdataとして持つ
       - EP1: The Python Menace
       - EP2: Attack of the Clones
       - EP3: Revenge of the Sith
       - EP4: A New Hoge
       - EF5: The Empire Strikes Back
       - EP6: Return of the Jedi
       - EP7: The Force Awakens
       - EP8: The Last Jedi
       - EP9: The Rise of Kubernetes
   - Deployment(1つ目)
     - メインコンテナはshが使用できれば何でも良い
     - 上記ConfigMapのKey-Valueをすべて環境変数として読み込む
     - メインコンテナのコマンドは以下を指定
       - command: ``['/bin/sh','-c','echo "Star Wars no title ichiran \n $EP1 \n $EP2 \n $EP3 \n $EP4 \n $EP5 \n $EP6 \n $EP7 \n $EP8 \n $EP9"; sleep 3600']``
   - Deployment(2つ目)
     - メインコンテナはshが使用できれば何でも良い
     - 上記ConfigMapのKey-Valueをすべて環境変数として読み込む
     - メインコンテナのコマンドは以下を指定
       - command: ``['/bin/sh','-c','echo "Star Wars no koukai jyun \n $EP4 \n $EP5 \n $EP6 \n $EP1 \n $EP2 \n $EP3 \n $EP7 \n $EP8 \n $EP9; sleep 3600']``
2. 各Deploymentで展開したPodのログを表示してください。
3. ConfigMapの値の誤りを修正し再デプロイしてください。なお、正しいvalueは[参考サイト](https://dic.nicovideo.jp/a/%E3%82%B9%E3%82%BF%E3%83%BC%E3%82%A6%E3%82%A9%E3%83%BC%E3%82%BA)などを見てください。
4. 各Deploymentで展開したPodのログを表示してください。表示が変わらないこと。
5. 各Deploymentで展開したPodのログを削除しセルフ・ヒーリングさせる。
6. 各Deploymentで展開したPodのログを表示してください。表示が変わること。
7. ConfigMapとDeployment2つを削除してください。

---

次： [ConfigMap(mount)](ConfigMap-mount.md)  
