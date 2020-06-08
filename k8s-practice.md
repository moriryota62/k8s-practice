---
tags:
  - k8s-practice
---

とくに指定がない場合はdefaultのnamespaceを対象に実行する。
AWS上のk8s（EKS等）を前提として記載している。（他のクラウドでもおそらく同様に動く）

## 超初級
Pod、Nodeといった必須な概念を学ぶ。また、基本的なkubectlの扱い方も学ぶ。なお、kubectlの使い方については[公式チートシート](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)や[kubectl公式サイト](https://kubectl.docs.kubernetes.io/)を参考にすると良い。

1. k8sクラスタのノードを表示するコマンドを答えよ
2. ノードに付与されたラベルを表示するコマンドを答えよ
3. Podリソースのオブジェクト一覧を表示するコマンドを答えよ
4. Serviceリソースのオブジェクト一覧を表示するコマンドを答えよ
5. サンプルマニフェストを使用してクラスタにPodをデプロイせよ
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
6. Podが起動しているワーカーノード名やPodのIPアドレスを確認するコマンドを答えよ
7. Podに付与されたラベルを表示するコマンドを答えよ
8. デプロイしたPodの詳細情報を表示するコマンドを答えよ
9. デプロイしたPodに含まれるコンテナのログを確認するコマンドを答えよ（今回のコンテナの場合、何も表示されないかもしれない）
10. デプロイしたPodに含まれるコンテナへログインするコマンドを答えよ
11. デプロイしたPodに含まれるコンテナへログインせずに「touch /mnt/test」でファイルを作成し「ls /mnt/test」で確認する追加コマンドを発行せよ
12. デプロイしたPodを削除して再作成せよ
13. 再作成したPodの/mnt/testを確認し、ファイルが``消えている``ことを確認せよ
14. サンプルマニフェストを使用してServiceをデプロイせよ
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
15.  Serviceを経由してPodにアクセスせよ。なお、接続確認のためにcurlコマンドが入ったコンテナを含むPodをデプロイしcurlコンテナにログインしてcurlコマンドを使用すること
``` sh
# curl の起動
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

# 確認完了後以下コマンドでPod(Deployment)を削除
kubectl delete deployment curl
```
16. 以下サンプルマニフェストを使い、改良して2つのコンテナを内包したPodをデプロイせよ。2つ目のコンテナは以下の記述を参考にすること
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
17. デプロイしたPodに含まれる``それぞれの``コンテナのログを確認するコマンドを答えよ
18. デプロイしたPodに含まれる``それぞれの``コンテナにログインするコマンドを答えよ
19. 以下サンプルマニフェストを使い、コンテナが同じemptyDirのボリュームをマウントするPodを作成せよ。作成したら一方のコンテナでマウントしたボリュームに書き込みを行い、もう一方のコンテナでボリュームを確認し``データが共有できている``ことを確認せよ
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
20. Pod:test、Service:test、Deployment:curlを削除せよ

## 初級
K8sの中でも基本的なリソースであるDeploymentとServiceについて学ぶ。
また、簡単なPod設定についても学ぶ。

出てくるキーワード
- K8sリソース
  - Deployment
  - ReplicaSet
  - Service(ClusterIP)
- Pod設定
  - volume
  - env

### DeploymetとServiceの基本
Deployment(ReplicaSet)はPodを常に定義した状態に保とうとするリソースである。ServiceはPodの前段に配置されるロードバランサの役割を担うリソースである。DeploymentとServiceはK8sの基本リソースであり``もっとも重要なリソース``といっても過言ではない。それぞれのリソースを実際に定義してその動作を確認する。

1. 以下を満たすDeploymentのマニフェストを作成しapplyせよ
   - Deploymentの名前は``nginx``
   - replicaは``1``
   - Pod
     - コンテナは``test``という名前のもの1つ
     - imageは``nginx:1.12``を使用
     - ラベルは``app: test``を付与する
2. Deployment,ReplicaSet,Podそれぞれのオブジェクト一覧を表示せよ
3. 作成したnginx-XXXXXXという名前のPodを削除せよ。(Deploymentは消しちゃだめ！)　再度Podを確認し、さきほどとは違う名前のPodが作成されていることを確認せよ
4. マニフェストを修正しDeploymentのreplicaを``2``にする。　再度Podを確認しPodが増えていることを確認せよ
5. 以下を満たすServiceのマニフェストを作成しapplyせよ
   - Serviceの名前は``nginx-svc``
   - Serviceのtypeは``指定なし``（明示的にtype: ClusterIPを指定しても良い）
   - portは``80``を指定
   - プロトコルは``TCP``
   - labelSelectorで``app: test``を指定
6. 作成したService:nginx-svcのCluster-IPを確認せよ
7. Serviceを経由してPodにアクセスせよ。なお、接続確認のためにcurlコマンドが入ったコンテナを含むPodをデプロイしcurlコンテナにログインしてcurlコマンドを使用すること。curlコマンドの宛先は確認した``Service:nginx-svcのCluster-IP``を指定する
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
8. Serviceを経由してPodにアクセスせよ。今度はcurlコマンドの宛先を``nginx-svc``にして実行せよ
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
9. 2つあるPod:nginx-XXXXXそれぞれに含まれるtestコンテナに対して追加コマンドを発行し、コンテナ内の/usr/share/nginx/html/index.htmlを以下内容に修正せよ
   - どちらかのコンテナ
     ```
     watashi no sentouryoku ha
     ```
   - もう一方のコンテナ
     ```
     530000 desu
     ```
10. curlコマンドの宛先を``nginx-svc``とし、Serviceを経由してPodに複数回アクセスせよ。表示される結果が``ランダムで変わる``ことを確認せよ
11. Service:nginx-svcのlabelSelectorを「app: test」から「``app: test2``」に修正しapplyせよ
12. curlコマンドの宛先を``nginx-svc``とし、Serviceを経由してPodに複数回アクセスせよ。``アクセスできない``ことを確認せよ
13. Deployment:nginxを削除せよ
14. Deployment:nginxのマニフェストを修正し、ラベルを「app: test」から「``app: test2``」に変えてapplyせよ
15. curlコマンドの宛先を``nginx-svc``とし、Serviceを経由してPodに複数回アクセスせよ。アクセスできるようにはなるが、/usr/share/nginx/html/index.htmlを修正する前の状態であることを確認せよ
16. Deployment:nginx、Deployment:curlおよびService:nginx-svcを削除せよ
17. Deployment、ReplicaSet、Podの関係を説明せよ
18. Serviceは転送先のPodをどのように判別しているか説明せよ

### Pod設定-初級編- (volume)
Podのボリュームは一時的なものであり、Podが消えると失われてしまう。消えては困るデータはPod外のボリュームに格納し、Podからそのボリュームをマウントする。マウントするボリュームの指定方法とPodからマウントする方法を学ぶ。

1. クラスタの参加ノードを確認し、ワーカーノードが1台だけの状態にせよ
2. 以下を満たすDeploymentとServiceをデプロイせよ
   - Deployment
     - Deploymentの名前は``volume``
     - replicas: ``1``
     - Pod
       - labelはすべて``app: volume``
       - containerのイメージは``nginx:1.12``
   - Service
     - Serviceの名前は``volume-svc``
     - typeは``指定なし``（type: ClusterIPでも可）
     - Portは80
     - 上記Deployment:volumeで展開したPodを対象とする
3. デプロイしたPod内のコンテナに対して追加コマンドを発行し、/usr/share/nginx/html/index.htmlを以下内容に修正せよ
  ```
  zettai ni nakunatte ha ikenai data ga kokoni aru
  ```
4. curlを実行できるPodを展開し、Service:volume-svcに対してcurlを実行せよ。さきほど修正したメッセージが表示されることを確認せよ
5. Podを削除（Deploymentは消さない！）しPodが再作成されたことを確認せよ
6. 再度Service:volume-svcに対してcurlを実行せよ。さきほど修正したメッセージが``表示されない``ことを確認せよ
7. Deploymentを削除せよ（Serviceはそのままで良い）
8. 以下を満たす様にDeploymentのマニフェストを修正しapplyせよ。（次の[公式ドキュメント](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)を参考にするとよい）
   - volumes
     - ``hostPath``のボリュームプラグインを使用
     - ホスト (worker node) の/mntなど任意のディレクトリを対象とする
   - volumeMount
     - 上記で定義したhostPathのボリュームを指定する
     - コンテナのマウントパスは/usr/share/nginx/html/を指定する
9. デプロイしたPod内のコンテナに対して追加コマンドを発行し、以下内容の/usr/share/nginx/html/index.htmlを作成せよ
  ```
  zettai ni nakunatte ha ikenai data ga kokoni aru
  ```
9. curlを実行できるPodを展開し、Service:volume-svcに対してcurlを実行せよ。さきほど修正したメッセージが表示されることを確認せよ
10. Podを削除（Deploymentは消さない！）しPodが再作成されたことを確認せよ
11. 再度Service:volume-svcに対してcurlを実行せよ。さきほど修正したメッセージが``表示される``ことを確認せよ
12. Deployment:volume、Deployment:curl、Service:volume-svcを削除せよ

### Pod設定-初級編-  (env)
コンテナ環境では同じコンテナイメージを各環境（開発/ステ/本番等）で使用するのが望ましい。こうすることで、テストを本番以外の環境で事前に行っておき、本番リリース時は使用するコンテナイメージを切り替えるだけで従来の様な複雑なリリース作業を省くことができる。しかしながら、各環境ごとに設定すべき値が違うといったことは往々にしてある。（たとえば本番と開発とではドメイン名が違う。外部DBのIPアドレスが違うなど）　コンテナ環境においてこのような環境差異を表現するには、環境間で変わる設定値はコンテナの環境変数を参照するようコンテナをbuildし、コンテナ起動時に環境変数を設定する方法が取られる。この章ではコンテナの環境変数を設定する基本的な方法を学ぶ。

1. 以下を満たすDeploymentをデプロイせよ
   - Deployment
     - Deploymentの名前は``env``
     - replicas: ``1``
     - labelはすべて``app: test``
     - Pod
       - containerのイメージは``busybox``
       - containerのコマンドは``['sh', '-c', 'echo $ENV1 $HOSTNAME $ENV2 && sleep 3600']``を指定
2. デプロイしたPod（コンテナ）のログを確認し「<Pod名>」が表示されること
3. デプロイしたPod（コンテナ）に追加コマンドを発行し、設定されている環境変数を確認せよ
4. Deployment:testを削除せよ
5. 以下を満たす様にDeploymentのマニフェストを修正しapplyせよ。（次の[公式ドキュメント](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/#define-an-environment-variable-for-a-container)を参考にするとよい）
   - ``ENV1``を定義し、valueは``"watashi no namae ha "``
   - ``ENV2``を定義し、valueは``" desuYO"``
6. デプロイしたPod（コンテナ）のログを確認し「watashi no namae ha <Pod名> desuYO」と表示されること
7. デプロイしたPod（コンテナ）に追加コマンドを発行し、設定されている環境変数を確認せよ
8. Deployment:envを削除せよ


## 中級
初級よりもより実践的な内容を学ぶ。
- CloudProviderと連携した機能を学び、k8sの威力を知る。
- 少し高度なPod設定についても学ぶ
- Deployment/Service以外の頻出リソースについても学ぶ。

出てくるキーワード
- K8sリソース
  - Persistent Volume Claim
  - Persistent Volume
  - Service(type:LoadBalancer)
  - Namespace
  - DaemonSet
  - StatefulSet
  - Job
  - CronJob
  - ConfigMap
  - Secret
  - LimitRange
  - ResourceQuota
  - RBAC関連（ServiceAccount/ClusterRole/ClusterRoleBindig/Role/RoleBinding）
- Pod設定
  - resources
  - liveness/readiness Probe
  - initContainer
  - Podスケジュール（nodeSelector）

### Dynamic Volume Provisioning
通常のオンプレやVMのインフラ基盤では、外部ボリュームをマウントしたい場合はあらかじめボリュームを作成し、マウント対象のサーバに割り当てる必要があった。Kubernetesの場合、これらの作業をKubernetesに任せることができる。このPod作成時にPodが必要となるボリュームを作成し、Pod起動ノードに割り当てる機能をDynamic Volume Provisioning（以下、DVP）という。（なお、DVPが使えるのはAWSなど対応したクラウドでK8sが動いている場合のみである。）　この章ではDVPの使用方法について学ぶ。

1. 以下を満たすPersistentVolumeClaim（以下、PVC）およびDeploymentをデプロイせよ。PVCは[公式ドキュメント](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)を参考にせよ。volumeプラグインは[公式ドキュメント](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)を参考にせよ。
   - PVC
     - 名前は``test-pvc``
     - storageClassNameは``指定しない``（デフォルトのStorageClassを使用する）
     - accessModesは``ReadWriteOnce``
     - ストレージ容量は``1Gi``
   - Deployment
     - 名前は``dvp``
     - replicas: ``1``
     - labelはすべて``app: dvp``
     - Pod
       - containerは一つでイメージは``nginx:1.12``
       - volumeプラグインで上記PVC:``test-pvc``を指定
       - 上記で定義したボリュームをコンテナの/usr/share/nginx/html/にマウント
2. PVCリソースのオブジェクト一覧を確認するコマンドを答えよ
3. PVリソースのオブジェクト一覧を確認するコマンドを答えよ
4. AWSでEBSボリュームを確認し、「kubernetes-dynamic-pvc-」から始まる名前のボリュームが作成されていることを確認せよ
5. デプロイしたPod内のコンテナに対して追加コマンドを発行し、以下内容の/usr/share/nginx/html/index.htmlを作成せよ
  ```
  Dynamic Volume Provisoning ha tottemo benri na kinou desu
  ```
6. Podを削除
7. 再作成されたPodに含まれるコンテナ内を確認し、/usr/share/nginx/html/index.htmlの内容が残っていることを確認せよ
8. Deployment:dvpを削除せよ（PVCは削除しない！）
9. PVC、PVおよびAWSのEBSを確認し、オブジェクトが``消えていない``ことを確認せよ
10. PVCを削除せよ
11. PVC、PVおよびAWSのEBSを確認し、オブジェクトが``消えている``ことを確認せよ
12. （上級問題）PVC、PV、EBSの関係を説明せよ

### Service Type:LoadBalancer
今までの内容ではPodに対する通信確認をK8sクラスタ内部でおこなっていた。これは、Podへの通信の前段に配置したServiceをデフォルトのType:ClusterIPでデプロイしたからである。Type:ClusterIPのIPアドレス（およびホスト名）はK8sクラスタ内部でしか使用できない。そのため、K8sクラスタ外部からの通信を受けるにはType:ClusterIP以外のTypeのServiceをデプロイする。この章では比較的簡単に外部公開を行えるType:LoadBalancerについて学ぶ。Type:LoadBalancerはクラウドプロバイダと連携し、クラウドのロードバランサーのデプロイやワーカーノードへの通信許可設定を自動で行う優れた機能である。（なお、Type:LoadBalancerが使えるのはAWSなど対応したクラウドでK8sが動いている場合のみである。）

1. 以下を満たすServiceおよびDeploymentをデプロイせよ。Service Type:LoadBalancerについては[公式ドキュメント](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)を参考にせよ。
   - Service
     - 名前は``lb-svc``
     - 対象のlabelは``app: lb``
     - プロトコルは``TCP``
     - Portは``80``
     - clusterIPは``指定なし``で良い
     - typeは``LoadBalancer``
   - Deployment
     - 名前は``lb``
     - replicas: ``1``
     - labelはすべて``app: lb``
     - Pod
       - containerは一つでイメージは``nginx:1.12``
2. Serviceリソースの一覧を確認し、デプロイしたlb-svcの``EXTERNAL-IP``を確認せよ。（あとで使うのでメモしておく）
3. デプロイしたPod内のコンテナに対して追加コマンドを発行し、以下内容の/usr/share/nginx/html/index.htmlを作成せよ。
  ```
  Type:LoadBalancer ha tottemo benri na kinou desu
  ```
4. インターネット接続可能な端末のwebブラウザからさきほど確認したlb-svcのEXTERNAL-IPにアクセスせよ。上記、修正した内容が表示されることを確認せよ。（なお、LBが使用可能になるまで2分くらいかかるので何度かアクセスしてみる。最長でも5分くらいすればアクセスできると思う。）
5. AWSでELBを確認し、EXTERNAL-IPと同じDNS名を持つclassicのLBがデプロイされていることを確認せよ。また、LBにアタッチされたsecurity group名を確認せよ。
6. AWSでK8sのワーカーにアタッチされているsecurity groupを確認し、LBにアタッチされたsecurity groupからのインバウンドが許可されていることを確認せよ。
7. curlを実行できるPodを展開し、Service:lb-svcに対してcurlを実行せよ。クラスタ内部からはtype:ClusterIPと同じようにアクセスできることを確認せよ。
8. Service:lb-svcおよびDeployment:lbを削除せよ。
9. AWSでELBおよびワーカーノードのsecurity groupを確認し、Type:LoadBalancerの設定が消えていることを確認せよ。

### Pod設定-中級編- (resources)
Podはとくに指定がない場合、ワーカーノードのHWリソース（CPU/メモリ）を使いたいだけ要求する。Podはワーカー上で複数実行するため、各Podが際限なくHWリソースを要求しワーカーノードのHWリソースが足りなくなると最悪の場合ワーカーノードが停止してしまう。このような事態を防ぐため、PodにHWリソースの``要求容量（requests）``と``上限容量（limits）``を設定することができる。この設定がresourcesである。resourcesはPodに含まれる各コンテナ単位で設定することができる。Podのresourcesは含まれるコンテナのresourcesを合算した値となる。  
要求容量（requests）はPod起動時に最低限確保が保証されるHWリソース容量である。Podはこの要求容量が確保できるワーカーノードで起動する。もし、どのワーカーノードでも要求容量が確保できなかった場合、Podは実行されず実行待ち状態（Pending）となる。  
上限容量（limits）はPodが使用できるHWリソース容量の上限である。上限までHWリソースを使用できるが確保は保証されない。つまり、上限容量は1つのワーカーノード上でオーバーコミットされる可能性がある。オーバーコミットを許容しない場合は要求容量と上限容量を同じにすればよい。（もしくは上限容量だけ設定すると自動で要求容量も同じ値で設定される）

1. 以下を満たすDeploymentのマニフェストを作成しデプロイせよ。なお、resourcesの設定については[公式ドキュメント](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)を参考にせよ。
   - Deployment
     - 名前は``resources``
     - replicas: ``1``
     - labelはすべて``app: resources``
     - Pod
       - コンテナの名前は``cpu-tukau-man``
       - イメージは``busybox``
       - commandは``['/bin/sh', '-c', 'sleep 3600']``
       - 以下のHWリソース容量を指定
         - CPU要求``500m``
         - CPU上限``600m``
2. デプロイしたPodの詳細を表示し、コンテナにrequestsとlimitsが設定されていることを確認せよ。
3. 作成したDeploymentを削除せよ。
4. 以下を満たすようにマニフェストを改良してデプロイせよ。
   - Podに以下containerを追加
     - 名前は``memory-tukau-man``
     - イメージは``busybox``
     - commandは``['/bin/sh', '-c', 'sleep 3600']``
     - 以下のHWリソース容量を指定
       - memory上限``100Mi``
5. デプロイしたPodの詳細を表示し、各コンテナにresourcesが設定さていることを確認せよ。また、memory-tukau-manにはmemoryの``requetsとlimitsが同じ値``で設定されていることを確認せよ。
6. Deploymentのreplica数を10に拡張せよ。
7. Podリソースのオブジェクト一覧を表示し、STATUS:PendingのPodがあること。（ない場合、Deploymentのreplica数をより大きな値にする）
8. STATUS:PendingのPodの詳細を表示しeventを確認せよ。以下の様なメッセージが出力されているはずである。（これは要求した量のCPUを確保できるノードが見つからなかった場合のメッセージ）
   ```
   0/2 nodes are available: 2 Insufficient cpu.
   ```
9. Deployment:resourcesを削除せよ

### Pod設定-中級編- (initContainer)
Pod内のメインコンテナを起動する前に一時的なコンテナをPod内で起動することができる。たとえば他Podとの依存関係を実装したり、メインコンテナ起動前の準備作業を実装したりなどができる。ポイントとしてはinitContainerで起動するコンテナはあくまでもメインコンテナ起動前の一時的なものであること。そのため、initContainerで指定するcommandはexit0で終わるものを指定する。initContainerはメインコンテナ起動前に消えてなくなる。

1. 以下を満たすマニフェストを作成しデプロイせよ。なお、initContainerについては[公式ドキュメント](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#init-containers-in-use)を参考にせよ。
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
3. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。（デプロイしてから接続できるまで2分、最長でも5分ほど時間がかかる）　initContainerで作成したindex.htmlの内容が表示されることを確認する
4. 作成したDeployment:initcontainerおよびService:initcontainer-svcを削除する。

### Pod設定-中級編- (livenessProbe/readinessProbe)
K8sが意識するPodの状態はコンテナのメインプロセスがあるか/ないかだけである。（Dockerの場合の話、コンテナランタイムによって違うかも）　これだとたとえばメインプロセスがハングしても異常と検知されず、セルフ・ヒーリングが行われない。また、プロセスの起動後に初期化処理が必要なコンテナであっても、Pod
起動（プロセス起動）と同時に通信を受けることとなり、通信が失敗する可能性もある。これらを回避するため、コンテナのヘルスチェックを追加で設定することができる。ヘルスチェックはコンテナの``正常性確認（livenessProbe）``と``待受状態確認（readinessProbe）``の2種類がある。（K8s1.16以上では
さらに追加でstartupProbeもあるがこのpracticeでは触れない）　ヘルスチェックはPodに含まれるコンテナ単位で設定できる。  
正常性確認（livenessProbe）はコンテナの稼働状態を確認するヘルスチェックである。このヘルスチェックに失敗した場合、コンテナの再作成を行う。Podデプロイ後に威力を発揮するヘルスチェックである。  
待受状態確認（readinessProbe）もコンテナの稼働状態を確認するヘルスチェックである。ただし、こちらのヘルスチェックに失敗してもコンテナの再作成は行わない。代わりに、ヘルスチェックに失敗したコンテナを含むPodをServiceのバランシング対象Podから一時的に除外する。ヘルスチェックに成功するとServiceのバランシング対象Podにまた追加される。Pod起動時に威力を発揮するヘルスチェックである。

1. 以下を満たすマニフェストを作成しデプロイせよ。なお、probeについては[公式ドキュメント](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)を参考にせよ。
   - Deploymet
     - 名前は``probe``
     - replicas: ``1``
     - labelはすべて``app: probe``
     - Pod
       - メインコンテナ
         - 名前は``probe``
         - イメージは``nginx:1.12``
         - readinessProbe
           - ``httpGet``のヘルスチェックを行う
           - pathは``/``
           - portは``80``
           - ヘルスチェックの詳細設定はとくに行わない
   - Service
     - 名前は``probe-svc``
     - Type: ``LoadBalancer``
     - Port: ``80``
     - 上記Deployment:probeで展開したPodを対象
2. Podの詳細を表示し、readinessProbeが設定されていることを確認。
3. Podリソースのオブジェクト一覧を確認し、上記マニフェストでデプロイしたPodの``READYが1/1``になっていることを確認する。また、PodのIPアドレスも確認しておく。
4. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。（デプロイしてから接続できるまで2分、最長でも5分ほど時間がかかる）　「Welcome to nginx!」と``表示される``こと
5. Service:probe-svcの詳細を表示し、EndpointsにPodのIPアドレスが``設定されている``ことを確認。
6. Podに以下の追加コマンドを発行する
   ``` sh
   mv /usr/share/nginx/html/index.html /usr/share/nginx/html/hoge.html
   ```
7. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの``READYが0/1``になっていることを確認する。
8. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。「Welcome to nginx!」が``表示されない``こと。
9.  Service:probe-svcの詳細を表示し、EndpointsにPodのIPアドレスが``設定されていない``ことを確認。
10. Podに以下の追加コマンドを発行する
   ``` sh
   mv /usr/share/nginx/html/hoge.html /usr/share/nginx/html/index.html
   ```
11. しばらく（10秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの``READYが1/1``になっていることを確認する。
12. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。「Welcome to nginx!」が``表示される``こと。
13. Service:probe-svcの詳細を表示し、EndpointsにPodのIPアドレスが``設定されている``ことを確認。
14. Deployment:probeを削除（Serviceも消して良いけどこのあとデプロイ待ちとかEXTERNAL-IP確認面倒なので残しておいた方がよい）
15. Deployment:probeのマニフェストを以下のように編集してデプロイする。
    - readinessProbeのブロックを丸コピペし、readinessProbeを``livenessProbe``に書き換える
    - readinessProbeのブロックを``コメントアウト``する
16. Podの詳細を表示し、livenessProbeが設定されていることを確認。
17. Podリソースのオブジェクト一覧を確認し、Podの``RESTARTS``の値を確認する。（たぶん0）
18. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。「Welcome to nginx!」が``表示される``こと。
19. Podに以下の追加コマンドを発行する。
   ``` sh
   rm /usr/share/nginx/html/index.html
   ```
18. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの``RESTARTSが増えている``ことを確認する。
19. ServiceのEXTERNAL-IPを確認し、作業端末のwebブラウザから接続する。「Welcome to nginx!」が``表示される``こと。
20. Deployment:probeを削除（ここではServiceは絶対消さない。あとの操作の切り分けが面倒になる）
21. Deployment:probeのマニフェストを以下のように編集してデプロイする。
    - メインコンテナのcommandを``['/bin/sh','-c','sleep 60;nginx -g "daemon off;"']``にする（nginxのプロセスがPod起動してから60秒後に起動する）
    - コメントアウトしていたreadinessProbeのブロックの``コメントアウトを解除``する
    - livenessProbeのヘルスチェックの詳細設定に``initialDelaySeconds: 120``を設定
22. （デプロイしてから60秒以内に実施）Podリソースのオブジェクト一覧を確認し、上記マニフェストでデプロイしたPodの``READYが0/1``になっていることを確認する。
23. （デプロイしてから60秒以内に実施）作業端末のwebブラウザからService経由で接続する。「Welcome to nginx!」が``表示されない``こと。
24. （デプロイしてから60秒以降に実施）Podリソースのオブジェクト一覧を確認し、上記マニフェストでデプロイしたPodの``READYが1/1``になっていることを確認する。
25. （デプロイしてから60秒以降に実施）作業端末のwebブラウザからService経由で接続する。「Welcome to nginx!」が``表示される``こと。
26. Podに以下の追加コマンドを発行する。
   ``` sh
   rm /usr/share/nginx/html/index.html
   ```
27. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの``RESTARTSが増えている``ことを確認する。
28. セルフ・ヒーリングされてから60秒ほどたったあとにPodの``READYが1/1``となり作業端末のwebブラウザからService経由で接続すると「Welcome to nginx!」が``表示される``こと。
29. Deployment:probeおよびService:prove-svcを削除

### Pod設定-中級編- (nodeSelector)
Podはマスターコンポーネントでのkube-schedulerによって起動するワーカーノードが決められます。このPodをワーカーノードに割り当てることをK8s用語でスケジュールといいます。Podにスケジュール関連のパラメータが設定されていない場合、kube-schedulerはノードの負荷状況などを考慮してPodをスケジュールします。しかし、どうしても起動先ノードを指定したい場合もあります。（たとえば、GPUを使いたいPodの場合はGPU搭載ノードで起動したいなど）　その様な場合にPodのスケジュール先を指定するもっとも簡単な方法がnodeSelectorです。

1. クラスタの参加ノードを確認し、ワーカーノードが2台以上の状態にせよ
2. ノードに付与されたラベルを表示せよ
3. どれか一台のノードに``node-spec:monster``のラベルを付与せよ
4. 他のノードには``node-spec:futuu``のラベルを付与せよ
5. key:node-specをカラムに追加してノードの一覧で表示せよ。一台のノードにmonsterのvalueがあることを確認せよ
6. 以下を満たすマニフェストを作成しデプロイせよ。なお、nodeSelectorについては[公式ドキュメント](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)を参考にせよ。
   - Deploymet
     - 名前は``node-selector``
     - replicas: ``1``
     - labelはすべて``app: node-selector``
     - Pod
       - メインコンテナ
         - 名前は``nginx``
         - イメージは``nginx:1.12``
       - nodeSelector
         - ラベル``node-spec``にvalue``monster``が付与されたノードを指定
7. Podリソースの一覧をwideで表示し、Podが``node-spec:monster``のラベルを付与したノードで起動していることを確認せよ
8. Deployment:node-selectorのPodレプリカ数を10に変更せよ。新たに作成されたPodもすべて``node-spec:monster``のラベルを付与したノードで起動していることを確認せよ
9. Deployment:node-selectorのnodeSelectorをkey``node-spec``にvalue``futuu``が付与されたノード指定に変更せよ
10. Podのローリングアップデートが実行され、すべてのPodが``node-spec:futuu``のラベルを付与したノードで起動していることを確認せよ
11. Deployment:node-selectorをいったん削除せよ
12. Deployment:node-selectorのnodeSelectorをkey``node-spec``にvalue``super``が付与されたノード指定に変更しデプロイせよ
13. すべてのPodのSTATUSが``Pending``となることを確認せよ
14. Deployment:node-selectorを削除せよ
15. ノードに付与されたkey``node-spec``のラベルを削除せよ

### Pod設定-中級編- (postStart/preStop)


### Namespace
NamespaceはK8sクラスタを論理的に分割した区画です。複数の顧客向けサービスの提供（マルチテナント）や複数チームでK8sクラスタを共有
する時に使います。デフォルトでは「kube-system」や「default」などのNamespaceが用意されています。（K8sサービスによっては他のNamespaceがあるかもしれません）　Namespace単位で使用できるHWリソースの上限を設定したり、Namespace間の通信制御を行うこともできます。ただし、それらを行うには別のK8sリソース（ResourceQuota、NetworkPolicy）を使用します。デフォルトでは同じK8sクラスタ内のNamespace間は自由に通信可能です。実は今まで意識して来ませんでしたが、Namespaceを指定しない場合はdefaultのNamespaceを使用しています。

1. Namespaceリソースのオブジェクト一覧を表示せよ
2. Namespace:kube-systemに属するPodリソースのオブジェクト一覧を表示せよ
3. 以下を満たすマニフェストを作成しデプロイせよ。なお、Namespaceについては[公式ドキュメント](https://kubernetes.io/docs/tasks/configure-pod-container/)を参考にせよ。
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
4. 上記マニフェストをコピー＆修正し以下を満たすマニフェストを作成しデプロイせよ。(変更箇所のみ``ハイライト``にする)
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
5. Namespaceリソースのオブジェクト一覧を表示し、``3-b``と``3-e``が作成されていることを確認せよ。
6. Namespace:defaultのPodおよびServiceリソースのオブジェクト一覧を表示し、上記手順で作成したPodやServiceが``ない``ことを確認せよ。
7. Namespace:3-bのPodおよびServiceリソースのオブジェクト一覧を表示し、``kinpachi``と名の付くPodとService:``sensei``があることを確認せよ。
8. Namespace:3-eのPodおよびServiceリソースのオブジェクト一覧を表示し、``koro``と名の付くPodとService:``sensei``があることを確認せよ。
9. Namespace:3-bのPodに含まれるcurlコンテナからService:senseiに対して「curl -s」を実行し、``The teacher in this class is Kinpachi-sensei``と表示されることを確認せよ
10. Namespace:3-bのPodに含まれるcurlコンテナからNamespace:3-eのService:senseiに対して「curl -s」を実行し、``The teacher in this class is Koro-sensei``と表示されることを確認せよ。なお、別NamespaceのServiceを指定する場合は``<service名>.<namespace名>``と指定すればよい。
11. Namespace:3-eのPodに含まれるcurlコンテナからService:senseiに対して「curl -s」を実行し、``The teacher in this class is Koro-sensei``と表示されることを確認せよ
12. Namespace:3-eのPodに含まれるcurlコンテナからNamespace:3-bのService:senseiに対して「curl -s」を実行し、``The teacher in this class is Kinpachi-sensei``と表示されることを確認せよ。なお、別NamespaceのServiceを指定する場合は``<service名>.<namespace名>``と指定すればよい。
13. Namespace:3-e``だけ``を削除せよ。その後、Namespace:3-eに属していたPodおよびServiceを確認し、``Namespaceと同時に削除された``ことを確認せよ
14. Namespace:3-bを削除せよ
15. すべてのNamespaceのPodおよびServiceリソースのオブジェクト一覧を表示し、作成したオブジェクトがすべて消えていることを確認せよ。

### 

### DaemonSet
DaemonSetは常にすべてのワーカーノードで同じPodを起動するようにするリソースです。クラスタ管理系機能（ログやメトリクスの収集）によく使われます。

1. クラスタの参加ノードを確認し、ワーカーノードが2台以上の状態にせよ
2. 以下を満たすマニフェストを作成しデプロイせよ。なお、DaemonSetについては[公式ドキュメント](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)を参考にせよ。
   - DaemonSet
     - 名前は``daemon``
     - labelはすべて``app: daemon``
     - Pod
       - メインコンテナ
         - 名前は``nginx``
         - イメージは``nginx:1.12``
3. DaemonSetリソースのオブジェクト一覧を表示し、``daemon``があることを確認せよ
4. Podリソースのオブジェクト一覧を起動ノード名も含み表示し、すべてのワーカーノードで``daemon``Podが起動していることを確認せよ
5. ``daemon``Podを一つ削除してからPodリソースのオブジェクト一覧を確認し、セルフヒーリングでPodが再作成されていることを確認せよ
6. DaemonSet:daemonを削除せよ

### StatefulSet
StatefulSetはDeployment（ReplicaSet）と同じく指定した数のPodを常に起動するようにするリソースですが少し動作が異なります。違いとしては以下の3つです。わかりやすい1つ目と2つ目の動作をまず確認します。3つ目はとても大事なのでさらに別の章で確認します。
- 起動するPod名に連番が付与される（Deploymentはランダム文字列）
- 名前の連番通りにPodを1つずつ起動し、前のPodが起動完了してから次のPodを起動する。（Deploymentは起動順を気にしない）
- 各PodごとにPVを割り当てるVolumeClaimTemplateが使える（Deploymentでは使えない）
StatefulSetはDBなどのワークロードを想定したリソースです。たとえば3ノードのDBだとMaster:1,Slave:2の構成などにすると思います。この際、Masterをまず起動し、次いでSlaveを起動します。このような順番を意識した起動はDeploymentではできないため、StatefulSetが用意されています。またHeadless Serviceを使った特定Podへのアクセスを行うこともあります。

1. 以下を満たすマニフェストを作成しデプロイせよ。なお、StatefulSetについては[公式ドキュメント](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)を参考にせよ。
   - StatefulSet
     - 名前は``stateful``
     - labelはすべて``app: stateful``
     - serviceNameは``stateful-svc``
     - replicasは ``3``
     - Pod
       - メインコンテナ1
         - 名前は``nginx``
         - イメージは``nginx:1.12``
       - メインコンテナ2
         - 名前は``curl``
         - イメージは``appropriate/curl``
         - commandは``['/bin/sh','-c','sleep 3600']``
   - Service
     - 名前は``stateful-svc``
     - typeは``ClusterIP``（``明示的に指定する``）
     - clusterIPに``None``
     - Portは``80``
     - selectorは``app: stateful``
2. StatefulSetリソースのオブジェクト一覧を表示し、``stateful``があることを確認せよ
3. Podリソースのオブジェクト一覧を表示し、``stateful-0,1,2``の３つのPodがあることを確認せよ。また起動時間が0,1,2の順で長いことを確認せよ
4. Serviceリソースのオブジェクト一覧を表示し、``stateful-svc``があること。また、``Headless Service``であることを確認せよ。（Headless ServiceはCluster-IPがNoneになっているService）
5. Pod:``stateful-0``に含まれるcurlコンテナに以下の追加コマンドを発行せよ
   ``` sh
   nslookup stateful-svc
   curl stateful-0.stateful-svc.default.svc.cluster.local
   curl stateful-1.stateful-svc.default.svc.cluster.local
   curl stateful-2.stateful-svc.default.svc.cluster.local
   ```
6. Pod:``stateful-1``のIPアドレスを確認してからPod:``stateful-1``を削除せよ。
7. Pod:``stateful-1``がセルフ・ヒーリングされていることを確認せよ。また、IPアドレスが変わっていることを確認せよ
8. Pod:``stateful-0``に含まれるcurlコンテナに以下の追加コマンドを発行せよ
   ``` sh
   nslookup stateful-svc
   curl stateful-1.stateful-svc.default.svc.cluster.local
   ```
9. StatefulSet:statefulおよびService:stateful-svcを削除せよ

上記の様に、Headless Serviceを使うことでPod IPアドレスが変わっても任意のPodにアクセスすることができます。たとえば、StatefulSetで Master:1,Slave:2 の3ノードで構成されたDBを想定します。このDB構成ではwriteはMasterでのみ行うとします。この時、通常のServiceではラウンドロビンで振り分けられるため、writeがSlaveのPodにいってしまうことがあります。Headless ServiceであればMasterを指定して接続することができます。また、単純に特定PodにアクセスするだけならPodのIPアドレスでも可能ですが、StatefulSetで展開しているPodでもセルフヒーリングで再作成されるとIPアドレスは変わってしまいます。Headless Serviceであればセルフヒーリング後も自動でPodを検出するため、宛先の指定を変える必要がありません。

とはいえ、Headless Serviceは少しイレギュラーなServiceです。replicas:2以上のStatefulSet以外で使うことは稀だと思います。replicas:1のStatefulSetやDeploymentの場合は通常のServiceを使うことがほとんどだと思います。

### StatefulSet（VolumeClaimTemplate）
VolumeClaimTemplateはPodごとにPVCおよびPVを作成する、StatefulSetで使える機能です。この機能を理解するため、Deploymentでのボリュームプロビジョニングの特徴をおさらいします。

1. 以下を満たすマニフェストを作成しデプロイせよ。
   - Deployment
     - 名前は``dep-dvp``
     - replicas: ``2``
     - labelはすべて``app: dep-dvp``
     - Pod
       - containerは一つでイメージは``nginx:1.12``
       - volumeプラグインでPVC:``dep-dvp-pvc``を指定
       - 上記で定義したボリュームをコンテナの/usr/share/nginx/html/にマウント
       - postStartで次のコマンドを実行``['/bin/sh','-c','echo $HOSTNAME >> /usr/share/nginx/html/index.html']``
   - PVC
     - 名前は``dep-dvp-pvc``
     - storageClassNameは``指定しない``（デフォルトのStorageClassを使用する）
     - accessModesは``ReadWriteOnce``
     - ストレージ容量は``1Gi``
   - Service
     - 名前は``dep-dvp-svc``
     - タイプは指定なし（ClusterIP）
     - Portは80
     - selectorは``app: dep-dvp``
2. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ``1つずつ``作成されていることを確認せよ。
3. Podリソースのオブジェクト一覧を表示し2つのPodの名前を確認せよ。
5. curlが実行できるコンテナを含むPodをデプロイし、Service``dep-dvp-svc``にcurlせよ。``2つ``のPod名が表示されることを確認せよ
6. Deployment:dep-dvpのreplicaを``5``に拡張せよ。
7. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ``1つのまま``であることを確認せよ。
8. Podリソースのオブジェクト一覧を表示し5つのPodの名前を確認せよ。また、すべてのPodが同じノードで起動していることも確認せよ
9. curlが実行できるコンテナを含むPodからService``dep-dvp-svc``にcurlせよ。``5つ``のPod名が表示されることを確認せよ
10. Deploymet:dep-dvp、PVC:dep-dvp-pvc、Service:dev-dvp-svcを削除せよ

ここまでがDeploymentでのボリュームプロビジョニングのおさらいです。注目するポイントとしてはDeploymentは展開したPodすべてで同じボリュームを共有する点です。そのため、PVCおよびPVは1つしか作られません。ボリュームの中身も当然すべてのPodで共有できています。なので、たとえばWEBサーバなどのワークロードでセッション情報をPod間で共有したいなどと言った場合にはこの構成が有効です。一方で、DBなどのワークロードで各Podが専用のボリュームを確保したい場合には使えません。また、今回はPVCでstorageClassNameを指定しなかったためデフォルトのStorageClassであるEBSのtype:gp2で実際のボリュームは作られています。EBSは単一のEC2インスタンスにしかボリュームを提供できません。そのため、replica数が増えても単一のワーカーノード（EC2）にしかPodがスケジュールできません。このことから、DeploymentのDVPとEBSはあまり相性が良くなく、EFSなどRead/Write Anyできるボリュームが使えるStorageClassを使用した方が良いです。  

つぎに、StatefulSetでVolumeClaimTemplateを使用した場合の挙動を確認します。

1. 以下を満たすマニフェストを作成しデプロイせよ。なお、VolumeClaimTemplateについては[公式ドキュメント](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#components)を参考にせよ。
   - StatefulSet
     - 名前は``ss-vct``
     - labelはすべて``app: ss-vct``
     - serviceNameは``ss-vct-svc``
     - replicasは``2``
     - Pod
       - containerは一つでイメージは``nginx:1.12``
       - ``index``ボリュームをコンテナの/usr/share/nginx/html/にマウント
       - postStartで次のコマンドを実行``['/bin/sh','-c','echo $HOSTNAME >> /usr/share/nginx/html/index.html']``
     - volumeClaimTemplates
       - 名前は``index``
       - storageClassNameは``指定しない``（デフォルトのStorageClassを使用する）
       - accessModesは``ReadWriteOnce``
       - ストレージ容量は``1Gi``
   - Service
     - 名前は``ss-vct-svc``
     - typeは``ClusterIP``（``明示的に指定する``）
     - clusterIPに``None``
     - Portは``80``
     - selectorは``app: ss-vct``
2. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ``2つずつ``作成されていることを確認せよ。
3. Podリソースのオブジェクト一覧を表示し2つのPodの名前を確認せよ。
4. curlが実行できるコンテナを含むPodをデプロイし、以下にcurlし``それぞれのPod名``が表示されることを確認せよ
   - ss-vct-0.ss-vct-svc.default.svc.cluster.local
   - ss-vct-1.ss-vct-svc.default.svc.cluster.local
5. StatefulSet:ss-vctのreplicaを``5``に拡張せよ。
6. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ``5つずつ``作成されていることを確認せよ。
7. Podリソースのオブジェクト一覧を表示し5つのPodの名前を確認せよ。また、Podの起動ノードが分散していることも確認せよ
8. curlが実行できるコンテナを含むPodから以下にcurlし``それぞれのPod名``が表示されることを確認せよ
   - ss-vct-0.ss-vct-svc.default.svc.cluster.local
   - ss-vct-1.ss-vct-svc.default.svc.cluster.local
   - ss-vct-2.ss-vct-svc.default.svc.cluster.local
   - ss-vct-3.ss-vct-svc.default.svc.cluster.local
   - ss-vct-4.ss-vct-svc.default.svc.cluster.local
9. StatefulSet:ss-vctおよびService:ss-vct-svcを削除せよ
10. PVCおよびPVリソースのオブジェクト一覧を表示し、5つ``まだ残っている``ことを確認せよ。
11. PVC:index-ss-vct-0~4を削除せよ。

以上がVolumeClaimTemplateを使用したボリュームプロビジョニングです。Deploymentの時とは違い、各Pod用にPVCおよびPVが作成されました。また、StorageClassはデフォルトのEBSですが、Podのスケジュール先も分散されました。この様にPodごとに専用のボリュームを確保したい時にVolumeClaimTemplateは有効です。また、Podがセルフ・ヒーリングで再作成された場合はそのPodに対応したボリュームが引き続き使用されるます。なお、VolumeClaimTemplateで作成されたPVCおよびPVはStatefulSetを削除しても``消えません``。（データを残すためにあえてこういう仕様になっています。）　もしボリュームが不要になった時は手動で消すのを忘れないようにしましょう。

また、StatefulSetでDeploymetと同じように１つのボリュームをPod間で共有するボリュームプロビジョニングも可能です。

1. 以下を満たすマニフェストを作成しデプロイせよ。
   - StatefulSet
     - 名前は``ss-dvp``
     - replicas: ``2``
     - labelはすべて``app: ss-dvp``
     - serviceNameは``ss-dvp-svc``
     - Pod
       - containerは一つでイメージは``nginx:1.12``
       - volumeプラグインでPVC:``ss-dvp-pvc``を指定
       - 上記で定義したボリュームをコンテナの/usr/share/nginx/html/にマウント
       - postStartで次のコマンドを実行``['/bin/sh','-c','echo $HOSTNAME >> /usr/share/nginx/html/index.html']``
   - PVC
     - 名前は``ss-dvp-pvc``
     - storageClassNameは``指定しない``（デフォルトのStorageClassを使用する）
     - accessModesは``ReadWriteOnce``
     - ストレージ容量は``1Gi``
   - Service
     - 名前は``ss-dvp-svc``
     - typeは``ClusterIP``（``明示的に指定する``）
     - clusterIPに``None``
     - Portは``80``
     - selectorは``app: ss-dvp``
2. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ``1つずつ``作成されていることを確認せよ。
3. Podリソースのオブジェクト一覧を表示し2つのPodの名前を確認せよ。
4. curlが実行できるコンテナを含むPodをデプロイし、以下にcurlせよ。``2つ``のPod名が表示されることを確認せよ
   - ss-dvp-0.ss-dvp-svc.default.svc.cluster.local
   - ss-dvp-1.ss-dvp-svc.default.svc.cluster.local
5. Deployment:dep-dvpのreplicaを``5``に拡張せよ。
6. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ``1つのまま``であることを確認せよ。
7. Podリソースのオブジェクト一覧を表示し5つのPodの名前を確認せよ。また、すべてのPodが同じノードで起動していることも確認せよ
8. curlが実行できるコンテナを含むPodから以下ににcurlせよ。``5つ``のPod名が表示されることを確認せよ
   - ss-dvp-0.ss-dvp-svc.default.svc.cluster.local
   - ss-dvp-1.ss-dvp-svc.default.svc.cluster.local
   - ss-dvp-2.ss-dvp-svc.default.svc.cluster.local
   - ss-dvp-3.ss-dvp-svc.default.svc.cluster.local
   - ss-dvp-4.ss-dvp-svc.default.svc.cluster.local
9. StatefulSet:ss-dvp、PVC:ss-dvp-pvc、Service:ss-dvp-svcを削除せよ。また、curlコンテナを含むPod（およびPodコントローラリソース）も削除せよ

以上のように、Deploymentと同じ様にPVCを指定すれば1つのボリュームをStatefulSetで展開した複数のPodで共有することができます。ただし、EBSボリュームは1つしか作成されておらず、1つのEBSボリュームは1つのEC2インスタンスにしか割り当てられません。そのため、Podが起動するノードはEBSボリュームが割り当てられている１つのワーカーに集約されてしまいます。この様に、EBSのようなブロックストレージボリュームとは相性がよくないのでEFSなどread/write any が可能なボリュームで使うようにしましょう。

========

ここまででだいぶK8sにも慣れてきたと思うので少し手順を簡略化して書いていきます。

========

### ConfigMap（環境変数）
ConfigMapはPodに設定する環境変数やファイルをPodとは切り離して扱うようにするリソースです。まずは環境変数としてのConfigMapの利用を確認します。

Podにパラメータを渡す方法として、Podのenvに指定する方法を紹介しましたが、1つ2つならまだしも、大量の環境変数が必要な場合に1つずつenvを定義するのは大変です。この様な場合、ConfigMapにパラメータを定義しておき、PodからConfigMapを読み込んで環境変数を設定することができます。また、ConfigMapは複数のPodから扱うことができるため、複数Podで使用する環境変数を1つで管理できるメリットもあります。

1. 以下を満たすマニフェストを作成しデプロイせよ。ConfigMapについては[公式ドキュメント](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)を参考にせよ。PodでConfigMapを環境変数として読み込む方法は[公式ドキュメント](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables)を参考にせよ。
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
     - 上記ConfigMapのKey-Valueをすべて環境変数として読み込め
     - メインコンテナのコマンドは以下を指定
       - command: ``['/bin/sh','-c','echo "Star Wars no title ichiran \n $EP1 \n $EP2 \n $EP3 \n $EP4 \n $EP5 \n $EP6 \n $EP7 \n $EP8 \n $EP9"; sleep 3600']``
   - Deployment(2つ目)
     - メインコンテナはshが使用できれば何でも良い
     - 上記ConfigMapのKey-Valueをすべて環境変数として読み込め
     - メインコンテナのコマンドは以下を指定
       - command: ``['/bin/sh','-c','echo "Star Wars no koukai jyun \n $EP4 \n $EP5 \n $EP6 \n $EP1 \n $EP2 \n $EP3 \n $EP7 \n $EP8 \n $EP9; sleep 3600']``
2. 各Deploymentで展開したPodのログを表示せよ
3. ConfigMapの値の誤りを修正し再デプロイせよ。なお、正しいvalueは[参考サイト](https://dic.nicovideo.jp/a/%E3%82%B9%E3%82%BF%E3%83%BC%E3%82%A6%E3%82%A9%E3%83%BC%E3%82%BA)を参考にせよ
4. 各Deploymentで展開したPodのログを表示せよ。表示が変わらないこと。
5. 各Deploymentで展開したPodのログを削除しセルフ・ヒーリングさせる。
6. 各Deploymentで展開したPodのログを表示せよ。表示が変わること
7. ConfigMapとDeployment2つを削除

### ConfigMap（ボリュームマウント）
ConfigMapをマウントすることもできます。たとえばPod（コンテナ）内のとあるファイルの内容を頻繁に書き換えたい場合、コンテナイメージの中にそのファイルがあるとファイル内容を変えてからコンテナイメージをビルドしなおすことになります。または、外部のEFSなどにそのファイルを置いておき、initContainerなどでコンテナ起動前にコピーする方法もあります。いずれの方法でも可能ですが、K8sにおいてもっとも一般的な方法は対象ファイルをConfigMapにすることです。PodからConfigMapをマウントすることでPod外に設定ファイルを保存する事ができます。ファイルの内容を変更したい場合はConfigMapを更新します。

1. 以下を満たすマニフェストを作成しデプロイせよ。
   - Secret
     - 以下の内容が記述されたindex.htmlをdataとして持つ
       - ConfigMap ni kaita naiyou dayo
   - Deployment
     - イメージはnginx:1.12
     - 上記ConfigMapをボリュームとしてマウント
     - マウント先は/usr/share/nginx/html
   - Service
     - 上記DeploymentをClusterIPのPort:80で公開
2. curlが実行可能なPodを展開し、Serviceを指定してcurlを実行する。ConfigMapの内容が表示されること。
3. ConfigMapのindex.htmlの内容を以下に修正しConfigMapを再デプロイせよ
   - ConfigMap wo henkou site mo Pod wo saisakusei sinaito hanei sarenaizo
4. curlが実行可能なPodからServiceを指定してcurlを実行する。ConfigMap修正前の内容が表示されること
5. Deploymentで展開したPodを削除しセルフ・ヒーリングさせる。
6. curlが実行可能なPodからServiceを指定してcurlを実行する。ConfigMap修正後の内容が表示されること
7. ConfigMap、Deployment、curlのPodを削除

なお、ボリュームマウントであるため、マウント先のディレクトリはConfigMap化したファイルのみになってしまう。もしも他のファイルがある場所にファイルを置きたい場合はtempDirのボリュームを作り、initContainerで元のディレクトリ内容をtempDirにコピー、ConfigMapもどこか別の場所にマウントしてからtempDirにコピー、メインコンテナでtempDirをしかるべき場所にマウントすればよい。

### Secret
SecretはConfigMapとほぼ一緒の使い方ができるリソースです。値を環境変数として読み込んだり、ファイルをボリュームマウントしたりできます。違いはdata部分がbase64エンコードされるか否かです。ConfigMapはbase64エンコードされていないため、K8sにデプロイした後にdescribeするとdataの内容がそのまま見えます。Secretはdata部分がbase64でエンコードされるためぱっと見はわかりません。（ただし、base64デコードしてしまえば誰でも見えます。）　ログイン情報など、Pod外で管理したいけどConfigMapの様に値が知られたくない情報を格納するのに使います。（ただし、ただのbase64エンコードなのでbase64デコードすれば誰でも見えてしまいます。）

1. 以下を満たすマニフェストを作成しデプロイせよ。Secretリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/configuration/secret/)を参考にせよ。
   - Secret
     - 以下のKey-Valueをdataとして持つ（以下はbase64デコードされた状態）
       - password: "zettai ni sirarete ha ikenai jyouhou"
   - Deployment
     - 上記Secretのpasswordの値を環境変数PASSWORDに格納
2. 上記Deploymentで展開したPodに「echo $PASSWORD」の追加コマンドを発行せよ。Secretの内容が表示されること
3. Secretの詳細を表示せよ。dataのvalue部分が見えないこと
4. Secretの内容をYAML形式で表示せよ。passwordのvalueが表示されるのでコピーする
5. どこでも良いので以下のコマンドを実行せよ。valueの中身が見えてしまうこと
   echo <前の手順でコピーしたvalue> | base64 --decode
6. SecretとDeploymentを削除

このように、SecretではK8sにアクセス可能な人ならだれでも値が知れてしまうので注意が必要です。また、Gitなどにマニフェストを置く時は万一の流出を考慮してSecretをさらに暗号化する仕組み(kubesec,sealedsecret,vault)を併用しましょう。

### Job
Deployment、DaemonSet、StatefulSetのPodコントローラは``Podの状態を維持すること``を目的としたPodコントローラです。そのため、Pod（コンテナ）も動き続けるようにしたものを使います。JobもPodコントローラの一種ですが少し異なり、``実行させること``を目的としたPodコントローラです。そのため、Pod（コンテナ）はexit 0で終了するようにしたものを使います。たとえばPod内の永続データのバックアップやアクセストークンの更新など、ある時にだけ必要となる処理を実行するのに使います。

1. 以下を満たすマニフェストを作成しデプロイせよ。Jobリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)を参考にせよ。
   - ConfigMap
     - 以下の内容が記述されたbackup.shをdataとして持つ
       ``` sh
       #!/bin/sh
       echo "Ore ha tensai teki na Backup Script."
       echo "imakara $DEST no Backup wo jikkou suru!"

       # Backup command no tumori desu
       curl $DEST >& /dev/null
       
       if [ $? -eq 0 ];then
         echo "Backup seikou! sasuga tensai!"
         exit 0
       else
         echo "Backup sippai... sonna.. bakana."
         exit 1
       fi
       ```
   - Deployment
     - nginxのPodを実行
   - Service
     - 上記DeploymentをClusterIPのPort80で公開
   - Job
     - コンテナイメージはcurlとshが使用可能なものなら何でも良い
     - 上記ConfigMapを適当な場所にマウント
     - 環境変数DESTに上記Serviceの名前を設定
     - commandでマウントしたbackup.shを実行
2. Podリソースのオブジェクト一覧を表示し、STATUS:CompletedになっているPodがあること。
3. STATUS:CompletedになっているPodのログを表示する。"Backup seikou!"が出力されていること
4. Jobリソースのオブジェクト一覧を表示し、COMPLETIONSが1/1になっていること。
5. Jobを削除してから以下の修正を加え、デプロイしなおせ
   - Job
     - 環境変数DESTの値をService名でない値にする
     - job失敗時の再試行回数を3にする
6. 30秒ほどしてからPodリソースのオブジェクト一覧を表示し、STATUS:ErrorになっているPodが3つあること。
7. STATUS:ErrorになっているPod3つのログを表示する。3つとも"Backup sippai"が表示されること
8. Jobリソースのオブジェクト一覧を表示し、COMPLETIONSが0/1になっていること。
9. ConfigMap、Deployment、Service、Jobを削除

### CronJob
CronJobはJobを定期的に自動実行するリソースです。時間はCronと同じ形式で指定します。

1. 以下を満たすマニフェストを作成しデプロイせよ。CronJobリソースについては[公式ドキュメント](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)を参考にせよ。
   - ConfigMap
     - 以下の内容が記述されたbackup.shをdataとして持つ
       ``` sh
       #!/bin/sh
       echo "Ore ha tensai teki na Backup Script."
       echo "imakara $DEST no Backup wo jikkou suru!"

       # Backup command no tumori desu
       curl $DEST >& /dev/null
       
       if [ $? -eq 0 ];then
         echo "Backup seikou! sasuga tensai!"
         exit 0
       else
         echo "Backup sippai... sonna.. bakana."
         exit 1
       fi
       ```
   - Deployment
     - nginxのPodを実行
   - Service
     - 上記DeploymentをClusterIPのPort80で公開
   - CronJob
     - スケジュールは1分間隔
     - コンテナイメージはcurlとshが使用可能なものなら何でも良い
     - 上記ConfigMapを適当な場所にマウント
     - 環境変数DESTに上記Serviceの名前を設定
     - commandでマウントしたbackup.shを実行
     - 成功したPodの保持数を2にせよ
2. デプロイしてから2分くらい待ってからCronJob、Job、Podリソースのオブジェクト一覧を表示せよ。STATUS:CompletedのPodが2つあり、起動時間が約1分ずれていること
3. さらにもう1分以上してからPodリソースのオブジェクト一覧を表示せよ。先程とは違うPod名だがSTATUS:CompletedのPodが2つあり、起動時間が約1分ずれていること。（タイミングによっては3つ表示されるかもしれない。その場合は再度確認すれば2つになっているはず）
4. ConfigMap、Deployment、Service、CronJobを削除

### LimitRange
LimitRangeはNamespace内のPodやPVCに対して設定できるHWリソース量の制限やデフォルト値を設けるリソースです。Namespaceに属します。ResourceQuotaとは違い、Podやコンテナに対して制限を設定します。Pod(コンテナ)のHWリソース量はlimitsとrequestsで指定できますが、LimitRangeはそのlimitsとrequestsで指定できる値の範囲に制限を設けます。制限はコンテナ単位やコンテナを合算したPod単位で設けることができます。また、PVCで指定するボリュームサイズに対しても制限を設けることができます。
さらに、LimitRangeはHWリソース量のデフォルト値を設定することもできます。後述するHPAを使う環境では基本的にすべてのPod（コンテナ）に対してresourcesを設定することが望ましいです。とはいえ、設定し忘れることもあるのでデフォルト値を指定しておけばより安全です。

まずはコンテナのHWリソース量の指定上限について確認します。

1. 以下を満たすマニフェストを作成せよ。（デプロイは次の手順でやる）　LimitRangeリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/policy/limit-range/)を参考にせよ。
   - Deployment
     - メインコンテナ1
       - イメージは何でも良い
       - resourcesで以下のHWリソース量を指定
         - limits
           - cpu: 400m
           - memory: 300Mi
         - requests
           - cpu: 100m
           - memory: 100Mi
   - LimitRange
     - 制限対象はコンテナ
     - max
       - cpuは300m
       - memoryは300Mi
     - min
       - cpuは100m
       - memoryは100Mi
2. 上記マニフェストをデプロイし、Podリソースの一覧を表示せよ。Podが作成されていないことを確認せよ。
3. ReplicaSetリソースの一覧を表示し、上記マニフェストでデプロイしたReplicaSetの詳細情報を表示せよ。以下のようにLimitRangeで指定した範囲に違反したためPodの作成に失敗した旨のメッセージを確認する。
   ```
   Error creating: pods "resources-6bf4ffd649-b9hb2" is forbidden: maximum cpu usage per Container is 300m, but limit is 400m.
   ```
4. Deploymentを削除し、以下のようにマニフェストを修正しデプロイせよ。（変更箇所を``ハイライト``にする）
   - Deployment
     - メインコンテナ1
       - イメージは何でも良い
       - resourcesで以下のHWリソース量を指定
         - limits
           - cpu: ``300m``
           - memory: 300Mi
         - requests
           - cpu: 100m
           - memory: ``10Mi``
5. Podリソースの一覧を表示せよ。Podが作成されていないことを確認せよ。
6. ReplicaSetリソースの一覧を表示し、上記マニフェストでデプロイしたReplicaSetの詳細情報を表示せよ。以下のようにLimitRangeで指定した範囲に違反したためPodの作成に失敗した旨のメッセージを確認する。
   ```
   Error creating: pods "resources-787f48c74f-2t5rd" is forbidden: minimum memory usage per Container is 100Mi, but request is 10Mi.
   ```
7. Deploymentを削除し、以下のようにマニフェストを修正しデプロイせよ。（変更箇所を``ハイライト``にする）
   - Deployment
     - メインコンテナ1
       - イメージは何でも良い
       - resourcesで以下のHWリソース量を指定
         - limits
           - cpu: 300m
           - memory: 300Mi
         - requests
           - cpu: 100m
           - memory: ``100Mi``
8. Podリソースの一覧を表示せよ。Podが``作成されている``ことを確認せよ。
9. DeploymentとLimitrangeを削除する。

このようにLimitRangeでPod内の各コンテナに対し、指定できるHWリソース量の範囲を制限することができる。  
次に、Podに対するHWリソース量の範囲制限について確認する。

1. 以下を満たすマニフェストを作成せよ。（デプロイは次の手順でやる）　LimitRangeリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/policy/limit-range/)を参考にせよ。
   - Deployment
     - メインコンテナ1
       - イメージは何でも良い
       - resourcesで以下のHWリソース量を指定
         - limits
           - cpu: 200m
           - memory: 200Mi
         - requests
           - cpu: 100m
           - memory: 100Mi
     - メインコンテナ2
       - イメージは何でも良い
       - resourcesで以下のHWリソース量を指定
         - limits
           - cpu: 100m
           - memory: 100Mi
         - requests
           - cpu: 100m
           - memory: ``10Mi``
   - LimitRange
     - 制限対象はPod
     - max
       - cpuは400m
       - memoryは300Mi
     - min
       - cpuは200m
       - memoryは200Mi
2. 上記マニフェストをデプロイし、Podリソースの一覧を表示せよ。Podが作成されていないことを確認せよ。
3. ReplicaSetリソースの一覧を表示し、上記マニフェストでデプロイしたReplicaSetの詳細情報を表示せよ。以下のようにLimitRangeで指定した範囲に違反したためPodの作成に失敗した旨のメッセージを確認する。
   ```
   Error creating: pods "resources-65fb6d7459-kkg7c" is forbidden: minimum memory usage per Pod is 200Mi, but request is 115343360.
   ```
4. Deploymentを削除し、以下のようにマニフェストを修正しデプロイせよ。（変更箇所を``ハイライト``にする）
   - Deployment
     - メインコンテナ1
       - イメージは何でも良い
       - resourcesで以下のHWリソース量を指定
         - limits
           - cpu: 200m
           - memory: 200Mi
         - requests
           - cpu: 100m
           - memory: 100Mi
     - メインコンテナ2
       - イメージは何でも良い
       - resourcesで以下のHWリソース量を指定
         - limits
           - cpu: 100m
           - memory: 100Mi
         - requests
           - cpu: 100m
           - memory: ``100Mi``
8. Podリソースの一覧を表示せよ。Podが``作成されている``ことを確認せよ。
9. DeploymentとLimitRangeを削除する。

このようにLimitRangeでPodのHWリソース量の範囲を制限することができる。  Podの場合はPodに含まれるすべてのコンテナを合算した値が比較対象となる。  
次に、デフォルトのHWリソース量設定に確認する。なお、デフォルトのHWリソース量設定はコンテナに対してのみ設定することができる。

1. 以下を満たすマニフェストを作成しデプロイせよ。LimitRangeリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/policy/limit-range/)を参考にせよ。
   - Deployment
     - メインコンテナ1
       - イメージは何でも良い
       - resourcesでHWリソース量を``指定しない``
   - LimitRange
     - 制限対象はコンテナ
     - デフォルトlimits
       - cpuは300m
       - memoryは300Mi
     - デフォルトrequests
       - cpuは100m
2. Podの詳細を確認し、limitsとrequestsが設定されていることを確認せよ。
3. DeploymentとLimitRangeを削除する。

このようにLimitRangeでコンテナのデフォルトresourcesを設定することができます。デフォルトlimitsのみでrequestsを指定しない場合はlimitsと同じ値が設定されます。（resourceの時と同じです。）  

次に、PVCに対する制限について確認します。

1. 以下を満たすマニフェストを作成せよ。（デプロイは次の手順でやる）　LimitRangeリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/policy/limit-range/)を参考にせよ。
   - Deployment
     - メインコンテナ1
       - イメージは何でも良い
       - DVPでボリュームを適当なディレクトリにマウント
   - PVC
     - storageClassNameは指定しない(デフォルトStorageClassを使用)
     - 容量は5Gi
   - LimitRange
     - max: 2Gi
     - min: 1Gi
