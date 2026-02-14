# Go Test Results Action

`go test -json` の出力をパースし、テスト結果を PR コメントおよび Actions Job Summary に表示する GitHub Composite Action です。

## Features

- PR コメントにテスト結果サマリーと失敗テストのログを投稿
- Actions Job Summary に全テストの詳細テーブルを表示
- 既存コメントの自動更新（同一 PR への再実行時）
- テスト数（pass / fail / skip）と実行時間の集計

## Usage

```yaml
name: Test

on:
  pull_request:

permissions:
  pull-requests: write

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: "1.22"

      - name: Run tests
        run: go test -json ./... > test-results.json 2>&1 || true

      - name: Show test output
        run: jq -rj 'select(.Output != null) | .Output' test-results.json || true

      - name: Post test results
        uses: ba58ajbse/testview-actions@main
        with:
          test-results-file: test-results.json
```

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `test-results-file` | Yes | - | `go test -json` の出力ファイルパス |
| `token` | No | `${{ github.token }}` | PR コメント投稿用の GitHub トークン |
| `post-comment` | No | `"true"` | PR コメントを投稿するかどうか (`"true"` / `"false"`) |
| `comment-tag` | No | `"go-test-results"` | コメント識別用タグ（同一タグのコメントを更新） |

## Outputs

| Name | Description |
|------|-------------|
| `total` | テスト総数 |
| `passed` | 成功したテスト数 |
| `failed` | 失敗したテスト数 |
| `skipped` | スキップされたテスト数 |
| `elapsed` | 合計実行時間（秒） |
| `summary` | Markdown 形式のテスト結果サマリー |

## Display

### PR Comment

サマリーテーブル（テスト数・実行時間）と失敗テストのログを表示します。

### Actions Job Summary

サマリーテーブルに加え、全テストのステータス・パッケージ・実行時間の詳細テーブルを表示します。

## License

MIT
