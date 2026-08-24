# Laravel バグ対応 自走トレーニング Playbook

> 目的: AIに聞かずに「原因を自力で特定できる」状態になること。
> 覚えるべきは「コマンド」ではなく「絞り込みの手順」。コマンドは手順の道具でしかない。

---

## 0. まず結論: あなたの切り分け方の評価

提示された流れ:

> フロント / バック / インフラのどこか判定 → DevTools・ログ・プロセス確認で範囲を絞る → デバッガで特定

**方向性は正しい。実務のデバッグはほぼこの形。** ただし、そのままだと2つの穴があるので補正する。

### 補正1: 「領域の判定」より前に「事実収集と再現」が来る

いきなり領域を当てにいくと、症状の思い込みで外す。まず「何が起きているか」を確定させる。
再現できないバグは直せないし、直っても直った証明ができない。

### 補正2: 領域分類はゴールではなく、二分探索の第1手にすぎない

「フロント/バック/インフラ」は**粒度が粗すぎる**。実際に速いのは、
**リクエストのライフサイクル上を、正常に動いている境界線がどこかで二分探索する**やり方。

```
ブラウザ → ネットワーク → Webサーバ → PHP-FPM → Laravel → DB / 外部API
          ↑ ここまでは正常   ↑ ここから異常   ← この境界を探すゲーム
```

領域分類は「その二分探索を、最初にざっくり3分割している」だけだと理解しておく。

### 補正3: デバッガは「最後の手段」ではない

「範囲が絞れてから使う」は正しいが、絞れたら**すぐ**使う。
dd() を10回打つより、ブレークポイント1個の方が早い場面は多い。

### 補正後の全体フロー

```
0. 事実収集     — 何が / いつから / 誰に / どの操作で / 常にか稀か
1. 再現         — 手元で再現手順を固定する(できないなら再現条件を探すのが仕事)
2. 変更差分     — 直前に何を変えたか(デプロイ / .env / ライブラリ / データ / インフラ)
3. 領域の粗切り — フロント / バック / インフラ(症状カタログで当たりをつける)
4. 二分探索     — 正常/異常の境界を1段ずつ狭める(curl, tinker, 生SQL, ログ)
5. 特定         — デバッガ or ログで、変数の実値と分岐を目で見て確定させる
6. 修正         — 一度に1変数だけ変える
7. 再発防止     — テストを書く / ログを足す / 原因をメモに残す
```

---

## 1. デバッグの原則(これが本体)

技術より、こっちを体に入れる方が効く。

| 原則 | 意味 | 破ると起きること |
|---|---|---|
| **推測するな、計測せよ** | 「たぶんここ」で修正しない。必ず「見て」から直す | 直ったように見えて別の場所が壊れる |
| **二分探索** | 正常と異常の境界を毎回半分にする | 端から総当たりして時間が溶ける |
| **一度に1変数** | 修正も検証も1つずつ | 何が効いたのか永遠に分からない |
| **事実と解釈を分ける** | 「500が出る」は事実。「DBが原因」は解釈 | 解釈を事実だと思い込んで迷子になる |
| **変更点を疑う** | 昨日動いてたなら、原因は昨日からの差分の中にある | 無関係なコードを読み続ける |
| **エラーメッセージを最後まで読む** | 大半は答えが書いてある | 読んでれば5分で終わった案件になる |
| **再現を固定する** | 「たまに」を「必ず」に変換できたら半分終わり | 直ったのか偶然なのか判定不能 |
| **元に戻せる状態で触る** | git stash / ブランチ / DBバックアップ | 調査中に傷口を広げる |
| **時間を区切る** | 15〜30分詰まったら手法を変える(人に聞く/寝る/ログを増やす) | 同じ場所を3時間ループする |

### 「事実収集」で必ず聞く/確認する項目

- **What**: 期待した挙動は? 実際の挙動は?(エラー文言・画面のスクショ・HTTPステータス)
- **When**: いつから? 直前に何かデプロイ/設定変更はあったか
- **Who**: 全員? 特定ユーザーだけ? 特定権限だけ?
- **Where**: 本番だけ? ステージングは? ローカルは? 特定ブラウザ/端末だけ?
- **How often**: 毎回? 何回に1回? 特定の時間帯?
- **Scope**: 特定のデータ(ID)だけ? 全データ?

この5つが埋まると、それだけで容疑者はかなり絞れる。

---

## 2. 症状カタログ: この症状ならまずここを疑う

**最初の粗切りはこの表で十分。** 暗記推奨。

