- [Zenn CLIをセットアップ](#zenn-cliをセットアップ)
  - [Zenn用のセットアップを行う](#zenn用のセットアップを行う)
- [CLIをアップデートする](#cliをアップデートする)
- [コマンド](#コマンド)
- [記事](#記事)
  - [ファイルの配置ルール](#ファイルの配置ルール)
  - [プレビューする](#プレビューする)
- [記事の作成](#記事の作成)
- [記事を公開する](#記事を公開する)
- [記事の更新](#記事の更新)

>[ZennとGitHub連携して実際に記事投稿してみる](https://zenn.dev/takuty/articles/5e1bf0fd6fc5a3)
>[Zenn公式](https://zenn.dev/zenn/articles/install-zenn-cli#zenn-cli%E3%81%A8%E3%81%AF)

# Zenn CLIをセットアップ
ZennとGitHubリポジトリを連携すると、ローカルの好きなエディターで投稿コンテンツの作成・編集ができるようになります。

ローカルでの執筆時には、スムーズにmarkdownファイルの作成したり、コンテンツをプレビューしたりするために「Zenn CLI」を導入しましょう。

Zennのコンテンツを管理したいディレクトリで、以下のコマンドを実行します。
```bash
npm init --yes # プロジェクトをデフォルト設定で初期化
npm install zenn-cli # zenn-cliを導入
```
これでディレクトリにCLIがインストールされます。
## Zenn用のセットアップを行う
続いて以下のnpxコマンドを実行します。
```bash
npx zenn init
```
README.mdや.gitignoreのほか、articlesとbooksという名前のディレクトリが作成されます。この中にmarkdownファイル（◯◯.md）を入れていくことになります。

これでZenn CLIの導入は完了です🎉
# CLIをアップデートする
Zenn CLIの表示がzenn.devと異なるときやCLI利用時に更新通知が表示されたときは下記のコマンドでアップデートを行ってください。
```bash
npm install zenn-cli@latest
```

# コマンド
👇  新しい記事を作成する
```bash
npx zenn new:article
```
👇  新しい本を作成する
```bash
npx zenn new:book
```
👇  投稿をプレビューする
```bash
npx zenn preview
```
# 記事
## ファイルの配置ルール
1 つの記事の内容は、1 つのmarkdownファイル（◯◯.md）で管理します。ファイルはarticlesという名前のディレクトリ内に含める必要があります。
```tree
.
└─ articles
   ├── example-article1.md
   └── example-article2.md
```
articles/ランダムなslug.mdというファイルが作成されます。slug（スラッグ）はその記事のユニークな ID のようなものです。詳しくは「Zenn の slug とは」をご覧ください。

作成されたファイルの中身は次のようになっています。
```markdown
---
title: "" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: [] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
ここから本文を書く
```
## プレビューする
本文の執筆は、ブラウザでプレビューしながら確認できます。ブラウザでプレビューするためには次のコマンドを実行します。
```bash
$ npx zenn preview # プレビュー開始
```

# 記事の作成
作成されたファイルは既に以下のような記載がされています。
![Title](https://res.cloudinary.com/zenn/image/fetch/s--JWohRtfL--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/5e54425ae6b46f5e830d9315.png%3Fsha%3D3dbf3d157b92d1958fba44b9e8684c8d293b9a85)
- title：この記事のタイトル
- emoji：タイトルの上に表示される絵文字(初期作成のままで良いと思います)
- type：tech 技術記事 / idea アイデア
- topics：記事のタグ(この記事であれば zenn と GitHub)
- publisjed：true 公開 / false 非公開

これらを編集し終えたら下部から本文を md で記載していきます。





# 記事を公開する
記事を zenn.dev 上で公開するにはpublishedオプションがtrueになっていることを確認したうえで、ファイルをコミットし、Zenn と連携されている GitHub リポジトリにプッシュします。
Zenn と連携したリポジトリの登録ブランチにプッシュされると、同期（デプロイ）が開始されます。

!!! warning デプロイ履歴はダッシュボードから見ることができます。デプロイ時にエラーが発生している場合もここから見る必要があります。

なおコミットメッセージに[ci skip]もしくは[skip ci]が含まれていると Zenn でのデプロイがスキップされます。

# 記事の更新
記事の更新を行う場合も、markdownファイルを編集し、GitHub リポジトリへプッシュするだけで OK です。このとき slug が同一のものでないと別の記事として作成されてしまうので注意しましょう。

!!! warning リポジトリの変更が zenn.dev に反映されるまでにしばらく時間がかかります。ダッシュボードからデプロイのステータスをご確認ください。

また、未ログイン状態ではしばらくキャッシュされた古い内容が表示される可能性があります。時間を置いた後にリロードしてご確認ください。






















