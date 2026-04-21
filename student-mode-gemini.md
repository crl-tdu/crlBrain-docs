---
title: "Student Mode — Gemini CLI 併用ガイド"
type: reference
status: active
date_created: 2026-04-21
project: crlBrain
tags:
  - topic/onboarding
  - topic/gemini
  - topic/student
---

# Student Mode — Gemini CLI 併用ガイド

> [!note] 初めて crlBrain に触れる学生は、先に [学生向け導入マニュアル](./student-onboarding.md) を読むことを推奨します。前提知識（ターミナル・git・SSH 鍵・Node.js）は [学生向け前提ガイド](./student-prereq.md) を参照してください。
> 本ガイドは **Gemini CLI 固有のセットアップ詳細** に焦点を当てた技術リファレンスです。

> このドキュメントは crlBrain を **Gemini CLI（無料・個人 Google アカウント）** で動かしたい学生向けの手順書です。
> 完全版（Claude Code）との違いを理解した上で、制約を自覚しながら使ってください。

---

## このドキュメント自体の主張分類メモ

本ドキュメントに記載された内容の認識論的階層を以下に示します。  
（主張 4 階層の詳細は [docs/constitution/constitution_claude.md 第9条](constitution/constitution_claude.md) を参照）

| 記述箇所 | 階層 | 根拠 |
|---------|------|------|
| Gemini CLI フリープラン制限（60 req/min, 1000 req/日） | [A] 学術的事実 | Gemini CLI 公式ドキュメント（一次ソース）に基づく |
| 大学 Workspace アカウントでは認証が通らないケースがある | [A] 学術的事実 | Google Workspace Education ポリシー（一次ソース）に基づく |
| 軽量 3 ペルソナ構成でも議論の質は確保できる | [C] 実験結果の尤もらしい見解 | Gemini CLI 実運用での観察に基づくが、統計的根拠なし |
| 1000 req/日はフル 25 名ペルソナには不十分 | [C] 実験結果の尤もらしい見解 | トークン消費パターンの観察から推測、統計的根拠なし |
| 重要セッションは CC ユーザーとの協業を推奨 | [D] 直感的なひらめきによる仮説 | 経験則・類推によるもの、データ未支持 |

---

## 1. 概要 — なぜ Student Mode が存在するか

crlBrain の標準環境は **Claude Code（CC）** を前提とします。CC は強力な Hook 自動実行・サブエージェント委任・メモリ永続化を備えていますが、有料プランが必要です。

**Student Mode** は、フリープランの学生でも crlBrain の核となる研究議論フローを体験できるよう、**Gemini CLI（無料）** を代替 AI クライアントとして使う構成です。

### 完全版（Claude Code）との主な違い

| 機能 | Claude Code（完全版） | Gemini CLI（Student Mode） |
|------|--------------------|--------------------------|
| AI クライアント | `claude` コマンド | `gemini` コマンド |
| 設定ファイル | `CLAUDE.md` + `.claude/settings.json` | `GEMINI.md` |
| Hook 自動実行 | あり（commit-msg, session-start-guard など） | **なし** → 手動チェックリストで代替 |
| スキル呼び出し | `/skill-name` 形式 | `@skill-name` 形式 |
| サブエージェント委任 | Agent ツールで委任可能 | **不可** → 段階的読み込みで代替 |
| MEMORY 自動更新 | Recorder が自動更新 | **手動記録** |
| ペルソナ数（推奨） | 25 名（Tiered Loading） | 3 名（軽量セット）|
| 無料枠 | 要確認 | 60 req/min, 1000 req/日 [A] |

---

## 2. 7 層アーキテクチャのカバー範囲

crlBrain は 7 層アーキテクチャで設計されています。Gemini CLI でカバーできる層・できない層を以下に示します。

