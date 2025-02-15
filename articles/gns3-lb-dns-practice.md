---
title: "【ネスペ対策】GNS3でロードバランサとDNSの検証環境を作った話"
emoji: "🦁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn","qiita","idea",”ネスペ"]
published: false
---
## 1.はじめに

ネットワークスペシャリスト（通称ネスぺ）を勉強中にロードバランサについての項目が出てきた。
漠然と知ってはいたが、いざどういう設定でどういう動きをしているかは全くわからなかったので、個人的に大好きなGNS3で検証環境を構築しました！



## 2. 検証シナリオ
端末からWebサーバに対してアクセスし、ロードバランサで負荷分散されているかを検証
（アクセスはブラウザ・ターミナル両方でアクセス）
ロードバランサついでにDNSサーバも配置し、名前解決できる環境も構築！

## 3. 使用するDockerイメージ
まずはじめに、各サーバの元となるイメージを作成します。
ウブンツの公式イメージを使用し、以下の項目をインストールするようにします。

```bash
iputils-ping
traceroute 
net-tools
```


```Dockerfile:Dockerfile
FROM ubuntu:latest


RUN apt-get update && \
    apt-get install -y \
      iproute2 \
      iputils-ping \
      traceroute \
      net-tools

CMD ["bash"]
```
インストールする理由としては、GNS3上にサーバを配置した時、インタフェースの有効化やpingを使えるようにするためです。

:::message
ウブンツ公式イメージなどは、最小構成のため、pingなどのコマンドがインストールされていないことが多いです。
:::


次に各サーバのDcokerfileを作成していきます。（一部ホストも）
- ロードバランサー
  nginxのイメージを使用
- DNSサーバ
　CoreDNSのイメージを使用
- Webサーバ
　pythonイメージを使用（サーバ機能はpython標準ライブラリのhttpコマンド）
　

## 4. Dcokerfileの準備
DcokerfileからGNS3で起動するイメージを作成する。
### ロードバランサー
```Dockerfile:Dockerfile
# 公式の nginx イメージをベースにする
FROM mynginx:nettools

# default.conf をイメージ内にコピー
COPY default.conf /etc/nginx/conf.d/default.conf
```
上記のDockerfile内にあるdefault.confは、ロードバランサーの動作を記述したファイルです。


```:default.conf
# /etc/nginx/conf.d/default.conf

# upstream でバックエンドサーバーを定義
upstream my_backend {
    server 192.168.1.10:80;
    server 192.168.1.20:80;
    server 192.168.1.30:80;
}

server {
    listen 80;
    server_name _;  # 任意のホスト名や _ など

    location / {
        # upstream 名を指定してプロキシ転送
        proxy_pass http://my_backend;
        # 必要に応じてヘッダーの付与や設定を追加
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
負荷分散するバックエンドサーバのIPアドレスを指定しています。
負荷分散のアルゴリズムを指定しない場合は、デフォルトのラウンドロビンになります。

### DNSサーバ
DNSサーバでは、CoreDNSというGO言語ベースの軽量DNSサーバを使用します。
Dockerfileの中でインストールしていきます。

```Dockerfile:Dockerfile
FROM ubuntu:latest

# HTTPSダウンロード用に ca-certificates をインストール
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        wget \
        ca-certificates \
        iproute2 && \
    rm -rf /var/lib/apt/lists/*

ENV COREDNS_VERSION=v1.12.0

RUN wget "https://github.com/coredns/coredns/releases/download/${COREDNS_VERSION}/coredns_1.12.0_linux_amd64.tgz" && \
    tar -zxvf coredns_1.12.0_linux_amd64.tgz -C /usr/bin && \
    rm coredns_1.12.0_linux_amd64.tgz

COPY Corefile /Corefile
CMD ["/usr/bin/coredns", "-conf", "/Corefile"]

```
Dockerfileの中にあるCorefileはDNS機能を記述しているファイルになります。

```:Corefile
.:53 {
    forward . 8.8.8.8
    log
    errors
}

example.local:53 {
    hosts {
        192.168.1.1  example.local
        fallthrough
    }
    log
    errors
}

```
ドメイン名「example.local」をIPアドレス「192.168.1.1」へ名前解決しています。

### Webサーバ
ロードバランサーのバックエンドで動作するWebサーバになります。
ここはPythonの標準イメージを使用しています。
```Dockerfile:Dockerfile
# ベースイメージとして公式の Python イメージを使用
FROM python:3.9-slim

# コンテナ内での作業ディレクトリを設定
WORKDIR /app

# ポート80番をコンテナ内で開放
EXPOSE 80


# python -m http.server 80 を起動コマンドとする
CMD ["python", "-m", "http.server", "80"]
```

### イメージの作成コマンド
作成したDockerfileはそれぞれディレクトリ分けしておきます。

```bash
例）
lb_gns3/
    Dockerfile
    default.conf
dns_gns3/
    Dockerfile
    Corefile
web-server-py_gns3
    Dockerfile
```
それぞれのディレクトリ内で下記のコマンドを実行します。

```bash
docker build -t <任意のイメージ名>:<任意のタグ名> .
```
