# Under Construction of Docker Projects


## Installation procedure (Initializing laravel project)

0. 開発中...
   - 常時SSL化
   - MailPitのdockerfile運用
        mailpitをDockerfileを用いてビルドするとき、Docker-composeは以下の通りとなる；
        ``` docker-compose.yml
        mailpit:
            container_name: ${FSMTP_NAME}
            build:
                context: .
                dockerfile: ./docker/mailpit/Dockerfile
            tty: true
            ports:
                - ${MAIL_PORT:-8025}:8025
                - ${SMTP_PORT:-1025}:1025
            environment:
                - MP_DATA_FILE: /home/mailpit/mails
                - MP_SMTP_SSL_CERT: /keys/cert.pem
                - MP_SMTP_SSL_KEY: /keys/privkey.pem
                - MP_SMTP_AUTH_FILE: /keys/.htpasswd
            volumes:
                - ./docker/mailpit:/home/mailpit/mails
                - ./keys:/keys
        ``` 

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
docker exec -it ${APP_NAME} bash
# .envファイルで設定したAP_NAMEを記入して実行する。
```

5. laravelプロジェクトを新規作成する
```bash
composer create-project "laravel/laravel=~9.0" --prefer-dist .
# 同時に下記が実行される；
# composer install
#cp .env.example .env
# php artisan key:generate
```

6. laravelプロジェクトのstorageディレクトリにRWX権限を付与
```bash
chmod -R 777 storage
```

7. php artisan初期処理
```bash
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

9. .env及び.env.exampleの設定変更
```env
APP_NAME=任意の名称
DB_HOST=docker-composeのdbコンテナ名(デフォルト：test_db)
DB_DATABASE=docker-composeのdbコンテナ名(デフォルト：test_db)
DB_USERNAME=docker-composeにて設定したユーザー名(デフォルト：user)
DB_PASSWORD=docker-composeにて設定したパスワード(デフォルト：pwd)
```

10. app.php編集
```php:/config/app.php
'timezone' => 'Asia/Tokyo',
'locale' => 'ja',
'faker_locale' => 'jp_JP',
```


## laravel first setup (root directory = src/)

