# YAML補足資料 - CloudFormation版

CloudFormationで必要なYAML知識

---

## 🎯 この資料について

### 対象
- CloudFormation初心者
- YAMLを初めて使う方

### 範囲
- **CloudFormationで必要な範囲**: 必須⭐
- **一般的なYAML知識**: 参考（プログラミング用途等）

---

## ⭐ CloudFormationで必須のYAML知識

### 1. インデント（超重要！）

**ルール**: **スペース2つ**（タブNG）

```yaml
# ✅ 正しい（スペース2つ）
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

# ❌ 間違い（タブ使用）
Resources:
	MyVPC:    # ← タブ（エラーになる）

# ❌ 間違い（スペース数がバラバラ）
Resources:
MyVPC:
   Type: AWS::EC2::VPC
```

**VSCode設定**（推奨）:
```json
{
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "editor.detectIndentation": false
}
```

---

### 2. キー・バリュー（基本）

```yaml
# 基本形
Key: Value

# CloudFormation例
AWSTemplateFormatVersion: '2010-09-09'
Description: My Template

# 階層構造
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: mybucket
```

**ポイント**:
- `:` の後にスペース1つ
- インデントでネスト構造を表現

---

### 3. 文字列

```yaml
# 通常（クォート不要）
BucketName: mybucket
Description: This is my bucket

# 数値を文字列として扱う場合（クォート必要）
Version: "2.0"
Port: "80"

# 特殊文字がある場合（クォート推奨）
Description: "Bucket for dev/test environment"

# 複数行（| = 改行保持）⭐ CloudFormationで頻出
UserData: |
  #!/bin/bash
  yum update -y
  echo "Hello World"

# 複数行（> = 改行を空白に）
Description: >
  This is a long description
  that spans multiple lines.
```

**CloudFormationでの使い分け**:
- 通常: クォート不要
- UserData: `|` を使う⭐
- 長い説明: `>` を使う

---

### 4. リスト

```yaml
# 方法1: ハイフン（CloudFormationで推奨）
SecurityGroupIds:
  - sg-12345
  - sg-67890

Tags:
  - Key: Name
    Value: MyInstance
  - Key: Environment
    Value: dev

# 方法2: JSON形式（短い場合のみ）
AllowedValues: [dev, stg, prod]
Ports: [80, 443]
```

**CloudFormationでの使用例**:
```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      SecurityGroupIds:
        - !Ref WebSG
        - !Ref AppSG
      Tags:
        - Key: Name
          Value: WebServer
```

---

### 5. コメント

```yaml
# これはコメント
Resources:
  MyBucket:    # 行末コメント
    Type: AWS::S3::Bucket
    # Properties:    # コメントアウト
    #   BucketName: mybucket
```

**ポイント**:
- `#` 以降がコメント
- 複数行コメントは各行に `#` が必要

---

### 6. 真偽値

```yaml
# true/false
Enabled: true
MultiAZ: false

# yes/no も使える（同じ意味）
Enabled: yes
MultiAZ: no
```

**CloudFormation例**:
```yaml
Resources:
  MyRDS:
    Type: AWS::RDS::DBInstance
    Properties:
      MultiAZ: true          # Multi-AZ有効
      PubliclyAccessible: false  # Public アクセス無効
```

---

### 7. 数値

```yaml
# 整数
Port: 80
Count: 3

# 小数
Threshold: 0.5

# 文字列にしたい場合はクォート
Version: "1.0"
```

---

### 8. null（空値）

```yaml
# null（値なし）
OptionalValue: null
OptionalValue: ~        # null の別表記

# 値がない場合は省略可能
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    # Properties は省略可能（リソースによる）
```

---

## 🔧 CloudFormation固有の記法

### 1. 組み込み関数（短縮形）

```yaml
# CloudFormation専用の記法
Resources:
  MySubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC              # 短縮形⭐
      # VpcId: Fn::Ref: MyVPC       # 完全形（使わない）
      
      CidrBlock: !Sub '10.0.${SubnetNumber}.0/24'  # 短縮形⭐
      # CidrBlock:                   # 完全形（使わない）
      #   Fn::Sub: '10.0.${SubnetNumber}.0/24'
```

**ポイント**:
- `!` から始まるのがCloudFormation固有
- 短縮形を使う⭐

---

### 2. クォートのルール

