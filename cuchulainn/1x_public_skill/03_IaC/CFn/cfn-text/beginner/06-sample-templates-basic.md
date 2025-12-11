# 06. 基礎サンプルテンプレート集

実際に動くテンプレートで学ぶ

---

## 🎯 学習目標

- 実際に動作するテンプレートを理解する
- 基本的なAWS構成パターンを習得する
- デプロイ・動作確認・削除の流れを体験する

**所要時間**: 90分

---

## 📚 サンプルテンプレート一覧

| # | 構成 | 学習内容 | 難易度 |
|---|------|---------|--------|
| 1 | S3バケット | 基本構文 | ★☆☆ |
| 2 | VPC + Subnet | ネットワーク基礎 | ★☆☆ |
| 3 | VPC + EC2 | Web サーバー | ★★☆ |
| 4 | VPC + EC2 + RDS | 3層アーキテクチャ | ★★★ |

---

## 📦 Sample 1: S3バケット

### 構成

```
S3 Bucket
├── Versioning有効
├── 暗号化有効
└── タグ設定
```

### テンプレート

```yaml
# sample-01-s3.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Simple S3 Bucket

Parameters:
  ProjectName:
    Type: String
    Default: myapp
    Description: Project name
  
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, stg, prod]
    Description: Environment name

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${ProjectName}-${Environment}-${AWS::AccountId}-bucket'
      VersioningConfiguration:
        Status: Enabled
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-bucket'
        - Key: Environment
          Value: !Ref Environment
        - Key: ManagedBy
          Value: CloudFormation

Outputs:
  BucketName:
    Description: S3 Bucket Name
    Value: !Ref MyBucket
  
  BucketArn:
    Description: S3 Bucket ARN
    Value: !GetAtt MyBucket.Arn
```

### デプロイ

```bash
aws cloudformation create-stack \
  --stack-name sample-01-s3 \
  --template-body file://sample-01-s3.yaml \
  --parameters \
      ParameterKey=ProjectName,ParameterValue=myapp \
      ParameterKey=Environment,ParameterValue=dev
```

### 確認

```bash
# スタック状態確認
aws cloudformation describe-stacks --stack-name sample-01-s3

# バケット確認
aws s3 ls | grep myapp

# 出力値確認
aws cloudformation describe-stacks \
  --stack-name sample-01-s3 \
  --query 'Stacks[0].Outputs'
```

### 削除

```bash
# バケットが空であることを確認
aws s3 rm s3://$(aws cloudformation describe-stacks \
  --stack-name sample-01-s3 \
  --query 'Stacks[0].Outputs[?OutputKey==`BucketName`].OutputValue' \
  --output text) --recursive

# スタック削除
aws cloudformation delete-stack --stack-name sample-01-s3
```

---

## 🌐 Sample 2: VPC + Subnet

### 構成

```
VPC (10.0.0.0/16)
├── Public Subnet 1 (10.0.1.0/24) - AZ-a
├── Public Subnet 2 (10.0.2.0/24) - AZ-c
├── Internet Gateway
└── Route Table
```

### テンプレート

```yaml
# sample-02-vpc.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: VPC with 2 Public Subnets

Parameters:
  ProjectName:
    Type: String
    Default: myapp
  
  Environment:
    Type: String
    Default: dev

Resources:
  # VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-vpc'
  
  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-igw'
  
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  
  # Public Subnet 1
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-public-1a'
  
  # Public Subnet 2
  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [1, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-public-1c'
  
  # Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-public-rt'
  
  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
  
  SubnetRouteTableAssociation1:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet1
      RouteTableId: !Ref PublicRouteTable
  
  SubnetRouteTableAssociation2:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet2
      RouteTableId: !Ref PublicRouteTable

Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref VPC
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
```

### デプロイ

```bash
aws cloudformation create-stack \
  --stack-name sample-02-vpc \
  --template-body file://sample-02-vpc.yaml
```

### 確認

