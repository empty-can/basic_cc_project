# basic_cc_project — Claude Code 活用プロジェクトのひな型

Claude Code をすぐ使い始められる**プロジェクトのひな型**。共有資産（skills / subagents /
rules / settings 等）を持つ `.claude/` は、共有資産リポジトリ `basic_dot_claude` を
**git submodule** として取り込む。

## 主な使い方

### A. 既存プロジェクトへ適用（主用途）
このリポジトリ直下の次を、対象プロジェクトへ**コピー**する（`.git` / `.gitmodules` は
持ち込まない＝中身だけをコピー）:

| コピーするもの | 無いとどうなるか |
|---|---|
| `.claude/`（**中身**） | 共有資産（skills / subagents / rules / settings / hooks / ランチャーの部品）が届かない |
| `start_claude_code.sh` / `start_claude_code.ps1` | **ランチャー本体**。無いと `.claude/launcher/` は部品だけ届いて起動する手段が無い |
| `.gitattributes` | `core.autocrlf=true`（Git for Windows 既定）の clone で `.sh` が CRLF 化し、`set -euo pipefail` 下の起動が `$'\r'` で落ちる |
| `CLAUDE.md.sample` | プロジェクト固有 `CLAUDE.md` のひな型。**`CLAUDE.md` にリネームして使う** |

`CLAUDE.md` はひな型段階では置かない（`.sample` のままなら memory として非ロード）。
コピー後に `CLAUDE.md.sample` を `CLAUDE.md` へ改名し、プロジェクト概要・開発コマンド・
MCP ポリシーを記入する。

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

## 起動ランチャー

環境変数と起動オプションを統制した状態で `claude` を起動する。

```bash
./start_claude_code.sh            # bash（要 bash 4.4+。macOS 同梱の 3.2 は不可）
.\start_claude_code.ps1           # PowerShell（5.1 / 7 両対応）
```

追加の引数はそのまま `claude` へ委譲される（例: `./start_claude_code.sh --model opus`）。
`.claude/` が submodule として未初期化だとランチャーは**理由を明示して停止する**
（`git submodule update --init --recursive` を促す）。

利用者が値を設定するファイルは**テンプレートからコピーして作る**（いずれも `.gitignore` 済みの個人実体）:

| 作るもの | テンプレート | 用途 |
|---|---|---|
| `.claude/custom.env` | `.claude/launcher/custom.env.template` | 環境変数（統制不要・非機微） |
| `.claude/option-settings.{sh,ps1}` | `.claude/launcher/option-settings.{sh,ps1}.template` | 起動オプション |

> **⚠ 置き場所に注意**: コピー先は **`.claude/` 直下**であって、テンプレートの隣（`.claude/launcher/`）ではない。
> 誤配置すると読み込まれない（ランチャーが警告する）。

> **⛔ 機微情報（API キー / 認証トークン / パスワード / 組織外秘の URL）は、`custom.env` を含む
> いかなるランチャーファイルにも書かない。** OS の環境変数として設定すること。`setup-environment` は
> 既知の機微キーを `custom.env` 内に見つけても**読み込まずに警告する**。

> **⚠ ここで設定した env が届くのは foreground 起動の `claude` だけ**。background / agent-view
> セッションは OS env・ディレクトリ設定から構成を読むため、そちらにも効かせたい値は OS 環境変数
> または `settings.json` で設定する。

チームで揃えたい値は、利用者が触らない場所に置く ―― env は `.claude/launcher/setup-environment.{sh,ps1}`、
起動オプションは `start_claude_code.{sh,ps1}` の `TEAM_OPTS` / `$TeamOpts`。ただしこれは
**既定値を揃える仕組みであって強制力はない**（起動時引数で上書きできる）。強制したい統制は
`settings.json` / managed settings 側で行う。

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

```
.claude/                    # basic_dot_claude（共有資産本体）の submodule
start_claude_code.sh        # 起動ランチャー本体（bash）
start_claude_code.ps1       # 起動ランチャー本体（PowerShell・UTF-8 BOM + CRLF）
.gitattributes              # 改行の固定（既定 LF ＋ *.ps1 / *.bat だけ CRLF）
CLAUDE.md.sample            # プロジェクト固有 CLAUDE.md のひな型（改名して使う）
.gitmodules
README.md                   # ← このファイル（リポ固有情報はここに隔離する）
```

- `.claude/` … `basic_dot_claude`（共有資産本体）の submodule。`.gitmodules` で `branch = main`
  を追従する（公開基準ブランチ）。**供給元は `base-dev-kit-for-cc`**（そこから `publish-share` で
  `basic_dot_claude` へ配布される）。
- `.gitattributes` … **`.claude/` には効かない**。submodule は別リポジトリなので、その中の改行は
  submodule 自身の `.gitattributes`（`basic_dot_claude` のルート）に従う。
- `CLAUDE.md.sample` / `CLAUDE.local.md.sample`（任意） … コピー先で改名して使うひな型
  （`.sample` のままなら memory として非ロード）。
- `start_claude_code.*` と `.gitattributes` は **submodule では運べない**（`.claude/` の外にあるため）。
  供給元のルートから**このリポジトリへ直接コミット**して同期する。
