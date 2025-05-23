# LLM ベースのエージェント（ReAct）

## エージェント（ReAct）とは

エージェントは、大規模言語モデル（LLM）をその中心的な計算エンジンとして使用する高度なAIシステムです。LLMの推論能力を、計画やツール使用などの追加機能と組み合わせて、複雑なタスクを自律的に実行します。エージェントは、複雑な問い合わせを分解し、ステップバイステップのソリューションを生成し、外部ツールやAPIと対話して情報を収集したり、サブタスクを実行したりできます。

このサンプルは、[ReAct（推論 + アクション）](https://www.promptingguide.ai/techniques/react)アプローチを使用してエージェントを実装しています。ReActにより、エージェントは推論とアクションを反復的なフィードバックループで組み合わせて、複雑なタスクを解決できます。エージェントは、思考、アクション、観察の3つの重要なステップを繰り返し実行します。LLMを使用して現在の状況を分析し、次に取るべきアクションを決定し、利用可能なツールやAPIを使用してアクションを実行し、観察された結果から学習します。この継続的なプロセスにより、エージェントは動的な環境に適応し、タスク解決の精度を向上させ、コンテキストを意識したソリューションを提供できます。

## 使用例

ReActを使用するエージェントは、さまざまなシナリオで正確で効率的なソリューションを提供できます。

### テキストからSQL

ユーザーが「最後の四半期の売上総額」を要求した場合、エージェントはこの要求を解釈し、SQL照会に変換し、データベースに対して実行して結果を提示します。

### 財務予測

財務アナリストが次の四半期の収益予測を行う必要がある場合、エージェントは関連するデータを収集し、財務モデルを使用して必要な計算を実行し、予測の正確性を保証する詳細な予測レポートを生成します。

## エージェント機能の使用方法

カスタムチャットボットでエージェント機能を有効にするには、次の手順に従ってください：

1. カスタムボット画面のエージェントセクションに移動します。

2. エージェントセクションで、エージェントが使用できる利用可能なツールのリストが表示されます。デフォルトでは、すべてのツールは無効になっています。

3. ツールを有効にするには、目的のツールの横にあるスイッチを切り替えるだけです。ツールが有効になると、エージェントはそのツールにアクセスでき、ユーザーの問い合わせを処理する際に使用できます。

![](./imgs/agent_tools.png)

> [!重要]
> エージェントセクションでツールを有効にすると、自動的に["知識"](https://aws.amazon.com/what-is/retrieval-augmented-generation/)機能もツールとして扱われることに注意してください。これは、LLMが自律的にユーザーの問い合わせに応答するために「知識」を使用するかどうかを決定し、利用可能なツールの1つとして考慮することを意味します。

4. デフォルトで、「インターネット検索」ツールが提供されています。このツールにより、エージェントはユーザーの質問に答えるためにインターネット上で情報を検索できます。

![](./imgs/agent1.png)
![](./imgs/agent2.png)

このツールは[DuckDuckGo](https://duckduckgo.com/)に依存しており、レート制限があります。コンセプト実証またはデモンストレーションには適していますが、本番環境で使用する場合は、別の検索APIを使用することをお勧めします。

5. エージェントの機能を拡張するために、独自のカスタムツールを開発して追加できます。カスタムツールの作成と統合の詳細については、[独自のツールを開発する方法](#how-to-develop-your-own-tools)のセクションを参照してください。

## 独自のツールを開発する方法

エージェント用のカスタムツールを開発するには、以下のガイドラインに従ってください：

- `AgentTool`クラスを継承する新しいクラスを作成します。インターフェースはLangChainと互換性がありますが、この実装例では独自の`AgentTool`クラスを提供しており、そこから継承する必要があります（[ソース](../backend/app/agents/tools/agent_tool.py)）。

- [BMI計算ツール](../examples/agents/tools/bmi/bmi.py)の実装例を参照してください。この例は、ユーザー入力に基づいてBMI（体格指数）を計算するツールの作成方法を示しています。

  - ツールで宣言された名前と説明は、LLMがユーザーの質問に応答するためにどのツールを使用すべきかを検討する際に使用されます。言い換えれば、LLM呼び出し時のプロンプトに埋め込まれます。したがって、可能な限り正確に記述することをお勧めします。

- [オプション] カスタムツールを実装したら、テストスクリプト（[例](../examples/agents/tools/bmi/test_bmi.py)）でその機能を確認することをお勧めします。このスクリプトは、ツールが期待通りに動作することを確認するのに役立ちます。

- カスタムツールの開発とテストが完了したら、実装ファイルを[backend/app/agents/tools/](../backend/app/agents/tools/)ディレクトリに移動します。次に、[backend/app/agents/utils.py](../backend/app/agents/utils.py)を開き、`get_available_tools`を編集して、ユーザーが開発したツールを選択できるようにします。

- [オプション] フロントエンド用に明確な名前と説明を追加します。このステップはオプションですが、追加しない場合は、ツールで宣言された名前と説明が使用されます。これらはLLM用であり、ユーザー用ではないため、ユーザーエクスペリエンスを向上させるために専用の説明を追加することをお勧めします。

  - i18nファイルを編集します。[en/index.ts](../frontend/src/i18n/en/index.ts)を開き、`agent.tools`に独自の`name`と`description`を追加します。
  - 希望する国コードの`xx/index.ts`も同様に編集します。

- `npx cdk deploy`を実行して変更を展開します。これにより、カスタムツールがカスタムボット画面で利用可能になります。

## 貢献

**リポジトリへの貢献を歓迎しています！** 役立つツールを適切に実装した場合は、issueやプルリクエストを送信してプロジェクトに貢献することを検討してください。