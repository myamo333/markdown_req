# Markdown要件定義書 品質保証ライン構築手順書
*開発環境構築ガイドライン*

---

## 1. 前提条件

### 🎯 目的
Markdown形式で要件定義書を記述し、**Lint／Prettier／pre-commit／CI** によって  
形式・文体・用語統一を自動化する再現性の高い環境を構築する。

### 👥 対象者
- 要件書をMarkdownで作成する開発チームメンバー  
- チームで品質統一を図るドキュメント管理者  
- Lintルール／CI整備担当者  

### 💻 システム要件

| 項目 | 推奨バージョン | 備考 |
|------|----------------|------|
| OS | Windows 11 / macOS 14 / Ubuntu 22 | いずれも可 |
| Node.js | 18以上 | npm利用のため必須 |
| Git | 2.30以上 | pre-commitフック動作用 |
| Python | 3.8以上 | pre-commit実行用 |
| VS Code | 最新版 | Lint拡張機能利用のため |

---

## 2. 基本環境のインストール

### 🎯 目的
Lintや自動整形ツールが動作する開発基盤を構築する。

### ⚙️ 操作手順

#### macOS / Linux
```bash
# Node.js / npm
brew install node

# Git
brew install git

# Python
brew install python3

# VS Code
open https://code.visualstudio.com/
```

#### Windows（PowerShell）
```powershell
winget install -e --id OpenJS.NodeJS.LTS
winget install -e --id Git.Git
winget install -e --id Python.Python.3.11
winget install -e --id Microsoft.VisualStudioCode
```

### 🔍 確認方法
```bash
node -v
npm -v
git --version
python --version
code --version
```

### 💡 補足
- Node.jsはLTS版を選択。  
- VS Codeは最新拡張機能を利用できるバージョンを推奨。

---

## 3. プロジェクト初期化

### 🎯 目的
共通ルールを共有できる開発リポジトリを初期化する。

### ⚙️ 操作手順
```bash
mkdir markdown-requirements
cd markdown-requirements
git init
npm init -y
npm install --save-dev markdownlint-cli textlint cspell prettier
```

### 🔍 確認方法
```bash
ls node_modules | grep markdownlint
```

### 💡 補足
`--save-dev` により、開発依存として登録。  
他メンバーも `npm ci` で同環境を再現可能。

---

## 4. Lint設定ファイル作成

### 🎯 目的
Markdown構文、文体、用語統一を自動で検出・修正できるよう設定。

### ⚙️ 操作手順
```bash
touch .markdownlint.jsonc .textlintrc .cspell.json prh.yml
```

### 🧩 各ファイル設定例

#### `.markdownlint.jsonc`
```jsonc
{
  "MD003": { "style": "atx" },
  "MD004": { "style": "dash" },
  "MD007": { "indent": 2 },
  "MD013": false,
  "MD029": { "style": "ordered" },
  "MD033": false,
  "MD041": false
}
```
**ポイント:**  
Markdown構文統一（見出し／箇条書き／改行）。  
誤検出の多いルールは無効化して使いやすく調整。

---

#### `.textlintrc`
```json
{
  "rules": {
    "preset-japanese": true,
    "max-ten": { "max": 3 },
    "no-mix-dearu-desumasu": true
  }
}
```
**ポイント:**  
文体統一（「である調」／「です・ます調」の混在防止）、冗長文や句読点の過多を自動検出。

---

#### `.cspell.json`
```json
{
  "version": "0.2",
  "language": "ja",
  "words": ["AEB", "CAN", "ECU", "PCS", "reqid"],
  "ignorePaths": ["node_modules"]
}
```
**ポイント:**  
信号名・略語のスペルミス防止。  
頻出専門語を辞書登録。

---

#### `prh.yml`
```yaml
rules:
  - expected: ドライバ
    patterns: ドライバー
  - expected: センサ
    patterns: センサー
```
**ポイント:**  
表記ゆれを防止。  
組織用語集と同期して管理。

---

### 🔍 確認方法
```bash
npx markdownlint '**/*.md'
npx textlint '**/*.md'
npx cspell '**/*.md'
```

### 💡 補足
- 各ファイルはプロジェクトルート直下に配置。  
- コメント可の `.jsonc` 形式を採用し説明追記も可能。

---

## 5. 自動整形とVS Code設定

### 🎯 目的
保存時の自動整形により、インデント・改行・表整形を統一。

### ⚙️ 操作手順

#### `.prettierrc`
```json
{
  "tabWidth": 2,
  "useTabs": false,
  "singleQuote": true,
  "trailingComma": "none",
  "proseWrap": "always"
}
```

