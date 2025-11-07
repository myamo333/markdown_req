# 品質チェックレポート (最新実行)

実行日時: 2025-11-07 21:14:21 JST

## 1. Markdown構文チェック

```bash
npm run lint:md
```

- **結果**: 失敗（既存ガイド2ファイルで多数のスタイル違反）
- **主な指摘**
  - `docs/01_markdown_quality_assurance_combined.md:1` 見出し前後の空行不足 (MD022)
  - `docs/01_markdown_quality_assurance_combined.md:82` 表の列数不一致 (MD056)
  - `docs/02_markdown_quality_assurance_formatted_guide.md:2`
    強調の見出し利用 (MD036)
  - `docs/02_markdown_quality_assurance_formatted_guide.md:37`
    コードブロック周辺の空行不足 (MD031)
- **対応メモ**: ガイド文書は資料として提供された元データのため、運用開始前に章構造・表整形をルールに合わせて修正するか、`config/lint/.markdownlint.jsonc` / `.pre-commit-config.yaml`
  の `exclude` に追加してください。

## 2. 文体・用語チェック

```bash
npm run lint:text
```

- **結果**: 失敗（`README.md` と `docs/01_markdown_quality_assurance_combined.md`
  で prh が表記ゆれを検出）
- **指摘内容**
  - `README.md:112` 「ドライバー / センサー」を「ドライバ / センサ」に統一
  - `docs/01_markdown_quality_assurance_combined.md:13`
    「ドライバー」を「ドライバ」に統一
- **対応メモ**: READMEは開発者向けドキュメントなので表記統一を行い、必要に応じて
  `config/lint/prh.yml` に例外語を追加してください。

## 3. スペルチェック

```bash
npm run lint:spell
```

- **結果**: 失敗（106語が辞書未登録）
- **主な未登録語カテゴリ**
  - ツール名・コマンド: `textlint`, `markdownlint`, `cspell`, `brew`, `winget`
  - 用語・略語: `flowchart`, `SPEED`, `SENSOR`, `Crash`, `Safety`,
    `requirements`
  - VS Code/CI 関連ID: `textlintrc`, `dearu`, `desumasu`, `checkout`, `setup`
- **対応メモ**: 頻出する技術用語は `config/lint/.cspell.json` の `words`
  に追記、一般単語はドキュメント側を日本語化するなどで解消できます。

---

このレポートを更新するには各コマンドを再度実行し、結果を本ファイルに追記または上書きしてください。
