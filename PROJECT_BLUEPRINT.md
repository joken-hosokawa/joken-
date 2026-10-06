# 静岡県立科学技術高等学校 情報処理研究部 (joken.jp)
## サイト全体構成 ＆ 開発マスターブループリント（完全最終決定版）

本ドキュメントは、`joken.jp` および部内ナレッジポータル `dash.joken.jp` の全構成・UIデザイン方針・DB仕様・機能詳細・アクセス制御・セキュリティ・BCP仕様を定めたマスター仕様書です。

---

## 1. コアコンセプト ＆ 解決する課題

### 💡 核心目的
1. **部員の「何をしていいかわからない」の排除**
   * 「1年生向けスタートガイド」や先輩の「HowTo・作例」を蓄積し、雑談で終わらせない道しるべを作る。
2. **学びと試行錯誤の資産化（知は力なり）**
   * 個人の環境構築メモ、エラー解決手順、Paiza/JOI/Blender等の学習ログを部内ナレッジとして継承する。
3. **個人実績（ポートフォリオ）の自動蓄積**
   * 3年生での進学・就職（履歴書・面接）時に「高校3年間で何を学んだか」を証明する実績ログとしてそのまま活用する。

---

## 2. 🎨 全体デザインシステム（デジタル庁 UD デザイン準拠）

* **デジタル庁デザインシステム (Universal Design)** に準拠し、`style.css` で共通変数・コンポーネントを定義。
* **タイポグラフィ**: `Noto Sans JP` + `Inter`（高コントラスト・読みやすさ重視）

---

## 3. 🔒 認証 ＆ セキュリティ ＆ アクセス制御仕様

### ① セッション管理 ＆ ユーザー承認制御
* **`sessions` テーブル**: ログイン成功時 `session_token` を発行し Cookie/Header で保持。
* **ユーザー `pending` 制御**:
  * アカウント申請直後（`status = 'pending'`）のユーザーはログイン可能だが、記事投稿・API書き込み・管理権限へのアクセスを遮断。

### ② 画像アクセス制御 ＆ 高速化 (R2プロキシ)
* **セキュリティ完全保護**:
  * R2 バケットはパブリック非公開（プライベート）。
  * Pages Functions プロキシ API（`/api/cdn/*`）で「ログイン済みセッション」を検証して画像を配信。
* **高速化対策**:
  * プロキシレスポンスに `Cache-Control: private, max-age=86400` を設定し、2回目以降はブラウザキャッシュで一瞬で表示。
  * 画像は自動的に `.webp` フォーマットに変換して保存。

---

## 4. 📝 記事投稿エディタ ＆ 機能詳細仕様

### ① エディタ ＆ 画像ドラッグ＆ドロップフロー
* **エディタ挿入フロー**:
  * 画像をドロップ ➔ R2 へ送信 ➔ 返ってきた `/api/cdn/xxx.webp` の Markdown コード（`![説明](URL)`）をエディタに自動挿入。
* **書式 ＆ 出力**: 太字・下線・斜体・打ち消し・見出し (`h1`〜`h4`)・目次 (TOC) 自動生成・コード構文ハイライト。
* **エクスポート**: `.md` ダウンロード ＆ **`.pdf`（A4印刷フォーマット）保存**。

### ② 下書き ＆ 同時編集競合対策 ＆ 履歴管理
* **下書き自動保存 (`localStorage`)**: 執筆中テキストの自動退避。
* **楽観的ロック（Optimistic Locking）による競合防止**:
  * 編集開始時の `updated_at` を保持し、保存時にDB側と不一致があれば「他のユーザーが更新済みです」と警告して上書き事故を防止。
* **記事の共同編集 ＆ リビジョン履歴管理**:
  * 修正時に `post_revisions` テーブルに過去バージョンを保持。過去の版へ即時復元可能。

### ③ slug（URL識別子）生成ルール
* タイトル変更によるリンク切れや文字化けを防ぐため、slug はタイトルに依存させず **`post-123` や `k-20261004-x8f2` のようにシステム側で自動採番・固定**（ユーザー変更不可）。

### ④ 承認待ち（`pending`）記事の閲覧制限
* 一般の一覧・検索 API では `WHERE status = 'published' AND is_deleted = 0` を強制適用。
* `pending` 状態の記事は、投稿者本人と管理者のみプレビュー可能。

### ⑤ リアクション機能 ＆ 重複防止
* 連打防止のため、`localStorage` に「自分がリアクションした記事ID」を記録し、ボタンを非活性化。

### 目次
目次をサイト側で自動生成する。←新規追加

---

## カレンダー機能 ←新規追加
現在スプレッドシートで管理している部活動計画をwebサイトに統合。見やすく、分かりやすく、管理しやすく（最も重要。スプレッドシートから乗り換えるレベルの利便性が必要。）を重点に置いた開発。
どんな設計になるかは要検討。ダッシュボードアクセスしたときその日にやるべきことが、一発でわかるように。
---

