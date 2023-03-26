# github_actions_sandbox
Test for GitHub Actions

Please refer to [yaml files](https://github.com/DharmaDoll/github_actions_sandbox/tree/main/.github/workflows).




例えば、releases/ で始まるブランチのときだけワークフローを実行し、relea ses/hoge-testing のように -testing で終わる形式のブランチのときはワークフ ローを実行しないようにするには、リスト 2.33 のように書きます。

```yaml
on: push:
  branches:
    - releases/**
    - '!releases/**-testing'
```

push イベントと pull_request イベントでは、変更されたファイルのパスによっ てワークフローの実行をフィルタできます。paths を指定すると、変更されたファイ ルのうち少なくとも一つのパスが指定されたパスにマッチするときのみワークフロー が実行されます。paths-ignore を指定すると、変更されたファイルのうち少なく とも一つのパスが指定されたパスにマッチしないときのみワークフローが実行されま す。使用できる特殊文字は、ブランチやタグのフィルタと一緒です。
例えば、リスト 2.34 のように書くと、任意の JavaScript ファイルが変更されたと きにワークフローが実行されます。

```yaml
on: push:
  paths:
   - '**.js'
```
docs ディレクトリ下のファイルだけが変更されたときは実行しない

```yaml
on: push:
  paths-ignore: - 'docs/**
```

paths か paths-ignore が指定されていると、ファイルが一つも変更されなかっ たときはワークフローが実行されません。
 変更されたファイル一覧は、次のように計算されます。
• pull_request: ベースとなるブランチとソースブランチが分岐するコミット から、ソースブランチの最新コミットまでの差分
• 既存ブランチへの push: push 前の HEAD からの差分
• 新規ブランチへの push: push されたコミットの祖先ですでに push されてた
   最新のコミットからの差分
差分に 1,000 コミット以上存在するときや、差分の計算に時間がかかりすぎると きは、指定した条件に関わらずワークフローが実行されてしまうらしいです。
