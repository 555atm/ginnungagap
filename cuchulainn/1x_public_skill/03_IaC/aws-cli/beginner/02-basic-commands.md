# 02. 基本コマンド構造

AWS CLIコマンドの基本構造、help、オプションの使い方

---

## 🎯 学習目標

- AWS CLIのコマンド構造を理解する
- helpコマンドを使いこなせる
- オプションとパラメータの違いを理解する
- ドライランモードを使える

**所要時間**: 30分

---

## 📐 コマンドの基本構造

### 基本形

```bash
aws <service> <command> [options] [parameters]
```

**構成要素**:

| 要素 | 説明 | 例 |
|------|------|-----|
| `aws` | AWS CLIの実行コマンド | `aws` |
| `<service>` | AWSサービス名 | `ec2`, `s3`, `iam` |
| `<command>` | 実行する操作 | `describe-instances`, `list-buckets` |
| `[options]` | オプション | `--region`, `--profile` |
| `[parameters]` | パラメータ | `--instance-ids`, `--bucket-name` |

---

### 具体例

```bash
# EC2インスタンス一覧を東京リージョンで取得
aws ec2 describe-instances --region ap-northeast-1

# 解説:
# aws           : AWS CLI実行
# ec2           : EC2サービス
# describe-instances : インスタンス一覧取得コマンド
# --region      : オプション（リージョン指定）
# ap-northeast-1 : パラメータ（東京リージョン）
```

---

## 📖 helpコマンドの使い方

### 3段階のhelpレベル

AWS CLIのhelpは3段階あります：

#### Level 1: サービス一覧

```bash
# AWS CLIで使えるサービス一覧
aws help
```

---

#### Level 2: サービスのコマンド一覧

```bash
# EC2サービスで使えるコマンド一覧
aws ec2 help

# S3サービスで使えるコマンド一覧
aws s3 help
```

---

#### Level 3: コマンドの詳細

```bash
# describe-instancesコマンドの詳細
aws ec2 describe-instances help

# list-bucketsコマンドの詳細
aws s3api list-buckets help
```

---

### helpの読み方

```bash
$ aws ec2 describe-instances help

NAME
       describe-instances - Describes one or more of your instances.

SYNOPSIS
            describe-instances
          [--filters <value>]
          [--instance-ids <value>]
          [--dry-run | --no-dry-run]
          [--cli-input-json <value>]
          ...

DESCRIPTION
       Describes the specified instances or all instances.
       ...

OPTIONS
       --filters (list)
              The filters.
              ...
```

**ポイント**:
- `[...]` = オプション（省略可能）
- `<value>` = 値を指定する必要がある

---

### 実践的なhelpの使い方

```bash
# 1. まずサービスのhelpを見る
aws ec2 help

# 2. 目的のコマンドを探す
# （例：インスタンスを停止したい → stop-instances）

# 3. コマンドの詳細を見る
aws ec2 stop-instances help

# 4. 必要なオプションを確認して実行
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
```

---

## ⚙️ オプションとパラメータ

### 共通オプション

すべてのコマンドで使えるオプション：

| オプション | 説明 | 例 |
|-----------|------|-----|
| `--region` | リージョン指定 | `--region ap-northeast-1` |
| `--profile` | プロファイル指定 | `--profile dev` |
| `--output` | 出力形式 | `--output json` |
| `--query` | 出力フィルタ | `--query 'Reservations[*].Instances[*]'` |
| `--dry-run` | ドライラン（実行しない） | `--dry-run` |

---

### サービス固有のパラメータ

各コマンドに固有のパラメータ：

```bash
# EC2: インスタンスIDを指定
aws ec2 describe-instances --instance-ids i-1234567890abcdef0

# S3: バケット名を指定
aws s3 ls s3://my-bucket

# IAM: ユーザー名を指定
aws iam get-user --user-name my-user
```

---

### パラメータの指定方法

#### 単一値

```bash
# 1つの値を指定
aws ec2 describe-instances --instance-ids i-1234567890abcdef0
```

---

#### 複数値（スペース区切り）

```bash
# 複数の値を指定
aws ec2 describe-instances --instance-ids i-12345 i-67890 i-abcde
```

---

#### JSON形式

```bash
# JSON形式で指定（複雑な構造）
aws ec2 create-tags \
  --resources i-1234567890abcdef0 \
  --tags '[{"Key":"Name","Value":"WebServer"},{"Key":"Env","Value":"dev"}]'
```

---

## 🔍 出力フォーマット

### 3つの出力形式

| 形式 | 説明 | 用途 |
|------|------|------|
| `json` | JSON形式（デフォルト） | プログラム処理、詳細確認 |
| `table` | テーブル形式 | 人間が見やすい |
| `text` | タブ区切りテキスト | シェルスクリプト |

---

### json形式（デフォルト）

```bash
$ aws ec2 describe-instances --output json

{
    "Reservations": [
        {
            "Instances": [
                {
                    "InstanceId": "i-1234567890abcdef0",
                    "InstanceType": "t3.micro",
                    ...
                }
            ]
        }
    ]
}
```

**特徴**:
- ✅ プログラムで処理しやすい
- ✅ 全ての情報が含まれる
- ❌ 人間には読みにくい

---

### table形式

```bash
$ aws ec2 describe-instances --output table

---------------------------------------
|        DescribeInstances             |
+-------------------------------------+
||           Reservations              ||
|+-----------------------------------+|
|||           Instances               |||
||+-------------+--------------------+||
|||  InstanceId | InstanceType       |||
||+-------------+--------------------+||
|||  i-123...   |  t3.micro          |||
||+-------------+--------------------+||
```