| 症状 | 第一容疑 | 最初に見る場所 |
|---|---|---|
| 画面が真っ白(何も出ない) | PHPの致命的エラー / メモリ切れ | `storage/logs/laravel.log`, PHP error_log |
| 500 Internal Server Error | アプリ例外 | `laravel.log` の最新スタックトレース |
| 502 Bad Gateway | PHP-FPM が落ちてる/繋がらない | nginx error.log, `systemctl status php8.x-fpm` |
| 504 Gateway Timeout | 処理が長すぎ | nginx timeout設定, slowlog, 重いSQL |
| 419 Page Expired | CSRFトークン / セッション | セッションドライバ, Cookie, `SESSION_DOMAIN` |
| 401 / 403 | 認証・認可 | ミドルウェア, Gate/Policy, トークン |
| 404 | ルーティング / URL | `php artisan route:list`, HTTPメソッド |
| 422 | バリデーション | FormRequest, レスポンスbodyの`errors` |
| CORSエラー | サーバ側のCORS設定 | `config/cors.php`, プリフライト(OPTIONS)の応答 |
| ボタン押しても何も起きない | JSエラー | DevTools Console |
| 画面が古いまま / CSSが効かない | キャッシュ / ビルド | ハードリロード, Viteビルド, `optimize:clear` |
| コードを直しても反映されない | 設定/OPcacheキャッシュ | `config:cache`済み, OPcache, worker再起動忘れ |
| 特定ユーザーだけ落ちる | データ起因 / 権限 | そのユーザーのレコードを tinker で確認 |
| たまに失敗する | 並行性 / 外部API / キュー / タイムアウト | 失敗時刻のログ突き合わせ, `failed_jobs` |
| 徐々に遅くなる | N+1 / インデックス欠如 / メモリリーク | Debugbar, Telescope, `EXPLAIN` |
| 本番だけ再現 | 環境差分 | `.env`, PHPバージョン, データ量, 権限 |
| メール/ジョブが飛ばない | キューworkerが死んでる | `supervisorctl status`, `failed_jobs` |

---

## 3. リクエストの流れの地図(これを頭に入れる)

切り分けとは、**この直線のどこで壊れたかを当てること**。

```
[1] ブラウザ(JS/フォーム)
     ↓ HTTPリクエスト
[2] DNS / ネットワーク / ロードバランサ
     ↓
[3] Nginx / Apache              ← 502/504/413/静的ファイル404はここ
     ↓ fastcgi
[4] PHP-FPM                     ← プロセス数枯渇・memory_limit・timeoutはここ
     ↓
[5] public/index.php → HTTP Kernel
     ↓
[6] Global Middleware           ← セッション, CSRF, TrustProxies
     ↓
[7] Routing                     ← 404, メソッド不一致
     ↓
[8] Route Middleware            ← auth, can, throttle → 401/403/429
     ↓
[9] FormRequest (バリデーション) ← 422
     ↓
[10] Controller → Service       ← ここが「自分のバグ」の本命
     ↓
[11] Eloquent / Query Builder   ← N+1, リレーション未ロード, スコープ
     ↓
[12] DB / 外部API / Redis       ← 接続エラー, タイムアウト, デッドロック
     ↓
[13] Response(Blade / JSON)     ← 未定義変数, null参照
     ↓
[14] ブラウザで描画 / JSで処理   ← 表示崩れ, JS例外
```

### 各段階で死んだときの「見え方」

| 段階 | 典型的な見え方 |
|---|---|
| [2] ネットワーク | DevTools Networkで `(failed)` / ERR_CONNECTION_REFUSED |
| [3] Nginx | Laravelのログに**何も残らない**まま 502/504/413 |
| [4] PHP-FPM | 502、またはPHPの `Allowed memory size exhausted` |
| [6][8] Middleware | Controllerに到達しない。dd()を置いても止まらない |
| [10] Controller | laravel.logにスタックトレースが出る(一番幸せなパターン) |
| [12] DB | `SQLSTATE[...]` を含む例外 |
| [13] View | `Undefined variable`, `Attempt to read property on null` |
| [14] JS | サーバは200を返しているのに画面がおかしい |

> **超重要な判定**: 「Laravelのログに何も出ていない」なら、**リクエストがLaravelに到達していない可能性が高い**([2]〜[4] か、そもそもリクエストが飛んでない[1])。

---

## 4. フロントエンド領域の切り分け

### 4.1 最初の3分岐

DevToolsを開いて、まずこの3つのどれかを確定させる。

1. **リクエストがそもそも飛んでいない** → JS側の問題(イベントが発火してない、JSエラーで止まった)
2. **リクエストは飛んだが、中身が間違っている** → JS側の問題(パラメータ、ヘッダ、URL)
3. **リクエストもレスポンスも正しいが、画面がおかしい** → 描画/JS処理の問題

