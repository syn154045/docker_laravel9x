FROM axllent/mailpit:1.9

# OpenSSLのインストール（SMTPSの有効化）
RUN apt-get update && apt-get install -y openssl && rm -rf /var/lib/apt/lists/*

# キー用のディレクトリを作成
RUN mkdir /keys

# OpenSSLを使用して証明書と秘密鍵を生成
RUN openssl req \
    -x509 \
    -newkey rsa:4096 \
    -nodes \
    -keyout /keys/privkey.pem \
    -out /keys/cert.pem \
    -sha256 -days 3650


# apache2-utilsのインストール（SMTP_AUTHの有効化）
# 認証形式＝basic認証と同じ（htpasswdを利用）
RUN apt-get update && apt-get install -y apache2-utils && rm -rf /var/lib/apt/lists/*

# htpasswdを使用して認証用のファイルを生成（ユーザー名：mailpit、パスワード：pwd）
RUN htpasswd -bc /keys/.htpasswd mailpit pwd
