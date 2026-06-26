# Webスケジュール管理アプリ 仕様書

## 1. 目的

PHP、JavaScript、SQLite を使い、ブラウザから予定を登録・閲覧・編集・削除できる Web スケジュール管理アプリを構築する。

初版では小規模チーム利用を主目的とし、以下を重視する。

- 月・週・日単位で予定を確認できる
- 予定の登録、編集、削除が簡単にできる
- 自分の予定とチーム共有予定を管理できる
- SQLite の単一DBファイルで導入しやすい
- PHP の標準機能と素の JavaScript を中心に実装できる

## 2. 想定ユーザー

- 小規模チーム内で予定を共有したいユーザー
- 社内やローカル環境に軽量な予定管理ツールを設置したい管理者
- チームメンバーの予定を確認しながら自分の予定を管理したいユーザー

## 3. 対象環境

### 3.1 サーバー

- PHP 8.2 以上
- SQLite 3
- PDO SQLite 拡張
- Apache
- 現在ディレクトリを Apache の公開ディレクトリとして利用
- HTTPS 環境を推奨

### 3.2 クライアント

- 最新版の Chrome、Edge、Firefox、Safari
- スマートフォンを優先し、PCとタブレットにもレスポンシブ対応
- JavaScript 有効環境

## 4. 技術構成

| 領域 | 採用技術 | 役割 |
| --- | --- | --- |
| サーバーサイド | PHP | 画面配信、API、認証、DB操作 |
| フロントエンド | HTML、CSS、JavaScript | カレンダーUI、フォーム、非同期通信 |
| データベース | SQLite | ユーザー、予定、共有情報の保存 |
| 通信 | Fetch API、JSON | 画面とAPI間のデータ送受信 |
| セッション | PHP Session | ログイン状態の保持 |

## 5. 主要機能

### 5.1 ユーザー機能

- ユーザー登録
- ログイン
- ログアウト
- パスワードハッシュ化保存
- ログイン中ユーザーの予定のみ表示
- 誰でもユーザー登録可能

### 5.2 スケジュール表示

- 月表示
- 週表示
- 日表示
- 今日へ移動
- 前月・翌月、前週・翌週、前日・翌日への移動
- 予定タイトル、開始時刻、終了時刻の表示
- 終日予定の表示
- 色分け表示

### 5.3 予定管理

- 予定登録
- 予定編集
- 予定削除
- 予定詳細表示
- 開始日時、終了日時の指定
- 終日予定の指定
- タイトル、説明、場所、色、公開範囲の指定
- 繰り返し予定は初版では対象外とし、将来拡張に回す
- 祝日表示、予定通知は初版では対象外とし、将来拡張に回す

### 5.4 検索・絞り込み

- キーワード検索
- 日付範囲検索
- 色またはカテゴリによる絞り込み
- 自分の予定のみ、共有予定のみの切り替え

### 5.5 共有機能

- 初版では以下の公開範囲を持つ。
- 共有範囲はログインユーザー全員のみとし、特定ユーザー・グループ指定は初版では扱わない。

| 公開範囲 | 説明 |
| --- | --- |
| private | 作成者のみ閲覧可能 |
| shared | ログインユーザー全員が閲覧可能 |

- 共有予定の編集・削除は作成者のみ可能とする。

### 5.6 管理機能

初版では管理者機能は実装対象外とし、将来拡張に回す。

## 6. 画面仕様

### 6.1 ログイン画面

URL: `/login.php`

#### 表示項目

- メールアドレス入力欄
- パスワード入力欄
- ログインボタン
- ユーザー登録画面へのリンク
- エラーメッセージ表示領域

#### 振る舞い

- 認証成功時は `/index.php` に遷移する。
- 認証失敗時は同一画面にエラーメッセージを表示する。
- ログイン済みユーザーがアクセスした場合は `/index.php` にリダイレクトする。

### 6.2 ユーザー登録画面

URL: `/register.php`

#### 表示項目

- 名前
- メールアドレス
- パスワード
- パスワード確認
- 登録ボタン

#### 振る舞い

- メールアドレスは一意とする。
- パスワードは 8 文字以上とする。
- 初版では誰でも登録可能とする。
- 登録直後のロールは `user` とする。
- 登録後はログイン画面に遷移する。

### 6.3 カレンダー画面

URL: `/index.php`

#### 表示項目

- ヘッダー
  - アプリ名
  - ログインユーザー名
  - ログアウトボタン