→ **1と2ならフロント。レスポンスが期待と違うならバックエンドへ移送。**

### 4.2 Chrome DevTools 実務での使い方

#### Console タブ
- 赤いエラーを**一番上(最初のエラー)から**読む。後続は連鎖しているだけのことが多い
- `Preserve log` をON(画面遷移でログが消えるのを防ぐ)
- エラー右側のファイル名:行番号をクリック → Sourcesに飛べる

#### Network タブ(バグ調査の主戦場)
チェック順:

| 見る場所 | 何が分かる |
|---|---|
| そもそも行が出るか | JSがリクエストを送っているか |
| **Status** | 200/302/419/422/500/504 → 症状カタログへ |
| **Request URL** | URL・クエリが意図通りか(タイポ、末尾スラッシュ、http/https) |
| **Request Method** | POSTのつもりがGETになってないか |
| **Payload / Request** | 送っているデータが正しいか(空、型違い、キー名違い) |
| **Headers** | `X-CSRF-TOKEN`, `Authorization`, `Accept: application/json`, `Content-Type` |
| **Cookie** | セッションCookieが送られているか |
| **Response** | サーバが実際に返した生データ(Previewは整形後) |
| **Timing** | どこで時間を食っているか(Waiting=TTFBが長い→サーバ処理が重い) |

小技:
- 対象リクエストを右クリック → **Copy as cURL** → ターミナルに貼って実行
  → **ターミナルで再現するならフロントは無罪。バックエンドの問題。**
- `Disable cache` をON にして、キャッシュ由来を排除
- フィルタで `Fetch/XHR` に絞る

#### Sources タブ(JSのデバッガ)
- 行番号クリックでブレークポイント
- 右クリック → **Conditional breakpoint**(`id === 5` のときだけ止める)
- ステップ実行: F10(次の行) / F11(関数に入る) / F8(次のブレークポイントまで)
- 止まった状態でConsoleに変数名を打つと**その場のスコープの値**が見られる(これが強力)
- `Scope` パネルで変数の実値、`Call Stack` でどこから呼ばれたかを確認

#### Application タブ
- Cookies: セッションCookieの有無、Domain、Secure、SameSite
- Local/Session Storage: トークンの保存状態
- 「ログインしたのにログアウトされる」系はここ

#### Elements タブ
- 表示崩れ → 要素を選んで Styles で「どのCSSが勝っているか」(打ち消し線が負けたルール)
- そもそもDOMに要素が存在するのか(存在しない=描画ロジックの問題、存在するがCSSで消えている=CSSの問題)

### 4.3 Laravel構成別の見どころ

**Blade(サーバレンダリング)**
- 「表示がおかしい」= ほぼバックエンド。Controllerが渡した変数を疑う
- ページのソースを表示(Ctrl+U)して、HTMLの時点でおかしいのか確認
  → HTMLの時点でおかしい = サーバ側 / HTMLは正しいのに画面がおかしい = CSS/JS

**Vite (Laravel Mix)**
- CSS/JSが反映されない: `npm run dev` が動いているか、`npm run build` したか
- 本番で404: `public/build/manifest.json` が存在するか、デプロイに含まれているか
- `@vite` ディレクティブのパス指定ミス

**Livewire**
- Networkで `livewire/update` へのPOSTを見る。ペイロードにコンポーネントの状態が丸ごと入っている
- 更新されない → プロパティが public か、`wire:model` の名前一致、`wire:key` の付け忘れ

**Inertia / SPA**
- Networkの `X-Inertia` ヘッダ、返ってくるJSONの `props` を確認
- 「propsに入っていない」= Controller側の問題

### 4.4 フロント/バックの判定に使う最強コマンド

```bash
# DevToolsのCopy as cURLを貼るのが基本。手書きするなら:
curl -i -X POST https://example.com/api/orders \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer xxxxx" \
  -d '{"item_id":1,"qty":2}'

# -i でレスポンスヘッダも表示。-v でTLS/接続まで詳細表示
```

判定:
- **curlで再現する → フロントは無実。バックエンドへ**
- **curlでは正常 → フロントが送っている内容が違う。Payload/Headerを比較**

---

## 5. バックエンド領域の切り分け

### 5.1 ログを読む(最優先)

```bash
# 追いかけながら操作を再現する。これが基本動作
tail -f storage/logs/laravel.log

# 見やすくする
tail -n 200 storage/logs/laravel.log

# 特定の時刻・キーワードで検索
grep -n "SQLSTATE" storage/logs/laravel.log
grep -n "2026-08-20 14:" storage/logs/laravel.log

# 調査前にログを空にすると、再現時の出力だけが残って読みやすい
: > storage/logs/laravel.log
```

