---
title: 【M1Mac】仮想化ソフトUTMにGNS3-VMをインストールする方法
tags:
  - Network
  - 勉強法
  - ネットワーク構築
  - GNS3
private: false
updated_at: '2024-06-19T08:22:56+09:00'
id: 764b25facff9fcf74bc3
organization_url_name: null
---
## 1.はじめに
GNS3でDockerやQemu建てたサーバ（Linux等）をトポロジに配置し、シスコIOSとのネットワークテストをやりたいと思いましたが、それらはGNS3-VMが必要とのことなので、ローカルサーバとは別にリモートサーバを仮想化ホストUTMで建てました。VMwareFusionでの構築はたくさん記事がありますが、UTMの記事はあまり見ないので今回書こうと決心しました！

## 2.M1Macのスペック及び仮想マシンの内容
◯スペック
OS：sonoma14.5
メモリ：16G
ストレージ：512GB
◯仮想マシン
OS：ubuntu18.04.6 LTS
メモリ（仮想）：8G
ストレージ（仮想）：30GB
→初期の20GBだとのちのちGNS3のproject数が増えるとストレージがすぐ圧迫するので！

## 3.GNS3-VMの構築のポイントその１
UTMでの仮想マシン作成は基本下記の通りに行えば問題ないです。

https://qiita.com/sakai00kou/items/f50e989d30d5e28c6dc9

今回のポイントとしては、M1Mac（ARM）アーキテクチャ上で**x86アーキテクチャ**をエミュレートする点です。x86ベースにすることで、アーキテクチャに起因するDcokerコンテナ起動、Qemu起動時のエラーが解消できます！！！（ARMに対応していないものが多い！！！）

![](https://raw.githubusercontent.com/bit-and-coffee/zenn-qiita-contents/main/images/utm-gns3vm-install/1.png)

上記の画像通りに「**Emulate**」を選択してください。

## 4.GNS3-VMの構築のポイントその２
上記の3項の通り仮想マシンをx86ベースで作成できればいよいよ**GNS3-VM**のインストールです。
インストールについては、下記の動画通りにやってもらえればOKです！！！

https://youtu.be/hHcQ5BCVHLg?si=qfYy5i_KSBGLLj2z

この動画では、シェルスクリップとで最新？のVMをインストールしてくれます。
無事にインストールが完了すればあとはGNS3上でリモートサーバ設定をするだけ。

![](https://raw.githubusercontent.com/bit-and-coffee/zenn-qiita-contents/main/images/utm-gns3vm-install/2.png)

リモート先を先程作成した仮想マシンのIPアドレスを指定すればOK！
（IPアドレスについては、UTMで仮想マシンを作成するときにNetoworkの項目をBridgeにしているので、自動的に自宅WiFiルータからのIPアドレスが払い出されるようにしています。）

接続はうまくいっていれば、ServerSummaryのランプが緑になります！

![](https://raw.githubusercontent.com/bit-and-coffee/zenn-qiita-contents/main/images/utm-gns3vm-install/3.png)

ちなみに、私のMainserverはVMWareの仮想マシンにインストールしてGNS3-VM（ARMアーキテクチャ）しています。
GNS3のアプライアンスによっては、ARM未対応のものもあるので色々試せるようにVM２本建てにしています。
下図のように色々と試せそうなのでワクワクしますよね！！！

![](https://raw.githubusercontent.com/bit-and-coffee/zenn-qiita-contents/main/images/utm-gns3vm-install/5.jpg)

## 5.さいごに
個人的にGNS3には大きな可能性を感じています。これからも開発が進んでGNS3上で色々なことが試せるようになることを期待しています。また、PCのスペックを積んでさらにたくさんのノードを配置したテスト等ができたらいいなと思っています。（M2Macmminiのメモリ24GBがほしいすぎる。。。）
