# AWS CLI 初級チートシート 📋

**よく使うコマンドのクイックリファレンス**

---

## 🎯 基本構造

```bash
aws <service> <command> [options] [parameters]
```

---

## ⚙️ 初期設定

```bash
# 設定
aws configure

# 設定確認
aws configure list

# 接続確認
aws sts get-caller-identity
```

---

## 🔧 共通オプション

| オプション | 説明 | 例 |
|-----------|------|-----|
| `--region` | リージョン指定 | `--region ap-northeast-1` |
| `--profile` | プロファイル指定 | `--profile dev` |
| `--output` | 出力形式（json/table/text） | `--output table` |
| `--query` | 出力フィルタ | `--query 'Instances[*].InstanceId'` |
| `--dry-run` | ドライラン | `--dry-run` |

---

## 💻 EC2

### インスタンス一覧

```bash
# 全インスタンス
aws ec2 describe-instances

# 実行中のみ
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"

# 特定インスタンス
aws ec2 describe-instances --instance-ids i-xxxxx
```

### インスタンス操作

```bash
# 起動
aws ec2 start-instances --instance-ids i-xxxxx

# 停止
aws ec2 stop-instances --instance-ids i-xxxxx

# 再起動
aws ec2 reboot-instances --instance-ids i-xxxxx

# 削除
aws ec2 terminate-instances --instance-ids i-xxxxx
```

### AMI

```bash
# AMI一覧（自分の）
aws ec2 describe-images --owners self

# Amazon Linux 2023 AMI検索
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=al2023-ami-*" \
  --query 'Images[*].[ImageId,Name,CreationDate]' \
  --output table
```

---

## 📦 S3

### バケット操作

```bash
# バケット一覧
aws s3 ls

# バケット作成
aws s3 mb s3://my-bucket

# バケット削除（空の場合）
aws s3 rb s3://my-bucket

# バケット削除（中身ごと）
aws s3 rb s3://my-bucket --force
```

### オブジェクト操作

```bash
# オブジェクト一覧
aws s3 ls s3://my-bucket
aws s3 ls s3://my-bucket/folder/

# アップロード
aws s3 cp file.txt s3://my-bucket/
aws s3 cp folder/ s3://my-bucket/folder/ --recursive

# ダウンロード
aws s3 cp s3://my-bucket/file.txt ./
aws s3 cp s3://my-bucket/folder/ ./folder/ --recursive

# 同期
aws s3 sync ./local-folder s3://my-bucket/remote-folder
aws s3 sync s3://my-bucket/remote-folder ./local-folder

# 削除
aws s3 rm s3://my-bucket/file.txt
aws s3 rm s3://my-bucket/folder/ --recursive
```

---

## 👤 IAM

### ユーザー

```bash
# ユーザー一覧
aws iam list-users

# ユーザー作成
aws iam create-user --user-name new-user

# ユーザー削除
aws iam delete-user --user-name old-user

# ユーザー情報取得
aws iam get-user --user-name my-user
```

### アクセスキー

```bash
# アクセスキー一覧
aws iam list-access-keys --user-name my-user

# アクセスキー作成
aws iam create-access-key --user-name my-user

# アクセスキー削除
aws iam delete-access-key --user-name my-user --access-key-id AKIAI...
```

### ポリシー

```bash
# ポリシー一覧（AWS管理）
aws iam list-policies --scope AWS

# ポリシー一覧（カスタム）
aws iam list-policies --scope Local

# ユーザーにポリシーをアタッチ
aws iam attach-user-policy \
  --user-name my-user \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

---

## 🌐 VPC

### VPC・Subnet

```bash
# VPC一覧
aws ec2 describe-vpcs

# Subnet一覧
aws ec2 describe-subnets

# 特定VPCのSubnet一覧
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-xxxxx"
```

### セキュリティグループ

```bash
# セキュリティグループ一覧
aws ec2 describe-security-groups

# 特定SG詳細
aws ec2 describe-security-groups --group-ids sg-xxxxx
```

---

## 🔍 出力フィルタリング

### --query（基本）

```bash
# インスタンスIDのみ
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text

# インスタンスID と タイプ
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId,InstanceType]' \
  --output table

