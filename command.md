

## ルーティングテーブルの確認
```
ip route show
ip r
ip route
```

## ipアドレスの追加
```
ip address add 192.0.2.254/24 dev gw-veth0
```
- dev: NICの指定をするよということを表すために必要

## routerの有効化(カーネルパラメータ設定)

routerに接続されているNIC間でパケットをバケツリレーするために必要な設定

```
sudo sysctl net.ipv4.ip_forward=1
sudo sysctl -a | grep net.ipv4.ip_forward
```

## default gatewayの追加
```
sudo ip route add default via 192.0.2.254
```
## iptables natテーブルの確認
```
sudo iptables -t nat -L
```

## iptables nat マスカレード設定
```
sudo iptables -t nat \
> -A POSTROUTING \
> -s 192.0.2.0/24 \
> -o gw-veth1 \
> -j MASQUERADE
```
* -A : 処理を追加するチェイン。POSTROUTINGはルーティングが終わってパケットがインターフェースから出ていく直前
* -s : 処理の対象となる送信元IPアドレスの範囲
* -o : 処理の対象とする出力先のNIC
* -j : 処理に一致したパケットをどのように処理するかのルール

## iptablesメモ
* 処理を適用するタイミング：`chain`

## tcpdump
```bash
sudo tcpdump -tnl -i wan-veth0 icmp
```
## Destination NAT
インターネットから到来したパケットをLANに繋がっているノードに転送できる

```bash
sudo iptables -t nat \
> -A PREROUTING \
> -p tcp \
> --dport 54321 \
> -d 203.0.113.254 \
> -j DNAT \
> --to-destination 192.0.2.1
```
* -A     : PREROUTINGは処理のタイミングとしてインターフェイスからパケットが入ってきた直後
* --dport           : 処理の対象となるポート番号
* -d                : 書き換える前の送信先IPアドレス
* --to-destination  : 書き換えた後の送信先IPアドレス