```bash
# VPC確認
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=myapp-dev-vpc"

# サブネット確認
aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$(aws cloudformation describe-stacks \
    --stack-name sample-02-vpc \
    --query 'Stacks[0].Outputs[?OutputKey==`VpcId`].OutputValue' \
    --output text)"
```

---

## 💻 Sample 3: VPC + EC2

### 構成

```
VPC (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24)
├── Internet Gateway
├── Security Group (Port 80, 443)
└── EC2 (Apache Web Server)
```

### テンプレート

```yaml
# sample-03-vpc-ec2.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: VPC with EC2 Web Server

Parameters:
  ProjectName:
    Type: String
    Default: myapp
  
  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64

Resources:
  # VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-vpc'
  
  # Internet Gateway
  IGW:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-igw'
  
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref IGW
  
  # Subnet
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-public-subnet'
  
  # Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-public-rt'
  
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
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Web Server Security Group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
      SecurityGroupEgress:
        - IpProtocol: -1
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-web-sg'
  
  # EC2 Instance
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: t3.micro
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      UserData: !Base64
        Fn::Sub: |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
          
          cat > /var/www/html/index.html << 'EOF'
          <!DOCTYPE html>
          <html>
          <head>
              <title>${ProjectName} Web Server</title>
              <style>
                  body {
                      font-family: Arial, sans-serif;
                      margin: 0;
                      padding: 0;
                      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                      display: flex;
                      justify-content: center;
                      align-items: center;
                      min-height: 100vh;
                  }
                  .container {
                      background: white;
                      border-radius: 15px;
                      padding: 40px;
                      box-shadow: 0 10px 40px rgba(0,0,0,0.3);
                      text-align: center;
                  }
                  h1 { color: #333; }
                  .info { margin: 20px 0; color: #666; }
              </style>
          </head>
          <body>
              <div class="container">
                  <h1>🚀 ${ProjectName} Web Server</h1>
                  <div class="info">
                      <p>Stack: ${AWS::StackName}</p>
                      <p>Region: ${AWS::Region}</p>
                      <p>Managed by CloudFormation</p>
                  </div>
              </div>
          </body>
          </html>
          EOF
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-web-server'

Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref VPC
  
  InstanceId:
    Description: EC2 Instance ID
    Value: !Ref WebServer
  
  PublicIp:
    Description: EC2 Public IP
    Value: !GetAtt WebServer.PublicIp
  
  WebUrl:
    Description: Web URL
    Value: !Sub 'http://${WebServer.PublicIp}'
```

### デプロイ

```bash
aws cloudformation create-stack \
  --stack-name sample-03-vpc-ec2 \
  --template-body file://sample-03-vpc-ec2.yaml
```

### 確認

```bash
# Web URLを取得してアクセス
WEB_URL=$(aws cloudformation describe-stacks \
  --stack-name sample-03-vpc-ec2 \
  --query 'Stacks[0].Outputs[?OutputKey==`WebUrl`].OutputValue' \
  --output text)

echo "Web URL: $WEB_URL"
curl $WEB_URL

# またはブラウザでアクセス
```

---

## 🏗️ Sample 4: VPC + EC2 + RDS（3層アーキテクチャ）

### 構成

```
VPC (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24)
│   └── EC2 (Web Server)
├── Private Subnet 1 (10.0.11.0/24)
│   └── RDS Primary
└── Private Subnet 2 (10.0.12.0/24)
    └── RDS (Multi-AZ)
```

### テンプレート

