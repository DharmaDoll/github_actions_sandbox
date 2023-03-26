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
  paths-ignore: 
    - 'docs/**
```

paths か paths-ignore が指定されていると、ファイルが一つも変更されなかっ たときはワークフローが実行されません。
 変更されたファイル一覧は、次のように計算されます。
• pull_request: ベースとなるブランチとソースブランチが分岐するコミット から、ソースブランチの最新コミットまでの差分
• 既存ブランチへの push: push 前の HEAD からの差分
• 新規ブランチへの push: push されたコミットの祖先ですでに push されてた
   最新のコミットからの差分
差分に 1,000 コミット以上存在するときや、差分の計算に時間がかかりすぎると きは、指定した条件に関わらずワークフローが実行されてしまうらしいです。

### キャッシュ
CI/CD では、リポジトリが依存するパッケージのダウンロードが原因でビルド時 間が長くなってしまうことがあります。近年の CI/CD サービスでは、ビルドごとに 完全にクリーンな実行環境が用意され、前回のビルドでダウンロードしたファイルが 持ち越されないからです。
この問題を解決するためには、CI/CD が提供するキャッシュ機能を用いて、異な るビルド間でダウンロードしたパッケージを使い回して高速化することが一般的で す。GitHub Actions でも actions/cache を使うことでキャッシュ機能が利用可能 です。この節では、GitHub Actions のキャッシュ機能について解説します。

```yaml
name: Continuous Integration on: push
jobs:
  unit-test:
    ...
    steps:
    ...
    - name: Get NPM cache directory 
      id: npm-cache
      run: |
        echo "::set-output name=dir::$(npm config get cache)" 
    - name: Cache NPM
      uses: actions/cache@v2.0.0
        with:
          path: ${{ steps.npm-cache.outputs.dir }}
          key: ${{ runner.os }}-node- ${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
     - name: Install dependencies 
      run: npm ci
```

- actions/cache アクションを実行すると、その時点で key に指定したキーに完全一致するキャッシュが存在するときは、path で指定したファイルパスに復元されます。
- keyで指定したキーでキャッシュが存在しない場合は、restore-keys で指定し たキーに前方一致するキーを持つキャッシュが復元されます。どちらのキーでもキャッシュが存在しない場合は、キャッシュの復元は行われません。
- また、actions/cache アクションを実行すると、ジョブの最後に指定したキーで 指定したファイルパスの内容がキャッシュとして保存されるようになります。すでに 同じキーでキャッシュが存在するときは保存されず、キャッシュの上書きは行われません。
- key では、コンテキストから複数の値を取得してキーを組み立てています。${{ runner.os }} でジョブ実行環境の OS を取得して、キーに含めています。このジョブでは複数の OS でマトリクスビルドを行っており、NPMのキャッ シュする内容は OSによって異なってくる可能性があるためです。
- また、${{ hashFiles(’**/package-lock.json’) }} でリポジトリ内の依存関係を定義するすべてのロックファイルのハッシュをキーに含めています。これにより、NPMの依存関係に変更があったときは新たにキャッシュが作成され直します。 このリポジトリにはルートディレクトリにしか package-lock.json が存在しないので **/ は必要ないですが、リポジトリ内で複数のモジュールを管理するケースを考えて全ディレクトリの package-lock.json を指定しています。

#### 制限事項
- キャッシュが利用できるイベントタイプ
  - GitHub Actions のキャッシュは、‘GITHUB_REF‘ が存在するすべてのイベン トで利用可能です。イベントの種類については、「2.7 イベントとアクティビティ」を 参照してください。
- ブランチ
  - ワークフローからアクセスできるキャッシュは、現在のブランチ、プルリクエスト
のベースブランチ、デフォルトブランチで作成されたキャッシュのみです。それ以外
のブランチで作成されたキャッシュにはアクセスできません。
- キャッシュの保存期間
  - 7日間アクセスされなかったキャッシュは、削除されます。
  
また、キャッシュの数には制限はありませんが、キャッシュのサイズには次のよう
な上限があります。
• リポジトリ全体で最大 5GB のキャッシュサイズ上限がある
– 圧縮した後のファイルサイズで判定される様子
– 5GB を超えた場合、キャッシュのアクセス時間が古い順に追い出される
サイズ上限がある代わりに、現時点では GitHub Actions のキャッシュ機能は完全に無料で利用できます。

#### プログラミング言語ごとの例
公式で主要なプログラミング言語とパッケージマネージャーごとの例が用意され ています。どのディレクトリをキャッシュすればいいか、どのファイルのハッシュを キーに含めればいいか、といったことがわからないときに参考になります。
https://github.com/actions/cache/blob/master/examples.md

### アーティファクト
CI/CD では、ビルド中に生成したファイルをビルド後に利用できるように保存し たいことがあります。例えば、バイナリやアーカイブなどの成果物や、デバッグ用の ログやテスト結果、カバレッジなどの情報を保存したいということがよくあります。
GitHub Actions では、アーティファクトという機能を用いてワークフロー実行時の成果物の保存が実現可能です。
