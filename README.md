# basic_cc_project — Claude Code 活用プロジェクトのひな型

Claude Code をすぐ使い始められる**プロジェクトのひな型**。共有資産（skills / subagents /
rules / settings 等）を持つ `.claude/` は、共有資産リポジトリ `basic_dot_claude` を
**git submodule** として取り込む。

## 主な使い方

### A. 既存プロジェクトへ適用（主用途）
このリポジトリ直下の `.claude/`（中身）・`README.md`・`*.sample` を、対象プロジェクトへ
**コピー**する（`.git`/`.gitmodules` は持ち込まない＝中身だけをコピー）。
`CLAUDE.md` はひな型段階では置かないので、コピー先で新規作成する
（`CLAUDE.md.sample` を用意してあれば改名して使う）。

### B. このリポジトリを単体で clone する
```
git clone --recurse-submodules <url>
# clone 済みなら:
git submodule update --init --remote
```
`.claude/` が空のときは submodule 未初期化（`--add-dir` してもエラーなく何も載らない。
`/skills`・`/memory` で確認）。

### C. 本リポジトリ配下の `.claude` を `--add-dir` で利用する
別の作業リポジトリから:
```
claude --add-dir <このリポジトリのパス>
```
- settings を効かせるなら: `--settings <このリポジトリのパス>/.claude/settings.json`
- `CLAUDE.md`/`rules` を読むなら: 環境変数 `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`

## 共有資産の最新化（refresh）
共有本体（`basic_dot_claude`）の更新を取り込む。**取得のみ（push しない）**ので、
日次の作業開始時にランチャー等で自動実行してよい:
```
git pull --ff-only
git submodule update --init --remote   # .claude を最新へ（pull --ff-only 相当・init も兼ねる）
```
> 共有本体への**公開（commit/push）**は供給元 `base-dev-kit-for-cc` の `publish-share` で行う
> 別操作。refresh では行わない。

## 構成メモ
- `.claude/` … `basic_dot_claude`（共有資産本体）の submodule。`.gitmodules` で `branch = main`
  を追従する（公開基準ブランチ）。
- `CLAUDE.md.sample` / `CLAUDE.local.md.sample`（任意） … コピー先で改名して使うひな型
  （`.sample` のままなら memory として非ロード）。
