# AWS CLI 中級編 🚀

**実務での活用と自動化**

---

## 🎯 学習目標

この中級編を修了すると、以下ができるようになります：

- ✅ JMESPathを使った高度なフィルタリング
- ✅ シェルスクリプトでの自動化
- ✅ 複数アカウント・環境の管理
- ✅ CloudFormationのCLI操作
- ✅ 実務でのベストプラクティスの実践

**所要時間**: 約5時間

---

## 📋 学習コンテンツ

### 01. 高度なフィルタリング
**[01-advanced-filtering.md](01-advanced-filtering.md)**

- JMESPathの基礎
- 配列操作
- 条件フィルタ
- 複雑なクエリ
- 実用例

**所要時間**: 45分

---

### 02. スクリプト作成
**[02-scripting.md](02-scripting.md)**

- シェルスクリプトの基礎
- エラーハンドリング
- ループと条件分岐
- 実用的なスクリプト例
- デバッグ方法

**所要時間**: 60分

---

### 03. プロファイルと認証管理
**[03-profiles-credentials.md](03-profiles-credentials.md)**

- 複数プロファイルの管理
- IAM Roleの使用
- MFA対応
- クレデンシャルの安全な管理
- 環境変数の活用

**所要時間**: 45分

---

### 04. CloudFormation CLI
**[04-cloudformation-cli.md](04-cloudformation-cli.md)**

- スタックの作成・更新・削除
- Change Setの活用
- パラメータ・タグの管理
- デプロイ自動化
- トラブルシューティング

**所要時間**: 45分

---

### 05. VPC・ネットワーク操作
**[05-vpc-networking.md](05-vpc-networking.md)**

- VPC・Subnet操作
- セキュリティグループ管理
- ネットワークACL
- ルートテーブル
- VPCピアリング

**所要時間**: 60分

---

### 06. 実務自動化Tips
**[06-automation-tips.md](06-automation-tips.md)**

- バックアップ自動化
- リソース監視
- コスト最適化
- セキュリティチェック
- CI/CD統合

**所要時間**: 45分

---

### 99. 中級チートシート
**[99-intermediate-cheatsheet.md](99-intermediate-cheatsheet.md)**

- JMESPath早見表
- スクリプトテンプレート
- 実務パターン集
- トラブルシューティング

**用途**: クイックリファレンス

---

## 🗺️ 推奨学習フロー

### 標準コース（全て学習）

```
前提：初級編修了

01. 高度なフィルタリング
  ↓
02. スクリプト作成（重要！）
  ↓
03. プロファイルと認証管理
  ↓
04. CloudFormation CLI
  ↓
05. VPC・ネットワーク操作
  ↓
06. 実務自動化Tips
  ↓
99. チートシートで復習
```

---

### 実務重点コース

```
【必須】
02. スクリプト作成
06. 実務自動化Tips

【選択】自分の業務に合わせて
  - IaC担当 → 04. CloudFormation CLI
  - ネットワーク担当 → 05. VPC・ネットワーク操作
  - 複数アカウント管理 → 03. プロファイルと認証管理
```

---

## ✅ 学習前の準備

### 必須

- [ ] 初級編を修了している
- [ ] 基本的なシェルスクリプトが書ける
- [ ] JSONの構造を理解している

### 推奨

- [ ] Gitの基本操作ができる
- [ ] テキストエディタ（VSCode等）を使える
- [ ] 実験用のAWS環境がある

---

## 📖 前提知識

### 必須
- AWS CLI初級編の内容
- Linux/Macのターミナル操作
- 基本的なシェルスクリプト（変数、if、for）

### あると便利
- jqの使い方
- 正規表現の基礎
- CI/CDツールの経験

---

## 💡 学習のコツ

### 1. 実際のユースケースで学ぶ

```bash
# ❌ ただコマンドを実行
# ✅ 実際の業務で使うシナリオを想定

# 例：毎日のバックアップスクリプトを作る
# 例：複数環境へのデプロイを自動化する
```

---

### 2. エラーハンドリングを忘れずに

```bash
# ❌ エラーチェックなし
aws ec2 stop-instances --instance-ids $ID

# ✅ エラーチェックあり
if aws ec2 stop-instances --instance-ids $ID; then
    echo "Success"
else
    echo "Failed" >&2
    exit 1
fi
```

---

### 3. ログを残す

```bash
# コマンドの実行ログを残す
aws ec2 describe-instances | tee instances.log

# エラーログも残す
./deploy.sh 2>&1 | tee deploy.log
```

---

### 4. バージョン管理

```bash
# スクリプトはGitで管理
git init
git add scripts/
git commit -m "Add backup script"
```

---

## 🎓 修了条件

以下ができるようになったら、中級編修了です！

- [ ] JMESPathで複雑なフィルタリングができる
- [ ] エラーハンドリング付きのスクリプトが書ける
- [ ] 複数のAWSアカウント・環境を管理できる
- [ ] CloudFormationスタックをCLIで操作できる
- [ ] 実務で使える自動化スクリプトを作成できる
- [ ] トラブルシューティングができる

---

## 🚀 次のステップ

### 中級編修了後

1. ✅ **実践**: 実際の業務で自動化スクリプトを作成
2. ✅ **共有**: チームでスクリプトを共有
3. ✅ **改善**: フィードバックを受けて改善
4. ✅ **発展**: Terraform、CDK等の他のIaCツールも学ぶ

---

## 📚 関連リソース

### 公式ドキュメント
- [AWS CLI公式ドキュメント](https://docs.aws.amazon.com/cli/)
- [JMESPath Tutorial](https://jmespath.org/tutorial.html)

### 関連学習ガイド
- [CloudFormation学習ガイド](../../CFn/cfn-text/README.md)
- [Terraform学習ガイド](../../Terraform/)

---

## 🔧 実践プロジェクト例

### プロジェクト1: 毎日のバックアップ自動化
```bash
# AMI自動作成スクリプト
# S3へのログバックアップ
# スナップショット管理
```

### プロジェクト2: 複数環境デプロイ
```bash
# dev/stg/prod へのデプロイ
# 環境ごとの設定管理
# ロールバック機能
```

### プロジェクト3: リソース監視・通知
```bash
# 未使用リソースの検出
# コスト分析
# Slack/Email通知
```

---

**準備ができたら、[01-advanced-filtering.md](01-advanced-filtering.md) から始めましょう！🚀**
