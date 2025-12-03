13. IaC（Terraform / CFN）
13-1. Terraform構成
/modules
/envs/dev
/envs/stg
/envs/prod

13-2. module設計

variables / outputs を明確に

変数で環境差分を吸収

13-3. remote backend

S3 + DynamoDB

チーム運用では必須

13-4. ECS/ALB/RDSコード例

（※必要なら別途作成）

13-5. CloudFormation 親子構成

親：全体オーケストレーション

子：個別コンポーネント

13-6. 組み込み関数

Ref / Sub / GetAtt は必須

13-7. CDK

TypeScriptでIaC

保守性が高い
