---
title: "Student Mode — OpenAI Codex CLI 併用ガイド (案 E)"
type: guide
status: active
date_created: 2026-04-21
date_modified: 2026-04-21
project: crlBrain
tags:
  - topic/crlBrain
  - topic/student-mode
  - method/codex-cli
related:
  - docs/student-mode-gemini.md
  - docs/student-mode-copilot-cli.md
  - sessions/discussions/crlBrain/20260421_1112_student-mode-gemini-pilot-plan.md
---

# Student Mode — OpenAI Codex CLI 併用ガイド (案 E)

> [!note] 初めて crlBrain に触れる学生は、先に [学生向け導入マニュアル](./student-onboarding.md) を読むことを推奨します。前提知識（ターミナル・git・SSH 鍵・Node.js）は [学生向け前提ガイド](./student-prereq.md) を参照してください。
> 本ガイドは **Codex CLI 固有のセットアップ詳細** に焦点を当てた技術リファレンスです。

> [!abstract]
> ChatGPT Plus サブスクリプション（$20/月）を保有する学生向けに、OpenAI Codex CLI 経由で crlBrain の主要機能を利用するための手順書。本ガイドは **案 E のパイロット検証前の試作版** であり、並行パイロット（案 A + 案 D + 案 E）で 3 案の優劣を比較評価してから正式版に昇格する。
>
> **案 A/D と異なり、案 E は恒久的な無料プランが存在しない**。ChatGPT Free での Codex 利用は 2026-04 現在「期間限定オファー」であるため、事前登録の前提として **ChatGPT Plus 前提で設計** する。

---

## このドキュメント自体の主張分類メモ

本ドキュメント内の主張は以下の階層を遵守している。

- **[A] 学術的事実 / 公式仕様** — OpenAI Developers Docs・公式 Changelog を一次ソースとして参照した記述
- **[C] 尤もらしい見解** — 公式情報から妥当に導かれる推測
- **[D] 直感的仮説** — crlBrain との整合性に関する未検証の仮説

本ガイド全体は Claude Code 前提の完全版 crlBrain から派生した **軽量運用モードの提案** であり、案 E の実運用可能性は並行パイロット（`sessions/discussions/crlBrain/20260421_1112_student-mode-gemini-pilot-plan.md`）で [B] 統計的根拠のある結論に昇格するかが判定される。

---

## 1. 概要 — 案 E の位置付け

### 1.1 なぜ案 E が候補になったか

- **AGENTS.md 規約が正式サポート** されており、crlBrain の `CLAUDE.md` を AGENTS.md として再配置することで規約が自然に共有可能 [A]
- **MCP (Model Context Protocol) 正式サポート** — `~/.codex/config.toml` または `.codex/config.toml` で設定し、crlBrain が将来 MCP 統合を進める際の共通基盤となる [A]
- GPT-5.3-Codex など Codex 系専用モデルへのアクセスが得られる [A]
- ChatGPT Plus 加入済み学生にとっては既存サブスクリプションの範囲内で利用可能 [C]

### 1.2 案 A / 案 D との違い

| 観点 | 案 A (Gemini CLI) | 案 D (Copilot CLI) | **案 E (Codex CLI)** |
|------|------------------|-------------------|---------------------|
| 利用料金 | 個人 Google 無料（1,000 req/日）| Student Pack で無料 | **ChatGPT Plus $20/月 必須**（Free は期間限定・非推奨）|
| 使用モデル | Gemini 2.x | GPT 系（Claude/GPT-5.4 不可）[A] | **GPT-5.4 / GPT-5.4-mini / GPT-5.3-Codex** [A] |
| 認証 | 個人 Google アカウント | GitHub アカウント（Student Pack）| **OpenAI / ChatGPT アカウント** |
| skills 対応 | あり（slash commands）| あり | **△**（slash commands 相当は Codex 側で別機構）|
| 規約メモリ | `GEMINI.md` | `COPILOT.md` / `.github/copilot-instructions.md` | **`AGENTS.md`**（precedence: `~/.codex/AGENTS.md` → project root → CWD）[A] |
| MCP サポート | ◯ | ◯ | **◎**（公式 Docs に詳細仕様あり）[A] |
| Hooks 互換性 | △（skills 内 pre/post 相当）| ❌ | **❌**（`notify` による `agent-turn-complete` イベントのみ）[A] |
| crlBrain 互換性 | △（モデル差異あり） | △（Claude 不可の影響大）| **△**（AGENTS.md で規約共有可、ただしペルソナの GPT 動作は未検証）[D] |
| Windows サポート | ◯ native | ◯ native | **△ experimental / WSL2 推奨** [A] |