#### `.vscode/settings.json`
```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  }
}
```

### 🔍 確認方法
Markdownを保存し、改行・スペースが自動整形されるか確認。

### 💡 補足
- `.vscode/`をチーム共有して環境差を防止。  
- フォーマット競合がある場合はPrettier拡張を優先設定。

---

## 6. pre-commit導入

### 🎯 目的
コミット前にLint／整形を自動実行し、エラーをリポジトリに残さない。

### ⚙️ 操作手順
```bash
pip install pre-commit
touch .pre-commit-config.yaml
```

#### `.pre-commit-config.yaml`
```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer

  - repo: local
    hooks:
      - id: markdownlint
        entry: npx markdownlint-cli '**/*.md'
        language: node
        pass_filenames: false
      - id: textlint
        entry: npx textlint '**/*.md'
        language: node
        pass_filenames: false
      - id: cspell
        entry: npx cspell '**/*.md'
        language: node
        pass_filenames: false
      - id: prettier
        entry: npx prettier --check '**/*.md'
        language: node
        pass_filenames: false
```

```bash
pre-commit install
```

### 🔍 確認方法
```bash
git commit -m "test"
```
→ エラーがあればコミット中断。修正後再実行。

### 💡 補足
- 差分ファイルのみチェックする場合は `pass_filenames: true` に変更可能。  
- Node実行環境がないサーバではCIで再検査を行う。

---

## 7. CI連携（GitHub Actions）

### 🎯 目的
Pull Request作成時にLint／整形チェックを自動実行し、品質を維持。

### ⚙️ 操作手順
```bash
mkdir -p .github/workflows
touch .github/workflows/lint-check.yml
```

#### `.github/workflows/lint-check.yml`
```yaml
name: Lint Check
on:
  pull_request:
    branches: [ main ]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 20

      - run: npm ci || npm install
      - run: npx markdownlint '**/*.md'
      - run: npx textlint '**/*.md'
      - run: npx cspell '**/*.md'
```

### 🔍 確認方法
1. Pull Requestを作成。  
2. 「Lint Check」ジョブが自動実行される。  
3. 成功時のみマージ可能に設定（保護ブランチ推奨）。

### 💡 補足
- CIで検出されたエラー内容はGitHubのPR画面上に表示。  
- ルール変更後は再実行で整合性を確認。

---

## 8. 動作確認

### 🎯 目的
構築した環境が一貫して動作しているかを検証する。

### ⚙️ 操作手順
```bash
echo "# 第1章 テスト要件" > test.md
git add test.md
git commit -m "Add sample markdown"
git push origin feature/test
```

### 🔍 確認項目
- VS Codeでリアルタイム警告が表示される  
- 保存時にPrettier整形が動作する  
- コミット時にpre-commitが実行される  
- PR作成時にCIが動作する  

---

## 9. トラブル対応（よくある事例）

| 症状 | 原因 | 対処方法 |
|------|------|----------|
| `npx`コマンドが見つからない | Node.js未導入 | `npm install -g npx` |
| textlint警告が多すぎる | ルール過剰 | `.textlintrc`で無効化 or 緩和 |
| CIが失敗 | YAMLインデント不正 | `yamllint .github/workflows` で検証 |
| 保存時に整形されない | VS Code拡張競合 | Prettierをデフォルトフォーマッタに設定 |

---

## 10. 構築後チェックリスト

| 項目 | 確認結果 |
|------|-----------|
| Node.js / npm / Git / Python / VS Codeが動作 | ✅ |
| Lint設定ファイル4種が存在 | ✅ |
| 保存時に自動整形が動作 | ✅ |
| pre-commitでLintチェックが実行 | ✅ |
| Pull Request時にCIが起動 | ✅ |
| 用語辞書`prh.yml`が参照される | ✅ |

---

## 11. 導入後の状態

- **執筆中：** VS Code拡張による波下線警告  
- **保存時：** Prettierによる自動整形  
- **コミット時：** pre-commitで自動Lintチェック  
- **PR時：** GitHub Actionsで再検査  
- **マージ前：** 品質統一が担保される  

---

## 12. まとめ

この手順により、「誰が書いても同じ品質のMarkdown要件書」を保証。

- 書けば整う（Lint＋整形）  
- 保存で揃う（Prettier）  
- コミットで守る（pre-commit）  
- マージで検証（CI）  

> **結果として、要件定義書が常に自動で正しい状態に保たれる開発基盤が完成します。**
