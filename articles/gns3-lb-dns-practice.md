---
title: "【ネスペ対策】GNS3でロードバランサとDNSの検証環境を作った話"
emoji: "🦁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn","qiita","idea",”ネスペ"]
published: false
---
##1.はじめに

ネットワークスペシャリスト（通称ネスぺ）を勉強中にロードバランサについての項目が出てきた。
漠然と知ってはいたが、いざどういう設定でどういう動きをしているかは全くわからなかったので、個人的に大好きなGNS3で検証環境を構築しました！

---

##2. 検証シナリオ
端末からWebサーバに対してアクセスし、ロードバランサで負荷分散されているかを検証
（アクセスはブラウザ・ターミナル両方でアクセス）
ロードバランサついでにDNSサーバも配置し、名前解決できる環境も構築！

##3. Dockerイメージの準備

GNS3上で配置するサーバについてはDockerイメージを使用します。（一部ホストも）
- ロードバランサー
  nginxのイメージを使用
- DNSサーバ
　CoreDNSのイメージを使用
- Webサーバ
　pythonイメージを使用（サーバ機能はpython標準ライブラリのhttpコマンド）
　

#### 例：UbuntuベースDockerfile

```dockerfile
FROM ubuntu:latest

# ネットワーク関連コマンド(ip, pingなど)やロードバランサー・DNSサーバのインストール
RUN apt-get update && \
    apt-get install -y iproute2 iputils-ping haproxy dnsmasq && \
    apt-get clean
