# AWS CLI 学習ガイド 🚀

**AWS初学者からCLI中級者への最短ルート**

---

## 🎯 このガイドについて

### 対象者
- AWS CLI を初めて使う方
- コマンドラインでAWSを操作したい方
- インフラ自動化の基礎を学びたい方

### 目標
- **初級**: AWS CLIの基本操作ができる
- **中級**: 実務で使える自動化スクリプトが書ける

---

## 📚 学習コンテンツ

### 初級編（AWS CLI基礎）

**[beginner/](beginner/)** - AWS CLIの基本から実用まで

| # | ファイル | 内容 | 所要時間 |
|---|---------|------|----------|
| 01 | [AWS CLIの基礎](beginner/01-aws-cli-basics.md) | インストール、設定、基本概念 | 30分 |
| 02 | [基本コマンド構造](beginner/02-basic-commands.md) | コマンド構造、help、基本操作 | 30分 |
| 03 | [EC2操作の基礎](beginner/03-ec2-basics.md) | インスタンス操作、確認 | 45分 |
| 04 | [S3操作の基礎](beginner/04-s3-basics.md) | バケット操作、オブジェクト操作 | 45分 |
| 05 | [IAM操作の基礎](beginner/05-iam-basics.md) | ユーザー、ロール、ポリシー | 45分 |
| 06 | [出力とフィルタリング](beginner/06-output-filtering.md) | JSON、テーブル、query | 45分 |
| 99 | [初級チートシート](beginner/99-beginner-cheatsheet.md) | クイックリファレンス | - |

**合計学習時間**: 約4時間

---

### 中級編（実務での活用）

**[intermediate/](intermediate/)** - 自動化と実務ノウハウ

| # | ファイル | 内容 | 所要時間 |
|---|---------|------|----------|
| 01 | [高度なフィルタリング](intermediate/01-advanced-filtering.md) | JMESPath、複雑なクエリ | 45分 |
| 02 | [スクリプト作成](intermediate/02-scripting.md) | シェルスクリプト、自動化 | 60分 |
| 03 | [プロファイルと認証](intermediate/03-profiles-credentials.md) | 複数アカウント、MFA | 45分 |
| 04 | [CloudFormation CLI](intermediate/04-cloudformation-cli.md) | スタック操作、デプロイ | 45分 |
| 05 | [VPC・ネットワーク](intermediate/05-vpc-networking.md) | VPC、Subnet、SG操作 | 60分 |
| 06 | [実務自動化Tips](intermediate/06-automation-tips.md) | ベストプラクティス | 45分 |
| 99 | [中級チートシート](intermediate/99-intermediate-cheatsheet.md) | クイックリファレンス | - |

**合計学習時間**: 約5時間

---

## 🗺️ 推奨学習パス

### パターン1: ゼロから学ぶ（初心者向け）

```
Step 1: 初級編を順番に学習（01→02→03→04→05→06）
  ↓
Step 2: 初級チートシートで復習
  ↓
Step 3: 実際のAWSアカウントで練習
  ↓
Step 4: 中級編へ進む
```

---

### パターン2: 特定サービスを重点的に学ぶ

```
Step 1: 01-aws-cli-basics.md（必須）
Step 2: 02-basic-commands.md（必須）
  ↓
Step 3: 必要なサービスを選択
  - EC2メイン → 03-ec2-basics.md
  - S3メイン → 04-s3-basics.md
  - 権限管理 → 05-iam-basics.md
  ↓
Step 4: 06-output-filtering.md（推奨）
  ↓
Step 5: 中級編の該当分野へ
```

---

### パターン3: 実務で使いながら学ぶ

```
Step 1: 初級チートシート（99-beginner-cheatsheet.md）で概要把握
  ↓
Step 2: 必要な操作を該当ファイルで確認
  ↓
Step 3: 自動化が必要になったら中級編へ
  ↓
Step 4: 02-scripting.md、06-automation-tips.md を重点的に
```

---

## 🎓 学習目標

### 初級編修了時

- ✅ AWS CLIのインストールと設定ができる
- ✅ 基本的なEC2、S3、IAM操作ができる
- ✅ 出力結果を読み取り、必要な情報を抽出できる
- ✅ ドキュメントを見ながら新しいコマンドを実行できる

### 中級編修了時

- ✅ 複雑なフィルタリング（JMESPath）ができる
- ✅ シェルスクリプトで自動化できる
- ✅ 複数アカウント・環境を管理できる
- ✅ CloudFormationをCLIで操作できる
- ✅ 実務でのベストプラクティスを理解している

---

## 📖 前提知識

### 必須
- ✅ Linux/Macのターミナル基本操作（cd、ls、cat等）
- ✅ AWSの基本概念（EC2、S3、IAMとは何か）

### 推奨
- ✅ JSON形式の基本的な読み方
- ✅ シェルスクリプトの基礎知識（中級編で必要）

---

## 🛠️ 事前準備

### 1. AWSアカウント
- [ ] AWSアカウントを作成済み
- [ ] IAMユーザーまたはAdministratorAccess権限がある

### 2. 環境
- [ ] ターミナルが使える環境（Linux、Mac、WSL2等）
- [ ] Python 3.7以降がインストール済み

### 3. 学習用リソース
- [ ] テスト用のAWSリソース作成権限
- [ ] 実験環境（本番環境とは別）

---

## 🔗 関連リソース

### 公式ドキュメント
- [AWS CLI公式ドキュメント](https://docs.aws.amazon.com/cli/)
- [AWS CLI Command Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)

### 関連学習ガイド
- [CloudFormation学習ガイド](../CFn/cfn-text/README.md)
- [Terraform学習ガイド](../Terraform/)

---

## 💡 学習のコツ

### 1. 実際に手を動かす
- ❌ 読むだけ
- ✅ 実際にコマンドを実行する
- ✅ エラーを経験する

### 2. ドキュメントを活用
```bash
# コマンドのヘルプを常に確認
aws ec2 help
aws ec2 describe-instances help
```

### 3. チートシートを手元に
- 初級チートシートを印刷またはサブディスプレイに表示
- よく使うコマンドをすぐ参照できるようにする

### 4. エラーを恐れない
- AWS CLIのエラーメッセージは親切
- エラーメッセージをよく読む習慣をつける

---

## 📝 学習記録テンプレート

```markdown
## AWS CLI 学習記録

### 日付: YYYY-MM-DD

#### 学習内容
- [ ] beginner/01-aws-cli-basics.md
- [ ] beginner/02-basic-commands.md
- [ ] ...

#### 実践したコマンド
- aws configure
- aws ec2 describe-instances
- ...

#### 躓いたポイント
- 問題: ...
- 解決策: ...

#### 次回やること
- [ ] ...
```

---

## 🎯 次のステップ

### 初心者の方
👉 **[beginner/01-aws-cli-basics.md](beginner/01-aws-cli-basics.md)** から始めましょう！

### AWS CLI経験者の方
👉 **[beginner/99-beginner-cheatsheet.md](beginner/99-beginner-cheatsheet.md)** で復習してから中級編へ

### 特定の操作を知りたい方
👉 **チートシート**から該当コマンドを探す

---

**AWS CLIをマスターして、効率的なクラウド運用を実現しましょう！🚀**