2. 上記マニフェストをデプロイせよ。以下のようにPVCのサイズがLimitRangeで指定した範囲に違反したメッセージが表示されること。
   ```
   persistentvolumeclaims "limitrange-pvc" is forbidden: maximum storage usage per PersistentVolumeClaim is 2Gi, but request is 5Gi.
   ```
3. 以下のようにマニフェストを修正しデプロイせよ。（変更箇所を``ハイライト``にする）
   - PVC
     - storageClassNameは指定しない(デフォルトStorageClassを使用)
     - 容量は``1Gi``
4. Deployment、PVC、LimitRangeを削除せよ。

最後に少し変わったHWリソース範囲の指定方法を紹介する。``maxLimitRequestRatio``という指定方法である。これは具体的な値で制限していた今までの制限方法とは異なり、``LimitsとRequestsの比``で制限を設ける指定方法である。今回は動作の確認まで行わないが詳しくは[公式ドキュメント](https://kubernetes.io/docs/concepts/policy/limit-range/#limits-requests-ratio)を参考にすること。

以上のようにLimitRangeでHWリソース量の制限やデフォルト値を設定することができる。LimitRangeでとく大事なのはHWリソース量のデフォルト値指定の方です。これによりresourceの設定忘れを防ぐことができます。制限については後述するResourceQuotaの方が設定しやすいかも知れない。

### ResourceQuota
ResourcesQuotaはNamespaceに対しHWリソース上限や作成できるK8sのオブジェクト数に制限を設けるリソースです。Namespaceに属します。LimitRangeとは違い、Namespaceに対する制限です。

まずはHWリソース量の制限について確認します。

1. 以下を満たすマニフェストを作成しデプロイせよ。ResourceQuotaリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/policy/resource-quotas/)を参考にせよ。
   - Deployment
     - イメージは何でもよい
     - replicas: 1
     - resources.limitsに以下を設定
       - cpu: 200m
       - memory: 100Mi
     - reqources.requestsに以下を設定
       - cpu: 100m
       - memory: 50Mi
   - ResourceQuota
     - cpuの制限を1core
     - memoryの制限を1G
