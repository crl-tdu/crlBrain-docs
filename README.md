# crlBrain-docs — 学生向け公開ミラー

本 repository は [crl-tdu/crlBrain](https://github.com/crl-tdu/crlBrain)（private）の学生向けドキュメントを公開するためのミラーです。

## 公開 URL

https://crl-tdu.github.io/crlBrain-docs/

## 更新方針

**本 repo を直接編集しないでください。** 上流の `crl-tdu/crlBrain` の `docs/` を更新し、そこから本 repo へ同期してください。同期手順は上流の `docs/superpowers/plans/` 以下のプランに従います。

## 含まれるドキュメント

- `student-onboarding.md` — 共通導入（4 CLI 共通）
- `student-prereq.md` — 前提ガイド（ターミナル・git・SSH 鍵）
- `student-mode-{claude-code,gemini,copilot-cli,codex}.md` — CLI 別詳細
- `vision.md` — crlBrain の思想

## ローカルプレビュー

```bash
bundle install
bundle exec jekyll serve
# → http://localhost:4000/
```

## ライセンス

上流 repo のライセンスに従います。
