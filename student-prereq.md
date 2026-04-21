---
title: "学生向け前提ガイド — crlBrain を始める前に"
type: reference
status: draft
date_created: 2026-04-21
project: crlBrain
tags:
  - topic/student-onboarding
  - topic/prerequisites
related:
  - docs/student-onboarding.md
  - docs/student-mode-claude-code.md
  - docs/student-mode-gemini.md
  - docs/student-mode-copilot-cli.md
  - docs/student-mode-codex.md
---

# 学生向け前提ガイド — crlBrain を始める前に

> このドキュメントは「研究室に配属されたばかりで、ターミナル・git・SSH 鍵などを扱うのは初めて」という学生向けの **前提知識ガイド** です。[D]
> crlBrain 本体の使い方は [学生向け導入マニュアル](./student-onboarding.md) を参照してください。

---

## 1. 対象読者

本ガイドは以下のいずれかに該当する学生を想定しています [D]:

- 2026 年度に研究室配属された学部 3-4 年生
- Mac / Windows / Linux の初期状態から crlBrain を始めたい
- ターミナル操作や git に不慣れで、「`git clone` と言われても何をすればいいか分からない」状態

慣れている学生は本ガイドを読み飛ばし、直接 [学生向け導入マニュアル](./student-onboarding.md) に進んで構いません。

---

## 2. ターミナルを開く

### macOS

`⌘ Space` で Spotlight を開き、"terminal" と入力してリターン。標準の **Terminal.app** が開きます。
お好みで [iTerm2](https://iterm2.com) をインストールしてもよい（任意）。

### Linux (Ubuntu)

`Ctrl+Alt+T` で標準のターミナルが開きます。

### Windows

**Windows Terminal** を使用（Windows 11 には標準搭載、Windows 10 はストアからインストール）。
crlBrain を本格的に使うなら **WSL2 + Ubuntu** の導入を強く推奨 [C]（参考: Microsoft Docs）。

---

## 3. Homebrew をインストール（macOS / Linux）

Homebrew は macOS / Linux の **パッケージマネージャ** です [A]（公式: brew.sh）。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

インストール後、表示された指示（`eval "$(/opt/homebrew/bin/brew shellenv)"` 等）を `~/.zshrc` または `~/.bashrc` に追記してください。

確認:

```bash
brew --version
# Homebrew 4.x.x と表示されれば成功
```

### Windows (WSL2) の場合

WSL2 上の Ubuntu で上記 Linuxbrew 手順を実行するか、Windows ネイティブなら `winget` や `choco` を使ってください。

---

## 4. Git をインストール・初期設定

### インストール

```bash
# macOS / Linux
brew install git

# Windows (native)
winget install Git.Git
```

### 初回設定

`YOUR_NAME` と `YOUR_EMAIL` を自分の値に置き換えて実行:

```bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_EMAIL"
git config --global init.defaultBranch main
git config --global pull.rebase false
```

メールアドレスは GitHub で公開されるので、大学メール以外を使いたい場合は GitHub の `<USERNAME>+<random>@users.noreply.github.com` 形式を使ってください [A]（公式: GitHub Docs）。

---

## 5. SSH 鍵を作成して GitHub に登録

crlBrain リポジトリを `git clone` する際、SSH 鍵認証を使うと毎回のパスワード入力が不要になります（推奨）[C]。

### 5.1 SSH 鍵を生成

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

パスフレーズは空でも可（空推奨ではない）。鍵は `~/.ssh/id_ed25519` と `~/.ssh/id_ed25519.pub` に保存されます。

### 5.2 公開鍵を GitHub に登録

公開鍵の内容をクリップボードにコピー:

```bash
# macOS
pbcopy < ~/.ssh/id_ed25519.pub

# Linux
xclip -selection clipboard < ~/.ssh/id_ed25519.pub

# Windows (WSL2)
cat ~/.ssh/id_ed25519.pub   # 表示してコピー
```

GitHub Web → Settings → SSH and GPG keys → New SSH key に貼り付けて保存。

### 5.3 接続確認

```bash
ssh -T git@github.com
# "Hi <username>! You've successfully authenticated..." と出れば成功
```

---

## 6. Node.js をインストール

一部の CLI（Gemini CLI / Copilot CLI / Codex CLI）が Node.js を必要とします。Homebrew の `node` フォーミュラは最新安定版をインストールするため、各 CLI の要件を満たせます。

```bash
# macOS / Linux (Homebrew)
brew install node

# Windows (winget)
winget install OpenJS.NodeJS.LTS
```

確認:

```bash
node --version   # v22 以上推奨（Copilot CLI 要件）、最低 v20
npm --version
```

> 必要な Node.js バージョンは CLI によって異なる:
> - Gemini CLI: Node.js 20+ [A]
> - Copilot CLI: Node.js 22+ [A]
> - Codex CLI: Node.js 18+ [A]（バイナリ版は Node.js 不要）
>
> 最新版をインストールしておけば全て満たせます [C]。

---

## 7. crlBrain リポジトリを clone

### SSH（推奨、§ 5 で鍵登録済の場合）

```bash
cd ~/local/project_workspace   # 任意の作業ディレクトリ
git clone git@github.com:crl-tdu/crlBrain.git
cd crlBrain
```

### HTTPS（鍵を登録していない場合）

```bash
git clone https://github.com/crl-tdu/crlBrain.git
cd crlBrain
```

`git remote -v` で `origin` が表示されれば clone 成功 [A]（git 公式挙動）。

---

## 8. エディタの推奨セットアップ（任意）

crlBrain は Markdown / YAML / Python / bash が混在するプロジェクト。以下いずれかを推奨:

- **VS Code** — 拡張: `Markdown All in One`, `YAML`, `ShellCheck`
- **Cursor** — VS Code 互換 + AI 支援が強化
- **Zed** — Markdown / 日本語の表示が美しい

いずれも `code .` / `cursor .` / `zed .` でリポジトリ直下から開くと効率的です。

---

## 9. 困ったとき — 先輩に聞く前のチェックリスト

問題が起きたら、以下を順に確認してから先輩に相談してください:

- [ ] ターミナルに出ているエラーメッセージをそのままコピーする（スクショより文字列が扱いやすい）[D]
- [ ] `git status` / `git branch --show-current` / `pwd` を実行して現状を把握する
- [ ] `echo $PATH` でパスが通っているか確認する
- [ ] 再起動で直ることも多い（特に `.zshrc` / `.bashrc` 変更後）
- [ ] 公式ドキュメントを先に読む（CLI 固有の問題は各 CLI の公式 Docs）
- [ ] 同じ問題の過去事例を GitHub Issue と過去議事録から検索する

---

## 10. 次のステップ

前提が揃ったら [学生向け導入マニュアル](./student-onboarding.md) に進んでください。

## 参照ドキュメント

| ドキュメント | 内容 |
|------------|------|
| `docs/student-onboarding.md` | crlBrain 学生向け導入マニュアル（共通導入）|
| `docs/student-mode-claude-code.md` | Claude Code 版 CLI ガイド |
| `docs/student-mode-gemini.md` | Gemini CLI 版 CLI ガイド |
| `docs/student-mode-copilot-cli.md` | Copilot CLI 版 CLI ガイド |
| `docs/student-mode-codex.md` | Codex CLI 版 CLI ガイド |

---

## 更新履歴

| 日付 | 変更者 | 変更内容 |
|------|------|---------|
| 2026-04-21 | Claude (session/20260421-crlbrain-student-onboarding-manual) | 初版作成 |