2. デプロイしたResourceQuotaオブジェクトの詳細を表示せよ。現在の利用量が表示される。（利用量はrequestsで計算される）
3. Deploymentのreplica数を10に拡張せよ
4. ResourceQuotaオブジェクトの詳細を表示せよ。現在の利用量が表示される。（利用量はrequestsで計算される）
5. Deploymentのreplica数を11に拡張せよ。
6. Podのオブジェクト一覧を表示せよ。Podの数が10のままであること。
7. ReplicaSetの詳細を表示し、以下のようにCPUの利用量がResourceQuotaで指定した範囲に違反したメッセージが表示されること。
   ```
   Error creating: pods "quota-5c4f499fb8-nxfqb" is forbidden: exceeded quota: quota, requested: cpu=100m, used: cpu=1, limited: cpu=1
   ```
8. DeploymentとResourceQuotaを削除せよ。

以上のようにNamespaceに対してHWリソース量の制限を設けることができます。ちなみに、ResourceQuotaで制限を設けている状態でrequestsを指定しないPodを起動させようとしても起動できません。（興味があればやってみてください。）

次にK8sリソースのオブジェクト数の制限について確認します。

1. 以下を満たすマニフェストを作成しデプロイせよ。ResourceQuotaリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/policy/resource-quotas/)を参考にせよ。
   - Deployment
     - イメージは何でもよい
     - replicas: 1
   - ResourceQuota
     - Podの制限を5
