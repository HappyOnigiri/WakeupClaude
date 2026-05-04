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

`workflow_dispatch` を叩くだけで、配列内の**全トークンに対して順番に実行**されます。

### gh CLI を使う場合

```bash
gh workflow run wakeup-claude.yml
```

### GitHub REST API を使う場合

```bash
curl -X POST \
  -H "Authorization: Bearer $GH_PAT" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/<owner>/<repo>/actions/workflows/wakeup-claude.yml/dispatches \
  -d '{"ref":"main"}'
```

## 挙動

1. Claude Code CLI (`@anthropic-ai/claude-code`) をランナーにインストールします。
2. `CLAUDE_CODE_OAUTH_TOKENS` 配列をループし、各トークンで `claude -p "hi" --model haiku --max-turns 1` を実行します。
3. トークン消費は最小限で、5 時間枠のリセット起算点が全アカウント分更新されます。
