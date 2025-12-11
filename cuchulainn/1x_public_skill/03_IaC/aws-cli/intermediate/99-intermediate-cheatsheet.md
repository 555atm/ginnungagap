# AWS CLI 中級チートシート 🚀

**実務で使える高度なコマンドと自動化**

---

## 🔍 JMESPath クエリ

### 基本構文

```bash
# 配列の全要素
--query 'Instances[*]'

# 特定フィールド抽出
--query 'Instances[*].InstanceId'

# 複数フィールド
--query 'Instances[*].[InstanceId,InstanceType]'

# フィールド名付き
--query 'Instances[*].{ID:InstanceId,Type:InstanceType}'
```

### 条件フィルタ

```bash
# 実行中のインスタンスのみ
--query 'Reservations[*].Instances[?State.Name==`running`]'

# t3.microのみ
--query 'Instances[?InstanceType==`t3.micro`]'

# 複数条件（AND）
--query 'Instances[?State.Name==`running` && InstanceType==`t3.micro`]'

# 複数条件（OR）
--query 'Instances[?State.Name==`running` || State.Name==`stopped`]'
```

### 配列操作

```bash
# 最初の要素
--query 'Instances[0]'

# 最後の要素
--query 'Instances[-1]'

# 範囲指定
--query 'Instances[0:3]'

# ソート
--query 'sort_by(Instances, &LaunchTime)'

# 長さ
--query 'length(Instances)'
```

### Projection

```bash
# 平坦化（flatten）
--query 'Reservations[*].Instances[*].[InstanceId]' --output text

# Nameタグ取得
--query 'Reservations[*].Instances[*].[InstanceId,Tags[?Key==`Name`].Value|[0]]'

# join関数
--query 'join(`, `, Instances[*].InstanceId)'
```

---

## 📝 スクリプトテンプレート

### 基本テンプレート

```bash
#!/bin/bash
set -euo pipefail  # エラーで即座に終了

# 変数定義
REGION="ap-northeast-1"
PROFILE="default"

# 色付きログ
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color

log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1" >&2
}

# メイン処理
main() {
    log_info "Starting script..."
    
    # AWS CLIコマンド
    if aws ec2 describe-instances --region "$REGION" --profile "$PROFILE" > /dev/null 2>&1; then
        log_info "Success"
    else
        log_error "Failed"
        exit 1
    fi
}

main "$@"
```

---

### ループ処理

```bash
# インスタンスIDのリストを取得してループ
instance_ids=$(aws ec2 describe-instances \
    --query 'Reservations[*].Instances[*].InstanceId' \
    --output text)

for instance_id in $instance_ids; do
    echo "Processing: $instance_id"
    aws ec2 describe-instances --instance-ids "$instance_id"
done
```

---

### 並列処理

```bash
# 複数リージョンで並列実行
regions=("ap-northeast-1" "us-east-1" "eu-west-1")

for region in "${regions[@]}"; do
    (
        echo "Checking $region..."
        aws ec2 describe-instances --region "$region"
    ) &
done

wait  # 全並列処理の完了を待つ
echo "All regions checked"
```

---

### エラーハンドリング

```bash
# エラーをキャッチして処理
if ! output=$(aws ec2 describe-instances 2>&1); then
    echo "Error occurred: $output"
    exit 1
fi

# リトライロジック
retry_count=0
max_retries=3

while [ $retry_count -lt $max_retries ]; do
    if aws s3 cp file.txt s3://bucket/; then
        echo "Success"
        break
    else
        retry_count=$((retry_count + 1))
        echo "Retry $retry_count/$max_retries"
        sleep 5
    fi
done
```

---

## 🔐 複数プロファイル管理

### プロファイル設定

```bash
# プロファイル作成
aws configure --profile dev
aws configure --profile stg
aws configure --profile prod

# プロファイル一覧確認
cat ~/.aws/config
cat ~/.aws/credentials
```

### プロファイル使用

```bash
# コマンドごとに指定
aws ec2 describe-instances --profile dev

# 環境変数で指定
export AWS_PROFILE=dev
aws ec2 describe-instances

# スクリプトで使用
PROFILE="dev"
aws ec2 describe-instances --profile "$PROFILE"
```

### Role切り替え

```bash
# ~/.aws/config
[profile dev]
region = ap-northeast-1
output = json

[profile prod]
region = ap-northeast-1
source_profile = dev
role_arn = arn:aws:iam::123456789012:role/ProdAccessRole
```

---

## ☁️ CloudFormation

### スタック操作

```bash
# スタック作成
aws cloudformation create-stack \
    --stack-name my-stack \
    --template-body file://template.yaml \
    --parameters file://params.json \
    --capabilities CAPABILITY_IAM

# スタック更新
aws cloudformation update-stack \
    --stack-name my-stack \
    --template-body file://template.yaml \
    --parameters file://params.json

# Change Set作成
aws cloudformation create-change-set \
    --stack-name my-stack \
    --change-set-name my-changeset \
    --template-body file://template.yaml

# Change Set実行
aws cloudformation execute-change-set \
    --change-set-name my-changeset \
    --stack-name my-stack

# スタック削除
aws cloudformation delete-stack --stack-name my-stack
```

### スタック監視

```bash
# スタック状態確認
aws cloudformation describe-stacks \
    --stack-name my-stack \
    --query 'Stacks[0].StackStatus'

# イベント監視
aws cloudformation describe-stack-events \
    --stack-name my-stack \
    --max-items 10

# 完了を待つ
aws cloudformation wait stack-create-complete \
    --stack-name my-stack
```

### パラメータファイル

```json
// params.json
[
  {
    "ParameterKey": "Environment",
    "ParameterValue": "dev"
  },
  {
    "ParameterKey": "InstanceType",
    "ParameterValue": "t3.micro"
  }
]
```