2. デプロイしたResourceQuotaオブジェクトの詳細を表示せよ。現在の利用量が表示される。
3. Deploymentのreplica数を5に拡張せよ
4. ResourceQuotaオブジェクトの詳細を表示せよ。現在の利用量が表示される。（利用量はrequestsで計算される）
5. Deploymentのreplica数を6に拡張せよ。
6. Podのオブジェクト一覧を表示せよ。Podの数が5のままであること。
7. ReplicaSetの詳細を表示し、以下のようにCPUの利用量がResourceQuotaで指定した範囲に違反したメッセージが表示されること。
   ```
   replicaset-controller  Error creating: pods "quota-68847df66-bplgc" is forbidden: exceeded quota: quota, requested: count/pods=1, used: count/pods=5, limited: count/pods=5
   ```
8. DeploymentとResourceQuotaを削除せよ。

以上のようにNamespace内のK8sオブジェクト数に制限を設けることができます。

Namespace単位で制限をかけられるのでLimitRangeよりも手っ取り早くリソース制限をかけることができます。しかし、各オブジェクト単位のリソース量制限はできためその場合はLimitRangeを使います。ResourceQuotaはたとえば1つのK8sクラスタを複数のチームで共有利用する場合などに使える機能です。しかし、ResourceQuotaを設定していなくても結局のところワーカーノードのHWリソースがクラスタとしての上限であるため、共有利用していないK8sクラスタの場合はあまり使用することはないと思います。