| Layer | 場所・役割 | Gemini CLI でのカバー状況 |
|-------|----------|------------------------|
| **Layer 1: Repo Memory** | `GEMINI.md` — プロジェクト設定 | **カバー可** `GEMINI.md` が自動読込される |
| **Layer 2: Expert Modes** | `.gemini-skills/` — スキルテンプレート | **カバー可** `@skill-name` で起動できる |
| **Layer 3: Guardrails** | `.claude/settings.json`（CC hook 宣言）+ `scripts/hooks/`（git hook スクリプト） | **カバー不可（CC hook のみ）** Hook は CC 専用機能。本ドキュメントの手動チェックリストで代替する（git hook は `scripts/hooks/install-git-hooks.sh` で別途導入可能） |
| **Layer 4: Progressive Context** | `docs/` — 詳細ドキュメント | **カバー可** 手動で参照・読み込みができる |
| **Layer 5: Local Context** | 各ディレクトリの `CLAUDE.md` | **部分的** GEMINI.md がある場合のみ自動読込、CLAUDE.md は手動参照 |
| **Layer 6: Context Management** | サブエージェント委任ルール | **カバー不可** Gemini CLI はサブエージェント非対応。段階的処理パターンで代替する |
| **Layer 7: Memory** | `~/.claude/projects/*/memory/` | **カバー不可** CC 専用パス。セッション終了時に手動で `MEMORY.md` を更新する |

---

## 3. 前提条件

### 3.1 Google アカウント

> **重要: 大学の @dendai.ac.jp アカウントでは Gemini CLI の OAuth 認証が通らない場合があります。[A]**

Google Workspace Education ポリシーにより、大学アカウントでは Gemini API の個人利用枠が制限されています。  
**個人の Google アカウント（gmail.com など）を使ってください。**

### 3.2 システム要件

| 要件 | バージョン |
|------|---------|
| Node.js | **20.0.0 以上**（LTS 推奨）[A] |
| npm | 9 以上 |
| Git | 2.x 以上 |
| OS | macOS / Linux（Ubuntu 20.04+）/ Windows 11 24H2+（WSL2 推奨） |
| Shell | Bash / Zsh / PowerShell |

確認コマンド:

```bash
node --version   # v20.x.x 以上であること
npm --version    # 9.x.x 以上であること
git --version
```

---

## 4. セットアップ手順

### 4.1 Gemini CLI のインストール

#### macOS

```bash
# 方法 A: Homebrew（推奨・Node.js の依存解決を自動化）
brew install gemini-cli

# 方法 B: npm（既に Node.js 20+ がある場合）
npm install -g @google/gemini-cli

# 方法 C: MacPorts
sudo port install gemini-cli
```

#### Linux（Ubuntu / Debian / Arch 等）

```bash
# 方法 A: Homebrew (Linuxbrew) — 推奨
brew install gemini-cli

# 方法 B: npm（Node.js 20+ 事前インストール）
#   Ubuntu/Debian の場合
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
npm install -g @google/gemini-cli
```

#### Windows

PowerShell を管理者権限で開き、以下のいずれかを実行:

```powershell
# 方法 A: Chocolatey で Node.js をインストール後 npm
choco install nodejs-lts
npm install -g @google/gemini-cli

# 方法 B: 公式インストーラー
#  1. https://nodejs.org/ から Node.js 20+ LTS (.msi) をダウンロード・インストール
#  2. PowerShell を再起動して以下を実行
npm install -g @google/gemini-cli
```

> **WSL2 を使う場合**: WSL2 上で Linux 手順を推奨。Windows と WSL2 の npm グローバルインストール先が別になるので、混在させないこと。

#### インストール確認（全 OS 共通）

```bash
gemini --version
```

バージョン番号が表示されれば OK。エラーが出る場合は Node.js のバージョンと PATH を確認してください。

### 4.2 初回認証（個人 Google アカウント）

Gemini CLI には独立した `auth login` サブコマンドはなく、**初回起動時にブラウザが自動で開く OAuth フロー**で認証します。[A]

```bash
gemini
```

