# 02. テンプレート基本構文

CloudFormationテンプレートの書き方をマスター

---

## 🎯 学習目標

- YAMLの基本文法を理解する
- テンプレートの各セクションの役割を理解する
- 実際にテンプレートを書けるようになる

**所要時間**: 45分

---

## 📝 YAML vs JSON

CloudFormationテンプレートは**YAML**または**JSON**で記述できます。

### YAML（推奨）

**メリット**:
- ✅ 読みやすい
- ✅ コメントが書ける
- ✅ 記述量が少ない

```yaml
# コメントが書ける
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-bucket
```

### JSON

**デメリット**:
- ❌ 読みにくい
- ❌ コメント不可
- ❌ 記述量が多い

```json
{
  "Resources": {
    "MyBucket": {
      "Type": "AWS::S3::Bucket",
      "Properties": {
        "BucketName": "my-bucket"
      }
    }
  }
}
```

**💡 結論**: **YAMLを使いましょう！**

---

## 📄 テンプレートの基本構造

### 完全版テンプレート

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: テンプレートの説明

# ==========================================
# Metadata: テンプレートのメタ情報
# ==========================================
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: "Network Configuration"
        Parameters:
          - VpcCidr
          - SubnetCidr

# ==========================================
# Parameters: 入力パラメータ
# ==========================================
Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - stg
      - prod
    Description: Environment name

# ==========================================
# Mappings: 環境別設定マップ
# ==========================================
Mappings:
  EnvironmentMap:
    dev:
      InstanceType: t3.small
    prod:
      InstanceType: m5.large

# ==========================================
# Conditions: 条件分岐
# ==========================================
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

# ==========================================
# Resources: リソース定義（必須）
# ==========================================
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${AWS::StackName}-bucket'

# ==========================================
# Outputs: 出力値
# ==========================================
Outputs:
  BucketName:
    Description: S3 Bucket Name
    Value: !Ref MyBucket
    Export:
      Name: !Sub '${AWS::StackName}-BucketName'
```

### セクション一覧

| セクション | 必須 | 説明 |
|----------|------|------|
| `AWSTemplateFormatVersion` | 推奨 | テンプレート形式のバージョン |
| `Description` | 推奨 | テンプレートの説明 |
| `Metadata` | 任意 | メタ情報（パラメータグループ等） |
| `Parameters` | 任意 | 入力パラメータ |
| `Mappings` | 任意 | キー・バリューマップ |
| `Conditions` | 任意 | 条件式 |
| `Resources` | **必須** | 作成するAWSリソース |
| `Outputs` | 推奨 | 出力値 |

---

## 🔤 YAMLの基本文法

### 1. インデント（重要！）

**スペース2つ**でインデント（タブNG）

```yaml
# ✅ 正しい（スペース2つ）
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-bucket

# ❌ 間違い（インデントがバラバラ）
Resources:
MyBucket:
   Type: AWS::S3::Bucket
```

### 2. キー・バリュー

```yaml
# 基本形
Key: Value

# 複数階層
Parent:
  Child: Value
  Child2: Value2
```

### 3. リスト

```yaml
# 方法1: ハイフン
SecurityGroupIds:
  - sg-12345
  - sg-67890

# 方法2: JSON形式
SecurityGroupIds: [sg-12345, sg-67890]
```

### 4. 文字列

```yaml
# クォート不要（通常）
BucketName: my-bucket

# クォート必要（特殊文字・数値を文字列として扱う場合）
BucketName: "my-bucket-123"
BucketName: 'my-bucket'

# 複数行（|: 改行保持）
UserData: |
  #!/bin/bash
  yum update -y
  echo "Hello World"

# 複数行（>: 改行を空白に変換）
Description: >
  This is a long description
  that spans multiple lines.
```

### 5. コメント

```yaml
# これはコメント
Resources:
  MyBucket:  # 行末コメント
    Type: AWS::S3::Bucket
```

### 6. 真偽値

```yaml
# true/false
Enabled: true
MultiAZ: false

# yes/no も使える
Enabled: yes
MultiAZ: no
```

---

## 📦 Resources セクション（最重要）

### 基本構造

```yaml
Resources:
  論理ID:
    Type: AWS::サービス名::リソースタイプ
    Properties:
      プロパティ名: 値
```

### 例: S3バケット

```yaml
Resources:
  MyBucket:                    # ← 論理ID（任意の名前）
    Type: AWS::S3::Bucket      # ← リソースタイプ（必須）
    Properties:                # ← プロパティ（リソースごとに異なる）
      BucketName: my-unique-bucket-name
      VersioningConfiguration:
        Status: Enabled
      Tags:
        - Key: Environment
          Value: dev
```

### 論理ID（Logical ID）の命名ルール

**ルール**:
- 英数字のみ（A-Z, a-z, 0-9）
- 最初は英字
- ハイフン・アンダースコア不可

```yaml
# ✅ 良い例
Resources:
  MyVPC:
  WebServer1:
  DatabasePrimary:

# ❌ 悪い例
Resources:
  My-VPC:         # ハイフン不可
  Web_Server:     # アンダースコア不可
  1stServer:      # 数字で始まる不可
```

**命名のベストプラクティス**:
```yaml
# パスカルケース推奨
Resources:
  MyVpc:              # VPC
  PublicSubnet1:      # サブネット
  WebServerInstance:  # EC2
  DatabasePrimary:    # RDS
```

---

## 🏷️ タグの書き方

### 方法1: リスト形式

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      Tags:
        - Key: Name
          Value: MyBucket
        - Key: Environment
          Value: dev
        - Key: Project
          Value: MyApp
```