### RBAC関連
K8sにはRBACの仕組みがあります。ユーザまたはK8s内のオブジェクトに対して権限を付与するServiceAccountがあります。ServiceAccountはK8sのリソースですが、ユーザはK8sのリソースではありません。ユーザはK8s外部の仕組みで認証され、K8sにアクセスします。ユーザやServiceAccountに許可する操作権限は4つのリソースで管理します。Namespaceに属するRole/RoleBinding、Namespaceに属さないClusterRole/ClusterRoleBindingです。これらのRBACはクラスタ操作権限を制限したい場合（たとえば、admin権限と参照権限だけ持つユーザを分ける、操作できるNamespaceの範囲を絞るなど）やK8sクラスタ内のPodからkube-apiserverへ通信（kubectl）したい場合などに用います。それ以外の場合としてはServiceAccountに対してデフォルトのimagePullSecretsを設定し、各Pod定義からimagePullSecretsの設定を省略したいときにServiceAccountを使います。ちなみに、Podは何かしらのServiceAccountで実行されますが、特に指定しない場合はそのPodが起動するNamespace内にあるdefaultと言う名前のServiceAccountで実行されます。デフォルトではServiceAccount:defaultは何も権限が付与されていないため、kube-apiserverへの通信は行なえません。

1. 以下を満たすマニフェストを作成しデプロイせよ。
   - Deployment
     - イメージは``bitnami/kubectl``
     - replicas: 1
     - commandは["/bin/sh","-c","sleep 3600"]