- 操作バー
  - 今日ボタン
  - 前へボタン
  - 次へボタン
  - 月・週・日表示切替
  - 新規予定ボタン
  - 検索入力
- カレンダー本体
- 予定詳細モーダル
- 予定登録・編集モーダル

#### 振る舞い

- 初期表示は月表示とする。
- 表示期間に該当する予定を API から取得する。
- 日付または時間帯をクリックすると新規予定モーダルを開く。
- 予定をクリックすると詳細モーダルを開く。
- 編集権限がある予定のみ編集・削除ボタンを表示する。

### 6.4 管理画面

初版では管理画面を実装しない。将来拡張で追加する場合は `/admin/index.php` 配下に配置する。

## 7. API仕様

API は `/api/` 配下に配置し、JSON を返す。

### 7.1 共通仕様

#### リクエスト

- 認証が必要な API は PHP Session によってログイン状態を確認する。
- 更新系 API は CSRF トークンを検証する。
- リクエストボディは `application/json` を基本とする。

#### レスポンス成功例

```json
{
  "ok": true,
  "data": {}
}
```

#### レスポンス失敗例

```json
{
  "ok": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "入力内容を確認してください。"
  }
}
```

### 7.2 予定一覧取得

`GET /api/events.php`

#### クエリ

| 名前 | 必須 | 説明 |
| --- | --- | --- |
| start | 必須 | 取得開始日。ISO 8601形式 |
| end | 必須 | 取得終了日。ISO 8601形式 |
| keyword | 任意 | タイトル・説明・場所の検索文字列 |
| visibility | 任意 | `private` または `shared` |

#### レスポンス

```json
{
  "ok": true,
  "data": [
    {
      "id": 1,
      "title": "定例会議",
      "description": "週次の進捗確認",
      "location": "会議室A",
      "start_at": "2026-07-01T10:00:00+09:00",
      "end_at": "2026-07-01T11:00:00+09:00",
      "all_day": false,
      "color": "#2563eb",
      "visibility": "shared",
      "owner_id": 1,
      "can_edit": true
    }
  ]
}
```

### 7.3 予定詳細取得

`GET /api/event.php?id={id}`

### 7.4 予定作成

`POST /api/event_create.php`

#### リクエスト

```json
{
  "title": "定例会議",
  "description": "週次の進捗確認",
  "location": "会議室A",
  "start_at": "2026-07-01T10:00:00+09:00",
  "end_at": "2026-07-01T11:00:00+09:00",
  "all_day": false,
  "color": "#2563eb",
  "visibility": "shared"
}
```

### 7.5 予定更新

`POST /api/event_update.php`

#### リクエスト

```json
{
  "id": 1,
  "title": "定例会議",
  "description": "週次の進捗確認",
  "location": "会議室A",
  "start_at": "2026-07-01T10:00:00+09:00",
  "end_at": "2026-07-01T11:00:00+09:00",
  "all_day": false,
  "color": "#2563eb",
  "visibility": "shared"
}
```

### 7.6 予定削除

`POST /api/event_delete.php`

#### リクエスト

```json
{
  "id": 1
}
```

## 8. データベース設計

DBファイル: `database/schedule.sqlite`

### 8.1 users

ユーザー情報を保存する。

| カラム | 型 | 制約 | 説明 |
| --- | --- | --- | --- |
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | ユーザーID |
| name | TEXT | NOT NULL | 表示名 |
| email | TEXT | NOT NULL UNIQUE | メールアドレス |
| password_hash | TEXT | NOT NULL | パスワードハッシュ |
| role | TEXT | NOT NULL DEFAULT 'user' | `user` または `admin` |
| is_active | INTEGER | NOT NULL DEFAULT 1 | 1: 有効、0: 無効 |
| created_at | TEXT | NOT NULL | 作成日時 |
| updated_at | TEXT | NOT NULL | 更新日時 |

### 8.2 events

予定情報を保存する。

| カラム | 型 | 制約 | 説明 |
| --- | --- | --- | --- |
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 予定ID |
| owner_id | INTEGER | NOT NULL | 作成者ID |
| title | TEXT | NOT NULL | タイトル |
| description | TEXT | NULL | 説明 |
| location | TEXT | NULL | 場所 |
| start_at | TEXT | NOT NULL | 開始日時 |
| end_at | TEXT | NOT NULL | 終了日時 |
| all_day | INTEGER | NOT NULL DEFAULT 0 | 1: 終日、0: 時刻指定 |
| color | TEXT | NOT NULL DEFAULT '#2563eb' | 表示色 |
| visibility | TEXT | NOT NULL DEFAULT 'private' | `private` または `shared` |
| created_at | TEXT | NOT NULL | 作成日時 |
| updated_at | TEXT | NOT NULL | 更新日時 |
| deleted_at | TEXT | NULL | 論理削除日時 |

