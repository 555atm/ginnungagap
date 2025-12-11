# 04. 組み込み関数（基礎）

CloudFormationの組み込み関数をマスター

---

## 🎯 学習目標

- 基本的な組み込み関数10種類を使いこなす
- リソース参照、文字列操作、リスト操作ができる
- 実践的な使い方を理解する

**所要時間**: 60分

---

## 📚 組み込み関数とは？

CloudFormationテンプレート内で使える**特殊な関数**

```yaml
# 普通の値
BucketName: mybucket

# 組み込み関数で動的に値を生成
BucketName: !Sub '${AWS::StackName}-bucket'
# 結果: my-stack-bucket
```

**メリット**:
- ✅ 動的に値を生成
- ✅ リソース間の参照
- ✅ 文字列・リスト操作
- ✅ 条件分岐

---

## 📖 組み込み関数一覧（初級編）

| 関数 | 用途 | 重要度 |
|------|------|--------|
| `!Ref` | リソース・パラメータ参照 | ★★★★★ |
| `!GetAtt` | リソース属性取得 | ★★★★★ |
| `!Sub` | 文字列展開 | ★★★★★ |
| `!Join` | 文字列結合 | ★★★☆☆ |
| `!Select` | リスト要素取得 | ★★★☆☆ |
| `!GetAZs` | AZ一覧取得 | ★★★☆☆ |
| `!FindInMap` | Mappings参照 | ★★★★☆ |
| `!If` | 条件分岐 | ★★★★☆ |
| `!Equals` | 等値判定 | ★★★☆☆ |
| `!Base64` | Base64エンコード | ★★☆☆☆ |

---

## 1️⃣ !Ref - リソース・パラメータ参照

### 概要

**リソースIDやパラメータ値を取得**

```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t3.small

Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
  
  MySubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC              # ← VPC IDを取得
      CidrBlock: 10.0.1.0/24
  
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType  # ← パラメータ値を取得
      SubnetId: !Ref MySubnet          # ← Subnet IDを取得
```

### 返り値

| リソースタイプ | !Ref の返り値 |
|--------------|-------------|
| `AWS::EC2::VPC` | VPC ID (vpc-xxxxx) |
| `AWS::EC2::Subnet` | Subnet ID (subnet-xxxxx) |
| `AWS::EC2::Instance` | Instance ID (i-xxxxx) |
| `AWS::EC2::SecurityGroup` | Security Group ID (sg-xxxxx) |
| `AWS::S3::Bucket` | Bucket Name |
| `AWS::RDS::DBInstance` | DB Instance Identifier |
| Parameters | パラメータの値 |

---

## 2️⃣ !GetAtt - リソース属性取得

### 概要

**リソースの詳細な属性を取得**

```yaml
Resources:
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-xxxxx
      InstanceType: t3.small

Outputs:
  PrivateIp:
    Value: !GetAtt MyEC2.PrivateIp    # Private IP取得
  
  PublicIp:
    Value: !GetAtt MyEC2.PublicIp     # Public IP取得
  
  AZ:
    Value: !GetAtt MyEC2.AvailabilityZone  # AZ取得
```

### よく使う属性

#### EC2

```yaml
!GetAtt MyEC2.PrivateIp
!GetAtt MyEC2.PublicIp
!GetAtt MyEC2.AvailabilityZone
!GetAtt MyEC2.PrivateDnsName
```

#### RDS

```yaml
!GetAtt MyRDS.Endpoint.Address     # ホスト名
!GetAtt MyRDS.Endpoint.Port        # ポート番号
```

#### S3

```yaml
!GetAtt MyBucket.Arn               # ARN
!GetAtt MyBucket.DomainName        # ドメイン名
!GetAtt MyBucket.WebsiteURL        # Website URL
```

#### ALB

```yaml
!GetAtt MyALB.DNSName              # DNS名
!GetAtt MyALB.CanonicalHostedZoneID
```

---

## 3️⃣ !Sub - 文字列展開

### 概要

**変数を文字列に埋め込む**

```yaml
Parameters:
  ProjectName:
    Type: String
    Default: myapp
  Environment:
    Type: String
    Default: dev

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${ProjectName}-${Environment}-bucket'
      # 結果: myapp-dev-bucket
```

