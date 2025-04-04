# キャッシュフロー表生成ワークフロー定義 (action_level.yaml)

このリポジトリには、指定されたパラメータに基づいて将来のキャッシュフロー表をHTML形式で生成するための抽象化されたワークフロー定義 (`action_level.yaml`) が含まれています。

## 目的

このワークフロー定義は、様々な家族構成、収入、支出、ライフイベントのシナリオに基づいて、キャッシュフローのシミュレーションを実行し、その結果を視覚化するための汎用的な手順を提供します。具体的な計算ロジックやHTML生成の実装は別途必要ですが、このYAMLファイルはその骨組みとなります。

## ワークフロー概要 (`action_level.yaml`)

このYAMLファイルは、ワークフローの入力パラメータと実行されるアクションのシーケンスを定義します。

### 入力 (Inputs)

ワークフローを実行するために必要なパラメータです。

- `template_file`: (必須) キャッシュフロー表の元となるHTMLテンプレートファイル。デフォルト: `index.html`
- `output_file`: (必須) 生成されるHTMLファイルの出力先。デフォルト: `cashflow_output.html`
- `years_to_simulate`: (必須) シミュレーションを行う年数。デフォルト: `20`
- `initial_savings`: (必須) シミュレーション開始時の貯蓄残高。
- `family_members`: (必須) 家族構成員のリスト (名前と初期年齢)。
    ```yaml
    # 例
    - name: '夫'
      initial_age: 35
    - name: '妻'
      initial_age: 33
    ```
- `income_sources`: (必須) 収入源のリスト (名前、初期年収、年次変動情報)。
    ```yaml
    # 例
    - name: '夫の収入'
      initial_amount: 400 # 単位: 万円など、実装に依存
      annual_change_rate: 0.02 # 2%昇給
    - name: '妻の収入'
      initial_amount: 200
      annual_change: 0 # 固定
    ```
- `recurring_expenses`: (必須) 定期的な支出項目のリスト (名前、初期年額、年次変動情報)。
    ```yaml
    # 例
    - name: '基本生活費'
      initial_amount: 250
      annual_change: 5 # 毎年5万円増加
    - name: '住居関連費'
      initial_amount: 100
      annual_change_rate: 0 # 固定
    ```
- `life_events`: (任意) ライフイベントのリスト (名前、発生年、一時的な支出/収入、対象者など)。
    ```yaml
    # 例
    - name: '長男大学入学'
      year: 13
      temporary_expense: 200
      target_member: '長男'
    - name: '住宅購入'
      year: 5
      temporary_expense: 3000
    ```
- `education_expenses_model`: (任意) 年齢や進学段階に応じた教育費の計算モデル。
    ```yaml
    # 例
    '未就学': 10
    '小学生': 15
    # ...
    ```

### アクション (Actions)

ワークフローが実行する手順です。

1.  **Initialize Workspace**: テンプレートファイルを出力ファイル名でコピーします。
2.  **Populate Initial Data**: 0年目の初期データをHTMLに設定します。
3.  **Simulate Annual Changes**: 指定年数分ループし、各年の年齢、収入、支出、収支、貯蓄を計算し、HTMLに追記します。ライフイベントも反映します。
4.  **Finalize Output**: 最終的な調整を行い、HTMLファイルを保存します。

## 使い方

1.  この `action_level.yaml` ファイルをワークフローエンジン（例: GitHub Actions, Argo Workflowsなど、YAMLを解釈・実行できるツール）で読み込みます。
2.  ワークフローを実行する際に、`Inputs` セクションで定義されたパラメータを指定します。
3.  各 `Actions` で定義されたステップ（`script` 部分など）に対応する具体的な処理（HTML操作、計算ロジック）を実装したスクリプトやコンテナイメージを用意します。
4.  ワークフローを実行すると、指定された `output_file` にキャッシュフロー表が生成されます。

## 注意点

- このYAMLファイルはワークフローの**定義**であり、単体で実行可能なプログラムではありません。
- 各ステップ内の `script:` 部分はプレースホルダーであり、実際の処理ロジックを記述する必要があります。
- 金額の単位（万円、円など）は、実装するスクリプト側で統一する必要があります。
- 年次変動の指定方法（変動率 `annual_change_rate` または固定額 `annual_change`）は、実装に合わせて選択・調整してください。 