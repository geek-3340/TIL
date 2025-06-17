# npm よく使うコマンド集

## 🛠 初期設定

```bash
npm init              # 対話形式で package.json を作成
npm init -y           # デフォルト設定で package.json を自動作成
```

---

## 📦 パッケージのインストール

```bash
npm install <package>       # パッケージをインストール
npm install <package>@<ver> # 特定のバージョンをインストール
npm install                 # package.json に基づいてすべての依存をインストール
```

---

## 💨 開発用パッケージのインストール

```bash
npm install --save-dev <package> # 開発用（devDependencies）としてインストール
```

---

## 🧹 アンインストール・更新

```bash
npm uninstall <package>     # パッケージを削除
npm update                  # すべてのパッケージを更新
npm outdated                # アップデート可能なパッケージを表示
```

---

## 🚀 スクリプト実行

```bash
npm run <script>            # package.json に定義されたスクリプトを実行
npm start                   # start スクリプトを実行（run は省略可）
npm test                    # test スクリプトを実行
```

---

## 🧪 パッケージの調査

```bash
npm list                    # インストール済みパッケージを表示（ローカル）
npm list -g                 # グローバルにインストールされたパッケージを表示
npm info <package>          # パッケージの詳細情報を表示
```

---

## 🌐 グローバル操作

```bash
npm install -g <package>    # グローバルにパッケージをインストール
npm uninstall -g <package>  # グローバルパッケージをアンインストール
```

---

## 🔧 その他便利系

```bash
npm ci                      # lockファイルに厳密に従ってインストール（CI向け）
npm audit                   # 脆弱性のスキャン
npm audit fix               # 自動で修正できる脆弱性を修正
npm cache clean --force     # npm キャッシュを強制的にクリア
```

---

## 📌 ワンポイントアドバイス

- グローバルにインストールしすぎると環境差異の原因になるので注意。
- `npm ci` は CI/CD 環境での高速かつ一貫性のあるインストールにおすすめ。
- `npx` を使えばグローバルにインストールせずに一時的にコマンド実行可能。

---

## ⚡ Vite の活用例

```bash
npm create vite@latest     # Vite プロジェクトを作成（対話形式で選択）
cd <project-name>
npm install                # 依存関係をインストール
npm run dev                # 開発サーバーを起動
npm run build              # 本番用にビルド
npm run preview            # 本番ビルドのプレビューをローカルで確認
```

---

## 📌 Vite 利用時のポイント

- `npm create vite@latest` を使うと素早くプロジェクトを立ち上げられる。
- TypeScript / React / Vue などの選択が可能なので、プロジェクトに応じてカスタマイズ可能。
- `vite.config.js` で設定を柔軟に変更できる（例：エイリアス、ポート、プラグイン等）。
- 開発中はホットリロード（HMR）が非常に高速で快適。
