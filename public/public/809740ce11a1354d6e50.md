---
title: Zenn・Qiita-CLIでのコマンドまとめ
tags:
  - Qiita
  - idea
  - Zenn
private: false
updated_at: '2024-06-22T01:22:54+09:00'
id: 809740ce11a1354d6e50
organization_url_name: null
---
## １．はじめに
最近ZennとQiitaのCLIを導入しました！
また、両記事をGithubと連携し、ひとつのリポジトリで管理できる環境構築にも成功しました！
構築にあたっては[Zenn公式の記事](https://zenn.dev/zenn/articles/zenn-cli-guide)及び以下の記事に大変お世話になりました。

https://zenn.dev/ot07/articles/zenn-qiita-article-centralized

おかげさまでGit・Githubについても勉強することができました！フォロー必須です！！！

https://zenn.dev/ot07

環境構築できたはいいが、CLIコマンドについてうろ覚えなので備忘録的な感じで記事にしようと思います。

## ２．Zenn-CLIコマンド
#### ◯CLIの起動
```
npx zenn init
```
任意のディレクトリでコマンドを実行することで、`article`と`books`ディレクトリが作成される。
#### ◯記事新規作成
```
npx zenn new:article
```
これでランダムな`マークダウンファイル（.md）`が作成されますが、ファイル名はわかりやすくしたいので自分は以下を多用
```
npx zenn new:article --slug <ファイル名>
```
これで＜ファイル名＞のマークダウンファイルが生成されます！

#### ◯プレビュー
```
npx zenn preview
```
`http://localhost:8000`にブラウザからアクセスしてプレビューが確認

とりあえずはこれらをおさせておけば一時は大丈夫かな？

細部は以下に頼ろう。。。

https://zenn.dev/zenn/articles/zenn-cli-guide

## ３．Qiita-CLIコマンド
#### ◯CLIの起動
```
npx qiita init
```
Zenn同様に関係ファイルが作成される。
-.gitignore
-GitHub Actions のワークフローファイル
-ユーザー設定ファイル（qiita.config.json）

#### ◯ログイン
発行したトークンで以下の要領でログイン
```
npx qiita login
```
```
発行したトークンを入力: トークンを入力しEnterキーを押す
Hi ユーザー名!
```
Zennのときログインしたっけ？？？（した記憶なし。。。）

#### ◯プレビュー
```
npx qiita preview
```
`http://localhost:8888`にブラウザからアクセスしてプレビュー

#### ◯記事新規作成
先ほども紹介しました[**神記事**](https://zenn.dev/ot07/articles/zenn-qiita-article-centralized)のおかげでQiitaはCLIからの新規作成は行いません！
これが非常に助かります！！！