2. デプロイしたPodに「kubectl get pod」の追加コマンドを発行し以下のようなメッセージで失敗すること
   ```
   Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:default" cannot list resource "pods" in API group "" in the namespace "default"
   ```
3. 以下を満たすマニフェストを作成しデプロイせよ。ServiceAccountリソースについては[公式ドキュメント](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)を参考にせよ。Roleリソースについては[公式ドキュメント](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole)を参考にせよ。RoleBindingリソースについては[公式ドキュメント](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)を参考にせよ。
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
4. 作成したServiceAccount,Role,RoleBindingを確認せよ
5. Deploymentのマニフェストを修正しServiceAccount:get-podでPodを起動するように修正しデプロイし直せ
6. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが``実行できる``こと
7. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行し以下のようなメッセージで失敗すること
   ```
   Error from server (Forbidden): deployments.extensions is forbidden: User "system:serviceaccount:default:get-pod" cannot list resource "deployments" in API group "extensions" in the namespace "default"
   ```
8. Deploymentのマニフェストを修正しServiceAccount:get-deployでPodを起動するように修正しデプロイし直せ
9. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが``実行できない``こと
10. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行しコマンドが``実行できる``こと
11. 1セット目のRoleBindingのマニフェストを修正し、紐付けるServiceAccountをget-podからget-deployに変更しデプロイし直せ
12. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが``実行できる``こと
13. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行しコマンドが``実行できる``こと
14. 作成したDeployment、ServiceAccount、Role、RoleBindingをすべて削除する。