1. ライブラリ・パッケージインストール
    - vite   .............Vite
        1. npm install
        
        2. configを少々修正
        ```js:/vite.config.js
        plugins: [
        // 略
        ],
        server: {
            host: true,
            hmr: {
                host: 'localhost',
            },
            watch: {
                usePolling: true,
            }
        },
        ```
        
        3. app.jsをちょこっと修正
        ```js:/resources/js/app.js
        // 略
        import '../css/app.css';
        ```
        
    
    - fortiry   ..........認証バックエンド
        1. fortifyインストール
        ```bash
        composer require laravel/fortify
        php artisan vendor:publish --provider="Laravel\Fortify\FortifyServiceProvider"
        ```
        
        2. app.phpのプロバイダに以下追加
        ```php:/config/app.php
        'providers' => [
            // 略
            /*
             * Fortify Service Providers...
             */
            App\Providers\FortifyServiceProvider::class,
        ],
        ```
        
        3. fortify.phpで各認証機能のON/OFFを切替（jetstreamと併用しない場合は、上３つのみが推奨）
        ```php:/config/fortify.php
        'features' => [
            Features::registration(),               // ユーザー登録
            Features::resetPasswords(),             // パスワードリセット
            Features::emailVerification(),          // メールアドレス確認
            // Features::updateProfileInformation(),   // 登録情報の更新
            // Features::updatePasswords(),            // パスワード更新
            // Features::twoFactorAuthentication([     // 二要素認証
                // 'confirm' => true,
                // 'confirmPassword' => true,
                // 'window' => 0,
            // ]),
        ],
        ```
        
        4. ビュー用のルーティングを無効化＆URLにプレフィックスを追加
        ```php:/config/fortify.php
        'prefix' => 'api',
        
        'views' => false,
        ```
        
        5. 各認証機能のカスタマイズ
        app/Actions/Fortifyにある各ファイルを修正可能
        CreateNewUser.php
        → ユーザー登録用
        PasswordValidationRules.php
        → パスワードのバリデーションルールが書かれたファイルで、各クラスの中でトレイトで呼ばれています。
        ResetUserPassword.php
        → パスワードリセット用
        UpdateUserPassword.php
        → パスワード更新用
        UpdateUserProfileInformation.php
        → 登録情報の更新用
        
        6. ログインビューを返す方法を指示
        ```php:/app/Providers/FortifyServiceProvide.php
        
        use Laravel\Fortify\Contracts\LogoutResponse;
        
        public function boot(): void
        {
            // 登録view
            Fortify::registerView(function () {
                return view('auth.register');
            });
            
            // ログインview
            Fortify::loginView(function () {
                return view('auth.login');
            });
            
            // パスワード忘れたview
            Fortify::requestPasswordResetLinkView(function () {
                return view('auth.forgot-password');
            });
            
            // パスワードリセットview
            Fortify::resetPasswordView(function (Request $request) {
                return view('auth.reset-password', ['request' => $request]);
            });
            
            // メール認証view
            Fortify::verifyEmailView(function () {
                return view('auth.verify-email');
            });
            
            // パスワード認証view
            Fortify::confirmPasswordView(function () {
                return view('auth.confirm-password');
            });
        }
        ```
        
        
    
    - 認証系バリデーションメッセージ日本語化
        1. 前提
        先述のapp.php編集において、言語周辺を"ja"に変更していること。
        
        2. インストール
        ```bash
        php -r "copy('https://readouble.com/laravel/8.x/ja/install-ja-lang-files.php', 'install-ja-lang.php');"
        php -f install-ja-lang.php
        php -r "unlink('install-ja-lang.php');"
        ```
        
        3. メッセージ修正
        ```php:/resources/lang/ja/validation
        // 略
        'custom' => [
            'terms' => [
                'required' => '登録には規約への同意が必須となります。',
            ],
        ],
        
        // 略
        
        'attributes' => [
            'name' => 'ユーザー名',
            'email' => 'メールアドレス',
            'password' => 'パスワード',
            'terms' => '規約',
        ],
        ```
        
        4. langディレクトリ直下にja.jsonを追加、最低以下の７つを追加(バリデーションメッセージ)
        ```json:/resources/lang/ja.json
        {
            "The :attribute must be at least :length characters and contain at least one number.": ":attribute は :length 文字以上で、1つ以上の数字が必要です",
            "The :attribute must be at least :length characters and contain at least one special character.": ":attribute は :length 文字以上で、1つ以上の記号が必要です",
            "The :attribute must be at least :length characters and contain at least one uppercase character and one number.": ":attribute は :length 文字以上で、1つ以上の大文字と数字が必要です",
            "The :attribute must be at least :length characters and contain at least one uppercase character and one special character.": ":attribute は :length 文字以上で、1つ以上の大文字と記号が必要です",
            "The :attribute must be at least :length characters and contain at least one uppercase character, one number, and one special character.": ":attribute は :length 文字以上で、1つ以上の数字と記号が必要です",
            "The :attribute must be at least :length characters and contain at least one uppercase character.": ":attribute は :length 文字以上で、1つ以上の大文字が必要です",
            "The :attribute must be at least :length characters.": ":attribute は :length 文字以上でなければなりません",
        }
        ```
        5. 各bladeの記述を英語⇒日本語の多言語対応にする場合；
        ```php:/resources/views/auth/login.php
        /*
         * 二重波括弧＋アンダーバー2本＋括弧＋シングルクォーテーションで記述。
         * ja.jsonに対応している言葉を記述している必要があります。
        */
        {{ __('Login') }}
        {{ __('Enter your e-mail.') }}
        ```
        
    
    - tailwind   .........CSSコンポーネント
        1. インストール
        ```bash
        npm install -D tailwindcss postcss autoprefixer
        npx tailwindcss init -p
         # ページネーションが必要な場合；
        php artisan vendor:publish --tag=laravel-pagination
        ```
        
        2. コンフィグの設定
        ```js:/tailwind.config.js
        content: [
            './storage/framework/views/*.php',
            './resources/**/*.blade.php',
            './resources/**/*.js',
            './resources/**/*.vue',
            "./vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php",
        ],
        ```
        
        3. スタイルシートの設定
        ```css:/resources/css/app.css
        @tailwind base;
        @tailwind components;
        @tailwind utilities;
        ```
        
        4. htmlのhead内にスタイル適用
        ```php:/resources/views/layout/app.blade.php
        <head>
            // 略
            @vite(['resources/css/app.css', 'resources/js/app.js'])
        </head>
        ```
        
        5. dockerの構造上devが動作しないので、新しいclassを追加した後に、以下更新を随時行う
        ```bash
        npm run build
        ```
    
    - daisyUI   ..........CSSライブラリ
        1. インストール
        ```bash
        npm i -D daisyui@latest
        ```
        
        2. tailwindコンフィグに追加
        ```js:/tailwind.config.js
        plugins: [require("daisyui")],
        ```
    
    - Vue   ..............フロントエンド
        1. viteでvue.jsを使用するためのプラグインをインストール
        ```bash
        npm install @vitejs/plugin-vue --save-dev
        ```
        
        2. configに設定を追加
        ```js:vite.config.js
        // 略
        import vue from "@vitejs/plugin-vue";
        
        // 略
            plugins: [
                vue(),
                
        // 略
        ```
        
        3. app.cssまたはbootstrap.cssに記述することで反映される
        
    
    - mobile middleware
        1. Middlewareを作成
        ```bash
        php artisan make:Middleware GetIsMobile
        ```
        
        2. 作成したMiddlewareを編集
        ```php:/app/Http/Middleware/GetIsMobile.php
        
        ```
        
        3. カーネルに追加
        ```php:/app/Http/Kernel.php
        protected $middleware = [
            \App\Http\Middleware\CheckForMaintenanceMode::class,
            // 略
            \App\Http\Middleware\GetIsMobile::class,
        ```
        
        4. viewで使用
        ```php:any.blade.php
        @if($isMobile)
            //モバイルだけで行う処理
        @endif
        
        @if(!$isMobile)
            //モバイル以外だけで行う処理
        endif
        ```



