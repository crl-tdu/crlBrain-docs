---
title: "Student Mode — GitHub Copilot CLI 併用ガイド (案 D)"
type: guide
status: active
date_created: 2026-04-21
date_modified: 2026-04-21
project: crlBrain
tags:
  - topic/crlBrain
  - topic/student-mode
  - method/copilot-cli
related:
  - docs/student-mode-gemini.md
  - sessions/discussions/crlBrain/20260421_1112_student-mode-gemini-pilot-plan.md
---

# Student Mode — GitHub Copilot CLI 併用ガイド (案 D)

> [!note] 初めて crlBrain に触れる学生は、先に [学生向け導入マニュアル](./student-onboarding.md) を読むことを推奨します。前提知識（ターミナル・git・SSH 鍵・Node.js）は [学生向け前提ガイド](./student-prereq.md) を参照してください。
> 本ガイドは **Copilot CLI 固有のセットアップ詳細** に焦点を当てた技術リファレンスです。

> [!abstract]
> GitHub Student Developer Pack を既に保有している学生向けに、GitHub Copilot CLI 経由で crlBrain の主要機能を利用するための手順書。本ガイドは **案 D のパイロット検証前の試作版** であり、並行パイロットで案 A (Gemini CLI) との優劣を比較評価してから正式版に昇格する。

---

## このドキュメント自体の主張分類メモ

本ドキュメント内の主張は以下の階層を遵守している。

- **[A] 学術的事実 / 公式仕様** — GitHub Docs・公式 Changelog を一次ソースとして参照した記述
- **[C] 尤もらしい見解** — 公式情報から妥当に導かれる推測
- **[D] 直感的仮説** — crlBrain との整合性に関する未検証の仮説

本ガイド全体は Claude Code 前提の完全版 crlBrain から派生した **軽量運用モードの提案** であり、案 D の実運用可能性は並行パイロット（`sessions/discussions/crlBrain/20260421_1112_student-mode-gemini-pilot-plan.md`）で [B] 統計的根拠のある結論に昇格するかが判定される。

---

## 1. 概要 — 案 D の位置付け

### 1.1 なぜ案 D が候補になったか

- 研究室の多くの学生が **GitHub Student Developer Pack** を既に保有している [C]
- Copilot CLI は 2026-02-25 に一般提供開始済み [A]
- `.claude/skills/` 資産が Copilot CLI の skills システムで再利用できる [A]（公式が agent skills 対応を明示）
- 既存 GitHub 認証をそのまま使えるため導入摩擦が低い [C]

### 1.2 Gemini CLI（案 A）との違い

| 観点 | 案 A (Gemini CLI) | 案 D (Copilot CLI) |
|------|------------------|-------------------|
| 利用料金 | 個人 Google アカウントで無料（1,000 req/日） | Student Pack で無料（premium request monthly allowance） |
| 使用モデル | Gemini 2.x | **GPT 系のみ**（Student Plan では Claude Opus/Sonnet 不可）[A] |
| 認証 | 個人 Google アカウント | GitHub アカウント（Student Pack 紐付け）|
| skills 対応 | あり | あり |
| crlBrain 互換性 | △（モデル差異あり、4 階層分類遵守は未検証） | △（Claude 前提の挙動と差異ありうる）[D] |
| 新規受付 | 可能 | **2026-04-20 以降、新規 Student Plan 登録は一時停止中** [A] |

### 1.3 対象読者（重要）

> [!warning] 本ガイドは以下の条件を **すべて** 満たす学生のみを対象とします
>
> 1. **GitHub Student Developer Pack が既に有効**（未申請の学生は 2026-04-20 以降、Copilot Student Plan の新規登録ができません）
> 2. **Copilot Free ではなく Copilot Student Plan（または Pro/Pro+）** のシートを保持している
> 3. 試験的な運用を受容できる（挙動差異によるペルソナ品質低下の可能性を理解）
>
> 新規学生は案 A (Gemini CLI) を推奨します。

---

## 2. 7 層アーキテクチャのカバー範囲

案 A (Gemini CLI) と同一です。詳細は `docs/student-mode-gemini.md` 第 2 節を参照。

