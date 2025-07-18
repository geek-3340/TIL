# envファイルとセキュリティ

## envとは
envとはenvironment(エンバイロンメント)の略で「環境」という意味

## envファイルの用途
プログラムが動くのに必要な設定を定義しておくファイル

例えば、、、

- アプリケーションの名前
- 動作環境（ローカル環境・本番環境 等）
- データベースがあるコンピューター
- データベースのパスワード

等、様々なアプリの動作に関わる設定が変数に格納されている　`（環境変数）`

envファイルの記載例
```env
APP_NAME=MyApp
APP_ENV=local
DB_HOST=localhost
DB_PASSWORD=secret
```

## なぜわざわざenvファイルに設定情報を書くのか
理由は主に２つ
1. ユーザーのログインパスワード、電子署名のプライベートキーなどの情報をコードに直接書くと、不正アクセス等の原因となり危険

2. 本番用、テスト用、自分のパソコン用で設定を切り替えたいとき、.envファイルの値だけ変えれば良く運用しやすい

## Githubにソースコードを公開するとき
.gitignoreファイルにて.envファイルを設定し、外部に漏らしたくない情報をリモートリポジトリへのコミット対象外にする

※チーム開発でGithubを用いる際は、.envの情報を別途連携する必要がある場合もある

## Laravelで環境変数を呼び出すとき
環境変数を`env()`で直接参照せずに`config()`を使って呼び出す

Laravelではアプリの公開直前や設定ファイル変更後に、キャッシュに設定情報を保存してアプリを速く・安定して動かす処理を行うことが多くあります

その際に使用するコマンドとして`sail artisan config:cache`がありますが、.envファイルはロードされない為**このコマンドを実行した後にenv()で環境変数を呼び出すとnullが返され結果アプリが正常に動作しません**
```bash
% sail artisan tinker
>>> env('SITE_LINK')
=> "https://sample_app.com"

% sail artisan config:cache
Configuration cache cleared successfully.
Configuration cached successfully.

% sail artisan tinker      
>>> env('SITE_LINK')
=> null
```

ではどのようにして.envファイル内の環境変数を参照するのか

前述のコマンドで.envファイルはセキュリティ上ロード対象外となるので、別の設定ファイル上で.envの環境変数を参照し、その設定ファイルを通して参照する必要があります

ここで使用するのが`config()`です

先ずはconfigディレクトリ配下にファイル（例：hoge.php）を作成し、以下のように環境変数を定義する
```php
<?php

return [
  'site_link' => env('SITE_LINK'),
];
```

configメソッドを使ってドット記法で呼び出す
```php
<?php

print config('hoge.site_link'); //"https://sample_app.com"
```

この方法を使用すればenvファイルの環境変数も事実上キャッシュされ参照可能となり、アプリも正常に動作します