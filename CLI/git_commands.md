
# Git よく使うコマンド集

## 🔧 初期設定

```bash
git config --global user.name "Your Name"       # ユーザー名を設定
git config --global user.email "your@email.com" # メールアドレスを設定
```

---

## 🗂 リポジトリ操作

```bash
git init                # 現在のフォルダをGitリポジトリにする（初期化）
git clone <URL>         # 既存のリポジトリをクローン
```

---

## ✅ 変更の記録（コミット）

```bash
git status              # 現在の状態を確認
git add <file>          # ファイルをステージに追加
git add .               # すべての変更をステージに追加
git commit -m "message" # コミット（メッセージ付き）
```

---

## 📤 リモートとの連携

```bash
git remote add origin <URL> # リモートリポジトリを登録
git push -u origin main     # 初回の push（以後は `git push` だけでOK）
git push                    # ローカルの変更をリモートに送信
git pull                    # リモートの変更を取得してマージ
```

---

## 🕘 履歴・差分の確認

```bash
git log                      # コミット履歴を表示
git log --oneline            # 簡易表示
git diff                     # 変更内容を確認
git diff <branchA> <branchB> # ブランチ間の差分を確認
```

---

## 🌿 ブランチ操作

```bash
git branch                 # ブランチ一覧を表示
git branch <name>          # 新しいブランチを作成
git checkout <name>        # ブランチを切り替え
git switch <name>          # checkout の代替（新しめ）
git checkout -b <name>     # 新しいブランチを作って移動
git merge <name>           # 指定ブランチを現在のブランチに統合
```

---

## ♻️ 取り消し・巻き戻し

```bash
git restore <file>         # 変更を取り消す（ステージ前）
git reset HEAD <file>      # ステージから外す
git reset --hard HEAD      # ローカル変更をすべて破棄（危険）
```

---

## 🧠 その他便利系

```bash
git stash                  # 一時的に変更を退避
git stash pop              # 退避した変更を戻す
git show                   # 最後のコミットの内容を確認
```

---

## 📌 ワンポイントアドバイス

- `git status` と `git log` は頻繁に使うクセをつけましょう。
- 操作に不安があるときは `--dry-run` オプションで事前確認（例：`git push --dry-run`）するのがおすすめです。