初回のみブラウザが起動し、Google ログインを求められます。**個人の Google アカウント**（gmail.com）でログインしてください。  
大学アカウント（@dendai.ac.jp）ではなく、個人アカウントを選ぶ点に注意してください。

認証成功の確認: ブラウザでの承認完了後、ターミナルにインタラクティブセッションのプロンプトが表示されれば OK です。認証情報は `~/.gemini/oauth_creds.json` に保存されます（以降の `gemini` 起動では自動ログイン状態）。[A]

### 4.3 フリープラン利用枠の確認

Gemini CLI フリープラン（2026-04-21 現在）: [A]

- **60 req/min** — 1 分あたりのリクエスト上限
- **1000 req/日** — 1 日あたりのリクエスト上限

> 上限は変更される場合があります。最新情報は公式ドキュメントを確認してください:  
> https://github.com/google-gemini/gemini-cli

### 4.4 crlBrain リポジトリのクローン

```bash
git clone <crlBrain リポジトリ URL>
cd crlBrain
```

### 4.5 Gemini CLI の起動

```bash
gemini
```

`GEMINI.md` が自動的に読み込まれ、crlBrain のコンテキストが設定されます。

---

## 5. 軽量ペルソナセット（推奨 3 名構成）

フリープランの 1000 req/日制限内で実用的な議論を行うため、以下の 3 名構成を推奨します。[C]

### 5.1 推奨 3 名

| ペルソナ | 役割 | 採用理由 |
|---------|------|---------|
| **Prof.Igarashi** | PI（指導教員） | 最終決定権者・議論の核心を問い直す役割 |
| **Dr.FEP** | 自由エネルギー原理専門家 | 認知科学・理論的枠組み提供 |
| **Dr.AI** | AI 研究者 | 実装・手法面のカバー |

> 3 名では評価（Dr.Veritas）・Engineering 観点が欠けます。  
> 重要なセッションでは CC ユーザー（指導教員・先輩）と協業することを推奨します。[D]

### 5.2 読み込み方法

#### 方法 A: GEMINI.md 経由（推奨）

`GEMINI.md` はすでに Tiered Loading を定義しています。Gemini CLI 起動後、以下のように指示してください:

```
「軽量セットで始めたい。Prof.Igarashi, Dr.FEP, Dr.AI の 3 名をロードしてください。」
```

その後、Manager（`agents/manager/SOUL.md` + `agents/manager/SKILL.md`）に Tier 1 として各 SOUL + SKILL を読み込ませます。

#### 方法 B: 手動プロンプト

セッション冒頭に以下のテキストをコピーして貼り付けます:

```
以下のペルソナファイルを読み込んでセッションを開始してください:
- agents/pi-igarashi/SOUL.md, SKILL.md
- agents/fep-neuroscientist/SOUL.md, SKILL.md
- agents/ai-researcher/SOUL.md, SKILL.md

room-config.md を読み込み、Manager として挨拶してください。
コンテキスト節約のため、MEMORY.md はオンデマンドで読み込みます。
```

### 5.3 ペルソナファイルのパス

```
agents/
├── pi-igarashi/       SOUL.md, SKILL.md, MEMORY.md
├── fep-neuroscientist/ SOUL.md, SKILL.md, MEMORY.md
└── ai-researcher/     SOUL.md, SKILL.md, MEMORY.md
```

---

## 6. 手動チェックリスト（Hooks の代替）

Claude Code 環境では `.claude/settings.json` で宣言された hook（`scripts/session-start-guard.sh` 等）と `scripts/hooks/` 由来の git hook が以下のルールを自動強制します。  
Gemini CLI ではこれらが動作しないため、**学生自身が毎セッション確認**してください。

### セッション開始前

- [ ] **ブランチ名の確認** — `main` ブランチで作業していないか確認する

  ```bash
  git branch --show-current
  # "main" と表示された場合は必ずブランチを切ること
  ```

