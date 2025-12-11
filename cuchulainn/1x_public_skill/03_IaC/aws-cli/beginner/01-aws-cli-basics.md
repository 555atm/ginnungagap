# 01. AWS CLIの基礎

AWS CLIの概要、インストール、初期設定

---

## 🎯 学習目標

- AWS CLIとは何かを理解する
- インストールができる
- 初期設定（aws configure）ができる
- 最初のコマンドを実行できる

**所要時間**: 30分

---

## 🔧 AWS CLIとは？

### 概要

**AWS CLI (Command Line Interface)** = コマンドラインからAWSサービスを操作するツール

```bash
# 例：EC2インスタンス一覧を取得
aws ec2 describe-instances
```

---

### なぜAWS CLIを使うのか？

| 方法 | メリット | デメリット |
|------|---------|-----------|
| **マネジメントコンソール** | 直感的、視覚的 | 手動操作、再現性低い |
| **AWS CLI** | 自動化可能、再現性高い | コマンド学習が必要 |
| **SDK（boto3等）** | プログラム統合可能 | コーディングが必要 |

**AWS CLIが適している場面**:
- ✅ 定型作業の自動化
- ✅ スクリプトでの一括処理
- ✅ CI/CDパイプライン
- ✅ 手動確認の高速化

---

## 💻 インストール

### システム要件

- Python 3.7以降（AWS CLI v2は不要）
- OS: Linux、macOS、Windows

---

### インストール方法

#### macOS

```bash
# Homebrew でインストール（推奨）
brew install awscli

# または公式インストーラー
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

---

#### Linux

```bash
# x86_64
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# ARM
curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

---

#### Windows

```powershell
# PowerShell
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

または、[公式インストーラー](https://awscli.amazonaws.com/AWSCLIV2.msi)をダウンロードして実行

---

### インストール確認

```bash
# バージョン確認
aws --version

# 出力例
# aws-cli/2.13.0 Python/3.11.4 Darwin/23.0.0 exe/x86_64
```

**ポイント**: v2系（2.x.x）が推奨！v1系はサポート終了予定

---

## ⚙️ 初期設定（aws configure）

### 必要な情報

AWS CLIを使うには、以下の情報が必要です：

| 項目 | 説明 | 取得方法 |
|------|------|----------|
| **Access Key ID** | アクセスキーID | IAMユーザー作成時に取得 |
| **Secret Access Key** | シークレットアクセスキー | IAMユーザー作成時に取得 |
| **Default region** | デフォルトリージョン | 例: `ap-northeast-1`（東京） |
| **Default output format** | 出力形式 | `json`, `table`, `text` |

---

### IAMユーザーとアクセスキーの作成

#### Step 1: IAMユーザー作成（マネジメントコンソール）

1. AWSマネジメントコンソール → IAM
2. 「ユーザー」→「ユーザーを追加」
3. ユーザー名を入力（例: `cli-user`）
4. アクセス権限を設定（例: `AdministratorAccess`）※実務では最小権限
5. 「ユーザーを作成」

#### Step 2: アクセスキー作成

1. 作成したユーザーを選択
2. 「セキュリティ認証情報」タブ
3. 「アクセスキーを作成」
4. 「コマンドラインインターフェイス（CLI）」を選択
5. **Access Key ID** と **Secret Access Key** をメモ

⚠️ **重要**: Secret Access Keyは作成時のみ表示されます！必ず保存してください。

---

### aws configure 実行

```bash
# 基本設定
aws configure

# 対話形式で入力
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: ap-northeast-1
Default output format [None]: json
```

**入力内容**:
- **Access Key ID**: IAMで取得したアクセスキーID
- **Secret Access Key**: IAMで取得したシークレットアクセスキー
- **Default region**: `ap-northeast-1`（東京）
- **Default output format**: `json`（推奨）

---

### 設定ファイルの確認

```bash
# 設定ファイルの場所
~/.aws/config         # リージョン、出力形式
~/.aws/credentials    # アクセスキー情報

# 内容確認
cat ~/.aws/config
cat ~/.aws/credentials
```

**~/.aws/config**:
```ini
[default]
region = ap-northeast-1
output = json
```

**~/.aws/credentials**:
```ini
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