**特徴**:
- ✅ 人間が見やすい
- ✅ ターミナルで確認しやすい
- ❌ プログラム処理には不向き

---

### text形式

```bash
$ aws ec2 describe-instances --output text

RESERVATIONS    123456789012    r-1234567890abcdef0
INSTANCES       ...     i-1234567890abcdef0     t3.micro
```

**特徴**:
- ✅ シェルスクリプトで処理しやすい
- ✅ `cut`, `awk` で加工しやすい
- ❌ 階層構造がわかりにくい

---

### 実践例：出力形式の使い分け

```bash
# 詳細確認（JSON）
aws ec2 describe-instances --output json

# ざっと見る（TABLE）
aws ec2 describe-instances --output table

# スクリプトで処理（TEXT）
instance_id=$(aws ec2 describe-instances --output text \
  --query 'Reservations[0].Instances[0].InstanceId')
```

---

## 🧪 ドライランモード

### --dry-run オプション

**実際には実行せず、権限チェックだけを行う**

```bash
# ドライラン（実行しない）
aws ec2 stop-instances \
  --instance-ids i-1234567890abcdef0 \
  --dry-run

# 成功例（権限がある場合）
An error occurred (DryRunOperation) when calling the StopInstances operation: 
Request would have succeeded, but DryRun flag is set.

# 失敗例（権限がない場合）
An error occurred (UnauthorizedOperation) when calling the StopInstances operation: 
You are not authorized to perform this operation.
```

**用途**:
- ✅ 本番環境で実行前の権限確認
- ✅ スクリプトのデバッグ
- ✅ 操作の影響範囲確認

---

### 実践例：本番環境での安全な操作

```bash
# Step 1: ドライランで確認
aws ec2 stop-instances \
  --instance-ids i-1234567890abcdef0 \
  --dry-run

# Step 2: エラーがなければ本実行
aws ec2 stop-instances \
  --instance-ids i-1234567890abcdef0
```

---

## 🔧 よく使うコマンドパターン

### パターン1: 一覧取得

```bash
# EC2インスタンス一覧
aws ec2 describe-instances

# S3バケット一覧
aws s3 ls

# IAMユーザー一覧
aws iam list-users
```

**命名規則**:
- `describe-*`: 詳細情報を取得
- `list-*`: 一覧を取得

---

### パターン2: 特定リソースの情報取得

```bash
# 特定のEC2インスタンス情報
aws ec2 describe-instances --instance-ids i-1234567890abcdef0

# 特定のS3バケット情報
aws s3api head-bucket --bucket my-bucket

# 特定のIAMユーザー情報
aws iam get-user --user-name my-user
```

---

### パターン3: リソース作成

```bash
# EC2インスタンス起動
aws ec2 run-instances \
  --image-id ami-xxxxx \
  --instance-type t3.micro

# S3バケット作成
aws s3 mb s3://my-new-bucket

# IAMユーザー作成
aws iam create-user --user-name new-user
```

---

### パターン4: リソース削除

```bash
# EC2インスタンス削除
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# S3バケット削除
aws s3 rb s3://my-bucket

# IAMユーザー削除
aws iam delete-user --user-name old-user
```

---

## ⚠️ よくあるエラーと対処法

### エラー1: パラメータ不足

```bash
$ aws ec2 stop-instances

usage: aws [options] <command> <subcommand> [<subcommand> ...] [parameters]
To see help text, you can run:
  aws help
  aws <command> help
  aws <command> <subcommand> help

aws: error: the following arguments are required: --instance-ids
```

**対処**: helpでrequired（必須）パラメータを確認

```bash
aws ec2 stop-instances help
```

---

### エラー2: 権限エラー

```bash
$ aws ec2 stop-instances --instance-ids i-1234567890abcdef0

An error occurred (UnauthorizedOperation) when calling the StopInstances operation: 
You are not authorized to perform this operation.
```

**対処**: IAMポリシーで権限を確認・付与

---

### エラー3: リソースが存在しない

```bash
$ aws ec2 describe-instances --instance-ids i-invalid

An error occurred (InvalidInstanceID.NotFound) when calling the DescribeInstances operation: 
The instance ID 'i-invalid' does not exist
```

**対処**: インスタンスIDを確認

```bash
# まず一覧を確認
aws ec2 describe-instances
```

---

## 💡 実践Tips

### Tip 1: エイリアスを設定

```bash
# ~/.bashrc または ~/.zshrc に追加
alias awsp='aws --profile'
alias awsr='aws --region'

# 使用例
awsp dev ec2 describe-instances
awsr ap-northeast-1 ec2 describe-instances
```

---

### Tip 2: historyを活用

```bash
# 過去のコマンドを検索
history | grep aws

# 番号指定で再実行
!123
```

---

### Tip 3: 長いコマンドは改行

```bash
# 読みやすく改行（\ で継続）
aws ec2 run-instances \
  --image-id ami-xxxxx \
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids sg-xxxxx \
  --subnet-id subnet-xxxxx
```

---

## ✅ このレッスンのチェックリスト

- [ ] AWS CLIのコマンド構造を理解した
- [ ] helpコマンドを3段階で使い分けられる
- [ ] オプションとパラメータの違いがわかる
- [ ] 3つの出力形式を使い分けられる
- [ ] ドライランモードを理解した
- [ ] よく使うコマンドパターンを理解した

---

## 📚 次のステップ

次は **[03. EC2操作の基礎](03-ec2-basics.md)** で、実際のEC2サービスを操作してみます！

---

**AWS CLIの基本構造をマスターしました！🎉**
