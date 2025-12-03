

# 未分類

・variable / output だけdescriptionに意味を書く文化
variable は  実務では必須
動作には影響しない
resource は description 書かない（Terraform にはその機能がない）


- tfvarsの正しい名前
Terraform は：
something.auto.tfvars → 自動読み込み
terraform.tfvars → 自動読み込み
それ以外 → 読み込みなし


- root module の意味
    - terraformコマンド打つ場所
    - ＝「Terraform プロジェクトのエントリーポイント」
すべての Terraform プロジェクトは少なくとも 1 つのモジュールを持ち、その起点になるモジュールが root module と呼ばれます。​

実際には「terraform init / plan / apply を打つカレントディレクトリの .tf ファイル一式」が root module で、その中から他の child module（modules/ 配下など）を呼び出します。