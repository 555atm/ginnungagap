1. AWS 基本概念
1-1. AWSの特徴（スケーラビリティ / 可用性 / 耐障害性）
● スケーラビリティ（Scaling）

必要なときにリソースを増やし、不要になれば減らせる

Auto Scaling（EC2 / ECS / RDS RR など）で自動化が基本

大前提：横に増やす（スケールアウト） 設計が前提
　→ 1台が高負荷になる前に分散させる

● 可用性（High Availability）

最低でもマルチAZ がAWSの基本思想

単一AZ運用は「テスト用」

可用性は「設計しない限り自動で高くならない」

EC2単体 → 耐障害性なし

RDS マルチAZ → 自動フェイルオーバー

ALB → AZ跨ぎで流す前提

● 耐障害性（Fault Tolerance）

インスタンス/ハード障害はいつでも起こる前提

代替リソースが自動で立ち上がる設計（AutoHeal / ALB / Multi-AZ）

「壊れてもサービスが落ちない構成」が AWS の正解

1-2. リージョン / アベイラビリティゾーン / エッジロケーション
● リージョン（物理的に離れた巨大データセンターの集合）

東京、バージニア、シンガポールなど

設計上は「国をまたぐレベルの距離」として扱う

リージョンまたぎはDR（災害対策）用途

● AZ（Availability Zone）

1リージョン内に複数存在する“独立データセンター”

電源・ネットワークが独立 → 片方が落ちても片方は生きる

AWSの高可用性は「マルチAZ」がすべての基礎

● エッジロケーション（CloudFront の配信拠点）

最もユーザーに近い場所でキャッシュ配信

パフォーマンス向上とDDoS吸収の役割

本番サイトは CloudFront + WAF + ALB/ECS が鉄板

1-3. アカウント構造（Organizations / メンバーアカウント）
● Organizations の役割

複数アカウントを 一元管理 する仕組み

大前提：AWSはアカウントを環境ごとに分ける方が安全
　（dev / stg / prod / shared / security …）

● メンバーアカウント

プロジェクト単位・環境単位で分割

アカウント分離は以下を明確にする

セキュリティ境界

請求の切り分け

権限管理（SCPで制御）

● ベストプラクティス

prod は専用アカウント

セキュリティアカウントは必須（GuardDuty, CloudTrail集約）

Shared アカウントにECR/S3等の共通リソースを置くことも多い

1-4. メタデータ（タグ・リソースID・ARN）
● タグ（Tag）

AWS設計で 最重要のメタデータ。
目的は：

コスト配分（Cost Allocation Tag）

運用管理（owner / system / env）

IaC連携（自動探索）

監視対象識別

セキュリティ分類

「名前よりタグで管理する」 が AWS 文化

● リソースID

i-xxxx, subnet-xxxx, sg-xxxx など

物理的な識別子であり、人間向けではない

● ARN（Amazon Resource Name）

AWSリソースを唯一識別する文字列

IAMポリシー、Lambda、S3 バケットポリシーなどで必須

「権限管理の核」となるので仕組み理解は重要

例：

arn:aws:s3:::my-bucket
arn:aws:ecs:ap-northeast-1:123456789012:service/my-service

1-5. 料金体系（従量課金 / 契約系 / Free Tierの注意点）
● 従量課金（使った分だけ請求）

代表例：

EC2（起動時間）

NAT Gateway（通信料が高額）

S3（保存量 / リクエスト）

Lambda（実行時間）

VPC Endpoint（時間＋通信量）

「使い続けると気づかず料金が増える」系は監視必須

● 契約系（利用コミットで割引）

Savings Plans（Compute / EC2）

Reserved Instance（RDS / EC2）

本番は Savings Plans 前提で考えると良い。

● Free Tier の落とし穴

Free Tier 対象外のサービスも多い（NATGW / ALB 等）

無料なのは「一定の条件下のみ」

複数リージョンに作ると無料枠外

課金アラームは必須（CloudWatch + SNS）

まとめ（AWS設計の最重要ポイント）

AWSは「壊れる前提」でマルチAZ設計をする

アカウントは分割が基本（prod分離は必須）

タグ設計がコスト・監視・IaCのすべてを支える

料金は「通信料」「データ転送」「常時稼働」の3つを最優先で見る

Free Tierは信用しない

高可用性の鍵は ALB / Auto Scaling / Multi-AZ
