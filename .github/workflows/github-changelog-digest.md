---
name: GitHub Changelog Digest
on:
  schedule:
    # 前半: 13日〜15日に実行（UTC）
    - cron: "0 9 13-15 * *"
    # 後半: 28日〜31日に実行（UTC）
    - cron: "0 9 28-31 * *"
  workflow_dispatch:
    inputs:
      period:
        description: "対象期間（auto / first / second）"
        required: false
        default: "auto"
        type: choice
        options:
          - auto
          - first
          - second
      year_month:
        description: "対象年月（YYYY-MM）。未指定なら実行日から推定"
        required: false
        type: string

permissions:
  contents: read
  discussions: read
  issues: read
  pull-requests: read

tools:
  web-fetch:
  bash:
    - "date *"
    - "curl *"

safe-outputs:
  create-discussion:
    category: "General"
    max: 1

timeout-minutes: 15

network:
  allowed:
    - "github.blog"
    - "github.com"

tracker-id: "gh-changelog-digest"
---

# GitHub Changelog Digest

あなたは GitHub Changelog（RSS）を収集し、期間ごと（前半/後半）にカテゴリ整理したダイジェストを GitHub Discussions に投稿します。

## 目標

- `https://github.blog/changelog/feed/` を取得し、対象期間内の記事だけを抽出
- 記事をカテゴリに分類してマークダウンに整形
- 同一期間の Discussion が既にあれば「追記」（コメント）または「更新」方針で重複を避ける
  - 既存検出は `tracker-id: gh-changelog-digest` とタイトルで行う

## 対象期間の決定

1. `workflow_dispatch` の inputs を読む
   - `period` が `first` / `second` ならそれを優先
   - `auto` の場合は実行日の日付から判定
     - 前半: 1〜15日
     - 後半: 16日〜末日
2. `year_month` が指定されていればその年月（YYYY-MM）を使う
3. Discussion タイトル案:
   - 前半: `GitHub Changelog Digest - YYYY年MM月 前半（1〜15日）`
   - 後半: `GitHub Changelog Digest - YYYY年MM月 後半（16日〜末日）`

## RSS の取得と抽出

1. RSS を取得
   - URL: `https://github.blog/changelog/feed/`
2. `<item>` から以下を抽出
   - タイトル / リンク / 公開日（pubDate） / カテゴリ（複数可）
3. 対象期間でフィルタ

## カテゴリ整理

- 可能ならリポジトリ内の `feed-categories.txt` を読み、この順番をカテゴリ見出し順として使う
- 見つからない場合は、以下のデフォルト順を使う

```
Copilot
Actions
Security
Project & Issues
Code
CI/CD
API
Enterprise
Billing
Mobile
Design
Docs
Community
Releases
Miscellaneous
```

### マッピング

RSS のカテゴリ名は完全一致しないため、意味的に最も近い見出しへ割り当ててください。
当てはまらないものは `Miscellaneous` に入れてください。

## Discussion 本文のフォーマット

```markdown
# GitHub Changelog Digest - YYYY年MM月 前半/後半

GitHub の Changelog から、YYYY年MM月の前半/後半の更新情報をまとめました。

---

## [カテゴリ名]

- [記事タイトル](URL)

---

**対象期間**: YYYY年MM月DD日〜DD日  
**最終更新**: YYYY-MM-DD HH:MM UTC  
**記事総数**: XX件
```

### 並び順

- カテゴリ内は公開日の昇順（古い→新しい）
- 該当記事がないカテゴリ見出しは出力しない

## 例外時の挙動

- RSS 取得やパースに失敗したら、原因（HTTP ステータス/エラー）を含めて Discussion に残す（可能な範囲で）
