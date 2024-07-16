# Docker Container for Laravel9.x


## Directory structure

```script
root/
├── docker/
│   ├── app/
│   │   ├── apache2/
│   │   │   ├──000-default.conf  ... http:port:80用基本設定ファイル
│   │   │   ├──default-ssl.conf  ... https:port:443用基本設定ファイル
│   │   │   └──fqdn.conf  .......... ドメイン設定ファイル
│   │   ├── Dockerfile  ............. PHP・Apache・node.js+npmの設定
│   │   ├── php.ini  ................ PHP設定ファイル
│   │   └── 000-default.conf  ....... Apache設定ファイル
│   ├── db/  ......................... Mysqlデータベース
│   │   ├── my.conf  ................ Mysql設定ファイル
│   │   ├── initdb/  ................ Mysql始動時に起動したいSQL格納
│   │   └── storage/  ............... Mysqlデータ保存用ディレクトリ
│   └── mailpit/  .................... メールサーバー（FakeSMTP）
│       └── Dockerfile  .............. SSL、認証の設定（ローカル環境では不要）※
├── src/  ............................. laravelソースディレクトリ（直下に配置すること）
├── .env  ............................. dockerコンテナ環境設定
├── docker-compose.yml
└── README. md
```


## Version manager
- Docker : 24.0.7
- Docker-compose : 2.23.3
- Dockerfile(app) ;
  - container env ;
    -  gnupg :                      GNU Privacy Guard、暗号化・署名の為のパッケージ（dockerイメージ構築時に必要）
    -  npm :                        npm command
    -  ca-certificates :            これも認証関係
    -  curl :                       データの送受信
    -  git :                        Git
    -  vim :                        Vimエディタ
    -  zip :                        zip関連ユーティリティ
    -  unzip :                      zip解凍ユーティリティ
    -  libzip-dev :                 zipアーカイブの読取・作成・変更する開発ヘッダ
    -  libfreetype6-dev :           画像処理ライブラリ拡張モジュール；TrueTypeフォントサポート(GD2版)->pngを含むので、libpngは不要
    -  libjpeg62-turbo-dev :        画像処理ライブラリ拡張モジュール；JPEGサポート(GD2版)
    -  libpng-dev :                 画像処理ライブラリ拡張モジュール；PNGサポート
    -  libonig-dev :                Oniguruma正規表現ライブラリ開発ヘッダ
    -  libssl-dev :                 OpenSSLライブラリ関連ヘッダ
    -  default-libmysqlclient-dev : MySQL開発ヘッダ
    -  libmagickwand-dev :          ImageMagick画像編集ライブラリヘッダ
  - PHP : 8.1 - apache2.x
  - nodejs : 20.x
  - npm : 9.5.1
  - xdebug : 3.2.2
- Mysql : 8.0
- mailpit : 1.9
- Inside the Container ;
  - laravel : 9.x


## Installation procedure (Cloning laravel project)
1. git clone
```bash
git clone https://${user_name}:${token}@github.com/${org/user_name}/${this_repository_name}.git .
# 末尾にドット(.)を付けること
```

2. .env設定
```bash
cp .env.example .env
# name, portは必要に応じて変更
```

3. dockerコンテナ起動
```bash
docker-compose up --build
# エラーが発生する場合はうしろに、 後ろにオプション(--no-cache) を付ける
# バックグラウンド起動する場合は、後ろにオプション(-d)を付ける
```

4. laravelプロジェクトをクローンする
```bash
cd src/
sudo chmod 777 .
git clone https://${user_name}:${token}@github.com/${org/user_name}/${repository_name}.git .
# 末尾にドット(.)を付けること
```

5. dockerコンテナ内に入る
```bash
docker exec -it ${APP_NAME} bash
# .envファイルで設定したAP_NAMEを記入して実行する。
```

6. laravelプロジェクトのstorageディレクトリにRWX権限を付与
```bash
chmod -R 777 storage
```

7. laravel初期処理
```bash
composer update

cp .env.example .env

php artisan key:generate

# シンボリックリンク
php artisan storage:link

# cache生成
php artisan optimize

# ※cacheを全て削除するときは；
php artisan optimize:clear
# ※画面が更新されないときは；
php artisan route:cache

php artisan migrate

npm i
# vite 起動
npm run dev
```

> ※うまくルーティングができないときは；以下のファイル削除；
>>./bootstrap/cache/routes-v7.php


## Laravel blade components directory structure
```script
! ".blade.php"は表記省略しています

resources/views/
├── components/
│   │   -----呼び出し先Blade（以下、子コンポーネント／孫コンポーネントと表記）-----
│   ├── layouts/
│   │   ├── global   ..........非ログイン時のページ土台
│   │   └── private   .........ログイン時のページ土台
│   ├── auth/
│   │   ├── session-status   ..
│   │   └── 
│   ├── modules/   ............＜動的ではないモジュールを格納、タイプによるif分岐をすることでファイル数を抑える＞
│   │   ├── button   ..........ボタン系（特に不要？）
│   │   └── modal   ...........モーダルウィンドウを表示させる（成功・失敗・警告など）
│   ├── app-logo   ............アプリケーションロゴ（svg形式）
│   ├── header   ..............ヘッダー
│   ├── footer   ..............フッター
│   └── navigation   ..........ナビゲーションバー
├── auth/
│   ├── register   ............初期登録
│   ├── login   ...............ログイン
│   ├── forgot-password   .....パスワードリセットメール送信ページ
│   ├── reset-password   ......パスワードリセットページ
│   └── verify-email   ........メール認証送信完了ページ（兼再送依頼）
│
│   -----呼び出し元Blade（以下、親コンポーネントと表記）-----
├── profile/
├── */   ......................各種ページを格納する場所
├── main   ....................メイン画面（ログアウト後のリダイレクト先）
└── home   ....................ログイン時のホーム画面
```

- コンポーネント呼び出し方例：

```php
1.
親=main
    <x-layouts.user>content</x-layouts.user>
子=components/layouts/user
    {{ $slot }}

2.
親=main
    <x-layouts.user title="hoge">content</x-layouts.user>
    or
    <x-layouts.user>
        <x-slot name="title">
            hoge
        </x-slot>
        content
    </x-layouts.user>
子=components/layouts/user
    {{ $title }}
    ->タイトルが表示される

3.
親=main
    <x-layouts.user>
        <x-slot name="header">(ここでの内容は、子にて{{ $header }}で反映される)</x-slot>
        content
    </x-layouts.user>
    ※必ず</x-slot>を書くこと！
子=components/layouts/user
    @if (isset($header))
        <x-header />
    @endif
孫=components/header
    ->ファイル内容が子に呼び出される
```