### 使用できる変数

#### 1. Parameters

```yaml
Parameters:
  ProjectName:
    Type: String

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${ProjectName}-bucket'
```

#### 2. Resources（リソースID）

```yaml
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-vpc'
```

#### 3. 疑似パラメータ

```yaml
Description: !Sub 'Stack in ${AWS::Region} account ${AWS::AccountId}'
# 結果: Stack in ap-northeast-1 account 123456789012

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${AWS::StackName}-${AWS::Region}-bucket'
      # 結果: my-stack-ap-northeast-1-bucket
```

**疑似パラメータ一覧**:
- `${AWS::AccountId}` - アカウントID
- `${AWS::Region}` - リージョン
- `${AWS::StackName}` - スタック名
- `${AWS::StackId}` - スタックID
- `${AWS::Partition}` - パーティション（aws）

#### 4. !GetAtt と組み合わせ

```yaml
Resources:
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-xxxxx
      InstanceType: t3.small

Outputs:
  InstanceInfo:
    Value: !Sub 'Instance ${MyEC2} has IP ${MyEC2.PrivateIp}'
    # 結果: Instance i-xxxxx has IP 10.0.1.100
```

---

## 4️⃣ !Join - 文字列結合

### 概要

**リストを区切り文字で結合**

```yaml
!Join [区切り文字, [要素1, 要素2, 要素3]]
```

### 例1: カンマ区切り

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      UserData: !Base64
        !Join
          - ','
          - - 'value1'
            - 'value2'
            - 'value3'
      # 結果: value1,value2,value3
```

### 例2: パス結合

```yaml
Outputs:
  BucketUrl:
    Value: !Join
      - ''
      - - 'https://s3.amazonaws.com/'
        - !Ref MyBucket
        - '/index.html'
    # 結果: https://s3.amazonaws.com/mybucket/index.html
```

---

## 5️⃣ !Select - リスト要素取得

### 概要

**リストの特定インデックスを取得**（0始まり）

```yaml
!Select [インデックス, リスト]
```

### 例1: 最初の要素

```yaml
Parameters:
  SubnetIds:
    Type: CommaDelimitedList
    Default: "subnet-aaa,subnet-bbb,subnet-ccc"

Resources:
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      SubnetId: !Select [0, !Ref SubnetIds]
      # 結果: subnet-aaa
```

### 例2: AZの選択

```yaml
Resources:
  Subnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      AvailabilityZone: !Select [0, !GetAZs '']  # 最初のAZ
      CidrBlock: 10.0.1.0/24
  
  Subnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      AvailabilityZone: !Select [1, !GetAZs '']  # 2番目のAZ
      CidrBlock: 10.0.2.0/24
```

---

## 6️⃣ !GetAZs - AZ一覧取得

### 概要

**リージョンのAZ一覧を取得**

```yaml
!GetAZs リージョン名
!GetAZs ''     # 現在のリージョン
```

### 例: 複数AZにサブネット作成

```yaml
Resources:
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: Public-1a
  
  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [1, !GetAZs '']
      Tags:
        - Key: Name
          Value: Public-1c
```

---

## 7️⃣ !FindInMap - Mappings参照

### 概要

**Mappingsから値を取得**

```yaml
!FindInMap [マップ名, トップキー, セカンドキー]
```

### 例: 環境別設定

```yaml
Mappings:
  EnvironmentMap:
    dev:
      InstanceType: t3.small
      DbClass: db.t3.micro
    prod:
      InstanceType: m5.large
      DbClass: db.r6i.large

Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, prod]

Resources:
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !FindInMap [EnvironmentMap, !Ref Environment, InstanceType]
      # dev → t3.small
      # prod → m5.large
```

---

## 8️⃣ !If - 条件分岐

### 概要

**条件によって値を切り替え**

```yaml
!If [条件名, 真の場合の値, 偽の場合の値]
```

### 例: 環境別設定

```yaml
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, prod]

Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

Resources:
  MyRDS:
    Type: AWS::RDS::DBInstance
    Properties:
      MultiAZ: !If [IsProduction, true, false]
      # prod → true
      # dev → false
      
      BackupRetentionPeriod: !If [IsProduction, 30, 7]
      # prod → 30日
      # dev → 7日
