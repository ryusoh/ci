# Shared CI (Reusable Workflows & Actions)

このリポジトリは、他のリポジトリ（例: `fund`, `ryusoh.github.io`）から再利用する GitHub Actions の共通ワークフローとコンポジットアクションを提供します。

## 使い方（Reusable Workflow）

呼び出し側リポジトリに以下のようなワークフローファイルを配置します。

```yaml
# .github/workflows/ci.yml（呼び出し側）
name: CI
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  web-ci:
    uses: ryusoh/ci/.github/workflows/web-ci.yml@main  # 安定したら @v1 に変更
    secrets: inherit
    with:
      node-version: '20'
      python-version: '3.12'
      run-precommit: true
      deploy: false
```

入力パラメータ:
- `node-version` (string): Node.js のバージョン
- `python-version` (string): Python のバージョン
- `run-precommit` (boolean): pre-commit を実行するか
- `deploy` (boolean, optional): 呼び出し側でデプロイが必要な場合に true（実装は呼び出し側で拡張）

> 注意: サブモジュールを使用しているリポジトリでも動作するよう、`web-ci.yml` は `fetch-depth: 0` と `submodules: recursive` で `checkout` します。

## 使い方（Composite Action: precommit）

ワークフロー内の任意のステップで下記のように呼び出せます。

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - uses: ryusoh/ci/actions/precommit@main
        # with:
        #   config: path/to/.pre-commit-config.yaml  # オプション
```

## タグ運用（推奨）

安定したらタグを打ち、呼び出し側はタグで参照してください。

```bash
git tag -a v1 -m "Reusable Web CI v1"
git push --tags
```

呼び出し側の `uses` を `@v1` に変更すると、変更の影響を受けにくくなります。

## 含まれるファイル

- `.github/workflows/web-ci.yml`: 再利用可能な Web CI ワークフロー（pre-commit、Node/Python セットアップ、サブモジュール対応）
- `actions/precommit/action.yml`: pre-commit を実行するためのコンポジットアクション

## 拡張案

- Pages デプロイ用の reusable workflow を追加（`deploy: true` の場合に発火 or 別ワークフローに分離）
- Node/PNPM/Yarn のマトリクス、Python のマトリクスに対応した派生ワークフロー