- [ ] **session/ ブランチを作成する**

  ```bash
  git checkout -b session/YYYYMMDD-{topic-id}-{slug}
  # 例: git checkout -b session/20260421-phs-gemini-test
  ```

  ブランチ命名規約:
  - `session/` プレフィックス必須
  - `YYYYMMDD` は JST の日付
  - `{topic-id}` はトピックレジストリの ID（例: `phs`, `sga`）
  - `{slug}` は英小文字+ハイフンで内容を簡潔に表す

### コミット時

- [ ] **`[topic-id]` prefix** — 全コミットメッセージに prefix を付ける

  ```bash
  git commit -m "[PHS] session: FEP の議論まとめ"
  git commit -m "[SGA] feat: swarm 実験計画 v2"
  git commit -m "[crlbrain] docs: student-mode マニュアル追加"
  ```

  トピック外の作業は `[crlbrain]` を使用します。  
  **"literal" という語はコミットメッセージに使わないこと。**（後述）

- [ ] **主張の 4 階層分類 [A/B/C/D] を明示する**（最重要・crlBrain の核）

  コミットメッセージや議事録で主張を書く際は、必ず `[A]`/`[B]`/`[C]`/`[D]` を接頭辞または後置括弧で明示してください。

  ```
  [C] n=20 で effect size は維持されると予測
  [B] Round 3 で Primary PASS（事前登録閾値超過）
  [D] σ_train が seed 依存性の原因と直感される
  ```

  4 階層の定義:

  | 階層 | 名称 | 使うとき |
  |------|------|---------|
  | [A] | 学術的事実 | 論文・教科書・数式・コード行など一次ソースから直接引用できる場合 |
  | [B] | 実験結果確認事項 | 事前登録した PASS 基準を達成した実験結果、または統計検定で帰無仮説を棄却した場合 |
  | [C] | 実験結果の尤もらしい見解 | 実験観察を外挿・内挿した推測。統計的根拠は伴わない |
  | [D] | 直感的なひらめきによる仮説 | 理論推論・類推・経験則。データで直接支持されていない |

- [ ] **"literal" 禁止語の確認** — コミットメッセージ・議事録・発言に "literal" を使っていないか確認する

  CC 環境ではこの語が commit-msg hook によって自動的に reject されます。  
  Gemini CLI では hook がないため、手動で使用を避けてください。  
  "literal" を使いたくなったときは、代わりに `[A]〜[D]` の階層で主張を明示してください。  
  詳細な経緯・理由は [docs/constitution/constitution_claude.md 第9条](constitution/constitution_claude.md) を参照してください。

### セッション終了時

- [ ] 議事録を `sessions/discussions/` に保存したか
- [ ] 参加ペルソナの `MEMORY.md` を手動で更新したか（自動更新されないため）
- [ ] 新用語があれば `@dictionary-manager` で辞書に追記したか
- [ ] `sessions/discussions/README.md` の目次を更新したか
- [ ] コミット＆プッシュを行ったか

---

## 7. セッション議事録テンプレート

Gemini CLI では `@discussion-memo-output` スキルを使って議事録を生成できます。  
以下のテンプレートは手動で議事録を作成する際の雛形です。  
**4 階層セクションは必須**です。省略すると MEMORY への記録が不完全になります。

```markdown
---
title: "[topic-id] セッションタイトル"
date: YYYY-MM-DD（JST）
branch: session/YYYYMMDD-{topic-id}-{slug}
participants:
  - Prof.Igarashi
  - Dr.FEP
  - Dr.AI
  - （ユーザー名）
topic_id: {topic-id}
---

# セッション議事録

## 概要（1〜3 文）

<!-- 今日のセッションで何を議論し、何が決まったか -->

---

## ラウンド記録

### Round 1: {テーマ}

**Segment A — 構造化発言**

- Prof.Igarashi [D]: ...
- Dr.FEP [A]: ...
- Dr.AI [C]: ...

**Segment B — 自由討論**

<!-- 議論の流れを記録 -->

**Segment C — まとめ**

<!-- 合意事項・未解決論点 -->

---

## 主張の 4 階層分類まとめ（必須セクション）

<!-- このセクションは省略不可。セッション中に出た主要な主張を 4 階層に分類して記録する -->

### [A] 学術的事実として確認された事項

-

### [B] 実験結果確認事項（統計的根拠あり）

-

### [C] 実験結果の尤もらしい見解（統計的根拠なし）

-

### [D] 直感的なひらめきによる仮説

-

---

## 未解決論点

- [ ] ...

## 次のアクション

- [ ] 担当者: アクション内容（期限）

---

## 品質スコアカード

| 軸 | スコア（1〜5） | 根拠 |
|---|------|------|
| 新規性 (Novelty) | | |
| 信頼性 (Rigor) | | |
| 意義 (Significance) | | |
```

