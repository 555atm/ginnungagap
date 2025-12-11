# 05. Outputs と ImportValue

スタック間でリソースを連携させる

---

## 🎯 学習目標

- Outputs でスタックから値を出力する
- Export で値を他スタックで使えるようにする
- ImportValue で他スタックの値を参照する
- スタック間連携の設計ができる

**所要時間**: 45分

---

## 📤 Outputs（出力値）

### 概要

**スタックから値を外部に出力**

```yaml
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref MyVPC
    # コンソールやCLIで確認可能
```

**用途**:
- ✅ スタック作成後に値を確認
- ✅ 他スタックで参照（Export使用）
- ✅ 外部システムへの情報提供

---

### Outputs の基本構文

```yaml
Outputs:
  出力名:
    Description: 説明文
    Value: 出力する値
    Export:              # ← 他スタックで使う場合のみ
      Name: エクスポート名
```

---

### 例1: 基本的な Outputs

```yaml
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
  
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-xxxxx
      InstanceType: t3.small

Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref MyVPC
  
  InstanceId:
    Description: EC2 Instance ID
    Value: !Ref MyEC2
  
  PublicIp:
    Description: EC2 Public IP Address
    Value: !GetAtt MyEC2.PublicIp
  
  PrivateIp:
    Description: EC2 Private IP Address
    Value: !GetAtt MyEC2.PrivateIp
```

**確認方法**:
```bash
# CLIで確認
aws cloudformation describe-stacks \
  --stack-name my-stack \
  --query 'Stacks[0].Outputs'

# 出力例:
# [
#   {
#     "OutputKey": "VpcId",
#     "OutputValue": "vpc-0123456789abcdef",
#     "Description": "VPC ID"
#   },
#   {
#     "OutputKey": "PublicIp",
#     "OutputValue": "18.182.123.45",
#     "Description": "EC2 Public IP Address"
#   }
# ]
```

---

## 🔗 Export（エクスポート）

### 概要

**Outputsの値を他スタックで使えるようにする**

```yaml
Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref MyVPC
    Export:
      Name: MyVPC-VpcId    # ← エクスポート名（他スタックから参照）
```

---

### Export の命名ルール

**ユニークな名前**が必要（同一リージョン内で重複不可）

```yaml
# ❌ 悪い例（重複しやすい）
Export:
  Name: VpcId

# ✅ 良い例（スタック名を含める）
Export:
  Name: !Sub '${AWS::StackName}-VpcId'
```

---

### 例2: Export を使った Outputs

```yaml
# ネットワークスタック (network-stack)
AWSTemplateFormatVersion: '2010-09-09'
Description: Network Stack

Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
  
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
  
  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [1, !GetAZs '']

Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref MyVPC
    Export:
      Name: !Sub '${AWS::StackName}-VpcId'
  
  PublicSubnet1Id:
    Description: Public Subnet 1 ID
    Value: !Ref PublicSubnet1
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnet1Id'
  
  PublicSubnet2Id:
    Description: Public Subnet 2 ID
    Value: !Ref PublicSubnet2
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnet2Id'
  
  # リストとして出力
  PublicSubnetIds:
    Description: Public Subnet IDs
    Value: !Join [',', [!Ref PublicSubnet1, !Ref PublicSubnet2]]
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnetIds'
```

---

## 📥 ImportValue（インポート）

### 概要

**他スタックのExportされた値を参照**

```yaml
# アプリケーションスタック (app-stack)
Resources:
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-xxxxx
      InstanceType: t3.small
      SubnetId: !ImportValue network-stack-PublicSubnet1Id    # ← 他スタックから参照
```

---

### ImportValue の基本構文

```yaml
!ImportValue エクスポート名
```

---

### 例3: ImportValue を使ったスタック

```yaml
# アプリケーションスタック (app-stack)
AWSTemplateFormatVersion: '2010-09-09'
Description: Application Stack

Parameters:
  NetworkStackName:
    Type: String
    Default: network-stack
    Description: Network stack name

Resources:
  # Security Group
  WebSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Web Server SG
      VpcId: !ImportValue 
        Fn::Sub: '${NetworkStackName}-VpcId'    # ← VPC IDをインポート
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
  
  # EC2 Instance
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-xxxxx
      InstanceType: t3.small
      SubnetId: !ImportValue
        Fn::Sub: '${NetworkStackName}-PublicSubnet1Id'  # ← Subnet IDをインポート
      SecurityGroupIds:
        - !Ref WebSG

Outputs:
  InstanceId:
    Description: EC2 Instance ID
    Value: !Ref MyEC2
```

---

## 🔄 スタック間連携のフロー

### 1. ネットワークスタック作成

```bash
# network-stack を作成
aws cloudformation create-stack \
  --stack-name network-stack \
  --template-body file://network.yaml

# 完了待ち
aws cloudformation wait stack-create-complete \
  --stack-name network-stack
```

### 2. Export確認

```bash
# Exportされた値を確認
aws cloudformation list-exports

# 出力例:
# {
#   "Exports": [
#     {
#       "ExportingStackId": "arn:aws:cloudformation:...:stack/network-stack/...",
#       "Name": "network-stack-VpcId",
#       "Value": "vpc-0123456789abcdef"
#     },
#     {
#       "Name": "network-stack-PublicSubnet1Id",
#       "Value": "subnet-aaa"
#     }
#   ]
# }
```

