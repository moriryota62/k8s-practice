前： [ConfigMap(env)](ConfigMap-env.md)  

---

# ConfigMap ファイルマウント

ConfigMapをマウントすることもできます。たとえばPod（コンテナ）内のとあるファイルの内容を頻繁に書き換えたい場合、コンテナイメージの中にそのファイルがあるとファイル内容を変えてからコンテナイメージをビルドしなおすことになります。または、外部のEFSなどにそのファイルを置いておき、initContainerなどでコンテナ起動前にコピーする方法もあります。いずれの方法でも可能ですが、対象ファイルをConfigMapにすることでも可能です。PodからConfigMapをマウントすることでPod外に設定ファイルを保存する事ができます。ファイルの内容を変更したい場合はConfigMapを更新します。

1. 以下を満たすマニフェストを作成しデプロイしてください。
   - ConfigMap
     - 以下の内容が記述されたindex.htmlをdataとして持つ
       - ConfigMap ni kaita naiyou dayo
   - Deployment
     - イメージはnginx:1.12
     - 上記ConfigMapをボリュームとしてマウント
     - マウント先は/usr/share/nginx/html
   - Service
     - 上記DeploymentをClusterIPのPort:80で公開
2. curlが実行可能なPodを展開し、Serviceを指定してcurlを実行してください。ConfigMapの内容が表示されること。
3. ConfigMapのindex.htmlの内容を以下に修正しConfigMapを再デプロイしてください。
   - ConfigMap wo henkou suruto 60byou kuraide Pod nimo hanei sareruzo
4. （configmapをデプロイして60秒以内）curlが実行可能なPodからServiceを指定してcurlを実行する。ConfigMap修正前の内容が表示されること。
5. （configmapをデプロイして60秒以降）curlが実行可能なPodからServiceを指定してcurlを実行する。ConfigMap修正後の内容が表示されること。
6. ConfigMap、Deployment、curlのPodを削除

なお、ボリュームマウントであるため、マウント先のディレクトリはConfigMap化したファイルのみになってしまう。もしも他のファイルがある場所にファイルを置きたい場合はtempDirのボリュームを作り、initContainerで元のディレクトリ内容をtempDirにコピー、ConfigMapもどこか別の場所にマウントしてからtempDirにコピー、メインコンテナでtempDirをしかるべき場所にマウントすればよいです。

---

次： [Secret](Secret.md)  
