---
title: "【GNS3検証】MikroTikルータでVPNを設定し、社内ネットワークにL2臨時接続する構成"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['network','gns3',cisco,juniper,chatgpt]
published: false
---
# 【GNS3検証】MikroTik × EoIP × RIPで社内ネットワークにL2臨時接続する構成

## はじめに

本記事では、GNS3とMikroTik RouterOSを使用して、**支店ルータ（Branch Router）を社内ネットワークにRIPで一時的に接続**し、**Edgeルータ同士をEoIPトンネルで接続してL2ブリッジを構成**する検証を紹介します。

> ※想定シナリオ：社内の既存RIPネットワークに、一時的に拠点（Branch）を追加し、Edgeルータを経由してL2で端末を接続する

---

## ネットワーク構成図

![構成図](./スクリーンショット%202025-09-21%209.17.22.png)

- **Branch-Router-1 / 2**  
  - 社内RIPネットワークに接続（ether1がWAN、ether2-4がLANブリッジ）
  - RIPによって0.0.0.0/0のデフォルトルートを学習
  - スタティックおよびコネクテッドルートを広報

- **Edge-VPN-Router-1 / 2**  
  - BranchルータのLAN側に接続（192.168.10.0/24 or 192.168.20.0/24）
  - それぞれのEdgeルータ間でEoIPトンネルを張り、LANをL2延伸
  - DHCP or 手動設定で端末へIP付与

- **PC1 ↔ PC2**  
  - EoIPトンネル越しにL2で疎通できるかを確認

---

## 使用技術

| 技術         | 用途                             |
|--------------|----------------------------------|
| MikroTik ROS | ルーティング、EoIP、Bridge構成   |
| RIP v2       | 社内ネットワークへの経路広報     |
| EoIP         | L2トンネルでのネットワーク延伸   |
| GNS3         | 検証環境の構築                   |

---

## 各ルータの設定概要

### Branch-Router の構成（共通）

```shell
/interface bridge
add name=bridge-lan

/interface bridge port
add bridge=bridge-lan interface=ether2
add bridge=bridge-lan interface=ether3
add bridge=bridge-lan interface=ether4

/ip address
add address=192.168.10.1/24 interface=bridge-lan  # Branch-1
# add address=192.168.20.1/24 interface=bridge-lan  # Branch-2
add address=10.0.10.2/30 interface=ether1
add address=192.168.100.200/24 interface=ether5

/routing rip instance
add name=rip-instance redistribute=connected,static routing-table=main

/routing rip interface-template
add instance=rip-instance interfaces=ether1
```

```shell
/interface bridge
add name=bridge-lan

/interface bridge port
add bridge=bridge-lan interface=ether2
add bridge=bridge-lan interface=ether3
add bridge=bridge-lan interface=ether4
add bridge=bridge-lan interface=eoip-tunnel

/ip address
add address=192.168.10.2/24 interface=ether1  # Edge-VPN-Router-1
# add address=192.168.20.2/24 interface=ether1  # Edge-VPN-Router-2
add address=192.168.100.200/24 interface=ether5

/interface eoip
add name=eoip-tunnel remote-address=192.168.20.2 tunnel-id=100 \
    local-address=192.168.10.2

/ip route
add dst-address=0.0.0.0/0 gateway=192.168.10.1  # Branch-RouterのLAN側を指定
```