### 3. アプリケーションスタック作成

```bash
# app-stack を作成（network-stackに依存）
aws cloudformation create-stack \
  --stack-name app-stack \
  --template-body file://app.yaml \
  --parameters ParameterKey=NetworkStackName,ParameterValue=network-stack
```

---

## 🎯 実践例: マイクロサービスアーキテクチャ

### スタック構成

```
1. network-stack         # VPC, Subnet, IGW
   ↓
2. database-stack        # RDS (network-stackに依存)
   ↓
3. app-stack             # EC2, ALB (network-stack, database-stackに依存)
```

### network-stack.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Network Infrastructure

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
  
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
  
  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.11.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
  
  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.12.0/24
      AvailabilityZone: !Select [1, !GetAZs '']

Outputs:
  VpcId:
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VpcId'
  
  PublicSubnet1Id:
    Value: !Ref PublicSubnet1
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnet1Id'
  
  PrivateSubnetIds:
    Value: !Join [',', [!Ref PrivateSubnet1, !Ref PrivateSubnet2]]
    Export:
      Name: !Sub '${AWS::StackName}-PrivateSubnetIds'
```

### database-stack.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Database Stack

Parameters:
  NetworkStackName:
    Type: String
    Default: network-stack

Resources:
  DBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: DB Subnet Group
      SubnetIds: !Split [',', !ImportValue 
        Fn::Sub: '${NetworkStackName}-PrivateSubnetIds']
  
  DBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: DB SG
      VpcId: !ImportValue
        Fn::Sub: '${NetworkStackName}-VpcId'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !ImportValue
            Fn::Sub: '${NetworkStackName}-AppSecurityGroupId'
  
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      Engine: mysql
      EngineVersion: '8.0.35'
      DBInstanceClass: db.t3.micro
      AllocatedStorage: 20
      DBSubnetGroupName: !Ref DBSubnetGroup
      VPCSecurityGroups:
        - !Ref DBSecurityGroup
      MasterUsername: admin
      MasterUserPassword: MyPassword123!

Outputs:
  DBEndpoint:
    Value: !GetAtt Database.Endpoint.Address
    Export:
      Name: !Sub '${AWS::StackName}-DBEndpoint'
```

### app-stack.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Application Stack

Parameters:
  NetworkStackName:
    Type: String
    Default: network-stack
  DatabaseStackName:
    Type: String
    Default: database-stack

Resources:
  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: App SG
      VpcId: !ImportValue
        Fn::Sub: '${NetworkStackName}-VpcId'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
  
  AppInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-xxxxx
      InstanceType: t3.small
      SubnetId: !ImportValue
        Fn::Sub: '${NetworkStackName}-PublicSubnet1Id'
      SecurityGroupIds:
        - !Ref AppSecurityGroup
      UserData: !Base64
        Fn::Sub:
          - |
            #!/bin/bash
            echo "DB_HOST=${DBHost}" > /etc/environment
          - DBHost: !ImportValue
              Fn::Sub: '${DatabaseStackName}-DBEndpoint'

Outputs:
  InstanceId:
    Value: !Ref AppInstance
```

---

## ⚠️ Import/Export の制約

### 1. Export削除の制約

**Exportを参照しているスタックがあると削除できない**

```bash
# エラー例
aws cloudformation delete-stack --stack-name network-stack

# Export network-stack-VpcId is still imported by app-stack
```

**対処**: 参照元を先に削除

```bash
# 正しい順序
aws cloudformation delete-stack --stack-name app-stack
aws cloudformation wait stack-delete-complete --stack-name app-stack

aws cloudformation delete-stack --stack-name network-stack
```

### 2. Export名の変更不可

**Exportを参照している間は名前変更できない**

---

## 💡 ベストプラクティス

### 1. Export名にスタック名を含める

```yaml
# ✅ 良い例
Export:
  Name: !Sub '${AWS::StackName}-VpcId'

# ❌ 悪い例（重複しやすい）
Export:
  Name: VpcId
```

### 2. スタックの依存関係を明確にする

```
ネットワーク → データベース → アプリケーション
                ↓
              監視
```

### 3. 削除順序を逆にする

```bash
# 作成順序
network-stack → database-stack → app-stack

# 削除順序（逆）
app-stack → database-stack → network-stack
```

### 4. 参照を確認してから削除

```bash
# Exportの参照状況を確認
aws cloudformation list-imports --export-name network-stack-VpcId
```

---

## ✅ このレッスンのチェックリスト

- [ ] Outputs の役割を理解した
- [ ] Export で値をエクスポートできる
- [ ] ImportValue で他スタックの値を参照できる
- [ ] スタック間の依存関係を設計できる
- [ ] Export削除の制約を理解した
- [ ] 正しい削除順序を理解した

---

## 📚 次のステップ

次は **[06. 基礎サンプルテンプレート](06-sample-templates-basic.md)** で、実践的なテンプレートを作成します！

---

**スタック間連携をマスターして、モジュール化されたインフラを作りましょう！🚀**
