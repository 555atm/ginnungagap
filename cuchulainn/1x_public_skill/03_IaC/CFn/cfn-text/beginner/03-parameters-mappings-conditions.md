# 03. Parameters, Mappings, Conditions

テンプレートを柔軟に、環境ごとに対応させる

---

## 🎯 学習目標

- Parameters で実行時にパラメータを指定できるようにする
- Mappings で環境別設定を管理する
- Conditions で条件分岐を実装する

**所要時間**: 60分

---

## 📥 Parameters（パラメータ）

### 概要

**実行時に値を指定**できるようにする機能

```yaml
Parameters:
  Environment:
    Type: String
    Default: dev

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub 'mybucket-${Environment}'
      # dev → mybucket-dev
      # prod → mybucket-prod
```

**メリット**:
- ✅ 同じテンプレートで複数環境に対応
- ✅ 実行時に値を変更可能
- ✅ 再利用性が高い

---

### Parameters の基本構文

```yaml
Parameters:
  パラメータ名:
    Type: データ型
    Default: デフォルト値
    Description: 説明
    AllowedValues: [許可する値のリスト]
    AllowedPattern: 正規表現
    ConstraintDescription: 制約違反時のメッセージ
```

---

### Type（データ型）

#### 基本型

```yaml
Parameters:
  # 文字列
  ProjectName:
    Type: String
    Default: myapp
  
  # 数値
  InstanceCount:
    Type: Number
    Default: 2
  
  # カンマ区切りリスト
  SubnetIds:
    Type: CommaDelimitedList
    Default: "subnet-aaa,subnet-bbb"
  
  # 数値リスト
  AllowedPorts:
    Type: List<Number>
    Default: [80, 443]
```

#### AWS固有型

```yaml
Parameters:
  # VPC ID
  VpcId:
    Type: AWS::EC2::VPC::Id
    Description: Select existing VPC
  
  # サブネットID（複数）
  SubnetIds:
    Type: List<AWS::EC2::Subnet::Id>
    Description: Select 2 or more subnets
  
  # キーペア名
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: EC2 key pair
  
  # AMI ID
  ImageId:
    Type: AWS::EC2::Image::Id
    Description: EC2 AMI ID
```

#### SSM Parameter Store型

```yaml
Parameters:
  # SSMパラメータストアから最新AMI取得
  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64
```

---

### 制約（Constraints）

#### AllowedValues - 許可する値を列挙

```yaml
Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - stg
      - prod
    Description: Environment name
```

#### AllowedPattern - 正規表現で制約

```yaml
Parameters:
  ProjectName:
    Type: String
    AllowedPattern: ^[a-z0-9-]+$
    ConstraintDescription: Only lowercase letters, numbers, and hyphens
    Description: Project name (lowercase only)
```

#### MinValue / MaxValue - 数値の範囲

```yaml
Parameters:
  InstanceCount:
    Type: Number
    Default: 2
    MinValue: 1
    MaxValue: 10
    Description: Number of instances (1-10)
```

#### MinLength / MaxLength - 文字列の長さ

```yaml
Parameters:
  ProjectName:
    Type: String
    MinLength: 3
    MaxLength: 20
    Description: Project name (3-20 characters)
```

---

### 実践例: Parameters

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Template with Parameters

Parameters:
  # プロジェクト名
  ProjectName:
    Type: String
    Default: myapp
    Description: Project name
  
  # 環境
  Environment:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - stg
      - prod
    Description: Environment name
  
  # インスタンスタイプ
  InstanceType:
    Type: String
    Default: t3.small
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
      - m5.large
    Description: EC2 instance type
  
  # キーペア
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: EC2 key pair name

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType    # ← パラメータ参照
      KeyName: !Ref KeyName
      ImageId: ami-0c3fd0f5d33134a76
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-web'
          # myapp-dev-web
```

**デプロイ**:
```bash
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters \
      ParameterKey=ProjectName,ParameterValue=webapp \
      ParameterKey=Environment,ParameterValue=prod \
      ParameterKey=InstanceType,ParameterValue=m5.large \
      ParameterKey=KeyName,ParameterValue=my-key
```

---

## 🗺️ Mappings（マッピング）

### 概要

**キー・バリューのマップ**を定義し、環境別設定を管理

```yaml
Mappings:
  EnvironmentMap:
    dev:
      InstanceType: t3.small
      DbClass: db.t3.micro
    prod:
      InstanceType: m5.large
      DbClass: db.r6i.large

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !FindInMap [EnvironmentMap, !Ref Environment, InstanceType]
      # dev → t3.small
      # prod → m5.large