| Layer | 案 D でのカバー状況 |
|-------|-------------------|
| 1. Repo Memory (CLAUDE.md) | △（`COPILOT.md` or プロンプトで手動読込）|
| 2. Expert Modes (.claude/skills/) | ✅（Copilot CLI は skills 対応）|
| 3. Guardrails (`.claude/settings.json` + `scripts/hooks/`) | ❌（Claude Code 専用の hook は未対応。手動チェックリストで代替。git hook は `scripts/hooks/install-git-hooks.sh` で別途導入可能） |
| 4. Progressive Context (docs/) | ✅ |
| 5. Local Context | △ |
| 6. Context Management | △ |
| 7. Memory | ❌（手動で MEMORY.md 更新）|

---

## 3. 前提条件

### 3.1 GitHub アカウントと Student Pack

> [!important]
> **GitHub Student Developer Pack が有効** である必要があります。
>
> 1. https://education.github.com/pack で Student Pack の状態を確認
> 2. 右上プロフィール → **Your plan** に "GitHub Copilot Student" と表示されていること
> 3. 認証は後述の `/login` で GitHub.com にリンクされる

Student Pack の有効性が切れている場合、Copilot CLI は起動できてもペルソナ実行時に利用枠エラーが発生します。

### 3.2 システム要件

| 要件 | バージョン |
|------|---------|
| Node.js | **22.0.0 以上** [A]（Copilot CLI は npm 経由の場合これが必須）|
| Git | 2.x 以上 |
| OS | macOS 12+ / Linux（Ubuntu 20.04+）/ Windows 10+ |
| Shell | Bash / Zsh / PowerShell |

> Gemini CLI は Node.js 20+ で動きますが、**Copilot CLI は 22+ が必要** な点に注意。

確認コマンド:

```bash
node --version   # v22.x.x 以上
git --version
```

---

## 4. セットアップ手順

### 4.1 Copilot CLI のインストール

#### macOS

```bash
# 方法 A: Homebrew（推奨）
brew install copilot-cli

# 方法 B: npm（Node.js 22+ が必要）
npm install -g @github/copilot

# 方法 C: インストールスクリプト
curl -fsSL https://gh.io/copilot-install | bash
```

#### Linux（Ubuntu / Debian / Arch 等）

```bash
# 方法 A: Homebrew (Linuxbrew) — 推奨
brew install copilot-cli

# 方法 B: インストールスクリプト
curl -fsSL https://gh.io/copilot-install | bash
#   システム全体にインストールする場合
curl -fsSL https://gh.io/copilot-install | sudo bash

# 方法 C: npm（Node.js 22+ 事前インストール）
#   Ubuntu/Debian の場合
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
npm install -g @github/copilot
```

#### Windows

PowerShell を管理者権限で開き、以下のいずれかを実行:

```powershell
# 方法 A: WinGet（推奨）
winget install GitHub.Copilot

# 方法 B: Chocolatey で Node.js をインストール後 npm
choco install nodejs-lts
npm install -g @github/copilot

# 方法 C: 公式インストーラー
#  1. https://nodejs.org/ から Node.js 22+ LTS (.msi) をダウンロード・インストール
#  2. PowerShell を再起動して以下を実行
npm install -g @github/copilot
```

> **WSL2 を使う場合**: WSL2 上で Linux 手順を推奨。Windows と WSL2 の PATH が混在しないよう注意。

#### インストール確認（全 OS 共通）

```bash
copilot --version
```

### 4.2 GitHub 認証（個人アカウント）

```bash
copilot
# インタラクティブ CLI が起動する
```

CLI 内で:

```
/login
```

ブラウザが開くので、**Student Pack が紐付いている GitHub アカウント** でログインしてください。Device Flow 認証が完了すると CLI 側に戻ります。[A]

認証成功の確認: CLI 側にインタラクティブプロンプトが戻り、以降 `/login` 再実行が求められなければ OK です。サインアウトには `/logout` を使います（Copilot CLI v1.0.34 時点で `/whoami` 系コマンドは存在しません）。[A]

### 4.3 利用枠の確認

Copilot Student Plan（2026-04-21 現在）[A]:

- **Unlimited code completions**
- **Premium models in Copilot Chat**（ただし Claude Opus/Sonnet・GPT-5.4 は自己選択不可）
- **Monthly allowance of premium requests**（具体値は GitHub 個人設定画面で確認）
- Copilot cloud agent access

> 最新情報: https://github.blog/changelog/2026-03-13-updates-to-github-copilot-for-students/

### 4.4 crlBrain リポジトリのクローン

```bash
git clone <crlBrain リポジトリ URL>
cd crlBrain
```

### 4.5 Copilot CLI の起動

```bash
copilot
```

プロジェクトディレクトリで起動すると、`COPILOT.md` または `.github/copilot-instructions.md`（存在する場合）が自動的に読み込まれます。

> [!note] crlBrain の Copilot 向け設定ファイルは 2026-04-21 時点では整備されていません。`CLAUDE.md` または `GEMINI.md` の内容を参考に `COPILOT.md` を作成する必要があります。これはパイロット結果の Deliverable として位置付けられています。

---

## 5. 軽量ペルソナセット

案 A と同一構成（Prof.Igarashi + Dr.FEP + Dr.AI の 3 名）を推奨します。詳細は `docs/student-mode-gemini.md` 第 5 節を参照。

### 5.1 Copilot CLI 固有の注意

- **モデル差異** [D]: Claude Opus を前提に調整されたペルソナ挙動が GPT 系モデルで再現される保証はない
- **context window**: GPT 系モデルは一般に Claude より context window が小さい。25 ペルソナ同時ロードは [D] 不可の見込み
- **skills の動作検証不足** [C]: `.claude/skills/` が Copilot CLI でも同一挙動で動くかは未検証

---

## 6. 手動チェックリスト（Hooks の代替）

案 A と同一の手動チェックリストを適用します。詳細は `docs/student-mode-gemini.md` 第 6 節を参照。

**ポイント**:
- `[topic-id]` commit prefix（例: `[PHS]`, `[crlBrain]`）
- 主張 4 階層分類 [A/B/C/D] の明示（**crlBrain 憲法第 9 条・最優先原則**）
- **"literal" 禁止語** の使用回避
- `session/YYYYMMDD-{topic-id}-{slug}` ブランチ命名

---

## 7. セッション議事録テンプレート

案 A と同一。詳細は `docs/student-mode-gemini.md` 第 7 節を参照。

---

## 8. 既知の制約

### 8.1 Claude モデルが使えない [A]

Student Plan では Claude Opus / Sonnet が自己選択不可。crlBrain は Claude 前提で調整されているため、以下の影響が予想される [D]:

- ペルソナの発言スタイルの差異
- 長文コンテキスト保持能力の差（GPT 系は 128k 程度、Claude Opus は 200k+）
- 4 階層分類の自発的遵守の再現度

### 8.2 新規 Student Plan 登録が停止中 [A]

2026-04-20 以降、GitHub Copilot Pro / Pro+ / Student プランの新規登録が一時停止中。**案 D は既存 Student Pack 保持者のみが対象**。新規学生は案 A を推奨。

### 8.3 Hooks が動かない

crlBrain の Claude Code hook (`.claude/settings.json` で宣言、`scripts/session-start-guard.sh` 等を起動) は Copilot CLI では実行されない。git hook (`scripts/hooks/`) は別途 `scripts/hooks/install-git-hooks.sh` で導入可能。以下は自律的な習慣形成または git hook で補う:

- `commit-msg` hook による `[topic-id]` prefix 強制
- `commit-msg` hook による "literal" 禁止語チェック
- `session-start-guard` による main ブランチ作業防止

### 8.4 MEMORY.md / Library の自動更新なし

Recorder が自動で MEMORY.md を更新する仕組みは CC 固有。Copilot CLI ではセッション終了時に学生が手動で追記する必要がある。

---

## 9. FAQ

### Q: Student Pack が未申請だが今から案 D を始めたい

**A**: 2026-04-20 以降、Copilot Student Plan の新規登録は一時停止中です [A]。以下のいずれかを検討してください:

1. **案 A (Gemini CLI) を使う** — 新規受付可、完全無料
2. **Copilot Pro の有料契約** — 月額 $10
3. **新規受付再開を待つ** — GitHub Changelog を定期確認

### Q: Claude モデルを使いたい