#### スタックトレースの読み方
1. **1行目のメッセージと例外クラス名**を読む(ここに答えが書いてあることが多い)
2. `at ...` の羅列の中から、**vendor/ ではない自分のコードの最初の行**を探す ← 実質の犯行現場
3. その行の変数の実値を確認しにいく

```
[2026-08-20 14:03:11] production.ERROR: Attempt to read property "name" on null
at /var/www/app/Http/Controllers/OrderController.php:42   ← ここを見る
#0 /var/www/vendor/laravel/framework/... (フレームワーク内部は基本スルー)
```

#### 自分でログを仕込む

```php
use Illuminate\Support\Facades\Log;

Log::debug('order create start', ['user_id' => $user->id, 'payload' => $request->all()]);
Log::error('payment failed', ['order_id' => $order->id, 'response' => $res->body()]);
```

- 第2引数(context)に変数を配列で渡すのが作法。文字列連結しない
- 出ない場合は `.env` の `LOG_LEVEL`(debugを出したいなら `LOG_LEVEL=debug`)
- チャンネル: `config/logging.php`。`Log::channel('slack')->error(...)` など

### 5.2 「どこまで到達しているか」を確かめる

一番原始的で一番速い方法。ライフサイクル上に目印を置いて二分探索する。

```php
Log::debug('=== middleware passed ===');   // ミドルウェアの後
Log::debug('=== controller entered ===');  // Controller冒頭
Log::debug('=== before save ===', $data);  // 保存直前
```

出力される最後の目印の直後が犯行現場。`dd()` でも同じことができるが、
Ajax・キュー・APIでは画面に出ないので**ログの方が確実**。

### 5.3 環境と設定の確認

```bash
php artisan about                 # 環境/ドライバ/キャッシュ状況を一望(まずこれ)
php artisan env                   # 現在のAPP_ENV
php artisan config:show database  # 実際に効いている設定値(.envではなく解決後の値)
php artisan route:list --path=orders
php artisan route:list --method=POST
php artisan migrate:status        # 未実行マイグレーションの有無
php artisan queue:failed          # 失敗ジョブ一覧
```

> `.env` を直接読むのではなく `config:show` を見る。
> `config:cache` 済みだと **.envの変更は無視される**ため、両者がズレる。これが事故の常連。

### 5.4 「反映されない」問題(Laravel最頻出のハマり)

```bash
php artisan optimize:clear   # config/route/view/cache/compiled を一括クリア
# 個別:
php artisan config:clear
php artisan route:clear
php artisan view:clear
php artisan cache:clear
php artisan queue:restart    # コード変更をworkerに反映(必須)
composer dump-autoload       # 新規クラスが見つからないとき
```

疑うもの:
- `config:cache` されている(→ .env 変更が効かない)
- OPcache(`opcache.validate_timestamps=0` の本番では PHP-FPM リロードが必要)
- キューworkerが**古いコードをメモリに抱えたまま**動いている
- ブラウザキャッシュ / CDN
- Viteのビルド成果物が古い

### 5.5 tinker(対話実行)

**HTTP層を丸ごと飛ばして、ロジックだけを検証できる**。切り分けの主力。

```bash
php artisan tinker
```

```php
// データの実態を確認
$u = App\Models\User::find(5);
$u->orders()->count();
$u->profile;               // null なら、View側の "on null" エラーの原因はこれ

// クエリの中身を目で見る
App\Models\Order::where('status','paid')->toSql();
App\Models\Order::where('status','paid')->toRawSql();  // 値を埋め込んだSQL(L11+)

// サービスクラスを直接叩く
app(App\Services\OrderService::class)->create($params);

// 設定値の確認
config('services.stripe.key');
```

判定:
- **tinkerで再現する → HTTP層(ルート/ミドルウェア/リクエスト)は無罪。ロジックかDK**
- **tinkerでは正常 → リクエストの入力値やミドルウェアが怪しい**

### 5.6 DBまわりの切り分け

#### 発行されたSQLを見る

```php
// AppServiceProvider@boot などに一時的に仕込む
DB::listen(function ($query) {
    Log::debug($query->sql, ['bindings' => $query->bindings, 'ms' => $query->time]);
});
```

```php
// 局所的に見る
DB::enableQueryLog();
$result = $service->run();
dd(DB::getQueryLog());
```

ツール: **Laravel Debugbar**(ローカルでクエリ本数・重複を可視化)、**Telescope**(リクエスト/クエリ/ジョブ/例外を後から追える)、**Clockwork**。

#### 判定
- 出たSQLをそのままDBクライアントで実行してみる
  - **生SQLでも期待と違う結果 → SQL(条件・JOIN)の問題**
  - **生SQLでは正しい → PHP側の加工・整形の問題**

