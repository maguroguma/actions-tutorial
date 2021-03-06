# GitHub Actionsの概要

Last Change: 2022-06-19 13:14:14.

[公式のドキュメント](https://docs.github.com/en/actions)のLearn GuitHub Actionsより。

## GitHub Actionsを理解する

https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions

- Workflows
  - 複数のジョブからなる自動化プロセス
  - YAMLファイルで定義される
  - 色々なイベントで実行可能
  - 1つのリポジトリに複数のワークフローを定義可能
  - ワークフローから他のワークフローを参照できる
- Runners
  - ワークフローがトリガーされたときに実行するサーバーのこと
    - Dockerコンテナのようなものだと思えば良い？
  - 各ランナーは一度に一つのジョブを実行できる
  - Ubuntu, Windows, Mac OSのランナーを提供している
    - これ以外のOSが必要な場合は、独自のランナーをホストすることもできるらしい
- Jobs
  - 同じランナーで実行されるワークフローのステップのセット
  - 各ステップは、実行されるシェルスクリプト、または実行されるアクション
  - ステップは順番に実行され、互いに依存しあっている
    - 前のステップで生成されたデータを、あとのステップに渡したりできる
  - ジョブは他のジョブとの依存関係を設定できる
    - デフォルトでは、ジョブ同士は依存せず、互いに並行して実行される
- Actions
  - GitHub Actionsプラットフォームのカスタムアプリケーションで、複雑だが頻繁に繰り返されるタスクを実行するもの
    - アクションによって、GitHubからgitリポジトリを取得したり、ビルド環境に適したツールチェインをセットアップしたり、クラウドプロバイダへの認証をセットアップしたりできる
    - 予め定義されたワークフローぐらいに思っていたが、ワークフローとは別の概念？
      - ワークフローからも呼び出せる、GitHub Actionsプラットフォーム専用のアプリ、ぐらいに思っておく
  - 自分でアクションを描くこともできるし、GitHub Marketplaceでワークフローで使うアクションを見つけることもできる
  - yamlでは `uses` というフィールドで指定される

## Actionsのカスタマイズ方法

https://docs.github.com/en/actions/learn-github-actions/finding-and-customizing-actions

- Marketplaceからのアクションの追加は、GitHub上のGUIから設定できる
- docker hubから提供されるアクションもある
  - `uses` フィールドを見れば判別可能
- actionもアプリケーションという位置づけなので、バージョンなども気にすること

## 特徴

https://docs.github.com/en/actions/learn-github-actions/essential-features-of-github-actions

変数の使用、スクリプトの実行、ジョブ間でのデータや成果物の共有。

- 環境変数は `env` フィールドで指定できる
- リポジトリ内のスクリプトは、以下のような記述で実行できる（パスに注目）
  - `run: ./.github/scripts/build.sh`
- ジョブ間での成果物の共有は、記述に若干の慣れが必要そう
  - `uses: actions/upload-artifact@v3` のようなアクションの利用が必要になる
  - 順序制御のための記述も必要になる

## 式

https://docs.github.com/en/actions/learn-github-actions/expressions

- ifキーワードと式を併用して、ステップを実行するかどうか制御できる
- シェルで使えるような組み込み関数もいくつかあるので、それらも併用することになる
- 以前のstepの成功か失敗か、などをチェックするstatus check functionsというものもあり、これらは利用しやすそう

## Contexts

https://docs.github.com/en/actions/learn-github-actions/contexts

コンテキストは、ワークフローの実行、ランナー環境、ジョブ、およびステップに関する情報にアクセスするための方法。

- `github` というコンテキストの場合、フィールドのアクセスは `github['sha']` とか `github.sha` のようになる
- 以下の例で `github.ref` はコンテキストにあたる
  - ブランチをチェックしており、これでジョブ全体を実行するかどうかを制御している
- デバッグのために、contextの内容をログに出力できる

```yaml
name: CI
on: push
jobs:
  prod-check:
    if: ${{ github.ref == 'refs/heads/main' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to production server on branch $GITHUB_REF"
```

## 環境変数

https://docs.github.com/en/actions/learn-github-actions/environment-variables

- デフォルトの環境変数と、yamlで指定するカスタム環境変数がある
- カスタム環境変数はスコープに注意
  - ジョブ単位だったり、ステップ単位で定義できたりする？
- 環境変数の参照の仕方は、ランナーOSによって変える必要がある
  - winの場合は慣れない表記になるので注意
- **contextはワークフローがスコープで、環境変数はジョブ（ランナーOS）がスコープ**
  - contextのほうが「広い、大きい」概念であり、実際 `env.DAY_OF_WEEK` みたいな感じでcontext経由で環境変数にアクセスすることもできる
- ジョブのあるステップで値を生成した場合、その値を既存または新規の環境変数に代入し、これを `GITHUB_ENV` 環境ファイルに書き込むことにより、同じジョブの後続ステップで値を使用できる
  - 任意の環境変数に代入できるわけではない（？）

## 使用上限、請求、管理

https://docs.github.com/en/actions/learn-github-actions/usage-limits-billing-and-administration

- 公開リポジトリであれば無料（無制限？）、非公開リポジトリだと上限がある
- 公開リポジトリでも、ジョブやワークフローの実行時間に上限があったり、APIリクエストのタイムレートがあったり、一度に実行できるジョブの上限数がある
  - GASの使用制限っぽい雰囲気
- 不必要なワークフローの実行を防ぐために、スケジュールされたワークフローは自動的に無効にあることがある
  - 公開リポジトリをフォークすると、スケジュールされたワークフローはデフォルトで無効化される
  - 公開リポジトリでは、60日間リポジトリのアクティビティが発生しないと、スケジュールされたワークフローは自動的に無効になる

