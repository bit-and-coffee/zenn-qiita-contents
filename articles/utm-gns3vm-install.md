---
title: "【M1Mac】仮想化ソフトUTMにGNS3-VMをインストールする方法"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [topics:"network","勉強法","ネットワーク","gns3","ネットワーク構築"]
published: false
---
## 1.はじめに
GNS3でDockerやQemu建てたサーバ（Linux等）をトポロジに配置し、シスコIOSとのネットワークテストをやりたいと思いましたが、それらはGNS3-VMが必要とのことなので、ローカルサーバとは別にリモートサーバを仮想化ホストUTMで建てました。VMwareFusionでの構築はたくさん記事がありますが、UTMの記事はあまり見ないので今回書こうと決心しました！

## 2.M1Macのスペック及び仮想マシンの内容
◯スペック
OS：sonoma14.5
メモリ：16G
ストレージ：512GB
◯仮想マシン
OS：ubuntu 18.04.6 LTS