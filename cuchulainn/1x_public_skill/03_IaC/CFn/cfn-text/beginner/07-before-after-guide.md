# 07. Before/After実践ガイド

初級編の総まとめ：ベタ書きコードから洗練されたコードへ

---

## 🎯 学習目標

- ベタ書きコードの問題点を理解する
- 組み込み関数の実践的な使い方を習得する
- パラメータ化・スタック間連携をマスターする
- **初級編で学んだすべての知識を統合**

**所要時間**: 120分

---

## 📚 この教材について

### Before版
**すべてハードコードされた保守性の低いコード**
- ❌ 環境変更が困難
- ❌ 重複コード多数
- ❌ 再利用性ゼロ

### After版
**組み込み関数とパラメータを活用した洗練されたコード**
- ✅ 環境変更が容易
- ✅ 重複削減
- ✅ 再利用性が高い

---

## 🏗️ 構成

```
VPC (10.0.0.0/16)
├── Public Subnet × 2 (AZ-a, AZ-c)
├── Private Subnet × 2 (AZ-a, AZ-c)
├── EC2 × 2台 (Web Server)
└── RDS × 2台 (Primary + Read Replica)
```

---

## 🔴 Before: ベタ書き版

### 詳細なファイル

**[before-basic.yaml](../before-basic.yaml)** を参照

### 主な問題点

#### 1. すべてハードコード

```yaml
# ❌ 環境名がハードコード
Tags:
  - Key: Name
    Value: myapp-dev-vpc

# ❌ インスタンスタイプがハードコード
Properties:
  InstanceType: t3.small
```

#### 2. 重複コード多数

```yaml
# ❌ EC2 x2 のコードが重複
resource "aws_instance" "web_1" {
  ami = "ami-xxxxx"
  instance_type = "t3.small"
  # ...
}

resource "aws_instance" "web_2" {
  ami = "ami-xxxxx"    # ← 同じAMI IDを重複記述
  instance_type = "t3.small"
  # ...
}
```

#### 3. リージョン依存

```yaml
# ❌ リージョンを変更すると動かない
AvailabilityZone: ap-northeast-1a
ImageId: ami-0c3fd0f5d33134a76    # ← リージョン固有のAMI
```

#### 4. パスワード平文

```yaml
# ❌ 超危険！
MasterUserPassword: MyPassword123!
```

---

## 🟢 After: 洗練版

### 詳細なファイル

**[after-advanced.yaml](../after-advanced.yaml)** を参照

### 主な改善点

#### 1. Parameters による柔軟性

```yaml
Parameters:
  ProjectName:
    Type: String
    Default: myapp
  
  Environment:
    Type: String
    AllowedValues: [dev, stg, prod]

# 使用
Tags:
  - Key: Name
    Value: !Sub '${ProjectName}-${Environment}-vpc'
    # 結果: myapp-dev-vpc
```

#### 2. Mappings による環境別設定

```yaml
Mappings:
  EnvironmentMap:
    dev:
      InstanceType: t3.small
      RdsClass: db.t3.micro
    prod:
      InstanceType: m5.large
      RdsClass: db.r6i.large

# 使用
Properties:
  InstanceType: !FindInMap [EnvironmentMap, !Ref Environment, InstanceType]
```

#### 3. Conditions による条件付きリソース

```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

Resources:
  # 本番のみ作成
  ReadReplica:
    Type: AWS::RDS::DBInstance
    Condition: IsProduction
```

#### 4. 組み込み関数の活用

```yaml
# !GetAZs で動的にAZ取得
AvailabilityZone: !Select [0, !GetAZs '']

# !Sub で文字列展開
BucketName: !Sub '${ProjectName}-${Environment}-bucket'

# !Join でリスト結合
SubnetIds: !Join [',', [!Ref Subnet1, !Ref Subnet2]]
```

#### 5. Outputs + Export によるスタック間連携

```yaml
Outputs:
  VpcId:
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VpcId'
```

---

## 📊 Before/After 比較表

| 項目 | Before | After | 改善効果 |
|------|--------|-------|---------|
| **環境変更** | 全箇所手動修正 | Parameters変更のみ | **10倍以上効率化** |
| **リージョン変更** | AMI等を全修正 | !GetAZs で自動対応 | **完全自動化** |
| **インスタンス追加** | コピペ | Count変更 | **自動化** |
| **保守性** | ★☆☆☆☆ | ★★★★★ | **5倍向上** |
| **再利用性** | ゼロ | スタック間連携可能 | **完全再利用** |
| **セキュリティ** | 平文パスワード | NoEcho, Secrets Manager | **セキュア** |

---

## 🎯 習得できる中級テクニック

### 1. Parameters（パラメータ化）

```yaml
Parameters:
  ProjectName:
    Type: String
  Environment:
    Type: String
    AllowedValues: [dev, stg, prod]
  DBPassword:
    Type: String
    NoEcho: true
    MinLength: 8
```

### 2. Mappings（環境別設定）

```yaml
Mappings:
  EnvironmentMap:
    dev:
      InstanceType: t3.small
    prod:
      InstanceType: m5.large
```

### 3. Conditions（条件分岐）

```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

Resources:
  ReadReplica:
    Condition: IsProduction
```

### 4. 組み込み関数（10種類以上）

