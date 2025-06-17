# Laravel Sail よく使う Sail コマンド集

## 🚀 Sail プロジェクト初期化

```bash
curl -s https://laravel.build <project-name> | bash   # Laravel + Sail プロジェクトを作成
cd <project-name>
./vendor/bin/sail up                                  # コンテナを起動
```

---

## ⚙️ Sail 基本コマンド

```bash
./vendor/bin/sail up             # コンテナ起動（バックグラウンドなし）
./vendor/bin/sail up -d          # コンテナをバックグラウンドで起動
./vendor/bin/sail down           # コンテナを停止・削除
./vendor/bin/sail restart        # コンテナを再起動
./vendor/bin/sail stop           # コンテナを停止
```

---

## 💻 アプリケーション操作

```bash
./vendor/bin/sail artisan        # Artisan コマンドの実行
./vendor/bin/sail artisan migrate    # マイグレーションの実行
./vendor/bin/sail php            # PHP を実行
./vendor/bin/sail composer       # Composer コマンドの実行
./vendor/bin/sail npm install    # npm インストール（Node関連の構築）
```

---

## 🧪 テスト・開発補助

```bash
./vendor/bin/sail test           # Laravel のテストを実行
./vendor/bin/sail node -v        # Node.js バージョン確認
./vendor/bin/sail bash           # コンテナ内のシェルに入る
```

---

## 🧠 その他便利系

```bash
alias sail="./vendor/bin/sail"  # sail コマンドを短縮（推奨）

sail up -d                      # 上記エイリアスを使うと簡略化
sail artisan migrate:fresh --seed   # DB初期化＋シーディング
sail share                     # ngrok経由で外部公開（ngrokが有効な場合）
```

---

## 📌 ワンポイントアドバイス

- プロジェクト直下で `alias sail="./vendor/bin/sail"` を `.bashrc` などに追加しておくと快適。
- Sail は Docker を前提にしているため、Docker Desktop が起動している必要があります。
- Laravel のバージョンやサービス構成（MySQL/PostgreSQL/Redisなど）は `docker-compose.yml` で柔軟に調整可能。
