# WSL よく使うコマンド集

## 🔧 インストール・有効化

```bash
# WSL機能の有効化
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# WSLのインストール
wsl --install

# WSLのアンインストール
wsl --uninstall
```

---

## 🗂 WSLの操作

```bash
wsl              # wslの実行
wsl --shutdown   # wslの強制終了
```

---

## ✅ ディストリビューションの操作

```bash
# 例：Ubuntu-24.04
# ディストリビューションのインストール
wsl --install -d Ubuntu-24.04

# ディストリビューションのアンインストール
wsl --unregister Ubuntu-24.04

# ディストリビューションの起動
wsl -d Ubuntu-24.04

# 登録済ディストリビューションの状態とバージョン確認
wsl -l -v

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
