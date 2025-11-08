# AGENTS.md

チーム合意事項と、このリポジトリを触るエージェント向けの運用ルールをまとめています。作業前に必ず確認してください。

## 1. Lint/Format運用の固定ルール

- `README.md` は常に lint / textlint / cspell / prettier の対象 **外** にする。`config/lint/.markdownlintignore` / `.textlintignore` / `.cspell.json` / `.pre-commit-config.yaml` に同じパターンを設定済みであり、今後も維持すること。
- `requirements/` 配下は `requirements/.markdownlint.jsonc` / `.textlintrc` でローカルルールを上書きできるが、`sample_requirement.md` は常にチェック対象に残す。
- `soft_requirements/` も同様に `.markdownlint.jsonc` / `.textlintrc` を持ち、ソフトウェア要件特有のルールを適用している。
- 新しい Markdown を生成・投入する際は、**必ず pre-commit のルール（markdownlint
  / textlint / cspell / prettier）を満たした状態で提出する**。自動生成でも
  `npx prettier --write`
  などを通し、フックで落ちないことを確認してからコミットする。
- 参照資料 `docs/01_markdown_quality_assurance_combined.md` /
  `docs/02_markdown_quality_assurance_formatted_guide.md` /
  `docs/CHECK_REPORT.md` / `docs/AGENTS.md` は既定で無視対象。新たに資料を増やして除外したい場合は、`config/lint/.markdownlintignore` / `.textlintignore` / `.cspell.json` を更新する。

## 2. pre-commit / CLI 実行時の注意

- `pre-commit` は
  `PATH="/Users/yuhei/Library/Python/3.9/bin:$PATH" pre-commit ...`
  のように PATH を通して実行する。権限制限のある環境では `SKIP=... git commit`
  などの例外運用が必要になるため、理由を必ず記録する。
- `pre-commit` の Node 系フックは `pass_filenames: true`
  で指定ファイルのみを対象にする設計。`pre-commit run --files <path>`
  を使って確認すること。

## 3. Markdown整形ポリシー

- Prettier 設定（`.prettierrc`: `tabWidth:2`, `proseWrap:"always"`
  など）に厳密に従う。Lint失敗の大半は整形漏れなので、コミット前に
  `npx prettier --write <file>` を実行する。
- `README.md`
  は整形しなくてもCIが落ちないが、内容更新する場合は人手で整合性をとる。必要なら一時的に lint 対象へ戻して整形→再び ignore 設定に戻す。

## 4. ドキュメントメンテ

- `docs/CHECK_REPORT.md` に最新の `markdownlint` / `textlint` / `cspell`
  実行結果をMarkdownで記録する運用。更新時は日付も書き換える。
- `README.md` は品質ライン全体の手順書として扱うが、Lint対象外とする制約を守る。
- `config/lint/prh.yml` で表記ゆれ（ドライバ/センサなど）を統制している。辞書追加時は
  `config/lint/.textlintrc` の `prh.rulePaths` が指すファイルのみを編集する。

## 5. 追加で守るべきこと

- `node_modules/`
  以下を直接編集・参照しない。Lintエラーは ignore 設定で回避する。
- 既存の設定を変更する場合は README と AGENTS.md に理由と手順を反映し、チーム内で共有する。
- ここに書かれていない例外運用が必要になったら、必ず AGENTS.md を更新して次のエージェントが迷わないようにする。