---

## 🚀 最初のコマンド実行

### 接続確認

```bash
# 現在の認証情報を確認
aws sts get-caller-identity

# 出力例
{
    "UserId": "AIDAI...",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/cli-user"
}
```

**成功すれば設定完了！** ✅

---

### よく使う最初のコマンド

#### EC2インスタンス一覧

```bash
# 自分のEC2インスタンス一覧を表示
aws ec2 describe-instances
```

#### S3バケット一覧

```bash
# 自分のS3バケット一覧を表示
aws s3 ls
```

#### IAMユーザー一覧

```bash
# IAMユーザー一覧を表示
aws iam list-users
```

---

## 🌍 リージョンについて

### デフォルトリージョン

`aws configure` で設定したリージョンがデフォルトになります。

```bash
# デフォルトリージョンで実行
aws ec2 describe-instances
```

---

### 一時的にリージョンを変更

```bash
# 大阪リージョンで実行
aws ec2 describe-instances --region ap-northeast-3

# バージニアリージョンで実行
aws ec2 describe-instances --region us-east-1
```

---

### 主なリージョン

| リージョン名 | コード | 用途 |
|-------------|--------|------|
| 東京 | `ap-northeast-1` | 日本のメイン |
| 大阪 | `ap-northeast-3` | 日本のサブ |
| バージニア | `us-east-1` | 米国東部 |
| オレゴン | `us-west-2` | 米国西部 |
| シンガポール | `ap-southeast-1` | 東南アジア |

---

## 📋 プロファイル（複数アカウント管理）

### プロファイルとは

複数のAWSアカウントや環境を切り替えて使うための機能

```bash
# プロファイルを指定して設定
aws configure --profile dev
aws configure --profile prod
```

---

### プロファイルの使い方

```bash
# プロファイルを指定してコマンド実行
aws ec2 describe-instances --profile dev
aws ec2 describe-instances --profile prod

# または環境変数で指定
export AWS_PROFILE=dev
aws ec2 describe-instances
```

---

### 設定ファイルの構造

**~/.aws/config**:
```ini
[default]
region = ap-northeast-1
output = json

[profile dev]
region = ap-northeast-1
output = json

[profile prod]
region = ap-northeast-1
output = table
```

**~/.aws/credentials**:
```ini
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[dev]
aws_access_key_id = AKIAI...DEV
aws_secret_access_key = wJalr...DEV

[prod]
aws_access_key_id = AKIAI...PROD
aws_secret_access_key = wJalr...PROD
```

**詳細は中級編で解説**

---

## ⚠️ よくあるエラーと対処法

### エラー1: コマンドが見つからない

```bash
$ aws --version
zsh: command not found: aws
```

**対処**: インストールが完了していない、またはPATHが通っていない

```bash
# インストール確認
which aws

# PATHを通す（~/.zshrc または ~/.bash_profile に追加）
export PATH="/usr/local/bin:$PATH"
```

---

### エラー2: 認証エラー

```bash
$ aws ec2 describe-instances

An error occurred (AuthFailure) when calling the DescribeInstances operation: 
AWS was not able to validate the provided access credentials
```

**対処**: アクセスキーが間違っているか、権限がない

```bash
# 設定を確認
aws configure list

# 再設定
aws configure
```

---

### エラー3: リージョンエラー

```bash
$ aws ec2 describe-instances --region invalid-region

Could not connect to the endpoint URL
```

**対処**: リージョン名が間違っている

```bash
# 正しいリージョン名を指定
aws ec2 describe-instances --region ap-northeast-1
```

---

## ✅ このレッスンのチェックリスト

- [ ] AWS CLIとは何かを理解した
- [ ] AWS CLIをインストールした
- [ ] IAMユーザーとアクセスキーを作成した
- [ ] `aws configure` で初期設定を完了した
- [ ] `aws sts get-caller-identity` で接続確認ができた
- [ ] リージョンの概念を理解した
- [ ] 最初のコマンドを実行できた

---

## 📚 次のステップ

次は **[02. 基本コマンド構造](02-basic-commands.md)** で、AWS CLIの基本的なコマンドの使い方を学びます！

---

**AWS CLIの第一歩を踏み出しました！🚀**
