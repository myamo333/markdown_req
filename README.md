# Markdown要件定義書 品質保証ライン README

このリポジトリは、以下の設計ドキュメントに基づき Markdown 要件定義書の品質保証ラインを構築・共有するためのテンプレートです。

- `01_markdown_quality_assurance_combined.md`
- `02_markdown_quality_assurance_formatted_guide.md`

開発者は本READMEに沿って環境をセットアップし、Lint／Prettier／pre-commit／GitHub
Actions を組み合わせたチェックフローを再現してください。

## 1. 前提条件

- **OS**: Windows 11 / macOS 14 / Ubuntu 22 いずれか
- **Node.js / npm**: 18 LTS 以上（`npm ci` で再現性を確保）
- **Python**: 3.8 以上（`pre-commit` の実行に必要）
- **Git**: 2.30 以上（フック動作用）
- **VS Code**: 最新安定版（Lint拡張と保存時整形に使用）
- **パッケージマネージャ**: npm を前提
- **リポジトリ構成**: ルート直下にLint/Formatter/辞書/CI設定、`.vscode/` と
  `.github/workflows/` を併設

## 2. セットアップ手順

### 2.1 リポジトリ取得

```bash
# macOS / Ubuntu
git clone git@github.com:your-org/markdown-qa.git
cd markdown-qa
```

```powershell
# Windows PowerShell
git clone git@github.com:your-org/markdown-qa.git
Set-Location markdown-qa
```

### 2.2 依存パッケージの導入

```bash
# macOS / Ubuntu
npm ci || npm install
python3 -m pip install --user pre-commit
pre-commit install
```

```powershell
# Windows PowerShell
npm ci
py -m pip install --user pre-commit
pre-commit install
```

> `pre-commit` をコマンドとして使えるよう、シェルの設定ファイルに
> `export PATH="/Users/<username>/Library/Python/3.9/bin:$PATH"`（macOS例）を追記してください。

### 2.3 初回チェック

```bash
npx markdownlint "**/*.md"
npx textlint "**/*.md"
npx cspell "**/*.md"
```

各コマンドで確認できる内容：

- `markdownlint`：見出しレベルや箇条書きの書き方、空行などMarkdown構文ルールの崩れを検出し、章構造やリストが崩れていないかを素早く把握できます。
- `textlint`：文体（である/ですます混在）や句読点、prh辞書による表記ゆれをチェックし、文章の品質と用語統一が守られているかを確認します。
- `cspell`：Markdown内の専門語・英単語を辞書参照で綴りチェックし、略語や信号名のタイプミスをコミット前に見つけられます。

Lint警告が出る既存ドキュメント（01/02のガイド）は `.markdownlintignore` /
`.textlintignore` / `.cspell.json` の `ignorePaths` で除外済みです。  
要件サンプル（`sample_requirement.md`）は常にLint対象として残してあり、動作確認に利用できます。別の参照資料を対象外にしたい場合は、これらのファイル群へパスを追記すると CLI /
pre-commit / CI が一括で無視します。必要に応じて `.pre-commit-config.yaml` の
`exclude` にも同じパターンを追加してください。

## 3. 設定ファイル一覧

| ファイル                           | 役割               | 備考                                                                    |
| ---------------------------------- | ------------------ | ----------------------------------------------------------------------- |
| `.markdownlint.jsonc`              | Markdown構文ルール | ATX見出し、2スペースインデント、長文許容などを定義                      |
| `.textlintrc`                      | 文体・表記統一     | `preset-japanese` + `max-ten` + `no-mix-dearu-desumasu` + `prh.yml`     |
| `.cspell.json`                     | スペルチェック辞書 | `words` にドメイン語を追加し、`ignorePaths` で `node_modules` を除外    |
| `prh.yml`                          | 用語辞書           | 「ドライバ/センサ/アクチュエータ/CAN」などの表記ゆれを吸収              |
| `.prettierrc`                      | 保存時整形         | 2スペース・`proseWrap: "always"` 等でMarkdownフォーマットを固定         |
| `.vscode/settings.json`            | VS Code共有設定    | `formatOnSave` と `cSpell.configFile` をプロジェクト全体に適用          |
| `.pre-commit-config.yaml`          | Gitフック設定      | `markdownlint`, `textlint`, `cspell`, `prettier` などをコミット前に実行 |
| `.github/workflows/lint-check.yml` | CI                 | PR作成時に `npm ci` → 各Lintを再実行                                    |

## 4. VS Code 推奨拡張

| 拡張名             | ID                                    | 用途                                             |
| ------------------ | ------------------------------------- | ------------------------------------------------ |
| Markdownlint       | DavidAnson.vscode-markdownlint        | `.markdownlint.jsonc` を読み込みリアルタイム警告 |
| textlint           | taichi.vscode-textlint                | `.textlintrc` + `prh.yml` に基づく文体チェック   |
| Code Spell Checker | streetsidesoftware.code-spell-checker | `.cspell.json` と連携したスペルチェック          |
| Prettier           | esbenp.prettier-vscode                | `.prettierrc` 通りに保存時整形                   |
| GitHub Actions     | GitHub.vscode-github-actions          | Workflow編集・結果確認をVS Codeで実施            |

## 5. 日常利用フロー

1. VS Codeで要件Markdownを編集（波線警告でLint違反を把握）。
2. 保存するとPrettierが整形。
3. `pre-commit` フックがコミット前に MarkdownLint / textlint / cspell /
   Prettier チェックを実行。
4. PR作成で GitHub Actions
   (`lint-check.yml`) が再度 Lint を実行し、成功しない限りマージ不可。

### よく使うコマンド

```bash
# 変更ファイルのみフックを手動実行
pre-commit run --all-files

# 単体チェック
npx markdownlint sample_requirement.md
npx textlint sample_requirement.md
npx cspell sample_requirement.md
```

## 6. 動作確認シナリオ

1. `sample_requirement.md`
   を編集して意図的に「ドライバー」「センサー」など表記ゆれを入れる。
2. VS Code 上で textlint/prh 警告が出ることを確認。
3. `git add sample_requirement.md` → `git commit`
   を実行し、pre-commit が違反をブロックすることを確認。
4. 修正後にコミットし、`git push` → PR作成。GitHub Actions の `Lint Check`
   が成功するまで監視。

## 7. トラブルシュート

| 症状                                        | 対処                                                                                                                    |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `pre-commit: command not found`             | `pip install --user pre-commit` 実行後、PATH に `~/Library/Python/<version>/bin` などを追加。                           |
| `npx markdownlint` で既存ガイドが大量エラー | `.pre-commit-config.yaml` と同じ `exclude` パターンを `package.json` の npm script やCIにも反映、もしくはガイドを整形。 |
| VS Code保存時に整形されない                 | コマンドパレットで「既定のフォーマッタ」を Prettier に設定、`editor.formatOnSave` が true か確認。                      |
| `pre-commit run` が遅い                     | `pre-commit clean && pre-commit install` でフックを再取得し、差分のみ処理したい場合は `pass_filenames: true` を検討。   |
| cspell が固有名詞を誤検出                   | `.cspell.json` の `words` に単語を追加し、チームで共有。                                                                |

## 8. 参考資料

- `01_markdown_quality_assurance_combined.md`: 品質保証ライン導入の背景と全体設計。
- `02_markdown_quality_assurance_formatted_guide.md`: 実際の構築手順・設定例。

上記ドキュメントを常に参照しながら、必要に応じてルールをアップデートし、Pull
Requestで共有してください。
