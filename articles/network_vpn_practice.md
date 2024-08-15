---
title: "【GNS3】CiscoとJniperを使ったVPN環境"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['network','gns3']
published: false
---
## はじめに
本記事では、OSPFで構成されたネットワークにRIPで加入し、エンドツーエンドをL2TPで接続するシナリオを検証します。加入ルーターの片側をJuniperにすることで、異なるメーカー間でもプロトコルが合致すれば、正しく通信が可能であることを確認します。

特に重要なポイントとして、OSPFネットワークでは、加入ルーターが公布するWAN・LAN側ネットワークのうち、**LAN側ネットワークのみ**をOSPFで再配布します。このため、加入ルーターはどの地域からでも接続可能であり、たとえエンドツーエンドでアドレスが重複してもVPNを構成できることが示されます。

## トポロジ図
以下のトポロジ図に従ってネットワークを構築します。

![](https://github.com/bit-and-coffee/zenn-qiita-contents/blob/main/images/network_vpn_practice/1.png) <!-- ここにトポロジ図を挿入 -->

## 検証手順

1. **OSPFネットワークの設定**
    - Ciscoルーター R1, R2, R3 を用いてOSPFネットワークを構築します。
    - OSPFルーティングプロトコルを設定し、各ルーター間の通信を確立します。

    ```Router1
    interface FastEthernet0/0
    ip address 172.16.1.1 255.255.255.0
    duplex auto
    speed auto
    !
    interface Serial0/0
    no ip address
    shutdown
    clock rate 2000000
    !
    interface FastEthernet0/1
    ip address 192.168.100.254 255.255.255.0
    duplex auto
    speed auto
    !
    router ospf 1
    log-adjacency-changes
    redistribute rip subnets route-map RIP_TO_OSPF
    network 172.16.1.0 0.0.0.255 area 0
    !
    router rip
    version 2
    redistribute ospf 1 metric 1
    network 192.168.100.0
    !
    ip forward-protocol nd
    !
    !
    no ip http server
    no ip http secure-server
    !
    !
    ip prefix-list KANYU-NW-1 seq 10 permit 192.168.1.0/24
    no cdp log mismatch duplex
    !
    !
    !
    route-map RIP_TO_OSPF permit 10
    match ip address prefix-list KANYU-NW-1
    ```

    ```ESW
    interface FastEthernet1/0
    no switchport
    ip address 172.16.1.254 255.255.255.0
    duplex full
    speed 100
    !
    interface FastEthernet1/1
    no switchport
    ip address 172.16.2.254 255.255.255.0
    duplex full
    speed 100

    router ospf 1
    log-adjacency-changes
    network 172.16.1.0 0.0.0.255 area 0
    network 172.16.2.0 0.0.0.255 area 0
    ```

    ```Router2
    interface FastEthernet0/0
    ip address 172.16.2.1 255.255.255.0
    duplex auto
    speed auto
    !
    interface Serial0/0
    no ip address
    shutdown
    clock rate 2000000
    !
    interface FastEthernet0/1
    ip address 192.168.100.254 255.255.255.0
    duplex auto
    speed auto
    !
    router ospf 1
    log-adjacency-changes
    redistribute rip subnets route-map RIP_TO_OSPF
    network 172.16.2.0 0.0.0.255 area 0
    !
    router rip
    version 2
    redistribute ospf 1 metric 1
    network 192.168.100.0
    !
    ip forward-protocol nd
    !
    !
    no ip http server
    no ip http secure-server
    !
    !
    ip prefix-list KANYU-NW-2 seq 10 permit 192.168.2.0/24
    no cdp log mismatch duplex
    !
    !
    !
    route-map RIP_TO_OSPF permit 10
    match ip address prefix-list KANYU-NW-2
    ```

2. **RIPルーターの追加**
    - R3のLAN側ネットワークにR4を追加し、RIPでOSPFネットワークに接続します。
    - Juniperルーターを使用して、LAN側のネットワークをRIPにより広報します。

    ```bash
    # R4 (Cisco) RIP Configuration
    router rip
     version 2
     network 192.168.4.0
     network 10.0.1.0
    ```

    ```bash
    # Juniper (R5) RIP Configuration
    set protocols rip group RIP-Group neighbor 10.0.1.2
    set protocols rip group RIP-Group export to-ospf
    ```

3. **OSPFへのLAN側ネットワークの再配布**
    - R3からOSPFにLAN側ネットワーク (例: 192.168.4.0/24) のみを再配布します。

    ```bash
    # R3 Configuration
    router ospf 1
     redistribute rip subnets
     distribute-list 1 in
    ```

    - 再配布時にWAN側ネットワークは含めないようにします。

    ```bash
    # Distribute-List Configuration
    access-list 1 permit 192.168.4.0 0.0.0.255
    access-list 1 deny any
    ```

4. **L2TPでのエンドツーエンドVPN構成**
    - JuniperとCiscoルーター間でL2TPトンネルを構築し、エンドツーエンドの通信を確立します。

    ```bash
    # Juniper (R5) L2TP Configuration
    set security ike proposal ike-proposal authentication-method pre-shared-keys
    set security ike policy ike-policy mode main
    set security ike gateway ike-gateway ike-policy ike-proposal

    set security ipsec vpn ipsec-vpn ike gateway ike-gateway
    set security ipsec vpn ipsec-vpn ike proxy-identity local 192.168.4.0/24
    ```

    ```bash
    # Cisco (R4) L2TP Configuration
    vpdn enable
    vpdn-group L2TP-Group
     request-dialin
     protocol l2tp
     domain default
     initiate-to ip 10.0.1.1
    ```

## 検証結果
この設定により、OSPFとRIP間でのネットワーク再配布が正しく行われ、L2TPトンネルを介してエンドツーエンドで通信が確立できることを確認しました。WAN側ネットワークアドレスの重複があっても、問題なくVPN接続ができることが証明されました。

## 結論
異なるベンダー間（CiscoとJuniper）のルーターを使用しても、適切にプロトコルを設定すれば、正確なネットワーク通信が可能です。OSPFでの再配布により、ネットワークの柔軟性が増し、セキュアなVPN接続が簡単に構築できることが確認されました。