```

---

### Mappings の基本構文

```yaml
Mappings:
  マップ名:
    キー1:
      サブキー1: 値
      サブキー2: 値
    キー2:
      サブキー1: 値
      サブキー2: 値
```

---

### 実践例1: 環境別設定

```yaml
Mappings:
  EnvironmentMap:
    dev:
      InstanceType: t3.small
      DbClass: db.t3.micro
      MultiAZ: false
      BackupRetention: 1
    stg:
      InstanceType: t3.medium
      DbClass: db.t3.small
      MultiAZ: false
      BackupRetention: 7
    prod:
      InstanceType: m5.large
      DbClass: db.r6i.large
      MultiAZ: true
      BackupRetention: 30

Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, stg, prod]

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !FindInMap [EnvironmentMap, !Ref Environment, InstanceType]
  
  MyDB:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: !FindInMap [EnvironmentMap, !Ref Environment, DbClass]
      MultiAZ: !FindInMap [EnvironmentMap, !Ref Environment, MultiAZ]
      BackupRetentionPeriod: !FindInMap [EnvironmentMap, !Ref Environment, BackupRetention]
```

---

### 実践例2: リージョン別AMI

```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0
    ap-northeast-1:
      AMI: ami-0c3fd0f5d33134a76
    eu-west-1:
      AMI: ami-0f29c8402f8cce65c

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref 'AWS::Region', AMI]
      # 現在のリージョンに応じてAMIを自動選択
      InstanceType: t3.small
```

---

### !FindInMap の使い方

```yaml
!FindInMap [マップ名, トップレベルキー, セカンドレベルキー]
```

**例**:
```yaml
Mappings:
  MyMap:
    Key1:
      Name: Value1
    Key2:
      Name: Value2

Resources:
  MyResource:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !FindInMap [MyMap, Key1, Name]
      # 結果: Value1
```

---

## 🔀 Conditions（条件）

### 概要

**条件によってリソースの作成を制御**

```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

Resources:
  # 本番環境のみ作成
  ReadReplica:
    Type: AWS::RDS::DBInstance
    Condition: IsProduction
    Properties:
      SourceDBInstanceIdentifier: !Ref PrimaryDB
```

---

### Condition の基本構文

```yaml
Conditions:
  条件名: 条件式
```

---

### 条件関数

#### !Equals - 等しいか判定

```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]
  IsDevelopment: !Equals [!Ref Environment, dev]
```

#### !Not - 否定

```yaml
Conditions:
  IsNotProduction: !Not [!Equals [!Ref Environment, prod]]
```

#### !And - AND条件

```yaml
Conditions:
  CreateResources: !And
    - !Equals [!Ref Environment, prod]
    - !Equals [!Ref CreateResources, 'true']
```

#### !Or - OR条件

```yaml
Conditions:
  IsProdOrStg: !Or
    - !Equals [!Ref Environment, prod]
    - !Equals [!Ref Environment, stg]
```

---

### Condition の使用方法

#### 1. リソース全体を条件付きで作成

```yaml
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, stg, prod]

Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

Resources:
  # 本番のみRDS Read Replicaを作成
  ReadReplica:
    Type: AWS::RDS::DBInstance
    Condition: IsProduction    # ← ここに指定
    Properties:
      SourceDBInstanceIdentifier: !Ref PrimaryDB
```

#### 2. プロパティ値を条件で切り替え

```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

Resources:
  MyDB:
    Type: AWS::RDS::DBInstance
    Properties:
      # 本番のみMulti-AZ
      MultiAZ: !If [IsProduction, true, false]
      
      # 本番は30日、それ以外は7日
      BackupRetentionPeriod: !If [IsProduction, 30, 7]