### 1.3 対象読者（重要）

> [!warning] 本ガイドは以下の条件を **すべて** 満たす学生のみを対象とします
>
> 1. **ChatGPT Plus / Pro / Business / Edu / Enterprise のいずれかに加入済** — または研究室が費用負担する方針が確定している
> 2. ChatGPT Free + Codex の「期間限定オファー」では **PASS 基準 M1 を事前登録できない** ため、本ガイドの利用対象外とする（[B] 昇格不能）
> 3. Windows ユーザーは **WSL2 環境の準備** が可能であること（native Windows 利用は experimental）
> 4. 試験的な運用を受容できる（挙動差異によるペルソナ品質低下の可能性を理解）
>
> ChatGPT Plus 未加入の学生は **案 A (Gemini CLI)** または **案 D (Copilot CLI)** を推奨します。

---

## 2. 7 層アーキテクチャのカバー範囲

| Layer | 案 E でのカバー状況 |
|-------|-------------------|
| 1. Repo Memory (CLAUDE.md) | ✅（`AGENTS.md` として再配置することで正式サポート）[A] |
| 2. Expert Modes (.claude/skills/) | △（slash commands 相当は Codex 側で別機構が必要、手動プロンプト貼付で代替）|
| 3. Guardrails (`.claude/settings.json` + `scripts/hooks/`) | ❌（Codex ネイティブの hook は `notify` agent-turn-complete のみ。commit-msg 等のチェックは `scripts/hooks/install-git-hooks.sh` で git hook を別途導入する） |
| 4. Progressive Context (docs/) | ✅ |
| 5. Local Context | △（各ディレクトリの CLAUDE.md を AGENTS.md にコピーまたはリンクする運用が必要）|
| 6. Context Management | △（Codex 側のコンテキスト管理に依存）|
| 7. Memory | ❌（手動で MEMORY.md 更新）|

---

## 3. 前提条件

### 3.1 ChatGPT サブスクリプション

> [!important]
> **ChatGPT Plus ($20/月) 以上のサブスクリプション** が推奨されます。
>
> 1. https://chatgpt.com/codex/pricing/ で現在の料金とプランを確認
> 2. 加入後、Codex CLI 内の `Sign in with ChatGPT` フローで認証
> 3. プラン情報は `/status` 系コマンドで確認可能

**学生が個人で加入できない場合の選択肢**:

| 選択肢 | 内容 | 留意点 |
|--------|------|--------|
| 個人負担 | 学生が自費で加入 | 任意。本ガイドは加入を強制しない |
| **案 A / 案 D へ移行** | 本ガイドではなく案 A/D を使う | 無料で crlBrain を試せる（推奨）|

> [!important] 2026-04-21 パイロットの方針確定事項
> 並行パイロット計画（`sessions/discussions/crlBrain/20260421_1112_student-mode-gemini-pilot-plan.md`）では、**Track E は ChatGPT Plus 加入済学生のみを対象** とする方針が確定しています。
>
> - 研究室費負担は行わない
> - パイロット参加のための新規加入も求めない
> - 加入済学生が研究室内に存在しない場合、Track E は実施せず 2-track (A + D) 運用に縮小
>
> 本ガイド自体は一般利用の技術ガイドとして残しますが、パイロット参加条件は上記の通り限定されます。