## 上級
今まではデフォルトのK8sでも使用できる標準的なリソースについて触れてきた。ここからはK8sにアドオンを追加しさらに実践的な機能について触れていく。

出てくるキーワード
- K8sリソース
  - Horizontal Pod Autoscaler
  - Ingress
  - NetworkPolicy
  - StorageClass
- Pod設定
  - postStart/preStop
  - Podスケジュール(affinity,taint/toleration)
- AddOn
  - metrics server
  - Cluster Autoscaller
  - descheduler
  - Ingress Controller
  - Calico
  - Kustomize
  - Helm

### metrics server
metrics serverはPodやワーカーノードのHWリソース量(CPU/メモリ)を取得するアドオン機能です。metrics serverはK8s内にPodとして起動させます。metrics serverを導入すると「kubectl top」コマンドが使用できるようになります。Pod数を自動で増減させるHorizontal Pod Autoscalerを使用するにはmetrics serverが必要となってきます。（なお、EKSの場合は利用者でデプロイが必要だが、AKSやGKEではマネージド・サービスの中でデプロイされている）

1. metrics serverをデプロイせよ。[公式ドキュメント](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/#metrics-server)を参考にせよ。
2. metrics serverが起動してから情報収集まで1分ほど時間がかかる。時間が経ってもkubectl topで結果がうまく出力されず、metrics serverのログに``unable to fetch node metrics for node ~``や``"unable to fetch node metrics for pod ~"``が出力されている場合はデバッグが必要になる。インターネットを調べてデバッグせよ。
3. 以下コマンドを実行しmetrics serverでメトリクスの取得ができていることを確認せよ。
   ``` sh
   kubectl top node
   kubectl top pod -n kube-system
   ```

### Horizontal Pod Autoscaler
Horizontal Pod Autoscaler（以下、HPA）はPodのHWリソース使用量を基にPodコントローラのreplica数を自動で増減させるリソースです。HPAを動かすにはmetrics serverが必要です。（EKSの場合は利用者でデプロイ、AKS/GKEの場合はマネージドサービスでデプロイ済）　また、Pod（コンテナ）にresources.requestsを指定していることも条件になります。HPAは目標の負荷状態状態を宣言します。この目標値に近づくようにPodの数量を変化させます。ターゲットとなるPod群の実際に消費しているHWリソース量の平均値をmetrics serverを使って計算します。この計算した値を目標値で割ることでどれくらいの数量のPodを追加/削除するか決定します。計算式で表すと以下の通りとなります。
```
desiredReplicas = ceil[currentReplicas * ( currentMetricValue / desiredMetricValue )]
```

1. 以下を満たすマニフェストを作成しデプロイせよ。HPAについては[公式ドキュメント](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)を参考にせよ。
   - Deployment
     - replica: 1
     - image:nginx: 1.12
     - resources.limits.cpu: 100m
   - Service
     - 上記Deploymentをtype:ClusterIPで公開
     - Portは80
   - HPA
     - 上記Deploymentを対象にする
     - Pod数を最小1 ~ 最大5の範囲とする
     - CPU使用率80%を目標値とする
2. 作成したオブジェクトを確認せよ。Podが１つであること。
3. 以下を満たすマニフェストを作成しデプロイせよ。
   - Deployment
     - image: httpd
4. 上記作成したPodに以下の様な追加コマンドを発行せよ。(宛先は作成したService名にする。最後の/を忘れずに。-nと-cの値はお好みで)
   ``` sh
   ab -n 1000000 -c 100 http://hpa/
   ```
5. 別ターミナルを開き、「kubectl top pod」と「kubectl get pod」をwatch等で監視する。しばらくするとPodの数が増えることを確認する。Podが5つに自動拡張されることを確認する。すべてのPodの負荷が80%(80m)を超えていてもPodが5つ以上増えないことも確認する。
6. 作成したオブジェクトをすべて削除する。


### Cluster Autoscaller
Cluster Autoscaller（以下、CA）は自動でワーカーノード数を調整するアドオン機能です。CAはK8s内にコントローラの役割を担うPodをデプロイして機能します。スケールアウトはHWリソースが足りずにどこにもスケジュールできないPod（STATUS:Pending）が発生した際に行われます。スケールインはクラスタ全体のHWリソース割当量を鑑みて行われます。なお、CAはEKS/AKS/GKEなど対応したクラウドプロバイダでないと使用できません。また、ワーカーノードをAWS AutoScalingなどノード台数を柔軟に変更できる機能で展開していないと使用できません。

1. ワーカーノードに付与されているIAMロールをマネジメントコンソール等で確認する。以下のようにAutoScalingGroupの権限がついていること。ついていない場合はワーカーのIAMロールにポリシーを追加すること。[ネタ元](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md)
   ``` json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "autoscaling:DescribeAutoScalingGroups",
                   "autoscaling:DescribeAutoScalingInstances",
                   "autoscaling:DescribeLaunchConfigurations",
                   "autoscaling:SetDesiredCapacity",
                   "autoscaling:TerminateInstanceInAutoScalingGroup"
               ],
               "Resource": "*"
           }
       ]
   }
   ```
2. AutoScalingGroupのワーカーノードのタグを確認する。以下の様なタグが付与されていること。付与されていない場合は付与すること。なお、タグはKeyのみでValueは不要。
   - k8s.io/cluster-autoscaler/enabled
   - k8s.io/cluster-autoscaler/<クラスタ名>
3. (https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml)のマニフェストをコピーする。
4. 上記コピーしたマニフェストの\<YOUR CLUSTER NAME\>の箇所を修正しapplyする。
5. ClusterAutoscalerのPodが起動していることを確認する。
6. 以下を満たすマニフェストを作成しデプロイせよ。
   - Deployment
     - イメージは何でもよい
     - resource.limits.cpu: 1000m
7. 上記作成したDeploymentのreplica数を１つずつ増やしていき、STATUS:PendingのPodが出るまで増やせ。
8. 「kubectl get node」および「kubectl get pod」をwatch等で監視し、Nodeが増えてSTATUS:PendingのPodがrunningになることを確認せよ。(PendingのPodが生まれてからEC2インスタンスを立ち上げるため数分かかる)
9. 上記作成したDeploymentのreplica数を１にせよ
10. 「kubectl get node」および「kubectl get pod」をwatch等で監視し、Nodeが減ることを確認せよ。(デフォルトではスケールアウト後のスケールインは10分以上経過しないと行われないため10分ほど待つ。時間が知りたい場合はCAのPodのlogを確認し「ノード名 was unneeded for 時間」が10分に達するとスケールインが始まる)
11. CA用のオブジェクト一式とDeploymentを削除せよ

### descheduler
deschedulerはクラスタを監視し、ルールに基づきPodを削除する機能です。この機能は削除されたPodがセルフ・ヒーリングで再配置されることを利用した機能です。deschedulerはK8s内にコントローラの役割を担うPodをデプロイして機能します。  
Podの起動先ノードを決めることをK8sではスケジュールと言います。このスケジュールはマスターコンポーネントであるkube-schedulerにより行われます。スケジュールはPodが起動するタイミングでしか行われません。たとえばノード障害などが発生し、あるノードで動いていたPodが別のノードに退避されたとします。その後、ノードを再度クラスタに参加させてもそのノードには何もPodが起動されません。（もし、ノードが参加した時点でPending状態のPodがあれば起動されます。）　PodのスケジュールはPod起動時にしか行われないため、このようにあとからノードを追加しても思うように負荷が分散されていない状況が起こります。これを防止する機能がdeschedulerです。deschedulerは定期的にクラスタを監視し、ルールに基づいてPodを削除します。Podを削除するとセルフ・ヒーリングされます。セルフヒーリング時にマスターコンポーネントのkube-shedulerによりノードの負荷状況などを考慮して再スケジュールが行われます。

1. ワーカーノード2台の状態から作業する。
2. 以下を満たすマニフェストを作成しデプロイせよ。
   - Deployment
     - イメージは何でもよい
     - replicas:10
3. 各Podが起動しているノードを一覧で表示し、2つのノードに分散配置されていることを確認せよ
4. どちらかのノードを停止せよ。（停止の仕方は何でも良い）
5. ノード停止から1分ほどしてからPodの起動ノードを確認せよ。1つのノードに片寄って配置されていることを確認せよ。
6. ワーカーノードを1台クラスタに再参加させよ。(ワーカーをAutoScalingGroupで構成している場合しばらく待っていれば自動で追加してくれる)
7. ワーカー2台の状態に戻ってもPodの起動ノードが片寄った状態のままで有ることを確認せよ
8. 以下コマンドで必要なマニフェストが含まれるgitリポジトリをクローン
   ``` sh
   git clone https://github.com/kubernetes-sigs/descheduler.git
   ```
9. 以下コマンドで必要なマニフェストをapply
   ``` sh
   kubectl create -f kubernetes/rbac.yaml
   kubectl create -f kubernetes/configmap.yaml
   kubectl create -f kubernetes/cronjob.yaml
   ```
10. 2分ほどたってからPodの起動ノードを確認し、2ノードに分散されていることを確認せよ。
11. デプロイしたDeploymentやdescheduler関連のオブジェクトを削除せよ

### Ingress Controller
IngressはServiceの外部公開を制御するリソースおよびアドオン機能です。L7ロードバランサの様なものです。Ingressはその動作をコントロールするIngress ControllerをPodとしてクラスタにアドオンします。さらに、Ingress Controllerの動作設定を行うIngressリソース（こちらはK8sのリソース）を組み合わせて使います。

Kubernetes外からアクセスを受ける方法として、Service Type:LBを以前紹介しました。Service Type:LBはとても便利ですが問題もあります。それはServiceごとにクラウドのLBができてしまう点です。たくさんのServiceを外部に公開したい場合、その数だけLBができてしまうのは管理、コストの面で問題となりえます。また、他の外部公開の方法としてService Type:NodePortという手段もありますが、この公開方法だとServiceの公開ポートで同じ番号が使えず、よく使われる80や443で公開できるServiceの数が限られてしまいます。

Ingressを使うとこれらの問題を解決する事ができます。外部への公開はすべてIngress用のService Type:LBが受け、Ingress Controllerに集約させます。Ingress ControllerはリクエストのURLを見てしかるべきServiceにリクエストを流します。このURLと転送先Serviceの設定はIngressリソースで定義します。

Ingress Controllerにはいくつかの種類があります。K8sがサポートしておりどのクラウドでも利用ができるのはNginx Ingress Controllerです。

1. Nginx Ingress Controllerをデプロイせよ。なお、[Nginx Ingress Controllerの公式](https://kubernetes.github.io/ingress-nginx/deploy/)を参考にせよ。なお、Serviceは「service-l7.yaml」および「patch-configmap-l7.yaml」を使用せよ。
2. AWSマネジメントコンソール（コマンドでも化）でNginx IngressのELBが作成されていることを確認せよ。ELBのタグを見ればNginx Ingressのものか判断できる。
3. Nginx Ingress用LBにアタッチされているセキュリティグループを確認せよ
4. ワーカーノードのセキュリティグループを確認し、Nginx Ingress用LBのセキュリティグループからのインバウンドが許可されていることを確認せよ。

### Ingress
Ingressを使い複数のサービスをhttpで公開してみる。

1. 以下を満たすマニフェストを作成しデプロイせよ。Ingressリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/services-networking/ingress/)を参考にせよ。
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
2. インターネットに接続可能でcurlが実行できる端末から以下コマンドを発行し、それぞれのServiceにIngressを経由してアクセスできていることを確認せよ。(社内プロキシ経由のアクセスだとリクエストがキャッシュされて表示が変わらない可能性がある。社内プロキシを通らない経路で試してみるとうまくいくかも)
   ``` sh
   curl -HHost:test-1.k8s.practice http://<nginx ingress用LBのDNS名>
   curl -HHost:test-2.k8s.practice http://<nginx ingress用LBのDNS名>
   ```
3. Nginx Ingress Controllerのログを確認し、上記2つのリクエストがIngress Controllerを経由していることを確認する。
4. （余裕があればやる）上記手順ではヘッダにホスト名を入れて実行したが、本来ならホスト名である「test-1.k8s.practice」および「test-2.k8s.practice」はRoute53などのDNSにレコードを追加して動作を確認するのが正しい。VPC内部限定のドメインで「.k8s.practice」を作成し、「*.k8s.practice」の宛先をNginx Ingress用LBとして以下のコマンドをVPC内のEC2インスタンス等から実行する。
   ``` sh
   curl http://test-1.k8s.practice
   curl http://test-2.k8s.practice
   ```

続いてhttpsの場合も確認する。TLSの終端はIngress用のService Type:LBで行う。証明書はオレオレを作成する。

1. 以下を実行してオレオレ証明書を作成せよ。
   ``` sh
   openssl req -x509 -sha256 -nodes -newkey rsa:2048 -subj '/CN=k8s.practice' -keyout k8s.practice.key -out k8s.practice.crt
   ```
2. 作成した「k8s.practice.key」と「k8s.practice.crt」の内容を表示せよ。（ACMに登録するのに使う）
3. AWS ACMにオレオレ証明書を登録せよ。
4. Nginx Ingress用のServiceを改良し、TLSで使用する証明書を設定せよ。（ヒント：証明書はACMのarnを指定する。）
5. 以下コマンドでhttpsで通信できることを確認せよ。（``httpではない``）
   ``` sh
   curl -k -HHost:test-1.k8s.practice https://<nginx ingress用LBのDNS名>
   curl -k -HHost:test-2.k8s.practice https://<nginx ingress用LBのDNS名>
   ```
6. 作成したtest-1、test-2関連のオブジェクトとNginx Ingress Controller関連のオブジェクトをすべて削除せよ。 


### NetworkPolicy
NetworkPolicyはK8sクラスタ内のセキュリティグループの様なものです。指定したラベルが付与されたPodやNamespaceに対するインバウンドおよびアウトバウンドの通信制限を設けることができます。デフォルトの状態では同一クラスタ内のPodはNamespaceが違っても制限なしに相互にアクセス可能です。たとえばNamespaceで顧客ごとに分割をしていても、ネットワークがアクセス可能だとあまり意味がありません。その様な場合にNetworkPolicyでアクセスを制限すればよりセキュアな構成にすることができます。  
なお、NetworkPolicyはK8sの標準リソースですが、使用するにはクラスタのネットワークが対応したCNIプラグインで構成されている必要があります。たとえばEKSの場合、デフォルトでは「Amazon VPC CNI plugin for Kubernetes」を使用していますが、2020/03時点で「Amazon VPC CNI plugin for Kubernetes」はNetworkPolicyに対応していません。（たしかGKEとAKSは標準のCNIで対応しているはず。）　NetworkPolicyに対応したCNIプラグインとしてはCalicoなどがあります。

**EKSの場合**  
まずはCalicoをクラスタにデプロイする。[公式の手順](https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/calico.html)を参考に作業する。

1. 以下コマンドを実行する
   ``` sh
   kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/release-1.5/config/v1.5/calico.yaml
   ```
2. 以下コマンドを実行し、Calicoがデプロイされたことを確認する。（calico-nodeとcalico-typhaがnodeの数だけデプロイされすべてRunningすればOK）
   ``` sh
   kubectl get daemonset calico-node --namespace kube-system
   kubectl get pod --namespace kube-system
   ```

続いてNamespace単位のNetworkPolicyについて確認する。

3. 以下を満たすマニフェストを作成しデプロイせよ。
   - 1セット目
     - Namespace
       - 1セット目とわかる名前にする。
       - 1セット目とわかるラベルをつける
     - Deployment
       - １つ目
         - namespaceは1セット目
         - 1つ目のDeployment固有のラベルをつける
         - nginx:1.12のコンテナ
         - /usr/share/nginx/html/index.htmlの内容を1つ目のアプリケーションであることがわかる内容に書き換える
       - 2つ目
         - namespaceは1セット目
         - 2つ目のDeployment固有のラベルをつける
         - curlが実行できるコンテナ
     - Service
       - namespaceは1セット目
       - 上記1つ目のDeploymentをClusterIPのPort:80で公開
   - 2セット目
     - 1セット目と基本同じでNamespaceの名前、ラベルとindex.htmlの内容を変える
4. 各NamespaceのPod:curlから各NamespaceのService:nginxに``通信できる``ことを確認せよ。（計4回curlする。他Namespaceへcurlする場合、アドレスにはNamespace名が必要になる）
5. 以下を満たすマニフェストを作成しデプロイせよ。NetworkPolicyリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/services-networking/network-policies/)を参考にせよ。
   - 1セット目
     - NetworkPolicy
       - 対象は同じnamespaceのPodすべて
       - 1セット目のNamespaceからのインバウンド通信を許可する。（言い方を変えると1セット目のNamespace以外からの通信を許可しない）
   - 2セット目
     - 1セット目と基本同じで2セット目のNamespaceからのインバウンド通信のみを許可する。
