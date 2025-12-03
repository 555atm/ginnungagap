3. ネットワーク設計（VPC）
3-1. VPC の基本概念

自分専用の仮想ネットワーク

サブネットはAZごとに切るのが基本

3-2. CIDR 設計

10.0.0.0/16 ← 1VPCのよくある例

サブネットは /24 や /20 が一般的

IPv6は今後必須（ALB/NLBは完全対応済）

3-3. サブネット（Public / Private / Isolated）

Public：ALB・踏み台

Private：EC2 / ECS / RDS

Isolated：DB, バッチ用（外部通信なし）

3-4. ルートテーブル設計

Public：0.0.0.0/0 -> IGW

Private：0.0.0.0/0 -> NATGW

Isolated：外部への通信なし

3-5. Internet Gateway (IGW)

インターネット出口

Publicサブネットは必ずIGWへルーティング

3-6. NAT Gateway

Privateサブネットが外部に出るために使用

高額になりやすい（通信課金）

冗長化：AZごとに配置がベスト

3-7. VPC Endpoint（Interface / Gateway）

AWS内部通信（S3/DynamoDB）を閉域で使う

コストは Interface > Gateway

セキュアかつNATGW削減に有効

3-8. Security Group

ファイアウォール（ステートフル）

サービス単位で作る

「相互参照」で構成を安全に管理

3-9. NACL

VPC全体の補助的フィルタ

基本はデフォルト許可のまま

厳密制御が必要な場合のみ設定

3-10. オンプレ接続

VPN：安価だが不安定

Direct Connect：高品質 + 高速

併用（DX + VPN）は企業でよく使う

3-11. マルチAZ設計の思想

すべての層がAZをまたげるように設計

ALB → マルチAZ

ECS/EC2 → マルチAZに配置

DB → マルチAZ














