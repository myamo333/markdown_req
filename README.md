# Markdown要件定義書 品質保証ライン README

このリポジトリは、以下の設計ドキュメントに基づき Markdown 要件定義書の品質保証ラインを構築・共有するためのテンプレートです。

- `docs/01_markdown_quality_assurance_combined.md`
- `docs/02_markdown_quality_assurance_formatted_guide.md`

開発者は本READMEに沿って環境をセットアップし、Lint／Prettier／pre-commit／GitHub Actions を組み合わせたチェックフローを再現してください。

## 1. 前提条件

- **OS**: Windows 11 / macOS 14 / Ubuntu 22 いずれか
- **Node.js / npm**: 18 LTS 以上（`npm ci` で再現性を確保）
- **Python**: 3.8 以上（`pre-commit` の実行に必要）
- **Git**: 2.30 以上（フック動作用）
- **VS Code**: 最新安定版（Lint拡張と保存時整形に使用）
- **パッケージマネージャ**: npm を前提
- **リポジトリ構成**:
  - `config/lint/`: markdownlint / textlint / cspell / prettier / prh などの設定
  - `requirements/`: 要件サンプル（`sample_requirement.md`）など実要件を配置。ディレクトリ直下の`.markdownlint.jsonc` / `.textlintrc`でルールを上書き可能
  - `soft_requirements/`: ソフトウェア要件（`autonomous_software_requirements.md` など）を配置し、独自のローカルLint設定を持つ
  - `docs/`: 参考資料 (`01/02` ガイド、`CHECK_REPORT.md`, `AGENTS.md` など)
  - `.github/workflows/`: CI
  - `.vscode/`: VS Code 共有設定

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
# requirements ディレクトリ向け
npm run lint:req:md
npm run lint:req:text

# soft_requirements ディレクトリ向け
npm run lint:soft:md
npm run lint:soft:text

# 両方まとめて & スペルチェック
npm run lint:md
npm run lint:text
npm run lint:spell
```

各コマンドで確認できる内容：

- `lint:req:md` / `lint:soft:md`：ディレクトリ固有の `.markdownlint.jsonc` を利用して、章構成や許容行長などのルールを検証。
- `lint:req:text` / `lint:soft:text`：各フォルダの `.textlintrc` を参照し、文体や句読点の厳しさをフォルダ単位で調整。
- `lint:spell`：`requirements` と `soft_requirements` のみを対象に cspell を実行し、ドメイン固有語を辞書で管理。

Lint警告が出る既存ドキュメント（docs配下の資料）は `config/lint/.markdownlintignore` / `.textlintignore` / `.cspell.json` の `ignorePaths` で除外済みです。  
各フォルダでルールを緩和したい場合は、`requirements/.markdownlint.jsonc` や `soft_requirements/.textlintrc` のみ編集すれば、他フォルダへ影響せずに運用できます。

## 3. 設定ファイル一覧

| ファイル                           | 役割               | 備考                                                                    |
| ---------------------------------- | ------------------ | ----------------------------------------------------------------------- |
| `config/lint/.markdownlint.jsonc`  | Markdown構文ルール | ATX見出し、2スペースインデント、長文許容などを定義                      |
| `config/lint/.textlintrc`          | 文体・表記統一     | `preset-japanese` + `max-ten` + `no-mix-dearu-desumasu` + `prh.yml`     |
| `config/lint/.cspell.json`         | スペルチェック辞書 | `words` にドメイン語を追加し、`ignorePaths` で `docs/` の資料を除外     |
| `config/lint/prh.yml`              | 用語辞書           | 「ドライバ/センサ/アクチュエータ/CAN」などの表記ゆれを吸収              |
| `config/lint/.prettierrc`          | 保存時整形         | 2スペース・`proseWrap: "always"` 等でMarkdownフォーマットを固定         |
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
npx markdownlint requirements/sample_requirement.md --config config/lint/.markdownlint.jsonc --ignore-path config/lint/.markdownlintignore
npx textlint requirements/sample_requirement.md --config config/lint/.textlintrc --ignore-path config/lint/.textlintignore
npx cspell requirements/sample_requirement.md --config config/lint/.cspell.json
```

### `pre-commit run --all-files` で行われるチェック

`pre-commit` は `.pre-commit-config.yaml` で定義された順に以下のフックを実行します。`--all-files` を付けるとステージ状態に関係なく対象ディレクトリ全体を検査します。

1. **trailing-whitespace**（末尾スペースを削除）
2. **end-of-file-fixer**（ファイル末尾の改行を強制）
3. **markdownlint (requirements / soft requirements)**（それぞれの `.markdownlint.jsonc` を使用）
4. **textlint (requirements / soft requirements)**（`prh` を含むルールで文体チェック）
5. **cspell**（`config/lint/.cspell.json` の辞書）
6. **prettier**（`config/lint/.prettierrc` に沿った整形差分が無いか確認）

### フックが `Failed` になった場合の対処

| フック名                            | 代表的なエラー例                                   | 解消手順                                                                                                                                         |
| ----------------------------------- | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| trailing-whitespace / end-of-file   | `Fixing newline at end of file` など               | VS Code の `Trim Trailing Whitespace` を実行、もしくは `pre-commit run trailing-whitespace --all-files` で自動修正。                              |
| markdownlint (requirements / soft)  | `MD0xx` 番号の警告                                | `npx markdownlint <file> --config <dir>/.markdownlint.jsonc --fix` で自動修正し、ルールに沿わない箇所を整形。                                     |
| textlint (requirements / soft)      | `[prh] ドライバー => ドライバ` 等のルール違反      | `npx textlint --config <dir>/.textlintrc --fix <file>` を実行し、自動修正または文面修正。                                                        |
| cspell                              | `Unknown word (xxx)`                               | 用語が正しいなら `config/lint/.cspell.json` の `words` に追記。誤字であれば本文を修正。                                                         |
| prettier                            | `Checking formatting... [warn] <file>`             | `npx prettier --config config/lint/.prettierrc --write <file>` または `npm run format` を実行して整形し、再度 `pre-commit run --all-files`。     |

> どのフックも基本的には「指摘箇所の修正 → 同じフックを再実行」で解消できます。  
> それでも失敗する場合は `pre-commit run <hook-id> --all-files` のように個別実行して詳細ログを確認してください。

## 6. 動作確認シナリオ

1. `requirements/sample_requirement.md`
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