- `!Ref` - リソース参照
- `!GetAtt` - 属性取得
- `!Sub` - 文字列展開
- `!Join` - リスト結合
- `!Select` - 要素取得
- `!GetAZs` - AZ一覧取得
- `!FindInMap` - Mappings参照
- `!If` - 条件分岐
- `!Split` - 文字列分割
- `!Base64` - エンコード

### 5. Outputs + Export（スタック間連携）

```yaml
Outputs:
  VpcId:
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VpcId'
```

### 6. ImportValue（他スタック参照）

```yaml
# 別のテンプレートで
Resources:
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      SubnetId: !ImportValue network-stack-PublicSubnetId
```

---

## 🚀 実践フロー

### Step 1: Before版を読む（15分）

```bash
# before-basic.yaml を開いて確認
# - ハードコードされた箇所を探す
# - 重複しているコードを見つける
# - 問題点をメモする
```

### Step 2: After版を読む（30分）

```bash
# after-advanced.yaml を開いて確認
# - Parameters の使い方
# - Mappings の設計
# - 組み込み関数の使い方
# - Outputs の設計
```

### Step 3: 比較する（10分）

**同じ機能を実現するのに、どう変わったか確認**

### Step 4: デプロイする（30分）

```bash
# After版をデプロイ
aws cloudformation create-stack \
  --stack-name after-stack \
  --template-body file://after-advanced.yaml \
  --parameters \
      ParameterKey=ProjectName,ParameterValue=myapp \
      ParameterKey=Environment,ParameterValue=dev \
      ParameterKey=DBPassword,ParameterValue=SecurePassword123!
```

### Step 5: ImportValue を試す（20分）

```bash
# import-example.yaml をデプロイ
aws cloudformation create-stack \
  --stack-name import-example \
  --template-body file://import-example.yaml \
  --parameters \
      ParameterKey=NetworkStackName,ParameterValue=after-stack
```

### Step 6: 削除する（10分）

```bash
# 逆順で削除
aws cloudformation delete-stack --stack-name import-example
aws cloudformation wait stack-delete-complete --stack-name import-example

aws cloudformation delete-stack --stack-name after-stack
```

---

## 💡 実務での使い方

### シナリオ1: 開発環境を本番環境に昇格

```bash
# 開発環境
aws cloudformation create-stack \
  --stack-name myapp-dev \
  --template-body file://after-advanced.yaml \
  --parameters \
      ParameterKey=Environment,ParameterValue=dev

# 本番環境（同じテンプレート！）
aws cloudformation create-stack \
  --stack-name myapp-prod \
  --template-body file://after-advanced.yaml \
  --parameters \
      ParameterKey=Environment,ParameterValue=prod
```

**自動で切り替わるもの**:
- インスタンスタイプ: t3.small → m5.large
- RDS: Single-AZ → Multi-AZ
- Read Replica: 作成しない → 作成する
- Backup: 7日 → 30日

---

### シナリオ2: 別リージョンへの展開

```bash
# 東京リージョン
aws cloudformation create-stack \
  --stack-name myapp-tokyo \
  --template-body file://after-advanced.yaml \
  --region ap-northeast-1

# バージニアリージョン（同じテンプレート！）
aws cloudformation create-stack \
  --stack-name myapp-virginia \
  --template-body file://after-advanced.yaml \
  --region us-east-1
```

**自動で対応するもの**:
- AZ: !GetAZs で自動取得
- AMI: SSM Parameter Store で最新取得

---

## 📝 詳細ファイル

### 関連ファイル

| ファイル | 内容 |
|---------|------|
| **[before-basic.yaml](../before-basic.yaml)** | Before版（ベタ書き） |
| **[after-advanced.yaml](../after-advanced.yaml)** | After版（洗練） |
| **[import-example.yaml](../import-example.yaml)** | ImportValue の実例 |
| **[deployment-guide.md](../deployment-guide.md)** | 詳細なデプロイ手順 |
| **[README-before-after.md](../README-before-after.md)** | Before/After教材ガイド |

---

## ✅ このレッスンのチェックリスト

### 理解度チェック
- [ ] Before版の5つの問題点を説明できる
- [ ] After版の6つの改善点を理解した
- [ ] Parameters, Mappings, Conditions を使い分けられる
- [ ] 組み込み関数を10種類以上使える
- [ ] Outputs + Export + ImportValue でスタック間連携できる

### 実践チェック
- [ ] Before版を読んで問題点を理解した
- [ ] After版を読んで改善点を理解した
- [ ] After版をデプロイして動作確認した
- [ ] ImportValue でスタック間連携を体験した
- [ ] 環境切り替え（dev→prod）を試した

---

## 🎓 初級編完了おめでとうございます！

### 習得したスキル

1. ✅ CloudFormationの基本概念
2. ✅ YAML構文
3. ✅ Parameters, Mappings, Conditions
4. ✅ 組み込み関数（10種類以上）
5. ✅ Outputs, Export, ImportValue
6. ✅ スタック間連携
7. ✅ 基本的なAWSリソース作成

### 到達レベル

**VPC + EC2 + RDS の基本構成を自力で作れる** ✨

---

## 📚 次のステップ

初級編を完了したら、**[中級編](../../intermediate/)** に進みましょう！

### 中級編で学ぶこと
- ネストスタックによるモジュール化
- 変更セットで安全な更新
- カスタムリソース（Lambda連携）
- StackSets（マルチアカウント）
- CI/CD統合

---

**初級編完了おめでとうございます！中級編でさらにスキルアップしましょう！🚀**