```yaml
# sample-04-3tier.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 3-Tier Architecture (VPC + EC2 + RDS)

Parameters:
  ProjectName:
    Type: String
    Default: myapp
  
  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64
  
  DBPassword:
    Type: String
    NoEcho: true
    Description: Database password (min 8 characters)
    MinLength: 8

Resources:
  # VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-vpc'
  
  # Internet Gateway
  IGW:
    Type: AWS::EC2::InternetGateway
  
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref IGW
  
  # Public Subnet
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-public'
  
  # Private Subnets (for RDS)
  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.11.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-private-1a'
  
  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.12.0/24
      AvailabilityZone: !Select [1, !GetAZs '']
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-private-1c'
  
  # Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
  
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
  
  # Security Groups
  WebServerSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Web Server SG
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-web-sg'
  
  DatabaseSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Database SG
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref WebServerSG
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-db-sg'
  
  # EC2 Instance
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: t3.micro
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds:
        - !Ref WebServerSG
      UserData: !Base64
        Fn::Sub:
          - |
            #!/bin/bash
            yum update -y
            yum install -y httpd mysql
            systemctl start httpd
            systemctl enable httpd
            
            echo "<h1>3-Tier Architecture</h1>" > /var/www/html/index.html
            echo "<p>DB Host: ${DBHost}</p>" >> /var/www/html/index.html
          - DBHost: !GetAtt Database.Endpoint.Address
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-web'
  
  # RDS DB Subnet Group
  DBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: DB Subnet Group
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
  
  # RDS Database
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: !Sub '${ProjectName}-db'
      Engine: mysql
      EngineVersion: '8.0.35'
      DBInstanceClass: db.t3.micro
      AllocatedStorage: 20
      StorageType: gp3
      StorageEncrypted: true
      DBName: appdb
      MasterUsername: admin
      MasterUserPassword: !Ref DBPassword
      VPCSecurityGroups:
        - !Ref DatabaseSG
      DBSubnetGroupName: !Ref DBSubnetGroup
      BackupRetentionPeriod: 7
      PreferredBackupWindow: '03:00-04:00'
      PreferredMaintenanceWindow: 'mon:04:00-mon:05:00'
      PubliclyAccessible: false
      MultiAZ: false
      DeletionProtection: false

Outputs:
  VpcId:
    Value: !Ref VPC
  
  WebServerPublicIp:
    Description: Web Server Public IP
    Value: !GetAtt WebServer.PublicIp
  
  WebUrl:
    Description: Web URL
    Value: !Sub 'http://${WebServer.PublicIp}'
  
  DatabaseEndpoint:
    Description: Database Endpoint
    Value: !GetAtt Database.Endpoint.Address
```

### デプロイ

```bash
aws cloudformation create-stack \
  --stack-name sample-04-3tier \
  --template-body file://sample-04-3tier.yaml \
  --parameters ParameterKey=DBPassword,ParameterValue=MySecurePassword123!
```

### 確認

```bash
# Web Server確認
WEB_URL=$(aws cloudformation describe-stacks \
  --stack-name sample-04-3tier \
  --query 'Stacks[0].Outputs[?OutputKey==`WebUrl`].OutputValue' \
  --output text)

curl $WEB_URL

# RDSエンドポイント確認
aws cloudformation describe-stacks \
  --stack-name sample-04-3tier \
  --query 'Stacks[0].Outputs[?OutputKey==`DatabaseEndpoint`].OutputValue' \
  --output text
```

---

## 🎓 学習ポイント

### Sample 1から学ぶこと
- 基本的なリソース作成
- Parameters の使い方
- !Sub による動的な名前生成

### Sample 2から学ぶこと
- VPCネットワーク構成
- Internet Gateway の設定
- Route Table の関連付け
- Export による値の公開

### Sample 3から学ぶこと
- EC2インスタンスの作成
- Security Group の設定
- UserData によるソフトウェアインストール
- !GetAtt による属性取得

### Sample 4から学ぶこと
- 3層アーキテクチャの設計
- Public/Private Subnet の分離
- RDS の作成と設定
- Security Group による通信制御

---

## ✅ このレッスンのチェックリスト

- [ ] Sample 1 をデプロイ・確認・削除できた
- [ ] Sample 2 をデプロイ・確認できた
- [ ] Sample 3 をデプロイしてWebページにアクセスできた
- [ ] Sample 4 をデプロイして3層構成を理解した
- [ ] スタックの削除順序を理解した

---

## 📚 次のステップ

次は **[07. Before/After実践ガイド](07-before-after-guide.md)** で、初級編の総まとめです！

---

**サンプルテンプレートで実践経験を積みましょう！🚀**
