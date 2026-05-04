# WakeupClaude

Claude Code のサブスクリプション枠は 5 時間ごとにリセットされます。このリポジトリは、任意の時刻に最小コストで API を 1 度呼び出し、5 時間枠のリセット時刻を能動的にコントロールするための GitHub Actions ワークフローです。

## セットアップ

### 1. OAuth トークンの取得

Claude Code CLI がインストール済みの環境で、アカウントごとに以下を実行します。

```bash
claude setup-token
```

表示されたトークン（`sk-ant-oat-...` 形式）をメモしておきます。

### 2. Secret の登録

リポジトリの **Settings → Secrets and variables → Actions** を開き、**New repository secret** から以下を作成します。

| Name | Value |
|------|-------|
| `CLAUDE_CODE_OAUTH_TOKENS` | JSON 配列の文字列（例は下記） |

**値の形式（JSON 配列の文字列）:**

```
["sk-ant-oat-アカウント1のトークン", "sk-ant-oat-アカウント2のトークン"]
```

トークンを追加したい場合は、配列に要素を追加するだけです。

## ワークフローのトリガー方法

### gh CLI を使う場合

```bash
# index=0 のアカウント（配列の先頭）をウェイクアップ
gh workflow run wakeup-claude.yml -f index=0

# index=1 のアカウント（配列の2番目）をウェイクアップ
gh workflow run wakeup-claude.yml -f index=1
```

### GitHub REST API を使う場合

```bash
curl -X POST \
  -H "Authorization: Bearer $GH_PAT" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/<owner>/<repo>/actions/workflows/wakeup-claude.yml/dispatches \
  -d '{"ref":"main","inputs":{"index":"0"}}'
```

> **注意:** REST API では `inputs` の値は文字列として渡す必要があります（`"0"` であり `0` ではない）。

## 挙動

`--model haiku --max-turns 1` で `"hi"` を 1 回送るだけなので、トークン消費は最小限です。5 時間枠のリセット起算点だけが更新されます。

## 複数アカウントの運用例

配列インデックスとアカウントが 1 対 1 で対応します。外部スケジューラ（例: 別リポジトリの cron ワークフロー、curl を叩く cron デーモンなど）でアカウントごとに時間をずらして dispatch することで、各アカウントの 5 時間枠を任意の時刻に張り直せます。

```
06:00 → index=0 dispatch（アカウントAの枠を 06:00 起算に固定）
07:30 → index=1 dispatch（アカウントBの枠を 07:30 起算に固定）
```