### 3.2 ChatGPT Free での利用可否（注意）

> [!warning] ChatGPT Free + Codex は「期間限定」
> 2026-04-21 現在、ChatGPT Free プランでも Codex CLI を一時的に使えるが、**具体的な req 上限は公式に公開されておらず、オファー終了時期も未定**。本ガイドでは Free プランでの利用を **非推奨** とし、パイロット対象から除外する。

### 3.3 システム要件

| 要件 | バージョン |
|------|---------|
| Node.js | **18.0.0 以上** [A]（npm 経由インストール時に必須）|
| Git | 2.x 以上 |
| OS | macOS 12+ / Linux（Ubuntu 20.04+）/ Windows 10+（WSL2 推奨）|
| Shell | Bash / Zsh / PowerShell |

> 案 D (Copilot CLI) の Node.js 22+ と比べて案 E は 18+ で緩い。Codex 本体は Rust 実装のため、バイナリ配布版を使えば Node.js すら不要。

確認コマンド:

```bash
node --version   # v18.x.x 以上
git --version
```

---

## 4. セットアップ手順

### 4.1 Codex CLI のインストール

#### macOS

```bash
# 方法 A: Homebrew（推奨）
brew install --cask codex

# 方法 B: npm（Node.js 18+ 事前インストール）
npm install -g @openai/codex

# 方法 C: バイナリ直接ダウンロード
#   https://github.com/openai/codex/releases から最新の darwin バイナリを取得
```

#### Linux（Ubuntu / Debian / Arch 等）

```bash
# 方法 A: Homebrew (Linuxbrew)
brew install --cask codex

# 方法 B: npm（Node.js 18+ 事前インストール）
#   Ubuntu/Debian の場合
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
npm install -g @openai/codex

# 方法 C: バイナリ直接ダウンロード
#   https://github.com/openai/codex/releases から linux-x86_64 / linux-arm64 を取得
tar -xzf codex-*.tar.gz
sudo mv codex /usr/local/bin/
```

#### Windows（experimental — WSL2 強く推奨）

**推奨: WSL2 で Linux 手順を実行**

```powershell
# PowerShell を管理者権限で起動
wsl --install -d Ubuntu
# 再起動後、Ubuntu シェルで上記の Linux 手順を実行
```

**WSL2 を使わず native Windows を試す場合（experimental）**:

```powershell
# 方法 A: Node.js LTS + npm
#   https://nodejs.org/ から Node.js 18+ LTS (.msi) をインストール後
npm install -g @openai/codex

# 方法 B: バイナリ直接ダウンロード
#   https://github.com/openai/codex/releases から windows-x86_64 を取得
```

#### インストール確認（全 OS 共通）

```bash
codex --version
```

### 4.2 OpenAI / ChatGPT 認証

```bash
codex
# インタラクティブ TUI が起動する
```

TUI 内で **Sign in with ChatGPT** を選択すると、ブラウザが開いて OAuth フローが完了します。

プラン情報の確認:

```
/status
```

### 4.3 利用枠の確認

ChatGPT Plus（2026-04-21 現在）[A]:

- **5 時間あたりのローカルメッセージ制限**:
  - GPT-5.4: 400〜2,000 local messages / 5h
  - GPT-5.4-mini: 1,200〜7,000 local messages / 5h
  - GPT-5.3-Codex: 600〜3,000 local messages + 200〜1,200 cloud tasks / 5h
- **Code Reviews**: 400〜1,000 / 5h
- **週次上限**あり（具体値は https://chatgpt.com/codex/pricing/ 参照）
- **2026-05-31 まで 2x promo** — パイロット期間中は追加余裕あり

> パイロット M1 閾値設計では **Plus GPT-5.3-Codex の下限 600 local messages / 5h** を基準として「各セッションで ≤ 400 local messages」を PASS 閾値とする（§pilot plan 参照）。

