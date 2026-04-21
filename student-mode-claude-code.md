---
title: "Student Mode — Claude Code 併用ガイド (完全版)"
type: guide
status: active
date_created: 2026-04-21
project: crlBrain
tags:
  - topic/crlBrain
  - topic/student-mode
  - method/claude-code
related:
  - docs/student-onboarding.md
  - docs/student-prereq.md
  - docs/student-mode-gemini.md
  - docs/student-mode-copilot-cli.md
  - docs/student-mode-codex.md
---

# Student Mode — Claude Code 併用ガイド (完全版)

> [!abstract]
> Claude Code は crlBrain の **標準環境（完全版）** です。Hook 自動実行・サブエージェント委任・Memory 永続化を含む全機能がネイティブに動作します。本ガイドは Claude Pro / Max 加入済または加入予定の学生向けに、セットアップ・運用・他 CLI との対比を解説します。

---

## このドキュメント自体の主張分類メモ

本ドキュメント内の主張は以下の階層を遵守します。

| 記述箇所 | 階層 | 根拠 |
|---------|------|------|
| Claude Code は Hook / skills / Agent ツールを標準サポート | [A] | Anthropic 公式 Docs（一次ソース）|
| Claude Pro / Max プランの料金体系 | [A] | Anthropic 公式 Pricing |
| 完全版 25 ペルソナは CC 環境で最も再現性高く動く | [C] | 研究室内運用観察、統計的根拠なし |
| crlBrain の `.claude/` 資産は CC 環境でフル機能する | [A] | 本リポジトリ CLAUDE.md の記述が一次ソース |
| 軽量 3 ペルソナで 30 分セッションを完遂できる | [C] | 軽量モード運用の観察、統計的根拠なし |

---

## 1. 概要 — Claude Code が標準環境である理由

crlBrain の `.claude/` 配下（`settings.json` / `skills/` / `hooks/` / `agents/`）は **Claude Code を前提に設計** されています [A]。具体的には:

- `commit-msg` hook が禁止語（憲法第 9 条指定の crutch word）を全ブランチで自動 reject する [A]
- `[topic-id]` commit prefix は CLAUDE.md 規約による必須要件（session/* ブランチでは hook 強制なし、規約上の義務）[A]
- `session-start-guard` が `main` ブランチでの作業を自動検知する [A]
- `/discussion-memo-output` や `/experiment-dispatch` 等の skills が議事録生成・実験投入を自動化する [A]
- Agent ツールでサブエージェントに探索タスクを委任し、メインセッションの context を保護できる [A]

他の CLI（Gemini / Copilot / Codex）では上記の一部が動作しません。本ガイドは Claude Code の完全機能を活かして軽量・フル両モードを動かす方法を解説します。

---

## 2. 7 層アーキテクチャのカバー範囲

crlBrain は 7 層アーキテクチャで設計されており、Claude Code は全層を完全サポートします。

| Layer | 場所・役割 | CC カバー状況 |
|-------|----------|------------|
| **Layer 1: Repo Memory** | `CLAUDE.md` | ✅ 自動読込 |
| **Layer 2: Expert Modes** | `.claude/skills/` | ✅ `/skill-name` で起動 |
| **Layer 3: Guardrails** | `.claude/settings.json`（hook 設定）/ `scripts/hooks/`（スクリプト）| ✅ 自動実行 |
| **Layer 4: Progressive Context** | `docs/` | ✅ 手動参照 |
| **Layer 5: Local Context** | 各ディレクトリの `CLAUDE.md` | ✅ 自動読込 |
| **Layer 6: Context Management** | サブエージェント委任 | ✅ Agent ツール |
| **Layer 7: Memory** | `~/.claude/projects/*/memory/` | ✅ 自動永続化 |

---

## 3. 前提条件

### 3.1 Claude サブスクリプション

> [!important]
> **Claude Pro ($20/月) または Claude Max** が必要です。[A]（Anthropic 公式 Pricing）
>
> - Pro: 個人向け・中規模利用
> - Max: 研究・開発向け大規模利用（研究室共通シートとして Prof.Igarashi が契約する場合あり）

### 3.2 システム要件

| 要件 | バージョン |
|------|---------|
| macOS | 12+ |
| Linux | Ubuntu 20.04+ 等 |
| Windows | Windows 10+（WSL2 推奨）|
| Git | 2.x 以上 |

Claude Code は CLI ツール（`claude` コマンド）として動作し、VS Code / JetBrains 拡張を併用できます [A]。

前提知識（ターミナル・git・SSH 鍵）は [学生向け前提ガイド](./student-prereq.md) を参照。

---

## 4. セットアップ手順

### 4.1 Claude Code のインストール

Claude Code は Node.js ベースの CLI ツールです。前提ガイド § 6 で Node.js インストール済であること。

#### 全 OS 共通（npm 経由、推奨）

```bash
npm install -g @anthropic-ai/claude-code
```

#### 動作確認

```bash
claude --version
```

#### IDE 拡張（任意・併用可）

VS Code / JetBrains Marketplace で "Claude" を検索してインストール。CLI 版と併用可能 [A]。

### 4.2 初回ログイン

```bash
claude login
```

ブラウザが開き OAuth 認証。完了後:

```bash
claude --version
claude /status    # プラン確認
```

### 4.3 crlBrain リポジトリ

前提ガイド § 7 の手順で clone 済であること。CC は起動時に **プロジェクト直下の `CLAUDE.md` と `.claude/settings.json` を自動読込** します [A]。

```bash
cd ~/local/project_workspace/crlBrain
claude
# → CLAUDE.md + .claude/settings.json が自動読込される
```

---

## 5. 軽量ペルソナセット（推奨 3 名構成）

まずは他 CLI と揃えた 3 固定で始め、慣れたら追加するのが良い [C]。

| ペルソナ | 役割 |
|---------|------|
| **Prof.Igarashi** | PI・最終決定 |
| **Dr.FEP** | 自由エネルギー原理 |
| **Dr.AI** | AI / 実装 |

CC では UserPromptSubmit hook が毎プロンプト時にセッション状態を提示し、必要に応じて `room-config.md` の Tiered Loading を促します [A]。呪文は `docs/student-onboarding.md` § 4.2 と同一。

### 5.1 フル版 25 ペルソナへの移行

1. 「軽量モードで 3-5 セッション経験」
2. Dr.Math を追加（数理の厳密性）[C]
3. `room-config.md` § 5 Tiered Loading に従い、テーマに応じて Dr.Cognition / Dr.Reviewer 等を追加
4. Phase 3 Integration で品質スコアカードを導入

詳細は [docs/manual-detailed.md](./manual-detailed.md) を参照。

---

## 6. 自動化されている項目（他 CLI との対比）

| 項目 | Claude Code | Gemini | Copilot | Codex |
|------|-------------|--------|---------|-------|
| `[topic-id]` commit prefix 強制 | △ main のみ (session/* は規約) | △ 同左 | △ 同左 | △ 同左 |
| 禁止語 `literal` reject | ✅ commit-msg hook (全ブランチ) | ✅ 同左 | ✅ 同左 | ✅ 同左 |
| `main` ブランチ作業警告 | ✅ session-start-guard | ❌ 手動 | ❌ 手動 | ❌ 手動 |
| MEMORY.md 自動更新 | ✅ Recorder | ❌ 手動 | ❌ 手動 | ❌ 手動 |
| Agent サブエージェント | ✅ | ❌ | △ | △ |
| skills `/<name>` 起動 | ✅ | ✅ `@<name>` | ✅ | △ 別機構 |

本表の詳細対比は [docs/student-mode-gemini.md § 1](./student-mode-gemini.md) を参照。

---

## 7. セッション議事録テンプレート

CC では `/discussion-memo-output` スキルで議事録を自動生成できます。

```
/discussion-memo-output
```

スキル内部では `sessions/discussions/{topic-id}/YYYYMMDD_HHMM_<slug>.md` に下書きを生成します [A]。4 階層分類セクションを含む完全テンプレートは [docs/student-onboarding.md § 4.4](./student-onboarding.md) を参照してください [A]。

---

## 8. 既知の制約

他 CLI と比べて CC は機能面で最も充実していますが、以下は留意点です:

- [A] **サブスクリプション必須** — Pro $20/月〜。Free で crlBrain を試したい学生は Gemini CLI を使う
- [C] **コンテキストウィンドウ** — 200k+ tokens あるが、25 ペルソナ全ロードはコンテキスト圧迫する。Tiered Loading 必須
- [D] **Hook の挙動差** — OS（macOS/Linux/Windows WSL2）で hook シェル実装に差が出ることがある。問題があれば `scripts/hooks/` のログを確認

---

## 9. FAQ

### Q: Pro と Max のどちらを契約すべきか

**A:** 軽量モード中心なら Pro で十分 [C]。フル版 25 ペルソナで週 10+ セッション回す場合は Max を検討。研究室共通 Max シート（案 C）が利用可能な場合もあるので Prof.Igarashi に相談。

### Q: Hook が動かない・無視される

**A:** `.claude/settings.json` が読み込まれているか確認:

```
claude /status
```

`scripts/session-start-guard.sh` の実行権限（`chmod +x scripts/session-start-guard.sh`）も確認してください [A]。

### Q: サブエージェントに何を委任すべきか

**A:** 以下のタスクで特に有効 [C]:

- 3 ファイル以上を読んで要約する探索タスク
- 複数ディレクトリを横断する grep 検索
- Library 候補の検証（Dr.Veritas の役割）

Agent ツールの使い方は `/help agent` または Anthropic 公式 Docs 参照。

### Q: 他の学生と同じ議事録を協同編集したい

**A:** Git ベースで編集 → PR → レビュー → merge の流れが標準 [A]。同時編集中は `git pull --rebase` の競合を気にせず済むように **session ブランチを個人ごとに分ける** のが安全。

### Q: skills や hooks を自分で追加したい

**A:** `.claude/skills/<name>/skill.md` を作成して commit すれば即座に使えます [A]。hooks 追加は `.claude/settings.json` の `hooks` キーに追記 + `scripts/hooks/<name>.sh` 作成。詳細は `docs/constitution/constitution_skills.md` 参照。

---

## 10. 並行ガイド

他 CLI のガイド:

| 案 | CLI | 対象 | 費用 | ガイド |
|----|-----|------|------|--------|
| 案 A | Gemini CLI | 全学生（新規可）| 無料 | [student-mode-gemini.md](./student-mode-gemini.md) |
| 案 D | Copilot CLI | Student Pack 既存保持者 | 無料 | [student-mode-copilot-cli.md](./student-mode-copilot-cli.md) |
| 案 E | Codex CLI | ChatGPT Plus 加入者 | $20/月 | [student-mode-codex.md](./student-mode-codex.md) |
| **標準** | **Claude Code** | **Pro/Max 加入者** | **$20/月〜** | **本ガイド** |

全 CLI の選択ガイドは [docs/student-onboarding.md § 3](./student-onboarding.md) を参照。

並行パイロット計画: [sessions/discussions/crlBrain/20260421_1112_student-mode-gemini-pilot-plan.md](../sessions/discussions/crlBrain/20260421_1112_student-mode-gemini-pilot-plan.md)

---

## 参照ドキュメント

| ドキュメント | 内容 |
|------------|------|
| `docs/student-onboarding.md` | 学生向け導入マニュアル（共通導入）|
| `docs/student-prereq.md` | 学生向け前提ガイド |
| `room-config.md` | 議論プロトコル・Tiered Loading |
| `CLAUDE.md` | プロジェクト規約・画像保存ルール |
| `.claude/settings.json` | hooks / skills 設定 |
| `docs/constitution/constitution_claude.md` | 4 階層分類（第 9 条）|

---

## 更新履歴

| 日付 | 変更者 | 変更内容 |
|------|------|---------|
| 2026-04-21 | Claude (session/20260421-crlbrain-student-onboarding-manual) | 初版作成（完全版ガイドとして既存 3 ガイドと同構造）|
