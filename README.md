# Construction of this Docker Project


## Directory structure

```script
root/
├── docker/
│   ├── app/
│   │   ├── apache2/
│   │   │   ├──000-default.conf  ...http:port:80用基本設定ファイル
│   │   │   ├──default-ssl.conf  ...https:port:443用基本設定ファイル
│   │   │   └──fqdn.conf  ..........ドメイン設定ファイル
│   │   ├── Dockerfile  ............PHP・Apache・node.js+npmの設定
│   │   ├── php.ini  ...............PHP設定ファイル
│   │   └── 000-default.conf  ......Apache設定ファイル
│   ├── db/  .......................Mysqlデータベース
│   │   ├── my.conf  ...............Mysql設定ファイル
│   │   ├── initdb/  ...............Mysql始動時に起動したいSQL格納
│   │   └── storage/  ..............Mysqlデータ保存用ディレクトリ
│   └── mailpit/  ..................メールサーバー（FakeSMTP）
│       └── Dockerfile  ............SSL、認証の設定（ローカル環境では不要）※
├── src/  ..........................laravelソースディレクトリ（直下に配置すること）
├── .env  ..........................docker-composeファイルの環境変数
├── docker-compose.yml
└── README. md
```

- ※備考：
    mailpitをDockerfileを用いてビルドするとき、Dockercomposeは以下の通りとなる；
    ``` docker-compose.yml
    mailpit:
        container_name: laravel_FSMTP
        build:
            context: .
            dockerfile: ./docker/mailpit/Dockerfile
        tty: true
        ports:
            - 8025:8025
            - 1025:1025
        environment:
            - MP_DATA_FILE: /home/mailpit/mails
            - MP_SMTP_SSL_CERT: /keys/cert.pem
            - MP_SMTP_SSL_KEY: /keys/privkey.pem
            - MP_SMTP_AUTH_FILE: /keys/.htpasswd
        volumes:
            - ./docker/mailpit:/home/mailpit/mails
            - ./keys:/keys
    ``` 


## Version manager

- Docker : 24.0.7
- Docker-compose : 2.23.3
- Dockerfile(app) ;
  - PHP : 8.1 - apache2.x
  - nodejs : 20.x
  - npm : 9.5.1
    - webpack : latest
  - xdebug : 3.2.2
  - OpenSSL : 3.0.8 *
  - 
- Mysql : 8.0
- mailpit : 1.9
- Inside the Container ;
  - laravel : 9.x
  - 


## Installation procedure (Cloning laravel project)

1. .envファイルにコンテナ名を設定
    -> ~_NAMEの箇所が要変更箇所
    -> PORT番号は必要に応じて変更すること

2. dockerイメージの生成
```bash
docker-compose build
# エラーが発生する場合はうしろに、 "--no-cache" を加えて実行する。
```

3. dockerコンテナ起動
```bash
docker-compose up
```

4. dockerコンテナ内に入る
```bash
docker exec -it test_react_app bash
# .envファイルで設定したAP_NAMEを記入して実行する。
```

5. laravelプロジェクトをクローンする
```bash
git clone git@{リポジトリサービス名}:{リポジトリ名}.git
# clone元からSSHコードをコピーすること
```

6. laravelプロジェクトのstorageディレクトリにRWX権限を付与
```bash
chmod -R 777 storage
```

7. php artisan初期処理
```bash
composer update

cp .env.example .env

php artisan key:generate

# シンボリックリンク
php artisan storage:link

# cache生成
php artisan optimize
# php artisan view:cache

# ※cache削除するときは；
php artisan optimize:clear

php artisan migrate
```

8. ルーティング自動反映のためにファイル削除(開発環境のみ)；
./bootstrap/cache/routes-v7.php

9. vite系(tailwindcss...)の適用について
ある程度新しいクラスを追加してから、適宜以下にて更新
```bash
npm run build
```


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
│   └── navigation   ..........ナビゲーションバー（プルダウン形式の場合は不要かもしれない）
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