### 4.4 crlBrain リポジトリのクローンと AGENTS.md 準備

```bash
git clone <crlBrain リポジトリ URL>
cd crlBrain
```

> [!note] AGENTS.md の準備
> crlBrain は 2026-04-21 時点で `AGENTS.md` を整備していない。Codex CLI は `CLAUDE.md` を自動読込しないため、以下のいずれかの対応が必要:
>
> 1. `CLAUDE.md` を `AGENTS.md` として symlink: `ln -s CLAUDE.md AGENTS.md`
> 2. `AGENTS.md` を新規作成し `CLAUDE.md` と同等の内容を記載（Codex 向けの挙動調整を加える）
> 3. パイロット参加学生が `AGENTS.md` 初稿を作成し、crlBrain への貢献 PR として提出
>
> 推奨は **3 番**（学生の crlBrain 貢献機会を兼ねる）。

### 4.5 Codex CLI の起動

```bash
codex
```

プロジェクトディレクトリで起動すると、precedence に従い AGENTS.md が自動読込される:

1. `~/.codex/AGENTS.override.md` → `~/.codex/AGENTS.md`
2. プロジェクト root の `AGENTS.override.md` → `AGENTS.md`
3. CWD までの各ディレクトリの `AGENTS.override.md` → `AGENTS.md`

> [!tip] ペルソナごとの override
> ディレクトリ単位の `AGENTS.override.md` を活用すると、「このディレクトリでは Dr.FEP として振る舞う」等のペルソナ切り替えを Codex ネイティブ機能で実現可能。skills の代替策として検討価値がある [D]。

### 4.6 MCP の設定（任意）

crlBrain 将来の MCP 統合に備え、設定例:

```toml
# ~/.codex/config.toml
[mcp_servers.crlbrain-library]
command = "node"
args = ["~/crlbrain-mcp/library-server.js"]
```

詳細は https://developers.openai.com/codex/mcp 参照。

---

## 5. 軽量ペルソナセット

案 A / 案 D と同一構成（Prof.Igarashi + Dr.FEP + Dr.AI の 3 名）を推奨します。詳細は `docs/student-mode-gemini.md` 第 5 節を参照。

### 5.1 Codex CLI 固有の注意

- **モデル差異** [D]: Claude Opus を前提に調整されたペルソナ挙動が GPT-5.4 / GPT-5.3-Codex で再現される保証はない
- **Codex は「コーディングエージェント」寄り** — crlBrain の「議論・計画策定・議事録作成」タスクが Codex のデフォルト動作と噛み合うかは要検証 [D]
- **ペルソナごとのサブエージェント**: `AGENTS.override.md` でディレクトリ単位の人格分離を試せる [D]

---

## 6. 手動チェックリスト（Hooks の代替）

案 D と同様、Codex CLI でも crlBrain の hooks は動作しない。手動チェックリストで代替する。詳細は `docs/student-mode-gemini.md` 第 6 節を参照。

**ポイント**:
- `[topic-id]` commit prefix（例: `[PHS]`, `[crlBrain]`）
- 主張 4 階層分類 [A/B/C/D] の明示（**crlBrain 憲法第 9 条・最優先原則**）
- **"literal" 禁止語** の使用回避
- `session/YYYYMMDD-{topic-id}-{slug}` ブランチ命名

### 6.1 Codex の notify hook 活用（任意）

Codex の `notify` 機能で `agent-turn-complete` イベントをトリガーに外部スクリプトを起動できる [A]。

```toml
# ~/.codex/config.toml
[notify]
command = ["/usr/local/bin/crlbrain-post-turn.sh"]
```

このスクリプトで `[topic-id]` prefix チェックなどの軽量ガードを実装可能（要実装）。crlBrain 側の Deliverables として案 E 成熟時に整備予定。

---

## 7. セッション議事録テンプレート

案 A / 案 D と同一。詳細は `docs/student-mode-gemini.md` 第 7 節を参照。

---

## 8. 既知の制約