#### よくあるDB起因バグ
| 症状 | 原因 |
|---|---|
| 一覧が異常に遅い | N+1(`with()` 忘れ)。Debugbarでクエリ数が件数に比例していたら確定 |
| 大量データでメモリ枯渇 | `all()`/`get()` で全件取得 → `chunk()` / `cursor()` / `lazy()` |
| 更新されない | `$fillable` 未設定でMass Assignment無視 |
| 保存したはずが消える | 例外でトランザクションがロールバック / `save()` の戻り値未確認 |
| たまにデッドロック | 同時更新。ロック順序、`lockForUpdate()` |
| 検索が遅い | インデックス欠如 → `EXPLAIN` で type=ALL(フルスキャン)を確認 |
| ソフトデリート絡みの不整合 | `withTrashed()` / グローバルスコープ |
| 日付がズレる | `APP_TIMEZONE` と DBのタイムゾーンの不一致 |

### 5.7 認証・セッション・CSRF

| 症状 | 確認 |
|---|---|
| 419 Page Expired | フォームに `@csrf` があるか / AjaxでX-CSRF-TOKENヘッダ / セッションドライバ(fileの権限、Redis接続)/ `SESSION_DOMAIN`, `SESSION_SECURE_COOKIE` |
| ログインが維持されない | Cookieが送られているか(Application タブ)、複数サーバでセッションが共有されていない(file→redis/dbへ) |
| APIで常に401 | `Authorization: Bearer` ヘッダ、`auth:sanctum` ミドルウェア、SPAなら `SANCTUM_STATEFUL_DOMAINS` と `withCredentials` |
| 403 | Policy/Gate。`Gate::allows()` を tinker で直接評価してみる |
| ログイン後リダイレクトループ | ミドルウェアの相互リダイレクト、`RedirectIfAuthenticated` |

### 5.8 キュー・スケジューラ

```bash
php artisan queue:work --once      # 手動で1件処理して例外を目で見る
php artisan queue:failed
php artisan queue:retry all
php artisan schedule:list
php artisan schedule:run           # 手動実行
supervisorctl status               # workerが生きているか
```

「ジョブが動かない」時の確認順:
1. `QUEUE_CONNECTION` が `sync` になっていないか(逆にローカルで動くのに本番で動かない典型)
2. workerプロセスが起動しているか
3. `jobs` テーブル / Redisにジョブが積まれているか(積まれてない=dispatch側の問題)
4. `failed_jobs` に例外が残っていないか
5. デプロイ後に `queue:restart` したか

### 5.9 デバッガ(Xdebug)で確定させる

範囲が絞れたらこれ。`dd()` を撒く作業より圧倒的に速い。

```ini
; php.ini
[xdebug]
zend_extension=xdebug
xdebug.mode=debug
xdebug.start_with_request=trigger   ; 常時ONにすると重い
xdebug.client_host=host.docker.internal  ; Dockerの場合
xdebug.client_port=9003
xdebug.idekey=VSCODE
```

VS Code の `.vscode/launch.json`(Dockerの場合の例):

```json
{
  "version": "0.2.0",
  "configurations": [{
    "name": "Listen for Xdebug",
    "type": "php",
    "request": "launch",
    "port": 9003,
    "pathMappings": { "/var/www/html": "${workspaceFolder}" }
  }]
}
```

使い方:
- 行番号左をクリックでブレークポイント
- **条件付きブレークポイント**: 右クリック → `$order->id === 42` のときだけ止める(ループ調査の必需品)
- **例外ブレークポイント**: 例外が投げられた瞬間に止める。「どこで例外が出たか分からない」に最強
- ステップオーバー / ステップイン / ステップアウト
- **Variables** で全変数の実値、**Watch** で式を常時監視、**Call Stack** で呼び出し経路
- 止まった状態で **Debug Console** に式を打って評価できる

`dd()` / `dump()` / `ray()` との使い分け:

| 手段 | 向いている場面 |
|---|---|
| `dd()` | 画面表示のある1箇所を即確認したい |
| `Log::debug()` | Ajax / API / キュー / 本番に近い環境 |
| Xdebug | 変数が多い、分岐が複雑、どこで死ぬか分からない |
| Telescope | 「あの時何が起きたか」を後から追う |

---

## 6. インフラ領域の切り分け

**Laravelのログに何も出ていないのに異常** なら、まずここ。

### 6.1 プロセスが生きているか

```bash
systemctl status nginx
systemctl status php8.3-fpm
systemctl status mysql
systemctl status redis

ps aux | grep php-fpm | head
supervisorctl status            # queue worker
docker compose ps               # Docker構成の場合
```

