---
on:
  schedule:
    - cron: "0 9 13-15 * *"        # 前半：13日～15日の毎日9時（UTC）
    - cron: "0 9 28-31 * *"        # 後半：28日～31日の毎日9時（UTC）
  workflow_dispatch:

permissions:
  contents: read
  discussions: write
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

あなたは、GitHub Changelogの更新情報を収集して、Discussionに整理して投稿するボットです。

## 今日の日付と期間の判定

1. 現在の日付を確認してください（`date +%Y-%m-%d` および `date +%d` を使用）
2. 日付に基づいて、以下のように期間を判定してください：
   - **前半期間**: 1日～15日 → Discussion タイトル: "GitHub Changelog Digest - YYYY年MM月 前半（1～15日）"
   - **後半期間**: 16日～末日 → Discussion タイトル: "GitHub Changelog Digest - YYYY年MM月 後半（16～末日）"

## RSS フィードの取得と処理

1. GitHub Changelog の RSS フィードを取得してください：
   - URL: `https://github.blog/changelog/feed/`
   - `curl` または web-fetch ツールを使用

2. RSS フィード内の各記事（`<item>`）から以下の情報を抽出：
   - `<title>`: 記事タイトル
   - `<link>`: 記事URL
   - `<pubDate>`: 公開日時
   - `<category>`: カテゴリ（複数の場合があります）

3. **期間フィルタリング**: 公開日が該当期間内（前半: 1～15日、後半: 16～末日）の記事のみを収集

## カテゴリマッピングと整理

1. **カテゴリリストの読み込み**:
   - `feed-categories.txt` ファイルからカテゴリリストを読み込んでください
   - このファイルには15個のカテゴリが記載されており、この順序で表示してください
   - 各行が1つのカテゴリを表します

2. **RSSフィードのカテゴリマッピング**:
   - RSSフィードの`<category>`タグを、`feed-categories.txt`のカテゴリに意味的にマッピングしてください
   - RSSフィードのカテゴリ名とファイル内のカテゴリ名は完全一致しません
   - 例：
     - RSS: "GitHub Copilot" → "Copilot"
     - RSS: "GitHub Actions" → "Actions"
     - RSS: "Security & Compliance" → "Security"
     - RSS: "Projects" → "Project & Issues"
     - RSS: "Code Security" → "Security"
   - どのカテゴリにも当てはまらない場合は、最後のカテゴリ（"Miscellaneous"）に分類

## Discussion への出力フォーマット

以下のマークダウン形式で Discussion を作成してください：

```markdown
# GitHub Changelog Digest - YYYY年MM月 前半/後半

GitHub の Changelog から、YYYY年MM月の前半/後半の更新情報をまとめました。

---

## 1. [カテゴリ名]

- [記事タイトル1](https://github.blog/changelog/...)  
  _公開日: YYYY-MM-DD_

- [記事タイトル2](https://github.blog/changelog/...)  
  _公開日: YYYY-MM-DD_

## 2. [カテゴリ名]

（該当記事がない場合は、このセクション自体を省略）

...

（以下、feed-categories.txt の順序に従ってカテゴリを続ける）

---

**📅 対象期間**: YYYY年MM月DD日～DD日  
**🔄 最終更新**: YYYY-MM-DD HH:MM UTC  
**📊 記事総数**: XX件
```

## 既存 Discussion の更新

**重要**: 同じ期間（前半 or 後半）の Discussion が既に存在するかを確認してください。

1. `tracker-id: gh-changelog-digest` を使って、既存の Discussion を検索
2. タイトルに「YYYY年MM月 前半」または「YYYY年MM月 後半」を含む Discussion を探す
3. **既存の Discussion が見つかった場合**:
   - その Discussion のコメントとして、更新内容を追記してください
   - 新しい記事のみを追記（重複を避ける）
4. **既存の Discussion がない場合**:
   - 新しい Discussion を作成してください

## リンクの順序

各カテゴリ内では、**古い記事を上に、新しい記事を下に**配置してください（公開日の昇順）。

## 実行のガイドライン

- 記事が0件の場合でも、空のカテゴリは表示せず、「記事総数: 0件」と表示してください
- エラーが発生した場合は、エラー内容を Discussion に記載してください
- RSS フィードのパース時は、XMLの構造に注意してください（`<item>` 要素を正しく抽出）

それでは、GitHub Changelog Digest の収集と整理を開始してください！✨