---

## 8. 既知の制約

Student Mode を使う前に以下の制約を理解してください。

### 自動化されない機能

| 機能 | CC 環境 | Student Mode |
|------|---------|-------------|
| commit-msg の `[topic-id]` 強制 | Hook で自動 reject | **手動確認** |
| "literal" 使用の自動 reject | Hook で自動 reject | **手動確認** |
| session-start-guard（main ブランチ警告） | Hook で自動プロンプト | **手動確認** |
| MEMORY.md 自動更新（Recorder） | セッション終了時に自動 | **手動記録が必要** |
| `/clear` 後のコンテキスト再構築 | CC コマンドで管理 | 新規 Gemini CLI セッションで代替 |

### リクエスト上限の実際的な影響

[C] 25 名フルペルソナを使った長時間セッションでは 1000 req/日の上限に達する可能性があります。  
以下の対策を推奨します:

- ペルソナを 3〜5 名に絞る（本ドキュメントの推奨 3 名構成）
- MEMORY.md はセッション開始時に読まず、必要時のみオンデマンドで読む
- 1 日に複数の長いセッションを行わない
- 上限に達した場合は翌日に継続する

### 重要セッションは CC ユーザーとの協業を推奨

[D] 以下の場面では CC 環境（指導教員・先輩研究者）との協業を推奨します:

- 品質ゲート（全 3 軸 >= 3）の判定が必要な場面
- `@research-proposal-generator` や `@research-requirements-engineering` を使う場面
- 実験計画書を正式に発行する場面（`@experiment-plan-bridge`）
- Dr.Veritas による知見ライブラリへのエントリ検証が必要な場面

---

## 9. FAQ

### Q: 大学の @dendai.ac.jp アカウントでログインしようとしたが認証が失敗する

**A**: 大学の Google Workspace Education アカウントでは Gemini CLI の個人利用 OAuth が制限されている場合があります。[A]  
**個人の Google アカウント（gmail.com など）でログインしてください。**

Gemini CLI には独立した `auth` サブコマンドがないため、認証情報ファイルを削除してから再起動する形で切り替えます。[A]

```bash
rm ~/.gemini/oauth_creds.json   # 既存認証を破棄
gemini                          # 個人アカウントでログインし直す
```

---

### Q: 1000 req/日の上限に達してしまったらどうすればいいか

**A**: 以下のいずれかで対応してください:

1. **翌日まで待つ** — 上限は UTC 0:00 にリセットされます（JST 9:00 に相当）
2. **有料プランへのアップグレードを検討する** — 研究が進んで CC ほどではないがより多くのリクエストが必要な場合
3. **セッションを短く分割する** — 1 セッションのペルソナ数・ラウンド数を減らす

---

### Q: Hooks が動かないがどうすればいいか

**A**: Gemini CLI は Claude Code の Hook 機能（`.claude/settings.json` で宣言される）に対応していません。  
本ドキュメントの「6. 手動チェックリスト」をセッションごとに確認してください。git hook（`scripts/hooks/install-git-hooks.sh`）は別途導入すればコミット時のチェックとして利用できます。  
特に以下の 3 点が重要です:

1. `session/` ブランチを切ってから作業する（main への直接コミット禁止）
2. 全コミットに `[topic-id]` prefix を付ける
3. "literal" という語を使わず、主張を `[A]/[B]/[C]/[D]` で分類する

---

### Q: `@discussion-memo-output` や `@topic` スキルが動かない

**A**: `.gemini-skills/` 配下のスキルが存在することを確認してください。

```bash
ls .gemini-skills/
```

スキルが見つからない場合、リポジトリが正しくクローンされているか確認してください。  
スキルの使い方は各スキルディレクトリ内の README または skill.md を参照してください。

---

### Q: MEMORY.md を更新するのを忘れた

**A**: セッション終了後に手動で更新してください。  
対象ファイル（推奨 3 名構成の場合）:

```
agents/pi-igarashi/MEMORY.md
agents/fep-neuroscientist/MEMORY.md
agents/ai-researcher/MEMORY.md
```

MEMORY.md のフォーマットは既存エントリを参考にしてください。  
更新後、必ずコミットしてください。

---

### Q: 主張の 4 階層分類は毎回書かないといけないか

**A**: crlBrain の根幹的な基盤であり、最優先原則です（憲法 claude 第9条）。  
発言・議事録・コミットメッセージの全ての主張に階層を明示してください。  
「これは [A] か [D] か」を自問する習慣をつけることで、研究の厳密性が高まります。  
CC 環境では Hook がこれを強制しますが、Gemini CLI では自律的な習慣形成が必要です。

---

### Q: 25 名全員を呼びたい場合はどうすればいいか

**A**: フリープランでは 1000 req/日の上限があるため、全員ロードは現実的ではありません。[C]  
以下の優先順位でペルソナを追加することを推奨します:

1. 基本セット（Prof.Igarashi + Dr.FEP + Dr.AI）
2. テーマに応じた専門家 1〜2 名を追加（例: 数理理論なら Dr.Math）
3. 評価フェーズのみ Dr.Veritas を追加

全 25 名が必要なフルセッションには CC 環境の使用を推奨します。

---

## 10. 並行検討: 他 CLI

Gemini CLI（案 A）と並行して、以下の CLI でも crlBrain を利用できます:

| 案 | CLI | 対象 | 費用 | ガイド |
|----|-----|------|------|--------|
| 標準 | Claude Code | Pro/Max 加入者 | $20/月〜 | [student-mode-claude-code.md](./student-mode-claude-code.md) |
| **案 A (本ガイド)** | **Gemini CLI** | **全学生（新規可）** | **無料** | **本ガイド** |
| 案 D | Copilot CLI | Student Pack 既存保持者 | 無料 | [student-mode-copilot-cli.md](./student-mode-copilot-cli.md) |
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
| `GEMINI.md` | Gemini CLI 向けプロジェクト設定 |
| `room-config.md` | 議論プロトコル・Tiered Loading の中核設定 |
| `docs/student-mode-copilot-cli.md` | 案 D: GitHub Copilot CLI 併用ガイド |
| `docs/manual-quickstart.md` | crlBrain クイックスタートガイド（CC 環境） |
| `docs/manual-detailed.md` | セッション詳細仕様（フェーズ・品質ゲート） |
| `docs/constitution/constitution_claude.md` | 主張 4 階層分類の詳細・事例（第9条） |
| `agents/README.md` | 全 25 名ペルソナ一覧 |

---

## 更新履歴

| 日付 | 変更者 | 変更内容 |
|------|------|---------|
| 2026-04-21 | Claude（session/20260421-crlbrain-karpathy-skills-review） | 初版作成 |
| 2026-04-21 | Claude（同セッション） | OS 別インストール手順（macOS/Linux/Windows）を追記、Node.js 要件を 20+ に更新、§10 案 D 並行検討節を追加 |
| 2026-04-21 | Claude (session/20260421-crlbrain-student-onboarding-manual) | 冒頭ノート + § 10 並行検討を 4 CLI 対応に更新 |