```

---

### 実践例: 環境別リソース作成

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Multi-environment template

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - stg
      - prod

Mappings:
  EnvironmentMap:
    dev:
      InstanceType: t3.small
      DbClass: db.t3.micro
    stg:
      InstanceType: t3.medium
      DbClass: db.t3.small
    prod:
      InstanceType: m5.large
      DbClass: db.r6i.large

Conditions:
  IsProduction: !Equals [!Ref Environment, prod]
  IsStaging: !Equals [!Ref Environment, stg]
  IsProdOrStg: !Or
    - !Condition IsProduction
    - !Condition IsStaging
  IsDevelopment: !Equals [!Ref Environment, dev]

Resources:
  # すべての環境で作成
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
  
  # すべての環境で作成（ただし設定が異なる）
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !FindInMap [EnvironmentMap, !Ref Environment, InstanceType]
      ImageId: ami-0c3fd0f5d33134a76
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-web-server'
  
  # 本番・ステージングのみ作成
  NATGateway:
    Type: AWS::EC2::NatGateway
    Condition: IsProdOrStg
    Properties:
      SubnetId: !Ref PublicSubnet
      AllocationId: !GetAtt EIP.AllocationId
  
  # 本番のみ作成
  ReadReplica:
    Type: AWS::RDS::DBInstance
    Condition: IsProduction
    Properties:
      SourceDBInstanceIdentifier: !Ref PrimaryDB
      DBInstanceClass: !FindInMap [EnvironmentMap, !Ref Environment, DbClass]
  
  # すべての環境で作成（MultiAZは本番のみ）
  PrimaryDB:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: !FindInMap [EnvironmentMap, !Ref Environment, DbClass]
      Engine: mysql
      MultiAZ: !If [IsProduction, true, false]
      BackupRetentionPeriod: !If [IsProduction, 30, 7]
      StorageEncrypted: !If [IsProduction, true, false]

Outputs:
  VpcId:
    Value: !Ref MyVPC
  
  EC2InstanceId:
    Value: !Ref MyEC2
  
  # 本番のみ出力
  ReadReplicaEndpoint:
    Condition: IsProduction
    Value: !GetAtt ReadReplica.Endpoint.Address
```

---

## 🎯 Parameters vs Mappings vs Conditions

| 項目 | Parameters | Mappings | Conditions |
|------|-----------|----------|-----------|
| **用途** | 実行時の入力 | 静的な設定マップ | 条件分岐 |
| **変更** | 実行時に指定 | テンプレート内に記述 | テンプレート内に記述 |
| **例** | 環境名、インスタンス数 | 環境別設定、リージョン別AMI | リソースの条件付き作成 |

**使い分け**:
- **Parameters**: ユーザーが変更する値
- **Mappings**: テンプレート作成者が定義する静的マップ
- **Conditions**: リソースの作成可否を制御

---

## 💡 実践パターン

### パターン1: 開発環境はコスト削減

```yaml
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, prod]

Mappings:
  EnvironmentMap:
    dev:
      InstanceType: t3.micro
      RdsMultiAZ: false
    prod:
      InstanceType: m5.large
      RdsMultiAZ: true

Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

Resources:
  # 本番のみNAT Gateway（開発はコスト削減）
  NATGateway:
    Type: AWS::EC2::NatGateway
    Condition: IsProduction
    Properties:
      SubnetId: !Ref PublicSubnet
      AllocationId: !GetAtt EIP.AllocationId
  
  # 本番はMulti-AZ、開発はSingle-AZ
  MyRDS:
    Type: AWS::RDS::DBInstance
    Properties:
      MultiAZ: !FindInMap [EnvironmentMap, !Ref Environment, RdsMultiAZ]
      DBInstanceClass: db.t3.small
```

---

### パターン2: リージョン対応

```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-xxxxx
      AZs: [us-east-1a, us-east-1b]
    ap-northeast-1:
      AMI: ami-yyyyy
      AZs: [ap-northeast-1a, ap-northeast-1c]

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref 'AWS::Region', AMI]
      AvailabilityZone: !Select [0, !FindInMap [RegionMap, !Ref 'AWS::Region', AZs]]
```

---

## ✅ このレッスンのチェックリスト

- [ ] Parameters の役割を理解した
- [ ] Parameters の Type を使い分けられる
- [ ] AllowedValues, AllowedPattern で制約を設定できる
- [ ] Mappings で環境別設定を管理できる
- [ ] !FindInMap で Mappings から値を取得できる
- [ ] Conditions で条件分岐ができる
- [ ] !Equals, !And, !Or, !Not を使える
- [ ] !If でプロパティ値を条件で切り替えられる

---

## 📚 次のステップ

次は **[04. 組み込み関数（基礎）](04-intrinsic-functions-basic.md)** で、より高度な値の操作を学びます！

---

**Parameters, Mappings, Conditions をマスターして、柔軟なテンプレートを作りましょう！🚀**