### 方法2: キー・バリュー形式

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      Tags:
        - Key: Name
          Value: WebServer
```

---

## 🎨 実践例: 最小のテンプレート

### 例1: S3バケットだけ

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Simple S3 Bucket

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
```

**デプロイ**:
```bash
aws cloudformation create-stack \
  --stack-name simple-bucket \
  --template-body file://simple-bucket.yaml
```

---

### 例2: VPCだけ

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Simple VPC

Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: MyVPC
```

---

### 例3: VPC + サブネット

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: VPC with Subnet

Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: MyVPC

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC              # ← VPCを参照
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select 
        - 0
        - !GetAZs ''                 # ← 最初のAZを取得
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: PublicSubnet

Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref MyVPC

  SubnetId:
    Description: Subnet ID
    Value: !Ref PublicSubnet
```

---

## 🔍 よく使うリソースタイプ

### ネットワーク

```yaml
Resources:
  # VPC
  MyVPC:
    Type: AWS::EC2::VPC
  
  # Subnet
  MySubnet:
    Type: AWS::EC2::Subnet
  
  # Internet Gateway
  MyIGW:
    Type: AWS::EC2::InternetGateway
  
  # Route Table
  MyRouteTable:
    Type: AWS::EC2::RouteTable
  
  # Security Group
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
```

### コンピューティング

```yaml
Resources:
  # EC2
  MyEC2:
    Type: AWS::EC2::Instance
  
  # Lambda
  MyFunction:
    Type: AWS::Lambda::Function
```

### ストレージ

```yaml
Resources:
  # S3
  MyBucket:
    Type: AWS::S3::Bucket
  
  # EBS
  MyVolume:
    Type: AWS::EC2::Volume
```

### データベース

```yaml
Resources:
  # RDS
  MyDB:
    Type: AWS::RDS::DBInstance
  
  # DynamoDB
  MyTable:
    Type: AWS::DynamoDB::Table
```

---

## 🔧 テンプレート検証

### AWS CLIで検証

```bash
# 構文チェック
aws cloudformation validate-template \
  --template-body file://template.yaml

# 成功時の出力
{
    "Parameters": [],
    "Description": "Simple VPC",
    "Capabilities": []
}
```

### cfn-lint で詳細チェック

```bash
# インストール
pip install cfn-lint

# チェック実行
cfn-lint template.yaml

# エラーがあれば表示
# E0000: 構文エラー
# W0000: 警告
```

---

## 💡 ベストプラクティス

### 1. Description は必ず書く

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: |
  VPC with public/private subnets
  Created: 2025-12-11
  Purpose: Learning CloudFormation
```

### 2. リソースにコメントを付ける

```yaml
Resources:
  # ==========================================
  # VPC - 10.0.0.0/16
  # ==========================================
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
```

### 3. タグを統一する

```yaml
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      Tags:
        - Key: Name
          Value: MyVPC
        - Key: Environment
          Value: dev
        - Key: Project
          Value: MyApp
        - Key: ManagedBy
          Value: CloudFormation
```

### 4. インデントを統一する

```yaml
# ✅ スペース2つで統一
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
```

---

## 🎓 実習: 初めてのテンプレート作成

### Step 1: テンプレート作成

```yaml
# my-first-template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: My First CloudFormation Template

Resources:
  # S3バケット
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${AWS::StackName}-bucket-${AWS::AccountId}'
      Tags:
        - Key: Name
          Value: MyFirstBucket
        - Key: Purpose
          Value: Learning

Outputs:
  BucketName:
    Description: Name of the S3 bucket
    Value: !Ref MyBucket
  
  BucketArn:
    Description: ARN of the S3 bucket
    Value: !GetAtt MyBucket.Arn
```

### Step 2: バリデーション

```bash
aws cloudformation validate-template \
  --template-body file://my-first-template.yaml
```

### Step 3: デプロイ

```bash
aws cloudformation create-stack \
  --stack-name my-first-stack \
  --template-body file://my-first-template.yaml
```

### Step 4: 確認

```bash
# スタック状態確認
aws cloudformation describe-stacks --stack-name my-first-stack

# 出力値確認
aws cloudformation describe-stacks \
  --stack-name my-first-stack \
  --query 'Stacks[0].Outputs'
```

### Step 5: 削除

```bash
aws cloudformation delete-stack --stack-name my-first-stack
```

---

## ⚠️ よくあるYAMLエラー

### エラー1: インデントミス

```yaml
# ❌ 間違い
Resources:
MyBucket:
  Type: AWS::S3::Bucket

# ✅ 正しい
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
```

### エラー2: タブを使用

```yaml
# ❌ タブ使用（見た目は同じでもエラー）
Resources:
	MyBucket:

# ✅ スペース使用
Resources:
  MyBucket:
```

### エラー3: クォートミス

```yaml
# ❌ 数値が文字列として扱われる
Port: "80"

# ✅ 数値
Port: 80

# ✅ 文字列（特殊文字がある場合）
BucketName: "my-bucket-2025"
```

---

## ✅ このレッスンのチェックリスト

- [ ] YAMLの基本文法を理解した
- [ ] インデント（スペース2つ）の重要性を理解した
- [ ] テンプレートの各セクションの役割を理解した
- [ ] Resources セクションの書き方を理解した
- [ ] 論理IDの命名ルールを理解した
- [ ] タグの書き方を理解した
- [ ] テンプレートを作成・デプロイ・削除できた

---

## 📚 次のステップ

次は **[03. Parameters, Mappings, Conditions](03-parameters-mappings-conditions.md)** で、テンプレートを柔軟にする方法を学びます！

---

**YAML構文をマスターして、次のステップへ！🚀**
