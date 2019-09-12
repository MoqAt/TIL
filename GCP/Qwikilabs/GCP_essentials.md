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