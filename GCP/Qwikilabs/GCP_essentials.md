# GCP Essentials

## A Tour of Qwiklabs and the Google Cloud Platform

基本的にはQwikilabsのハンドアウトの進め方を説明している

#### Role and Permission

各Roleの持っている権限を確認

- viewer  
閲覧権限。存在するリソースの確認が可能。起動、編集などの権限は持っていない。

- Editor  
編集権限。Viwer権限に加え、状態を変更するアクション（存在するリソースを編集するなど）の権限を有する。

- Owner  
編集権限に加えて以下の権限を有する。

  - プロジェクトとプロジェクト内のすべての権限を管理する。
  - 支払情報などを設定する。


## 仮想マシンの作成

ここで日本語に変更できることに気づく。
Google Cloud EngineのConsoleと`gcloud`を利用した作成方法、操作方法を学ぶ。

`gcloud`の概要を掴む。（次の章でもやるけど）

#### `gcloud`を利用したインスタンスの作成
仮想マシンの作成コマンドは基本的に以下。

```sh
$ gcloud compute instances create {instanceName} --machine-type {machineType} --zone {ZONE}
```

*`gloud`コマンド以下にコンポーネントとしてここでは`compute`を使用していることをチェック*

指定していない項目は以下のようになっている
- OS  
`Debian9(stretch)`イメージが使用される
- 永続ディスク  
インスタンス名と同じで作成されてインスタンスに組み込まれる。

#### 作成したインスタンスへのssh接続

`gcloud`を使用することで作成したインスタンスに簡単に接続できる。

```sh
$ gcloud compute ssh {instanceName} --zone {zone}
```

ここから先は普通のssh接続と同じ流れ。

## Cloud Shell と gcloud のスタートガイド

`gcloud`コマンドを使う。

### `gcloud`と`components`

`gcloud`コマンドは`compoents`の機能を利用できる。

```sh
$ gcloud components list
```

例えば...
- compute  
Google Compute Engineを操作する。
- beta  
Cloud Functionsを操作する機能がここにある。

## Kubernetes Engine: Qwik Start

GKEの基礎について確認。

### Kubernetes(GKE)を利用するメリット

**なぜ純粋にVMを使用するのではなくGKEを利用するのか**　　要確認！

- Compute Engineの負荷分散（ロードバランサが動かしやすい）
- ノードプールを使用する柔軟性（クラスタ内でいろんなマシンのスペックなどを使用可能）（https://cloudplatform-jp.googleblog.com/2016/06/google-container-enginegke.html）
- 自動スケーリング（ノード内の負荷が高まると自動で限界までVMが起動する）
- 自動アップグレード（ノードごとに動くためローリングアップデート可能、マスターを更新するとクラスター内のノードが更新される）
- ノードの自動修復（Deploymentと同じ定義で動作しているか常に監視、動いていなければ直す）
- Stackdriverのロギングとモニタリングでクラスタの状況確認が可能

クラスタの作成
```
gcloud container clusters create [CLUSTER-NAME]
```

クラスタの認証情報取得（kubectlで操作するクラスタの指定）
```
gcloud container clusters get-credentials [CLUSTER-NAME]
```

Deploymentオブジェクトの作成
```
kubectl run hello-server --image=gcr.io/google-samples/hello-app:1.0 --port 8080
```

Deployment, Podなどの公開
```
kubectl expose deployment hello-server --type="LoadBalancer"
```
`--type="LoadBalancer"`を渡すとコンテナのCompute Engineロードバランサを作成できる

service, deployment, podなどの情報取得
```
kubectl get {対象}
```

### k8sのオブジェクト

- Deployment
- Service
- Pod

## ネットワーク ロードバランサと HTTP ロードバランサを設定する

- L3 ネットワークロードバランサ
- L7 HTTP(S)ロードバランサ

### ロードバランサの種類とその選択

![ロードバランサの選択](https://cloud.google.com/load-balancing/images/choose-lb.svg?hl=ja)
https://cloud.google.com/load-balancing/docs/choosing-load-balancer

- L3 ネットワークロードバランサ
  - TCPトラフィック（の一部）
  - UDPトラフィック
  - **アプリケーションパケットをチェックできる**
  - Regional(あるリージョンに設定し、そのリージョン内で振り分ける)
- L7 HTTP(S)ロードバランサ
  - HTTP/HTTPSトラフィック
  - Global(リクエストをリジョンをまたいで振り分け可能)
  - URLのパスによる振り分けも可能

```
まずTCPとHTTPって何が違う？
→TCPがHTTPより下位のレイヤーにある。TCP接続が確立されてその上でHTTPリクエスト/レスポンスを行う。という認識
```

#### インスタンスをテンプレートから作成する

例えばnginxをインストールして起動するスクリプト`startup.sh`を用意する
```
gcloud compute instance-templates create nginx-template \
         --metadata-from-file startup-script=startup.sh
```
でテンプレートを作成できる。  
以下のように作成することで同一のインスタンスが作成できる。

```
gcloud compute instance-groups managed create nginx-group \
         --base-instance-name nginx \
         --size 2 \
         --template nginx-template \
         --target-pool nginx-pool
```

`target-pool`とは、グループ内のすべてのインスタンスに一箇所からアクセスできるようにする仕組み。  
作成方法
```
gcloud compute target-pools create nginx-pool
```

#### ファイアウォールの作成  
そのネットワーク全体に適用される
```
gcloud compute firewall-rules create www-firewall --allow tcp:80
```

#### L3ネットワークロードバランサの作成
```
gcloud compute forwarding-rules create nginx-lb \
         --region us-central1 \
         --ports=80 \
         --target-pool nginx-pool
```
ここでtarget-poolを指定している  
ロードバランサに対するアクセスがtarget-pool内の通信に対して振り分けられる。

#### HTTP(S)ロードバランサの設定

ヘルスチェックの作成
```
gcloud compute http-health-checks create http-basic-check
```

HTTPサービスを作成し、インスタンスグループのポートにマッピング
```
gcloud compute instance-groups managed \
       set-named-ports nginx-group \
       --named-ports http:80
```

バックエンドサービスの作成
```
gcloud compute backend-services create nginx-backend \
      --protocol HTTP --http-health-checks http-basic-check --global
```

バックエンドサービスに対してインスタンスグループの追加
```
gcloud compute backend-services add-backend nginx-backend \
    --instance-group nginx-group \
    --instance-group-zone us-central1-a \
    --global
```

URLマップの作成
```
gcloud compute url-maps create web-map \
    --default-service nginx-backend
```

HTTPプロキシの作成
```
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map
```

グローバルの転送ルール作成
```
gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
```

うーん、よくわからん！  
↓こっちで確認します

[Google Cloud Platform HTTP Load Balancers Explained via the CLI](https://www.ianlewis.org/en/google-cloud-platform-http-load-balancers-explaine)

[GCPのロードバランサの仕組みの復習](https://road288.hatenablog.com/entry/2018/07/17/201156)