6. 各NamespaceのPod:curlから各NamespaceのService:nginxに``curl -s -m 10``で通信せよ。（-mでタイムアウトを短くしている）　Namespaceを跨いだ通信は失敗することを確認せよ。
7. デプロイしたオブジェクトをすべて削除せよ。

上記のようにNetworkPolicyで通信制御をNamespaceにかけることができる。さらに、同じNamespace内でもPodごとに通信制御をかけることもできる。

1. 1セット目のマニフェストを以下のように修正しデプロイせよ。
   - Deployment
     - 3つ目
       - 3つ目のDeployment固有のラベルをつける
       - curlが実行できるコンテナ
   - NetworkPolicy
     - 対象は1つ目のDeployment
     - 3つ目のDeploymentのみ通信を許可する。
2. 2つ目と3つ目のDeploymentでデプロイしたPodからService:nginxに対して通信せよ。3つ目でデプロイしたPodからのみアクセスできることを確認せよ。（もしできてしまう場合、同一Namespaceからの通信を許可するNetworkPolicyが残っていないか確認し、残っていれば削除する）
3. デプロイしたすべてのオブジェクトを削除する。Calicoも削除する。

### StorageClass
以前、DynamicVolumeProvisioning(DVP)に触れた時にはデフォルトのStorageClassを使用してDVPを行いました。StorageClassはDVPの際に使用するストレージの種類を定義したリソースです。EKSの場合はEBS(gp2)のStorageClassがデフォルトで定義されています。EBS(gp2)以外のディスクタイプを使用してDVPを行うには追加でStorageClassを定義し、PVCでそのStorageClassを指定すれば良いです。なお、StorageClassとして定義可能なストレージサービスには制限があります。Provisionerというストレージサービスと連携をとるための機能が提供されているもののみになります。K8sにはデフォルトでAWS EBSやAzure Diskなど代表的なストレージサービスのProvisionerが組み込まれています。デフォルトで組み込み済のProvisionerは[公式ドキュメント](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner)で確認できます。組み込まれていないストレージサービス(たとえばAWS EFSなど)を使用してDVPしたい場合は利用者でProvisonerを導入してからStorageClassを定義します。なお、DVPしない場合はProvisonerおよびStorageClassは特に必要なく、Podの定義からvolumeプラグインを使用してマウントすれば良いです。

1. 以下を満たすマニフェストを作成しデプロイせよ。StorageClassリソースについては[公式ドキュメント](https://kubernetes.io/docs/concepts/storage/storage-classes/)を参考にせよ。なお、デプロイする際は必ずStorageClassを最初にデプロイすること。（他は適当で良い。）
   - Deployment
     - イメージは何でもよい
     - volumeMountsで2つのボリュームを別々の適当なpathにマウントせよ
     - volumeで以下2つのpvcをそれぞれ指定せよ。
       - normal-disk-pvc
       - fast-disk-pvc
   - PVC
     - normal-disk-pvc
       - storageClassNameはデフォルト
       - readwriteonce
       - 容量1G
     - fast-disk-pvc
       - storageClassNameはio1
       - readwriteonce
       - 容量4G
   - StorageClass
     - 名前はio1
     - ebsのprovisionerを使用
     - reclaimPolicyはDelete
2. 作成したオブジェクトを確認せよ。なお、StorageClassのgp2はデフォルトで作成されているものである。
3. マネジメントコンソールからEBSを確認し、gp1とio1のボリュームがあることを確認せよ。
4. デプロイしたオブジェクトをすべて削除せよ。

### Kustomize
Kustomizeはマニフェスト管理に役立つコマンドラインツールである。v1.14のkubectlからは標準で使用することができる。K8sを複数の環境（開発/ステ/本番等）で使用する場合を考える。多くの場合、環境間では差異が生じる。（環境変数のドメイン名が違う。HWリソース量が違う。など）　差異がある場合は各環境ごとにマニフェストを用意することになるが、修正が発生した場合にすべての環境のマニフェストをいちいち修正するのは手間である。そこでKustomizeを使用すると、ベースとなるマニフェストを作成しておき、環境差異が生じる部分だけ各環境で上書きするということができる。

1. 以下の構造のkustomizeディレクトリを作成せよ
   ```
   kustomize
   ├── base
   └── overlay
       ├── dev
       └── prod
   ```
2. kustomize/base配下に以下を満たすマニフェストを作成せよ。（applyはしなくてよい）
   - Deployment
     - replica:1
     - コンテナイメージはnginx:1.12
     - ConfigMap:env-configのENVを環境変数ENVに格納
   - Service
     - 上記DeploymentをClusterIP:80で公開
   - ConfigMap
     - 名前はenv-config
     - dataはENV: base
3. 上記マニフェストをresourcesとして指定したkustomize/base/kustomization.yamlを書け。kustomizeについては[kustomize公式ドキュメント](https://github.com/kubernetes-sigs/kustomize)や[k8s公式ドキュメント](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)を参考にせよ。（公式はあまりわかりやすく無いのでインターネットを検索しても良い）
4. 以下コマンドでkustomizeでapplyせよ。（以下コマンドはkustomizeディレクトリで実行した場合）
   ``` sh
   kubectl apply -k base/
   ```
5. 作成したオブジェクトを確認し、以下コマンドで削除せよ。（以下コマンドはkustomizeディレクトリで実行した場合）
   ``` sh
   kubectl delete -k base/
   ```
6. overlay/dev/およびoverlay/prod/に以下を満たすマニフェストおよびkustomization.yamlを作成せよ
   - prod
     - baseは../../base
     - 作成するオブジェクトのプレフィックスに``prod-``をつける
     - 作成するオブジェクトに``env: prod``のラベルを追加
     - ConfigMap:env-configをオーバーライド(patches)し``ENV: prod``に変更せよ
   - dev
     - baseは../../base
     - 作成するオブジェクトのプレフィックスに``dev-``をつける
     - 作成するオブジェクトに``env: dev``のラベルを追加
     - ConfigMap:env-configをオーバーライド(patches)し``ENV: dev``に変更せよ
7. 以下コマンドでprodとdevをデプロイせよ.（以下コマンドはkustomizeディレクトリで実行した場合）
   ``` sh
   kubectl apply -k overlay/prod
   kubectl apply -k overlay/dev
   ```
8. 作成したオブジェクトを確認。pordおよびdevのPodに対して以下追加コマンドを発行しオーバーライドできていることを確認せよ。
   ``` sh
   echo $ENV
   ```
9.  deploymentを以下の様にオーバーライドして再デプロイせよ。なお、オーバーライドする時のマニフェストは必要最小限の記載にすること
    - prod
      - replicas: 3
      - resources.requests.memory: 100Mi
    - dev
      - replicas: 2
      - resources.requests.memory: 50Mi
10. 上記オーバーライドした内容が反映されていることを確認せよ
11. Podの/usr/share/nginx/html/index.htmlの内容を環境変数ENVの値とし、各環境で表示を変えたい。kustomize/base配下のみを修正し実装せよ。
12. curlが実行可能なPodを展開し、prodとdevそれぞれにcurlせよ。各環境名が表示されること
13. curl、dev、prodをすべて削除せよ

## 超上級
より高度な応用機能について学ぶ

istio
argoCD
CRD


## 試験対策
CKA/CKADを目指す上でしっておくと良い項目

色々なkubectlコマンド
K8sのアーキテクチャ
コンポーネントの設定