#### インデックス

```sql
CREATE INDEX idx_events_owner_id ON events(owner_id);
CREATE INDEX idx_events_start_end ON events(start_at, end_at);
CREATE INDEX idx_events_visibility ON events(visibility);
```

### 8.3 audit_logs

監査ログを保存する。

| カラム | 型 | 制約 | 説明 |
| --- | --- | --- | --- |
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | ログID |
| user_id | INTEGER | NULL | 操作ユーザーID |
| action | TEXT | NOT NULL | 操作種別 |
| target_type | TEXT | NOT NULL | 対象種別 |
| target_id | INTEGER | NULL | 対象ID |
| detail_json | TEXT | NULL | 詳細JSON |
| created_at | TEXT | NOT NULL | 作成日時 |

## 9. 初期DDL

```sql
PRAGMA foreign_keys = ON;

CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL DEFAULT 'user',
  is_active INTEGER NOT NULL DEFAULT 1,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  CHECK (role IN ('user', 'admin'))
);

CREATE TABLE events (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  owner_id INTEGER NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  location TEXT,
  start_at TEXT NOT NULL,
  end_at TEXT NOT NULL,
  all_day INTEGER NOT NULL DEFAULT 0,
  color TEXT NOT NULL DEFAULT '#2563eb',
  visibility TEXT NOT NULL DEFAULT 'private',
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  deleted_at TEXT,
  FOREIGN KEY (owner_id) REFERENCES users(id),
  CHECK (all_day IN (0, 1)),
  CHECK (visibility IN ('private', 'shared'))
);

CREATE TABLE audit_logs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER,
  action TEXT NOT NULL,
  target_type TEXT NOT NULL,
  target_id INTEGER,
  detail_json TEXT,
  created_at TEXT NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_events_owner_id ON events(owner_id);
CREATE INDEX idx_events_start_end ON events(start_at, end_at);
CREATE INDEX idx_events_visibility ON events(visibility);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at);
```

## 10. ディレクトリ構成案

現在ディレクトリをそのまま Apache で公開する前提とする。公開不要な `src/`、`database/`、`docs/` は `.htaccess` で直接アクセスを拒否する。

```text
/
  .htaccess
  index.php
  login.php
  register.php
  logout.php
  assets/
    css/
      app.css
    js/
      app.js
      calendar.js
      events-api.js
  api/
    events.php
    event.php
    event_create.php
    event_update.php
    event_delete.php
  src/
    Auth.php
    Database.php
    EventRepository.php
    UserRepository.php
    Csrf.php
    Response.php
    Validator.php
  database/
    schema.sql
    schedule.sqlite
  docs/
    specification.md
```

`.htaccess` には最低限、次のアクセス拒否を設定する。

```apache
RedirectMatch 403 ^/(src|database|docs)(/.*)?$
```

## 11. バリデーション仕様

### 11.1 ユーザー

| 項目 | ルール |
| --- | --- |
| 名前 | 必須、1から50文字 |
| メールアドレス | 必須、メール形式、重複不可 |
| パスワード | 必須、8文字以上 |

### 11.2 予定

| 項目 | ルール |
| --- | --- |
| タイトル | 必須、1から100文字 |
| 説明 | 任意、最大2000文字 |
| 場所 | 任意、最大200文字 |
| 開始日時 | 必須、有効な日時 |
| 終了日時 | 必須、有効な日時、開始日時以降 |
| 終日 | 必須、真偽値 |
| 色 | 必須、HEXカラー形式 |
| 公開範囲 | 必須、`private` または `shared` |

## 12. 認証・認可仕様

### 12.1 認証

- パスワードは `password_hash()` で保存する。
- ログイン時は `password_verify()` で検証する。
- ログイン成功時は `session_regenerate_id(true)` を実行する。
- ログアウト時はセッションを破棄する。

### 12.2 認可

| 操作 | 条件 |
| --- | --- |
| private 予定の閲覧 | 作成者本人 |
| shared 予定の閲覧 | ログインユーザー |
| 予定作成 | ログインユーザー |
| 予定編集 | 作成者本人 |
| 予定削除 | 作成者本人 |

