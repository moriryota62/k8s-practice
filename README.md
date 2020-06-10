# K8s practice
## まえがき
K8s practice はこれからKubernetes(以下、K8s)に入門する方を対象とした練習教材です。kubectlによる操作やマニフェストの読み書きなどの基本を抑えることを目的としています。

K8sリソースごとに動作や使い方を知る問題を用意しています。問題の指示や公式ドキュメントを参考に自身で考えながら進めてください。

## レベルについて
K8s practice は大きく以下の3つのレベルに分けています。レベル順に進んでいくことを想定していますが、気になる/学びたいリソースのところだけ問題を説くこともできます。

|レベル|内容|
|-|-|
|Beiginner|初級レベルです。kubectlやPodなど基礎となるリソースについて扱います。|
|Intermediate|中級レベルです。K8s標準で使えるリソースを扱います。|
|Advanced|上級レベルです。アドオンを追加することで使用できるリソースを扱います。|

## 前提
K8sクラスタを用意してください。また、kubectlでクラスタに接続できる状態としてください。これらを準備する手順としては[Amazon EKSの開始方法](https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/getting-started.html)などを参考にしてください。

なお、K8s practice の問題は EKS v1.14 をベースに作成しています。そのため、クラウドプロバイダもAWSを前提としています。他のマネージド・サービスやベアメタルのK8sでも問題を解くことはできると思いますが、クラウドプロバイダの部分はそれぞれのクラウドに置き換えてください。


