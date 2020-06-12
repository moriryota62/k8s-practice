前： [HorizontalPodAutoscaler](HorizontalPodAutoscaler.md)  

---

# Cluster Autoscaller
Cluster Autoscaller（以下、CA）は自動でワーカーノード数を調整するアドオン機能です。CAはK8s内にコントローラの役割を担うPodをデプロイして機能します。スケールアウトはHWリソースが足りずにどこにもスケジュールできないPod（STATUS:Pending）が発生した際に行われます。スケールインはクラスタ全体のHWリソース割当量をみて行われます。なお、CAはEKS/AKS/GKEなど対応したクラウドプロバイダでないと使用できません。また、ワーカーノードをAWS AutoScalingなどノード台数を柔軟に変更できる機能で展開していないと使用できません。

1. ワーカーノードに付与されているIAMロールをマネジメントコンソール等で確認してください。以下のようにAutoScalingGroupの権限がついていること。ついていない場合はワーカーのIAMロールにポリシーを追加すること。[ネタ元](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md)
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
2. AutoScalingGroupのワーカーノードのタグを確認してください。以下の様なタグが付与されていること。付与されていない場合は付与すること。なお、タグはKeyのみでValueは不要です。
   - k8s.io/cluster-autoscaler/enabled
   - k8s.io/cluster-autoscaler/<クラスタ名>
3. [CAのマニフェスト](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml)をコピーしてください。
4. 上記コピーしたマニフェストの\<YOUR CLUSTER NAME\>の箇所を修正しapplyしてください。
5. ClusterAutoscalerのPodが起動していることを確認してください。
6. 以下を満たすマニフェストを作成しデプロイしてください。
   - Deployment
     - イメージは何でもよい
     - resource.limits.cpu: 1000m
7. 上記作成したDeploymentのreplica数を1つずつ増やしていき、STATUS:PendingのPodが出るまで増やしてください。
8. 「kubectl get node」および「kubectl get pod」をwatch等で監視し、Nodeが増えてSTATUS:PendingのPodがrunningになることを確認してください。（PendingのPodが生まれてからEC2インスタンスを立ち上げるため数分かかります）
9. 上記作成したDeploymentのreplica数を1にしてください。
10. 「kubectl get node」および「kubectl get pod」をwatch等で監視し、Nodeが減ることを確認してください。(デフォルトではスケールアウト後のスケールインは10分以上経過しないと行われないため10分ほど待つ。時間が知りたい場合はCAのPodのlogを確認し「ノード名 was unneeded for 時間」が10分に達するとスケールインが始まる)
11. CA用のオブジェクト一式とDeploymentを削除してください。

---

次： [descheduler](descheduler.md)  