---

## 🌐 VPC・ネットワーク

### セキュリティグループ

```bash
# SG作成
aws ec2 create-security-group \
    --group-name my-sg \
    --description "My security group" \
    --vpc-id vpc-xxxxx

# インバウンドルール追加
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxx \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

# ルール削除
aws ec2 revoke-security-group-ingress \
    --group-id sg-xxxxx \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

### VPCピアリング

```bash
# ピアリング接続作成
aws ec2 create-vpc-peering-connection \
    --vpc-id vpc-xxxxx \
    --peer-vpc-id vpc-yyyyy

# ピアリング承認
aws ec2 accept-vpc-peering-connection \
    --vpc-peering-connection-id pcx-xxxxx
```

---

## 🔄 自動化パターン

### バックアップ自動化

```bash
#!/bin/bash
# AMI自動作成

INSTANCE_ID="i-xxxxx"
DATE=$(date +%Y%m%d-%H%M%S)
AMI_NAME="backup-$INSTANCE_ID-$DATE"

# AMI作成
ami_id=$(aws ec2 create-image \
    --instance-id "$INSTANCE_ID" \
    --name "$AMI_NAME" \
    --no-reboot \
    --query 'ImageId' \
    --output text)

echo "Created AMI: $ami_id"

# 古いAMI削除（7日以上前）
aws ec2 describe-images \
    --owners self \
    --filters "Name=name,Values=backup-$INSTANCE_ID-*" \
    --query "Images[?CreationDate<='$(date -d '7 days ago' -Iseconds)'].[ImageId]" \
    --output text | while read old_ami; do
        echo "Deleting old AMI: $old_ami"
        aws ec2 deregister-image --image-id "$old_ami"
    done
```

---

### リソース監視

```bash
#!/bin/bash
# 未使用EIP検出

unassociated_eips=$(aws ec2 describe-addresses \
    --query 'Addresses[?AssociationId==null].[PublicIp,AllocationId]' \
    --output text)

if [ -n "$unassociated_eips" ]; then
    echo "Unassociated EIPs found:"
    echo "$unassociated_eips"
    # Slack通知などを追加
fi
```

---

### コスト最適化

```bash
#!/bin/bash
# 停止中のインスタンスを検出

stopped_instances=$(aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=stopped" \
    --query 'Reservations[*].Instances[*].[InstanceId,Tags[?Key==`Name`].Value|[0]]' \
    --output text)

if [ -n "$stopped_instances" ]; then
    echo "Stopped instances found:"
    echo "$stopped_instances"
fi
```

---

## 🛠️ デバッグ

### デバッグモード

```bash
# 詳細ログ
aws ec2 describe-instances --debug

# HTTPリクエスト/レスポンスを表示
aws ec2 describe-instances --debug 2>&1 | grep -A 20 "Making request"
```

### トラブルシューティング

```bash
# 認証情報確認
aws sts get-caller-identity

# リージョン確認
aws configure get region

# プロファイル確認
aws configure list --profile dev

# 設定ファイル確認
cat ~/.aws/config
cat ~/.aws/credentials
```

---

## 📊 jq との連携

```bash
# インストール
brew install jq  # Mac
sudo apt install jq  # Linux

# 基本的な使い方
aws ec2 describe-instances | jq '.'

# フィルタリング
aws ec2 describe-instances | jq '.Reservations[].Instances[] | {InstanceId, State}'

# 配列操作
aws ec2 describe-instances | jq '.Reservations[].Instances[].InstanceId'

# 条件フィルタ
aws ec2 describe-instances | jq '.Reservations[].Instances[] | select(.State.Name=="running")'
```

---

## 💡 実務Tips

### ログ出力

```bash
# コマンドログを残す
script_name=$(basename "$0")
log_file="/var/log/${script_name%.*}.log"

exec > >(tee -a "$log_file")
exec 2>&1

echo "[$(date)] Script started"
```

### 環境変数管理

```bash
# .env ファイル
# .env
AWS_PROFILE=dev
AWS_REGION=ap-northeast-1
ENVIRONMENT=development

# 読み込み
source .env
aws ec2 describe-instances --profile "$AWS_PROFILE" --region "$AWS_REGION"
```

### Git管理

```bash
# .gitignore
.env
*.log
credentials
*.pem

# スクリプトをバージョン管理
git init
git add scripts/
git commit -m "Add automation scripts"
```

---

## ⚠️ セキュリティ

### クレデンシャル管理

```bash
# ❌ ハードコードしない
ACCESS_KEY="AKIAI..."  # NG!

# ✅ プロファイルを使う
aws ec2 describe-instances --profile dev

# ✅ IAM Roleを使う（EC2、Lambda等）
aws ec2 describe-instances  # IAM Roleから自動取得
```

### MFA

```bash
# MFAトークン取得
aws sts get-session-token \
    --serial-number arn:aws:iam::123456789012:mfa/user \
    --token-code 123456

# 一時的な認証情報を環境変数にセット
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
```

---

## ✅ 中級修了チェックリスト

- [ ] JMESPathで複雑なクエリが書ける
- [ ] エラーハンドリング付きスクリプトが書ける
- [ ] 複数プロファイルを使い分けられる
- [ ] CloudFormationスタックをCLIで操作できる
- [ ] 実務で使える自動化スクリプトを作成できる
- [ ] トラブルシューティングができる

---

## 📚 さらに学ぶ

- [CloudFormation学習ガイド](../../CFn/cfn-text/README.md)
- [Terraform学習ガイド](../../Terraform/)
- [AWS CDK](https://aws.amazon.com/jp/cdk/)

---

**このチートシートを活用して、効率的なAWS運用を実現しましょう！🚀**