### 6.2 ポートが空いているか / 届くか

```bash
ss -lntp                        # 待ち受けポート一覧(何のプロセスが何番か)
ss -lntp | grep 9000            # php-fpm
lsof -i :80

# 到達性テスト
curl -I http://localhost
nc -zv db-host 3306             # DBへ届くか
ping db-host
dig example.com
```

### 6.3 Webサーバ(Nginx)

```bash
tail -f /var/log/nginx/error.log
tail -f /var/log/nginx/access.log
nginx -t                        # 設定の文法チェック
systemctl reload nginx
```

チェックポイント:
- `root` が `/path/to/public` を指しているか(プロジェクトルートを指すと事故)
- `fastcgi_pass` の向き先(unixソケット or 127.0.0.1:9000)が実際のFPMと一致
- `client_max_body_size`(アップロードで **413**)
- `fastcgi_read_timeout`(長い処理で **504**)
- `try_files $uri $uri/ /index.php?$query_string;`

| 症状 | 見るべき |
|---|---|
| 502 | error.logに `connect() failed` / `No such file or directory`(ソケットパス違い、FPM停止) |
| 504 | `upstream timed out` → タイムアウト値 or 処理が重い |
| 413 | `client_max_body_size` |
| 静的ファイルだけ404 | root設定、ファイル権限、シンボリックリンク(`storage:link`) |

### 6.4 PHP-FPM / PHP設定

```bash
php -v
php -m                          # 拡張が入っているか(pdo_mysql, gd, intl...)
php -i | grep memory_limit
php --ini                       # どのphp.iniが読まれているか(CLIとFPMで別物なので注意)
tail -f /var/log/php8.3-fpm.log
```

重要な設定値:

| 設定 | 症状 |
|---|---|
| `memory_limit` | `Allowed memory size exhausted` / 白画面 |
| `max_execution_time` | 途中で処理が切れる |
| `upload_max_filesize` / `post_max_size` | ファイルアップロード失敗、`$request->all()` が空 |
| `pm.max_children` | 高負荷時に `server reached pm.max_children` → 502/待ち行列 |
| `slowlog` / `request_slowlog_timeout` | 遅い処理のスタックトレースが取れる。パフォーマンス調査の切り札 |

> **CLI と FPM で php.ini が違う**。「artisanでは動くのにブラウザでは動かない」の原因常連。

### 6.5 権限・所有者

```bash
ls -la storage bootstrap/cache
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```

- 「ログが書けない」「viewキャッシュが作れない」→ 500 or 白画面
- `php artisan` を root で実行した後、www-data が書けなくなるのが典型パターン
- `php artisan storage:link` 忘れで画像404

### 6.6 リソース枯渇

```bash
df -h                  # ディスク使用率(100%でログもDBも死ぬ)
df -i                  # inode枯渇(ファイル数が多いと起きる)
free -h                # メモリ
top / htop
dmesg -T | grep -i "killed process"    # OOM Killer に殺されていないか
du -sh storage/logs/*  # ログが肥大化していないか
```

### 6.7 Docker特有

```bash
docker compose ps
docker compose logs -f app
docker compose logs --tail=100 nginx
docker compose exec app bash        # 中に入って確認
docker compose exec app php artisan about
```

ハマりどころ:
- **ホスト名**: `DB_HOST=127.0.0.1` ではなくサービス名(`db`, `mysql`)
- **ポートマッピング**: `3306:3306` のホスト側とコンテナ側の混同
- **ボリューム**: マウントされておらず、コンテナ内が古いコード
- **`host.docker.internal`**: Xdebug接続で必要(Linuxでは `extra_hosts` 追加)
- ビルドキャッシュで古いイメージのまま(`docker compose build --no-cache`)

### 6.8 環境差分の洗い出し(「本番だけ再現」の対処)

| 観点 | 確認 |
|---|---|
| `.env` | `APP_ENV`, `APP_DEBUG`, `APP_URL`, DB/キュー/メールのドライバ |
| PHPバージョン・拡張 | `php -v`, `php -m` を両環境で比較 |
| データ量・データ内容 | 本番だけNULLが存在する、桁が大きい、文字コード |
| キャッシュ状態 | `config:cache` の有無、OPcache設定 |
| タイムゾーン | `APP_TIMEZONE`, DBのtime_zone, OSのtimedatectl |
| 権限・ユーザー | 実行ユーザーが違う |
| HTTPS / プロキシ配下 | `TrustProxies`, `X-Forwarded-Proto` → リダイレクトがhttpになる |
| 外部サービス | 本番キー vs テストキー、IP制限 |

---

## 7. 領域境界の判定チートシート(暗記推奨)