## 13. セキュリティ要件

- SQL は PDO のプリペアドステートメントで実行する。
- HTML 出力時は `htmlspecialchars()` でエスケープする。
- 更新系リクエストは CSRF トークンを検証する。
- セッションCookieには `HttpOnly`、`Secure`、`SameSite=Lax` を設定する。
- ログイン試行の連続失敗に対する簡易レート制限を設ける。
- エラーメッセージには内部情報を含めない。
- SQLite DBファイルは `database/` に配置し、Apache から直接ダウンロードできないよう `.htaccess` でアクセス拒否する。
- `src/` と `docs/` も直接アクセスを拒否する。

## 14. フロントエンド仕様

### 14.1 UI方針

- 業務利用に適したシンプルで静かなデザインにする。
- 操作ボタンは明確なアイコンとラベルを併用する。
- 予定の色は視認性を優先する。
- スマートフォンを主対象にし、片手操作しやすい配置と読みやすい予定一覧を優先する。
- スマートフォンでは月表示の情報量を抑え、日別一覧を併用する。

### 14.2 JavaScriptモジュール

| ファイル | 役割 |
| --- | --- |
| `app.js` | 初期化、共通イベント、画面状態管理 |
| `calendar.js` | 月・週・日表示の描画 |
| `events-api.js` | API通信 |

### 14.3 状態管理

ブラウザ側では以下の状態を保持する。

- 現在の表示日
- 表示モード。`month`、`week`、`day`
- 検索キーワード
- 表示中の予定一覧
- 編集中の予定

## 15. エラー処理

| 種別 | 表示・処理 |
| --- | --- |
| 入力エラー | フォーム項目の近くにメッセージを表示 |
| 認証エラー | ログイン画面へ遷移 |
| 認可エラー | 403 メッセージを表示 |
| 通信エラー | 画面上部に再試行可能な通知を表示 |
| サーバーエラー | 汎用エラーを表示し、詳細はログに記録 |

## 16. ログ仕様

### 16.1 監査ログに記録する操作

- ログイン成功
- ログイン失敗
- ログアウト
- 予定作成
- 予定更新
- 予定削除

### 16.2 PHPエラーログ

- 本番環境では画面にエラー詳細を表示しない。
- PHP の `error_log` に詳細を出力する。

## 17. テスト方針

### 17.1 単体テスト

- バリデーション
- 認証処理
- 予定の権限判定
- Repository のDB操作

### 17.2 結合テスト

- ユーザー登録からログインまで
- 予定の作成、表示、編集、削除
- private 予定と shared 予定の閲覧権限

### 17.3 手動確認

- スマートフォン表示
- PC表示
- 月表示、週表示、日表示
- 日本語入力
- 不正な日時入力
- セッション切れ時の動作

## 18. MVP範囲

最初に実装する最小機能は以下とする。

1. ユーザー登録、ログイン、ログアウト
2. 月表示カレンダー
3. 予定一覧取得
4. 予定作成
5. 予定編集
6. 予定削除
7. private、shared の公開範囲
8. SQLite schema 作成

初版では以下は実装対象外とする。

- 繰り返し予定
- 祝日表示
- 予定通知
- 特定ユーザー・グループへの限定共有
- 管理者機能

## 19. 将来拡張

- 繰り返し予定
- 祝日表示
- 通知・リマインダー
- iCalendar インポート・エクスポート
- Googleカレンダー連携
- 管理者機能
- チーム、グループ、部署単位の共有
- 添付ファイル
- 予定への参加者登録
- ドラッグアンドドロップによる予定移動
- ダークモード

## 20. 未確定事項・確認したい内容

詳細仕様を固めるため、次の項目は確認が必要。

### 20.1 確定済み

| 項目 | 内容 |
| --- | --- |
| 主目的 | チーム用 |
| ユーザー登録 | 誰でも登録可能 |
| 繰り返し予定 | 初版では不要 |
| 祝日表示 | 初版では不要 |
| 予定通知 | 初版では不要 |
| 予定共有 | 全員共有のみ |
| 優先画面 | スマートフォン優先 |
| 管理者機能 | 初版では不要 |
| デザイン | シンプルな業務向けデザイン |
| サーバー配置 | 現在ディレクトリを Apache で公開 |

### 20.2 未確定

現時点で未確定事項はない。