```

---

## 9️⃣ !Equals - 等値判定

### 概要

**2つの値が等しいか判定**（Conditionsで使用）

```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, prod]
```

---

## 🔟 !Base64 - Base64エンコード

### 概要

**文字列をBase64エンコード**（UserDataで使用）

```yaml
Resources:
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-xxxxx
      InstanceType: t3.small
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          echo "<h1>Hello from ${AWS::StackName}</h1>" > /var/www/html/index.html
```

**省略形**:
```yaml
UserData: !Base64
  !Sub |
    #!/bin/bash
    echo "Hello World"
```

---

## 🎯 実践例: 組み込み関数の組み合わせ

### 例1: VPC + サブネット + EC2

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: VPC with EC2 using intrinsic functions

Parameters:
  ProjectName:
    Type: String
    Default: myapp
  
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, stg, prod]

Mappings:
  EnvironmentMap:
    dev:
      InstanceType: t3.small
    stg:
      InstanceType: t3.medium
    prod:
      InstanceType: m5.large

Conditions:
  IsProduction: !Equals [!Ref Environment, prod]

Resources:
  # VPC
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-vpc'
  
  # Public Subnet
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC                              # !Ref
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']      # !Select + !GetAZs
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-public'  # !Sub
  
  # Internet Gateway
  IGW:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-igw'
  
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref MyVPC
      InternetGatewayId: !Ref IGW
  
  # Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref MyVPC
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-public-rt'
  
  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref IGW
  
  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable
  
  # Security Group
  WebSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Web Server SG
      VpcId: !Ref MyVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-web-sg'
  
  # EC2
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c3fd0f5d33134a76
      InstanceType: !FindInMap [EnvironmentMap, !Ref Environment, InstanceType]  # !FindInMap
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds:
        - !Ref WebSG
      UserData: !Base64                                # !Base64
        !Sub |                                         # !Sub
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
          echo "<h1>Hello from ${ProjectName}-${Environment}</h1>" > /var/www/html/index.html
          echo "<p>Instance ID: $(ec2-metadata --instance-id | cut -d ' ' -f 2)</p>" >> /var/www/html/index.html
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-web'

Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref MyVPC
  
  InstanceId:
    Description: EC2 Instance ID
    Value: !Ref MyEC2
  
  PublicIp:
    Description: EC2 Public IP
    Value: !GetAtt MyEC2.PublicIp          # !GetAtt
  
  WebUrl:
    Description: Web URL
    Value: !Sub 'http://${MyEC2.PublicIp}'  # !Sub + !GetAtt
```

---

## 💡 ベストプラクティス

### 1. !Sub を積極的に使う

```yaml
# ❌ 悪い例（ハードコード）
BucketName: myapp-dev-bucket

# ✅ 良い例（動的生成）
BucketName: !Sub '${ProjectName}-${Environment}-bucket'
```

### 2. !Ref と !GetAtt を使い分ける

```yaml
# !Ref: リソースID取得
SubnetId: !Ref MySubnet

# !GetAtt: 詳細な属性取得
PrivateIp: !GetAtt MyEC2.PrivateIp
```

### 3. 疑似パラメータを活用

```yaml
# ユニークな名前を生成
BucketName: !Sub '${AWS::StackName}-${AWS::AccountId}-bucket'
```

---

## ✅ このレッスンのチェックリスト

- [ ] !Ref でリソース・パラメータを参照できる
- [ ] !GetAtt でリソース属性を取得できる
- [ ] !Sub で文字列展開ができる
- [ ] !Join でリスト結合ができる
- [ ] !Select でリスト要素を取得できる
- [ ] !GetAZs でAZ一覧を取得できる
- [ ] !FindInMap で Mappings から値を取得できる
- [ ] !If で条件分岐ができる
- [ ] 複数の組み込み関数を組み合わせて使える

---

## 📚 次のステップ

次は **[05. Outputs と ImportValue](05-outputs-imports.md)** で、スタック間連携を学びます！

---

**組み込み関数をマスターして、動的なテンプレートを作りましょう！🚀**