## 5. 🛡️ プチBCP（データ保全・一方通行バックアップ・焚書防止）

1. **GitHub への一方通行定期バックアップ（Cron Trigger）**:
   * API Limit 回避のため、1日1回〜3日に1回の Cron バッチ処理で GitHub へ一方通行自動コミット（編集はサイト側オンリー）。
2. **焚書防止（ソフトデリート ＋ 履歴復元）**:
   * 削除操作を行ってもDB上は論理削除（`is_deleted = 1`）とし、`post_revisions` からいつでも復元可能。
3. **ワンクリック完全引き継ぎパッケージ**:
   * 管理画面から全ユーザー・全ナレッジデータを ZIP 一括エクスポート。

---

## 6. 🗄️ データベース設計 (Cloudflare D1 - SQLite)

### ユーザーテーブル (`users`)
```sql
CREATE TABLE IF NOT EXISTS users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  display_name TEXT NOT NULL,
  team TEXT NOT NULL,
  role TEXT DEFAULT 'member',
  status TEXT DEFAULT 'pending',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### セッション管理テーブル (`sessions`)
```sql
CREATE TABLE IF NOT EXISTS sessions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  session_token TEXT UNIQUE NOT NULL,
  expires_at DATETIME NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### ナレッジ・記事・ログテーブル (`posts`)
```sql
CREATE TABLE IF NOT EXISTS posts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  author_id INTEGER NOT NULL,
  title TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,       -- システム自動採番・固定 (例: k-20261004-x8f2)
  content TEXT NOT NULL,           -- 本文 (Markdown)
  category TEXT NOT NULL,          -- 'guide' / 'knowledge' / 'log'
  field TEXT NOT NULL,             -- 分野: 'programming' / 'infrastructure' / 'creative' / 'cert'
  tags TEXT,                       -- カンマ区切りタグ (#C言語, #Blender など)
  status TEXT DEFAULT 'pending',   -- 'draft' / 'pending' / 'published'
  is_ai_generated INTEGER DEFAULT 0, -- 0: 人間作成, 1: AI生成スタブ記事
  is_deleted INTEGER DEFAULT 0,    -- 0: 生存, 1: 論理削除 (焚書防止)
  reactions TEXT DEFAULT '{}',     -- リアクションJSON {"like": 5, "fire": 2}
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (author_id) REFERENCES users(id)
);
```

### 記事変更履歴・リビジョンテーブル (`post_revisions`)
```sql
CREATE TABLE IF NOT EXISTS post_revisions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  post_id INTEGER NOT NULL,
  editor_id INTEGER NOT NULL,      -- 修正を行った部員のID
  title TEXT NOT NULL,
  content TEXT NOT NULL,           -- 修正時点のMarkdown本文
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
  FOREIGN KEY (editor_id) REFERENCES users(id)
);
```

---

## 7. 🚩 開発ロードマップ (Development Roadmap)

### Phase 1: 共通CSS ＆ クラウドデプロイ基盤
* [x] デジタル庁 UD デザイン準拠 `style.css` の策定
* [ ] GitHub リポジトリ (`joken-jp`) への構成整理と push
* [ ] Cloudflare Pages デプロイ & D1 / R2 バインディング設定

### Phase 2: エディタ・認証・R2画像プロキシ ＆ 履歴管理・BCP
* [ ] `sessions` テーブルによるログイン状態保持 API 実装（`pending` 制限付）
* [ ] R2画像認証プロキシ API（`/api/cdn/*` - WebP変換・Cache-Control設定）
* [ ] リッチMarkdownエディタ（自動slug採番・.md/.pdf保存・下書き保存）
* [ ] 楽観的ロックによる同時編集競合防止 ＆ `post_revisions` 履歴復元機能
* [ ] 重複防止付きリアクション機能
* [ ] 「タグ ＋ 分野 ＋ タイトルLIKE」検索機能
* [ ] Cron Trigger による GitHub 一方通行バックアップ API

### Phase 3: AI初期記事の量産 ＆ ガイド・マイページ充実
* [ ] AIプロンプトを使った「初期シード記事」の量産（`is_ai_generated = 1`）
* [ ] 人間が書く「1年生向けスタートガイド」初期コンテンツ作成
* [ ] マイページ (`/profile`) での活動ログ・実績一覧画面（PDF出力対応）

### Phase 4: 外部ポータル (`joken.jp`) 統合 ＆ 完全引き継ぎテスト
* [ ] 外部向け `index.html` の1ページ完結仕上げ
* [ ] 1クリック完全バックアップZIPエクスポート検証

---

**更新日時**: 2026年10月6日
**プロジェクト**: 静岡県立科学技術高等学校 情報処理研究部 ナレッジポータル