## Under Investigating...

### SSL接続
1. OpeSSLを使用する場合；
```bash
    # ディレクトリ移動 #
cd (docker-composeのあるディレクトリ)/docker/ssl

    # 秘密鍵生成 (server.key)
openssl genrsa 2048 > server.key

    # CSRファイル生成 (server.csr)
openssl req -new -sha256 -key server.key > server.csr
>> Country Name (2 letter code) [AU]:JP
>> Common Name (e.g. server FQDN or YOUR name) []:localhost
>> 上記以外はenterキーを押してください

    # 証明書作成（server.txt） #
echo "subjectAltName = DNS:localhost" > server.txt
    # (server.crt) #
openssl x509 -days 365 -req -in server.csr -sha256 -signkey server.key -extfile server.txt > server.crt

server.txtのみ削除してください
```

2. Let's encryptを使用する場合；
```bash

```

### tailwindcssのdev環境での動作
```docker-compose.yml
service:
    app:
        ports:
            - ${APP_VITE_PORT:-5173}:5173
```
```Dockerfile:./docker/app/Dockerfile
# 最終行
EXPOSE 5173
```
```bash
# dockerコンテナ内にて
npm run dev --host
```
これで動作するかもしれないが、networkがbridge接続だと重くて動作しない可能性大


## Laravel tips

### 大枠の流れ
1. ブラウザからアクセス

2. public\index.php
    ```php
    $app = require_once __DIR__.'/../bootstrap/app.php';
    ```

3. bootstrap\app.php
    bootstrap/app.phpからLaravelのコアであるApplicationクラスがインスタンス化されています。
    ```php
    $app = new Illuminate\Foundation\Application(
        $_ENV['APP_BASE_PATH'] ?? dirname(__DIR__)
    );
    ```

4. illuminate\foundation\application.php
    このコードからサービプロバイダーはconfig[‘app.providers’]を使って読み込まれています。
    ```php
    public function registerConfiguredProviders()
    {
    $providers = Collection::make($this->config['app.providers'])
        ->partition(function ($provider) {
            return Str::startsWith($provider, 'Illuminate\\');
        });
    ```

5. config\app.php
    ```php
    'providers' = [
        // ...
    ]
    ```



### artisanコマンド一覧

#### makeコマンド（使いそうなもののみ）
1. controller
2. model
3. mail (メール送信)
4. request (varidation)
5. migration
6. seeder
7. policy (複数権限があるケース)
8. auth (ログイン機能ができる)
9. provider (初期起動処理)
10. middleware


### CRUD機能

#### ルーティング
```script
controller
    -> service
        -> interface
            -> repository
                ( -> model )
                    -> entity
```

#### ディレクトリ構成
- Entities
    1. 
- Repositories
    - Interfaces
- Services
- Utils
- その他（関連事項）
    - model

#### ルール
1. ディレクトリ下にディレクトリを大量生産しない

2. 1-Entity 1-Repository
    - 状態更新に関わる振る舞いはEntityやServiceに実装し、可視性を制御して無理に変更ができないようにする
    - Repositoryは単なるストレージへのアクセス手段として利用し、複雑なビジネス上のロジックを持たないようにする

3. DDD集約の原則
    - Entityと関連するオブジェクトを一つの塊として捉え、集約の更新は必ず集約ルートを経由してしか行えないようにすることで、集約内でのデータ整合性を担保する（＝子Entityに対してはRepositoryを持たない）
    - ※ただし、これは変更(command)操作を行うときの原則

4. CQRS (Command Query Responsibility Segregation) の原則
    - Query操作を行うときは、4. と異なり複雑になりがちなので、直接アプリケーションレイヤーのサービスからクエリビルダーを介して直接クエリを発行する





### useのあれこれ
- use Laravel\Passport\HasApiTokens;
    - Passportパッケージ
    - apiトークン認証をサポートするためのメソッド・リレーションシップを提供


### サービスプロバイダについて


### controller記述
- returnパターン
    1. ダイレクトにViewを指定する（attributesを渡す）；
    ```php
    use App\Domain\Services\サービス名;
    use Illuminate\Contracts\View\View;
    
    $result = $this->サービス名->メソッド名();
    return view('(ディレクトリ.)ビュー名', compact('result'));
    ```
    
    2. ルートにリダイレクトする
    ```php
    // 処理
    return redirect()->route('ルート名')->with('message','〇〇しました');
    ```
    
    3. リダイレクトバックする
    ```php
    return redirect()->back()->with('message','〇〇しました');
    ```


### その他
isset, empty, is_nullの違い
```php
値              if($var)	isset	empty	is_null
$var=1          true        true	false	false
$var="";        false       true	true	false
$var="0";       false       true	true	false
$var=0;         false       true	true	false
$var=NULL;      false       false	true	true
$var            false       false	true	true
$var=array()    false       true	true	false
$var=array(1)   true        true	false	false
```