```yaml
# ✅ 推奨（シングルクォート）
Description: 'My CloudFormation Template'
BucketName: !Sub '${ProjectName}-bucket'

# ✅ OK（ダブルクォート）
Description: "My CloudFormation Template"

# ✅ OK（クォートなし・シンプルな場合）
Description: My CloudFormation Template
BucketName: mybucket

# ❌ 避ける（!Sub では必ずクォート）
BucketName: !Sub ${ProjectName}-bucket    # エラーになる可能性
```

**CloudFormationでの推奨**:
- 通常: クォートなし
- `!Sub`: シングルクォート⭐
- 特殊文字: シングルクォート

---

## 📚 参考：一般的なYAML知識

### アンカー・エイリアス（CloudFormationでは非対応）

```yaml
# 一般的なYAMLでは使えるが、CloudFormationでは使えない
defaults: &defaults
  InstanceType: t3.small
  
dev:
  <<: *defaults    # CloudFormationでは使えない！
```

**CloudFormationの代替**:
- Mappings を使う
- Parameters を使う

---

### 複雑なデータ型（CloudFormationでは限定的）

```yaml
# 一般的なYAML
date: 2025-12-11
binary: !!binary base64data

# CloudFormationでは
# - 文字列、数値、真偽値、リスト、マップのみ
# - 日付型、バイナリ型は使わない
```

---

## ⚠️ よくあるエラーと対処法

### エラー1: インデントミス

```yaml
# ❌ エラー
Resources:
MyBucket:
  Type: AWS::S3::Bucket

# エラーメッセージ:
# Template format error: YAML not well-formed
```

**対処**: インデントをスペース2つに統一

```yaml
# ✅ 修正後
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
```

---

### エラー2: タブ使用

```yaml
# ❌ エラー（見た目では分からない）
Resources:
	MyBucket:    # ← タブ

# エラーメッセージ:
# mapping values are not allowed here
```

**対処**: タブをスペースに変換（VSCodeの設定）

---

### エラー3: クォートミス

```yaml
# ❌ エラー
BucketName: !Sub ${ProjectName}-bucket

# エラーメッセージ:
# Template error: variable names in Fn::Sub syntax must be unique
```

**対処**: !Sub は必ずクォート

```yaml
# ✅ 修正後
BucketName: !Sub '${ProjectName}-bucket'
```

---

### エラー4: リストのインデント

```yaml
# ❌ エラー
Tags:
- Key: Name
  Value: MyBucket

# エラーメッセージ:
# Template format error: YAML not well-formed
```

**対処**: ハイフンのインデントを揃える

```yaml
# ✅ 修正後
Tags:
  - Key: Name
    Value: MyBucket
```

---

## ✅ CloudFormation用YAML チェックリスト

### 基本
- [ ] インデントはスペース2つ
- [ ] タブは使っていない
- [ ] `:` の後にスペースがある
- [ ] コメントは `#` で始まる

### CloudFormation固有
- [ ] `!Ref`, `!Sub` 等の短縮形を使っている
- [ ] `!Sub` はシングルクォートで囲んでいる
- [ ] UserData は `|` を使っている

### 構造
- [ ] `Resources:` セクションが必須
- [ ] `Type:` が各リソースに必須
- [ ] インデントが正しい

---

## 🔧 便利なツール

### VSCode拡張機能

```bash
# YAML拡張
code --install-extension redhat.vscode-yaml

# CloudFormation拡張
code --install-extension aws-cloudformation.yaml-schema
```

### CLIツール

```bash
# cfn-lint（テンプレート検証）
pip install cfn-lint
cfn-lint template.yaml

# YAML構文チェック
python -c "import yaml; yaml.safe_load(open('template.yaml'))"
```

---

## 📖 まとめ

### CloudFormationで必要なYAML知識

| 項目 | 重要度 | ポイント |
|------|--------|---------|
| インデント | ★★★★★ | スペース2つ、タブNG |
| キー・バリュー | ★★★★★ | `: ` 後にスペース |
| 文字列 | ★★★★☆ | 通常クォート不要 |
| リスト | ★★★★★ | ハイフン形式 |
| コメント | ★★★☆☆ | `#` |
| 複数行 | ★★★★☆ | UserDataで `|` |
| 組み込み関数 | ★★★★★ | `!Ref`, `!Sub` 等 |

### 学習の流れ

1. ✅ このYAMLガイドを読む
2. ✅ [01. CFn基礎](01-cfn-basics.md) に戻る
3. ✅ [02. 基本構文](02-basic-syntax.md) で実践
4. ✅ サンプルテンプレートで練習

---

**CloudFormationに必要なYAML知識をマスターしましょう！📚**