| 質問 | Yes | No |
|---|---|---|
| curl(Copy as cURL)で再現する? | フロント無罪 → サーバ側へ | フロントが送る内容が違う |
| tinkerで同じ処理を呼んで再現する? | ロジック/DBの問題 | HTTP層(入力値・ミドルウェア)の問題 |
| 生SQLを直接叩いて同じ結果? | SQL/データの問題 | PHP側の加工の問題 |
| laravel.log に痕跡がある? | アプリ層 | Laravelに到達していない → Web/FPM/ネットワーク |
| ローカルで再現する? | コードの問題 | 環境差分 → インフラ/設定/データ |
| 特定ユーザー/レコードだけ? | データ起因(そのデータをtinkerで観察) | ロジック全般 |
| 毎回再現する? | ロジック | 並行性 / 外部依存 / キャッシュ / タイムアウト |
| 直前のコミットをrevertすると直る? | その差分が原因 | もっと前 or 環境要因 |

### 二分探索の実例(git bisect)

「いつからか分からないが昔は動いていた」場合、コード履歴を二分探索する。

```bash
git bisect start
git bisect bad                # 現在は壊れている
git bisect good v1.2.0        # このタグでは動いていた
# → 中間コミットがcheckoutされるので、動作確認して good / bad を答える
git bisect good   /   git bisect bad
# 繰り返すと原因コミットが特定される
git bisect reset
```

---

## 8. Laravel頻出バグ 原因→確認手順

| # | 症状 | ありがちな原因 | 確認 |
|---|---|---|---|
| 1 | 419 Page Expired | `@csrf` 忘れ / セッション保存先の不具合 / SESSION_DOMAIN不一致 | フォームHTML、Cookie、`storage/framework/sessions` の権限 |
| 2 | 500だがログが空 | ログ書き込み権限がない / 例外ハンドラより前で死んだ | `ls -la storage/logs`, nginx error.log, php-fpm.log |
| 3 | 白画面 | memory_limit超過 / 構文エラー / APP_DEBUG=false | `php -l file.php`, php error_log |
| 4 | 修正が反映されない | `config:cache` / OPcache / worker未再起動 | `php artisan about` のキャッシュ欄 |
| 5 | `Attempt to read property on null` | リレーションがnull、`first()` がnull | tinkerで該当データ確認、`?->` と存在チェック |
| 6 | 一覧が遅い | N+1 | Debugbarのクエリ数 → `with()` 追加 |
| 7 | `Class not found` | オートロード / 名前空間タイポ | `composer dump-autoload`, use文 |
| 8 | バリデーションが効かない | FormRequestの `authorize()` が false / ルールのキー名ミス | 422レスポンスのerrors、`authorize()` の戻り値 |
| 9 | 保存されない | `$fillable` 未定義 / トランザクションのロールバック | `$model->getDirty()`, 例外の有無 |
| 10 | 日時が9時間ズレる | `APP_TIMEZONE` とDBの不一致、UTC保存 | `config('app.timezone')`, DBの `SELECT NOW()` |
| 11 | メールが飛ばない | `MAIL_MAILER=log` / キュー未処理 | `storage/logs`, `failed_jobs` |
| 12 | 画像が404 | `storage:link` 忘れ / ディスク設定 | `ls -la public/storage` |
| 13 | 本番でCSSが崩れる | Viteビルド未実行 / manifest欠落 | `public/build/manifest.json` |
| 14 | ローカルは動くが本番だけ500 | `.env`差分 / 権限 / PHPバージョン | `php artisan about` を両方で比較 |
| 15 | リダイレクトがhttpになる | プロキシ配下でスキーム誤検出 | `TrustProxies`, `APP_URL`, `X-Forwarded-Proto` |
| 16 | 429 Too Many Requests | throttleミドルウェア | `route:list` でミドルウェア確認 |

---

## 9. 自走トレーニングメニュー

### ルール
- **AIに聞く前に必ず15分は自力で追う**。そして「聞く」ときも答えではなく**確認方法**を聞く
- 調査の途中経過を必ず**書きながら**やる(頭の中だけでやると同じ場所をループする)

### デバッグジャーナル(テンプレ)

毎回これを埋めながら調査する。埋まらない欄が、次に調べるべき場所。

```markdown
## 事象
- 期待:
- 実際:
- 再現手順:
- 再現率:      /10
- 環境:        local / stg / prod

## 事実(観測できたことだけ)
- HTTPステータス:
- laravel.log:
- Networkのリクエスト内容:

## 仮説と検証
| # | 仮説 | 検証方法 | 結果 |
|---|---|---|---|
| 1 |      |          |      |
| 2 |      |          |      |

## 原因
## 対処
## なぜ気付くのが遅れたか(次回の改善)
```

