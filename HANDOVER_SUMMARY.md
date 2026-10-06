# joken.jp プロジェクト引き継ぎ・完全ステータスサマリー

**更新日時**: 2026年10月4日
**対象ドメイン**: `joken.jp` / `dash.joken.jp`（静岡県立科学技術高等学校 情報処理研究部）

---

## 1. プロジェクト基本方針

* **コア目的**: 部員の「何をしていいかわからない」を排除し、学びや試行錯誤を「部内ナレッジ（知）」として後輩へ引き継ぐ。3年生での進路・履歴書用「個人実績（ポートフォリオ）」を自動蓄積する。
* **セキュリティ・認証**: D1 `sessions` テーブル管理。`pending` ユーザーは投稿・管理APIへの書き込み遮断。
* **画像プロキシ (R2)**: 非公開 R2 バケット ＋ セッション認証プロキシ。WebP変換 ＋ `Cache-Control: private` で表示超高速化。
* **知の保護・競合防止**: 楽観的ロック（Optimistic Locking）による上書き事故防止。`post_revisions` テーブルによる履歴保持。
* **URL構造**: slug はシステム自動採番（`k-20261004-x8f2`）で固定。
* **検索システム**: タグ ＋ 分野 ＋ タイトルLIKE による複合検索。
* **リアクション**: `localStorage` 重複防止制御。
* **コンテンツ戦略**: 人間（部風・体験談）× AI（基礎解説・スタブ記事）の2大分類。

---

## 2. ディレクトリ構造 ＆ 成果物

```text
C:\Users\awaho\antigravity\joken\
│
├── PROJECT_BLUEPRINT.md        # ★マスター仕様書 & 開発ロードマップ（完全最終決定版）
├── HANDOVER_SUMMARY.md         # ★本引き継ぎサマリー
├── ARTICLE_IDEAS.md            # ★部内ナレッジ記事ネタ帳
│
├── 🌐 portal-site/             # 公開ポータル (1ページ完結向けに修正中)
│   ├── index.html              # TOPページ (UDデザイン・ニュース自動表示)
│   ├── about.html              # 活動紹介 (5つの特化班 / 参加大会表)
│   ├── contact.html            # お問い合わせ (GAS連携)
│   └── style.css
│
├── 🔒 system/                  # 部員認証モジュール
│   ├── schema.sql              # D1 テーブル定義 (users / sessions / posts / post_revisions)
│   ├── login-modal.html        # ログイン ＆ 承認申請UI
│   └── auth-api.js             # 認証機能 API
│
└── 📝 post-system/             # 部員専用ダッシュボード・グループウェア
    ├── dashboard.html          # サイドバー付き部員ポータル
    ├── calendar.js             # カレンダーの閲覧取得のAPI　←新規追加
    └── posts-api.js            # 記事投稿・一覧取得 API
```

---

## 3. 開発ロードマップ（進捗状況）

- [x] **Phase 1-1**: 公開ポータルサイト（index, about, contact）の文言・情報修正
- [x] **Phase 1-2**: 部内ポータルのコア方針決定（認証プロキシWebPキャッシュ、楽観的ロック、自動採番slug、`pending`制限、リビジョン履歴、Cronバックアップ）
- [ ] **Phase 1-3**: GitHub リポジトリ接続 & Cloudflare Pages デプロイ (`login.joken.jp` / `dash.joken.jp`)
- [ ] **Phase 2**: `sessions`認証、R2画像プロキシ、エディタ実装（.md/.pdf出力、楽観的ロック、更新・履歴保存）、Cronバックアップ API
- [ ] **Phase 3**: AI初期記事の量産 & 人間による1年生ガイド、マイページ実績PDF出力
- [ ] **Phase 4**: 外部向け1ページトップ（`joken.jp`）の完成

---

**備考**: 
本ファイルと `PROJECT_BLUEPRINT.md` が最新状態に更新されているため、別セッションや別AIモデルへの引き継ぎ時にも100%同等のクオリティで開発を継続可能です。