### 8.1 ChatGPT Plus $20/月 が必須 [A]

- 恒久的な無料運用は不可能（ChatGPT Free の Codex 利用は期間限定オファー）
- 研究室費負担または学生個人負担の合意が前提
- 案 A (完全無料) / 案 D (Student Pack で無料) と比べた最大の弱点

### 8.2 Claude モデルが使えない [A]

案 D と同じく、案 E でも Claude Opus / Sonnet は利用不可。crlBrain は Claude 前提で調整されているため、以下の影響が予想される [D]:

- ペルソナの発言スタイルの差異
- 長文コンテキスト保持能力の差
- 4 階層分類の自発的遵守率の再現度

### 8.3 Hooks が動かない [A]

crlBrain の Claude Code hook（`.claude/settings.json` で宣言、`scripts/session-start-guard.sh` 等）と git hook（`scripts/hooks/`）のうち、Codex CLI が実行するのは Codex ネイティブの `notify` (`agent-turn-complete`) のみ。CC hook はカバーされない。git hook は `scripts/hooks/install-git-hooks.sh` で別途導入可能。以下は手動または git hook で補う:

- `commit-msg` hook による `[topic-id]` prefix 強制 → **Git hook で別途実装**
- `commit-msg` hook による "literal" 禁止語チェック → **Git hook で別途実装**
- `session-start-guard` による main ブランチ作業防止 → **Git pre-commit hook で別途実装**

### 8.4 skills の slash command 互換性

Codex CLI は `/login`, `/status` 等の slash command を持つが、crlBrain `.claude/skills/` の `Skill` tool とは別機構。skills 内のプロンプト本文を手動貼付して動作させる運用になる [C]。

### 8.5 Windows native は experimental [A]

OpenAI 公式が WSL2 を推奨。Windows ネイティブ環境でのパイロットは設計外とする。

### 8.6 MEMORY.md / Library の自動更新なし

Recorder が自動で MEMORY.md を更新する仕組みは Claude Code 固有。Codex CLI ではセッション終了時に学生が手動で追記する必要がある。

---

## 9. FAQ

### Q: ChatGPT Plus に加入していないが案 E を試したい

**A**: 以下のいずれかを検討してください:

1. **案 A (Gemini CLI) を使う** — 完全無料、新規学生も可
2. **案 D (Copilot CLI) を使う** — 既存 Student Pack 保持者のみ
3. **研究室費負担の相談** — Prof.Igarashi に相談（パイロット参加条件）
4. **ChatGPT Free + Codex の期間限定オファー** — パイロット対象外だが、個人学習には使える

### Q: Claude モデルを使いたい

**A**: 案 E では選択不可 [A]。以下が選択肢:

1. 研究室共通の Claude Code Max シート（案 C）を利用
2. Claude.ai Web Free で一時的に Claude と対話（skills 非対応）
3. 案 A (Gemini CLI) を使用

### Q: Gemini CLI / Copilot CLI と併用できるか

**A**: できます [C]。3 つのモデルで同じ議論を走らせて結果を比較するのは、**モデル横断ベンチマーク** として有用（パイロット X1〜X3）。ただし 1 セッション内で同時に走らせるのは認知負荷が高いため、セッション単位で使い分けることを推奨。

### Q: `AGENTS.md` が crlBrain に存在しないがどうすればいい

**A**: 2026-04-21 時点で `AGENTS.md` は整備されていません。以下のいずれかを実施:

1. **最小対応**: `CLAUDE.md` を AGENTS.md として symlink
   ```bash
   ln -s CLAUDE.md AGENTS.md
   ```
2. **推奨対応**: プロジェクトルートに `AGENTS.md` を新規作成し、パイロット参加者が GitHub PR として提出（副次的成果物）。CLAUDE.md の内容に加え、Codex 向けの挙動調整（GPT-5 系の特性・slash command 代替案など）を含める。

### Q: 5 時間制限を超えたらどうなる