### 練習ドリル

1. **わざと壊す**: 動いているコードに1箇所バグを仕込み、翌日ログだけを頼りに特定する
2. **エラー暗記**: 419 / 500 / 502 / 504 / 422 / 403 について、「どの階層で誰が出したエラーか」を説明できるようにする
3. **ログ縛り**: `dd()` を封印し、`Log::debug()` とログ閲覧だけで原因特定する
4. **Xdebug縛り**: 逆に `dd()` を封印してブレークポイントだけで追う。ステップ実行に慣れる
5. **タイムアタック**: 症状を見てから「原因の階層」を宣言するまで3分。外れたら、なぜ外したかを記録
6. **フレームワークを1段潜る**: 例外が出たら vendor/ の中まで読んで、なぜその例外が投げられたかを追う
7. **git bisect練習**: 意図的に過去コミットにバグを仕込んで bisect で見つける
8. **障害訓練**: DBを止める / storage の権限を落とす / FPMを止める → それぞれの見え方をログとブラウザで観察して表にする(**症状カタログを自分の手で作る**のが最強の学習)

### 到達目標

- [ ] 症状を見た瞬間に「疑うべき階層」を2つ以内に挙げられる
- [ ] laravel.log を開いて、10秒で犯行現場の行番号を特定できる
- [ ] Copy as cURL で、フロント/バックの切り分けが即座にできる
- [ ] tinker でロジックを単体で叩いて検証できる
- [ ] Xdebugの条件付きブレークポイントを日常的に使える
- [ ] 502 / 504 / 白画面 を見て、Laravel以外を調べに行ける
- [ ] 「本番だけ再現」で、環境差分チェックリストを回せる
- [ ] 直したあとに再発防止テストを書ける

---

## 10. コマンド早見表

```bash
### ログ
tail -f storage/logs/laravel.log
: > storage/logs/laravel.log
tail -f /var/log/nginx/error.log
tail -f /var/log/php8.3-fpm.log
journalctl -u nginx -n 100 --no-pager

### Laravel 状態確認
php artisan about
php artisan route:list --path=xxx
php artisan config:show database
php artisan migrate:status
php artisan queue:failed
php artisan schedule:list

### キャッシュ
php artisan optimize:clear
php artisan queue:restart
composer dump-autoload

### 検証
php artisan tinker
php artisan queue:work --once
php -l path/to/file.php
curl -i -X POST url -H "Accept: application/json" -d '...'

### プロセス / ネットワーク
systemctl status nginx php8.3-fpm mysql
ps aux | grep php-fpm
ss -lntp
nc -zv db 3306
supervisorctl status

### リソース
df -h ; df -i ; free -h ; top
dmesg -T | grep -i "killed process"

### Docker
docker compose ps
docker compose logs -f app
docker compose exec app php artisan about

### Git
git log --oneline -20
git diff HEAD~1
git bisect start / bad / good
```

---

## 11. アンチパターン(これをやると自走できなくなる)

- **とりあえず直してみる**: 原因未特定のまま修正すると、直った理由が分からず再発する
- **複数箇所を同時に変える**: 何が効いたか分からなくなる
- **エラーメッセージを読まずに検索/AIに投げる**: 読解力が育たず、いつまでも自走できない
- **`try { } catch { }` で握りつぶす**: 情報を捨てる行為。最悪の対処
- **本番で `APP_DEBUG=true`**: 情報漏洩。ログを見る習慣で代替する
- **`dd()` を消し忘れて本番へ**: コミット前に `git diff` を必ず読む
- **再現手順を固定しないまま調査開始**: 直った判定ができない
- **ログを増やさずに悩む**: 情報が足りないなら情報を増やす。悩むのは情報を得た後
- **調査メモを取らない**: 30分後に同じ仮説を再検証している

---

## 12. 一枚でまとめると

```
症状を言語化 → 再現手順を固定 → 直前の変更を確認
        ↓
Laravelのログに痕跡がある?
  ├ ない  → Networkで到達確認 → nginx/php-fpm/プロセス/権限(インフラ)
  └ ある  → スタックトレースの自分のコード最初の行へ
              ↓
        curlで再現する? ─ No → フロント(送信内容/JS)
              ↓ Yes
        tinkerで再現する? ─ No → HTTP層(入力値/ミドルウェア/認証)
              ↓ Yes
        生SQLで再現する? ─ Yes → SQL/データ/インデックス
              ↓ No
        アプリのロジック → Xdebugで変数と分岐を目視 → 確定
              ↓
        1変数だけ修正 → 再現手順で検証 → テスト追加 → メモを残す
```

**デバッグは才能ではなく手順。手順を回した回数だけ速くなる。**