# まほうの図書館 — PLAN.md

## プロダクト概要

子供の読書習慣を楽しく育てる読書記録PWA。
「5冊読む → スタンプ1個 → スタンプが貯まったらごほうび」というシンプルなゲームループが核。

---

## ゲームループ（確定仕様）

```
5冊読む
  └→ スタンプ1個獲得（アプリ内演出）
       └→ 本トラックリセット、また5冊へ
            └→ スタンプが親設定数に達する
                 └→ ごほうびゲット（親が設定したメモを表示）
```

---

## ターゲット・運用

| 項目 | 内容 |
|---|---|
| 対象 | 読書習慣をつけたい子供を持つ親（一般公開） |
| 操作者 | 親子どちらでも使う |
| 認証 | メール＋パスワード |
| 子供 | 1アカウントに複数プロフィール（兄弟姉妹対応） |
| プラットフォーム | PWA（iOS/Androidホーム画面追加） |

---

## 技術スタック

| レイヤー | 技術 |
|---|---|
| フロントエンド | HTML / CSS / Vanilla JS（シングルファイル） |
| バックエンド | Supabase（DB + 認証） |
| ホスティング | GitHub Pages または Netlify |
| スタイル | 魔法少女トーン（M PLUS Rounded 1c + Dancing Script） |

---

## DBスキーマ（v1確定）

### `users`
```sql
id            uuid PK
email         text
display_name  text
created_at    timestamp
```

### `children`
```sql
id              uuid PK
user_id         uuid FK → users
name            text
avatar_emoji    text
level           int       DEFAULT 1
xp              int       DEFAULT 0
total_books     int       DEFAULT 0
current_streak  int       DEFAULT 0
longest_streak  int       DEFAULT 0
last_read_date  date
stamp_count     int       DEFAULT 0   -- 累計スタンプ数（リセットしない）
stamp_goal      int       DEFAULT 5   -- 何個でごほうび（親設定）
reward_memo     text                  -- ごほうびの内容（親メモ）
created_at      timestamp
```

### `books`
```sql
id          uuid PK
child_id    uuid FK → children
title       text
author      text
pages       int
created_at  timestamp
updated_at  timestamp
```

### `reading_logs`
```sql
id          uuid PK
child_id    uuid FK → children
book_id     uuid FK → books
impression  text
rating      int        -- 1〜5
read_at     timestamp
created_at  timestamp
updated_at  timestamp
```

> **バッジ・ストリークはchildrenに統合**（独立テーブル不要、MVP簡略化）

---

## UI構成（現在の実装）

### 画面一覧
| 画面 | 状態 |
|---|---|
| 子供ダッシュボード | ✅ 実装済み（yomyom-magical-v4.html） |
| ログイン / 新規登録 | ⬜ 未実装 |
| 子供プロフィール追加 | ⬜ 未実装 |
| 親設定（スタンプ目標・ごほうびメモ） | ⬜ 未実装 |

### ダッシュボード構成（上から順）
1. **ナビバー** — `✦ Yomyom ✦` ロゴ ＋ 子供切り替え
2. **ヒーローカード** — 子供名・称号 ＋ 5冊トラック（魔法少女の本イラスト5枚）
3. **スタンプカード** — 現在周回の進捗 ＋ ごほうびプレビュー ＋ 消費済み回数
4. **ストリークエリア** — 連続日数 / 最高記録 / 総冊数
5. **よんだほん** — 最新3冊表示 ＋「もっと見る」展開

---

## MVP機能スコープ

### ✅ 含む（MVP）
- [ ] ユーザー認証（メール＋パスワード）
- [ ] 子供プロフィール追加・切り替え（複数対応）
- [ ] 本の記録（タイトル・著者・感想・星評価）
- [ ] 本の編集・削除
- [ ] 5冊ゴール進捗UI（魔法少女本イラスト）
- [ ] スタンプ獲得演出（紙吹雪）
- [ ] スタンプカード（使用済みグレーアウト）
- [ ] ごほうびゴール達成演出
- [ ] 連続ログインストリーク表示
- [ ] よんだ本リスト（最新3件 ＋ もっと見る）
- [ ] 親設定（スタンプ目標数・ごほうびメモ）
- [ ] PWAマニフェスト・Service Worker

### ⬜ v2以降
- バッジコレクション（冊数・連続・スタンプ達成）
- 親ダッシュボード（子供の記録一覧）
- 読書統計グラフ
- プッシュ通知（読み忘れリマインド）

---

## ゲームデザイン

### レベル称号
| レベル | 称号 | 条件（総冊数） |
|---|---|---|
| 1 | よみはじめ | 0〜4 |
| 2 | ほしよみ師 | 5〜9 |
| 3 | まほうよみ | 10〜19 |
| 4 | ほんのまじょ | 20〜29 |
| 5 | でんせつの読み手 | 30〜 |

### スタンプ設計
- **累計カウント方式**（リセットしない）
- `stamp_count % stamp_goal` = 現在周回の進捗
- `Math.floor(stamp_count / stamp_goal)` = ごほうび獲得回数
- UIでは「獲得済み/未獲得/使用済み」を色分け

---

## 実装ステップ

```
STEP 1  Supabase テーブル作成 SQL
STEP 2  認証画面（ログイン / 新規登録）
STEP 3  supabase-js 組み込み・認証フロー
STEP 4  子供プロフィール CRUD
STEP 5  本の記録 CRUD（reading_logs）
STEP 6  ストリーク自動更新ロジック
STEP 7  スタンプ判定・ごほうびフロー
STEP 8  親設定画面（stamp_goal / reward_memo）
STEP 9  PWAマニフェスト・Service Worker
STEP 10 デプロイ（GitHub Pages / Netlify）
```

---

## ファイル構成（予定）

```
/
├── index.html          # メインアプリ（シングルファイル）
├── manifest.json       # PWAマニフェスト
├── sw.js               # Service Worker
└── assets/
    └── books/          # 魔法少女本イラスト（5枚）
```

---

## デザイントーン

- **参考**: magical-girl-design-guide.md
- **カラー**: ミルクピンク(#FFF5F8) / ビビッドローズ(#FF6BA8) / ラベンダー(#C87DE8)
- **フォント**: M PLUS Rounded 1c（本文）/ Dancing Script（ロゴ）
- **特徴**: グラデーションボタン・オフセット影・カード半透明・浮遊パーティクル・クリックスパークル
- **本トラック**: 魔法少女の本イラスト5枚（読了=カラー・次=金グロー・未読=グレースケール）