**A**: Plus プランの 5h 制限を超えると、5h rolling window が回復するまで Codex が使えなくなります [A]。パイロットでは 1 セッションを 1 rolling window 内に収める設計。超過した場合は Session 完遂失敗としてカウント（M3 に反映）。

### Q: Windows native でパイロット参加できるか

**A**: 公式が experimental のため、**案 E パイロットでは WSL2 環境を必須** とします。WSL2 が使えない Windows 環境の学生は案 A / 案 D を推奨。

---

## 10. 並行検討: 他 CLI

Codex CLI（案 E）と並行して、以下の CLI でも crlBrain を利用できます:

| 案 | CLI | 対象 | 費用 | ガイド |
|----|-----|------|------|--------|
| 標準 | Claude Code | Pro/Max 加入者 | $20/月〜 | [student-mode-claude-code.md](./student-mode-claude-code.md) |
| 案 A | Gemini CLI | 全学生（新規可）| 無料 | [student-mode-gemini.md](./student-mode-gemini.md) |
| 案 D | Copilot CLI | Student Pack 既存保持者 | 無料 | [student-mode-copilot-cli.md](./student-mode-copilot-cli.md) |
| **案 E (本ガイド)** | **Codex CLI** | **ChatGPT Plus 加入者** | **$20/月** | **本ガイド** |

全 CLI の選択ガイドと比較表は [docs/student-onboarding.md § 3](./student-onboarding.md) を参照。

並行パイロット計画: [sessions/discussions/crlBrain/20260421_1112_student-mode-gemini-pilot-plan.md](../sessions/discussions/crlBrain/20260421_1112_student-mode-gemini-pilot-plan.md)

---

## 参照ドキュメント

| ドキュメント | 内容 |
|------------|------|
| `docs/student-onboarding.md` | 学生向け導入マニュアル（共通導入・最初に読むべし）|
| `docs/student-prereq.md` | 学生向け前提ガイド（ターミナル・git・SSH 鍵）|
| `docs/student-mode-claude-code.md` | Claude Code 版（完全版）ガイド |
| `docs/student-mode-gemini.md` | 案 A: Gemini CLI 併用ガイド |
| `docs/student-mode-copilot-cli.md` | 案 D: Copilot CLI 併用ガイド |
| `room-config.md` | 議論プロトコル・Tiered Loading の中核設定 |
| `docs/manual-quickstart.md` | crlBrain クイックスタートガイド（CC 環境）|
| `docs/manual-detailed.md` | セッション詳細仕様（フェーズ・品質ゲート）|
| `docs/constitution/constitution_claude.md` | 主張 4 階層分類の詳細・事例（第 9 条）|
| `agents/README.md` | 全 25 名ペルソナ一覧 |

## 外部参照

- [Codex CLI – OpenAI Developers](https://developers.openai.com/codex/cli)
- [Codex CLI Features](https://developers.openai.com/codex/cli/features)
- [Custom instructions with AGENTS.md](https://developers.openai.com/codex/guides/agents-md)
- [Model Context Protocol – Codex](https://developers.openai.com/codex/mcp)
- [Advanced Configuration – Codex](https://developers.openai.com/codex/config-advanced)
- [Codex Pricing](https://chatgpt.com/codex/pricing/)
- [Using Codex with your ChatGPT plan](https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan)
- [openai/codex GitHub リポジトリ](https://github.com/openai/codex)
- [@openai/codex – npm](https://www.npmjs.com/package/@openai/codex)

---

## 更新履歴

| 日付 | 変更者 | 変更内容 |
|------|------|---------|
| 2026-04-21 | Claude（session/20260421-crlbrain-karpathy-skills-review）| 初版作成（案 E 試作版。案 A + 案 D + 案 E 並行パイロットの一環）|
| 2026-04-21 | Claude (session/20260421-crlbrain-student-onboarding-manual) | 冒頭ノート + § 10 並行検討を 4 CLI 対応に更新 |