# Name タグを持つインスタンス
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId,Tags[?Key==`Name`].Value|[0]]' \
  --output table
```

### --filters

```bash
# 実行中のインスタンス
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running"

# Name タグでフィルタ
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=WebServer"

# 複数条件
aws ec2 describe-instances \
  --filters \
    "Name=instance-state-name,Values=running" \
    "Name=instance-type,Values=t3.micro"
```

---

## 🔄 CloudFormation

```bash
# スタック一覧
aws cloudformation list-stacks

# スタック作成
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# スタック更新
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# スタック削除
aws cloudformation delete-stack --stack-name my-stack

# スタック詳細
aws cloudformation describe-stacks --stack-name my-stack

# スタックイベント
aws cloudformation describe-stack-events --stack-name my-stack
```

---

## 📊 その他のサービス

### RDS

```bash
# DBインスタンス一覧
aws rds describe-db-instances

# DBスナップショット一覧
aws rds describe-db-snapshots
```

### Lambda

```bash
# Lambda関数一覧
aws lambda list-functions

# Lambda関数実行
aws lambda invoke \
  --function-name my-function \
  response.json
```

### CloudWatch Logs

```bash
# ロググループ一覧
aws logs describe-log-groups

# ログストリーム一覧
aws logs describe-log-streams --log-group-name /aws/lambda/my-function

# ログイベント取得
aws logs tail /aws/lambda/my-function --follow
```

---

## 🔐 認証・プロファイル

```bash
# プロファイル設定
aws configure --profile dev

# プロファイル使用
aws ec2 describe-instances --profile dev

# 環境変数でプロファイル指定
export AWS_PROFILE=dev
aws ec2 describe-instances

# 現在の認証情報確認
aws sts get-caller-identity
```

---

## 🌍 リージョン

| リージョン名 | コード |
|-------------|--------|
| 東京 | `ap-northeast-1` |
| 大阪 | `ap-northeast-3` |
| バージニア | `us-east-1` |
| オレゴン | `us-west-2` |

```bash
# リージョン指定
aws ec2 describe-instances --region ap-northeast-1

# 環境変数でリージョン指定
export AWS_DEFAULT_REGION=ap-northeast-1
```

---

## 💡 便利Tips

### エイリアス

```bash
# ~/.bashrc または ~/.zshrc
alias awsp='aws --profile'
alias awsr='aws --region'
alias awst='aws --output table'
```

### jq で整形

```bash
# jqインストール（Mac）
brew install jq

# 使用例
aws ec2 describe-instances | jq '.Reservations[].Instances[] | {InstanceId, State}'
```

### 複数コマンド実行

```bash
# 全リージョンのEC2インスタンス数を確認
for region in $(aws ec2 describe-regions --query 'Regions[].RegionName' --output text); do
  echo "Region: $region"
  aws ec2 describe-instances --region $region --query 'length(Reservations[].Instances[])'
done
```

---

## ⚠️ よくあるエラー

| エラー | 原因 | 対処 |
|--------|------|------|
| `AuthFailure` | 認証エラー | アクセスキーを確認 |
| `UnauthorizedOperation` | 権限不足 | IAMポリシーを確認 |
| `InvalidInstanceID.NotFound` | リソースが存在しない | IDを確認 |
| `DryRunOperation` | ドライラン成功 | `--dry-run` を外して実行 |

---

## 📚 helpコマンド

```bash
# サービス一覧
aws help

# サービスのコマンド一覧
aws ec2 help

# コマンドの詳細
aws ec2 describe-instances help
```

---

## ✅ 初級修了チェックリスト

- [ ] `aws configure` で設定できる
- [ ] EC2インスタンスの起動・停止ができる
- [ ] S3にファイルをアップロード・ダウンロードできる
- [ ] IAMユーザーの一覧を確認できる
- [ ] `--query` で必要な情報を抽出できる
- [ ] `help` コマンドでドキュメントを参照できる

---

## 🚀 次のステップ

初級編を修了したら、**[中級編](../intermediate/README.md)** へ進みましょう！

- 高度なフィルタリング（JMESPath）
- シェルスクリプトでの自動化
- 複数アカウント管理
- 実務での活用Tips

---

**このチートシートを手元に置いて、AWS CLIを使いこなしましょう！📚**
