# WSL よく使うコマンド集

## 🔧 インストール・有効化

```bash
# WSL機能の有効化
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# WSLのインストール
wsl --install

# WSLのアンインストール
wsl --uninstall

# ディストリビューションのインストール
wsl --install --distribution Ubuntu-24.04
wsl --install -d Ubuntu-24.04  # 省略記法

# ディストリビューションのアンインストール
wsl --unregister Ubuntu-24.04
```

---

## 🗂 WSL、ディストリビューションの操作

```bash
# 例：Ubuntu-24.04

wsl              # wslの実行
wsl --shutdown   # wslの強制終了

# WSLの既定のバージョン、ディストリビューションを表示
wsl --status

# WSL2を規定値に変更
wsl --set-default-version 2

# ディストリビューションの起動
wsl -d Ubuntu-24.04

# ディストリビューションを強制終了（wslの強制終了でも可）
wsl --terminate Ubuntu-24.04

# WSL、ディストリビューションをリスト表示
wsl --list
wsl -l  # 省略記法

# WSL、ディストリビューションの詳細情報（状態、バージョン）をリスト表示
wsl --list --verbose
wsl -l -v  # 省略記法

# WSLのインストール可能なディストリビューションをリスト表示
wsl --list --online
wsl -l -o  # 省略記法

# デフォルトディストリビューションを設定
wsl --set-default Ubuntu-24.04
wsl -s Ubuntu-24.04  # 省略記法
```

## 💡 コマンドの処理や省略記法の有無を照会

```bash
# コマンドの処理や省略記法の有無を一覧表示
wsl --help
```