**A**: Student Plan では自己選択不可 [A]。以下が選択肢:

1. 研究室共通の Claude Code Max シート（案 C）を利用
2. Claude.ai Web Free で一時的に Claude と対話（skills 非対応）
3. 案 A (Gemini CLI) を使用

### Q: Gemini CLI（案 A）と併用できるか

**A**: できます [C]。両方のモデルで同じ議論を走らせて結果を比較するのは、モデル横断ベンチマークとして有用。ただし 1 セッション内で両方を同時に走らせるのは認知負荷が高いため、セッション単位で使い分けることを推奨。

### Q: `COPILOT.md` が存在しないがどうすればいい

**A**: 2026-04-21 時点で crlBrain には `COPILOT.md` が整備されていません。`CLAUDE.md` または `GEMINI.md` の内容を元に、プロジェクトルートに `COPILOT.md` を作成してください。パイロット参加者は作成した `COPILOT.md` を GitHub に PR として提出することで、crlBrain への貢献が記録されます。

---

## 10. 並行検討: 他 CLI

Copilot CLI（案 D）と並行して、以下の CLI でも crlBrain を利用できます:

| 案 | CLI | 対象 | 費用 | ガイド |
|----|-----|------|------|--------|
| 標準 | Claude Code | Pro/Max 加入者 | $20/月〜 | [student-mode-claude-code.md](./student-mode-claude-code.md) |
| 案 A | Gemini CLI | 全学生（新規可）| 無料 | [student-mode-gemini.md](./student-mode-gemini.md) |
| **案 D (本ガイド)** | **Copilot CLI** | **Student Pack 既存保持者** | **無料** | **本ガイド** |
| 案 E | Codex CLI | ChatGPT Plus 加入者 | $20/月 | [student-mode-codex.md](./student-mode-codex.md) |

全 CLI の選択ガイドと比較表は [docs/student-onboarding.md § 3](./student-onboarding.md) を参照。

並行パイロット計画: [sessions/discussions/crlBrain/20260421_1112_student-mode-gemini-pilot-plan.md](../sessions/discussions/crlBrain/20260421_1112_student-mode-gemini-pilot-plan.md)

---

## 参照ドキュメント

| ドキュメント | 内容 |
|------------|------|
| `docs/student-onboarding.md` | 学生向け導入マニュアル（共通導入・最初に読むべし）|
| `docs/student-prereq.md` | 学生向け前提ガイド（ターミナル・git・SSH 鍵）|
| `docs/student-mode-claude-code.md` | Claude Code 版（完全版）ガイド |
| `docs/student-mode-gemini.md` | 案 A: Gemini CLI 併用ガイド（本ガイドと相補的）|
| `room-config.md` | 議論プロトコル・Tiered Loading の中核設定 |
| `docs/manual-quickstart.md` | crlBrain クイックスタートガイド（CC 環境）|
| `docs/manual-detailed.md` | セッション詳細仕様（フェーズ・品質ゲート）|
| `docs/constitution/constitution_claude.md` | 主張 4 階層分類の詳細・事例（第 9 条）|
| `agents/README.md` | 全 25 名ペルソナ一覧 |

## 外部参照

- [GitHub Copilot CLI - GitHub Docs](https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli)
- [GitHub Copilot CLI is now generally available (Changelog)](https://github.blog/changelog/2026-02-25-github-copilot-cli-is-now-generally-available/)
- [Updates to GitHub Copilot for students (Changelog)](https://github.blog/changelog/2026-03-13-updates-to-github-copilot-for-students/)
- [Changes to GitHub Copilot plans for individuals (2026-04-20)](https://github.blog/changelog/2026-04-20-changes-to-github-copilot-plans-for-individuals/)
- [GitHub Student Developer Pack](https://education.github.com/pack)

---

## 更新履歴

| 日付 | 変更者 | 変更内容 |
|------|------|---------|
| 2026-04-21 | Claude（session/20260421-crlbrain-karpathy-skills-review）| 初版作成（案 D 試作版。案 A + 案 D 並行パイロットの一環）|
| 2026-04-21 | Claude (session/20260421-crlbrain-student-onboarding-manual) | 冒頭ノート + § 10 並行検討を 4 CLI 対応に更新 |
