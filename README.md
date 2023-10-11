# Nginx を用いた Web サーバーの構築。

## 概要

- 久しぶりの docker-compose なので思い出しつつメモ。
- 久しぶりなのに一気に作ろうとして失敗したので、1 個ずつやる。
- ssl を用いたセキュアな接続も行う。
- リバースプロキシも設定する。

## docker-compose.yaml の作成

1. スタートは簡単な形で作成していく。

- docker-compose.yaml

```
version: "3"
services:
  ngweb1:
    image: nginx
    ports:
      - 80:80
    volumes:
      - ./web-server1/html:/usr/share/nginx/html
```

2. docker-compose up -d で起動する。

3. ブラウザで確認する。

- http://localhost:80
- 404 エラーが出るので、html ファイルを作成する。
- ./web-server1/html/index.html

```html
<html>
  <body>
    <h1>hello world</h1>
  </body>
</html>
```

- 再度ブラウザで確認する。

4. コンテナから nginx の関連ファイルをコピーする。

- docker ps でコンテナ ID を確認する
- docker cp コンテナ ID:/etc/nginx/conf.d/ ./web-server1/conf.d
- docker cp コンテナ ID:/etc/nginx/nginx.conf ./web-server1/nginx/nginx.conf

ひとまず関連する conf のみ取得。conf.d 配下にある default.conf が 80 ポートで起動していることが確認できる。

5. docker-compose down で停止する。

## 証明書の作成（自己証明書）

SSL 通信を行うために証明書を作成する。今回は検証なので自己証明書。

1. openssl のインストール

- OpenSSL をインストールしていない場合は、[OpenSSL の公式ウェブサイト](https://www.openssl.org/) から入手し、インストールします。

```bash
$ openssl version
OpenSSL 3.1.3 19 Sep 2023 (Library: OpenSSL 3.1.3 19 Sep 2023)
```

2. 証明書のプライベートキーを生成する
   次のコマンドを使用して、プライベートキーを生成します。

```bash
openssl genpkey -algorithm RSA -out private.key
```

3. CSR（証明書署名要求）を生成する
   次のコマンドを使用して、CSR を生成します。Common Name（CN）の部分にワイルドカードドメインを指定します（例: `.example.com`）。

```bash
openssl req -new -key private.key -out csr.pem
```

このコマンドを実行すると、いくつかの質問が表示されます。必要事項を入力してください。Common Name には、ワイルドカードを含めたドメイン名を指定してください。

**※ここの CN の設定を間違えないこと**

4. 自己署名証明書を生成する
   次のコマンドを使用して、自己署名証明書を生成します。

```bash
openssl x509 -req -days 365 -in csr.pem -signkey private.key -out certificate.pem
```

このコマンドでは、有効期限が 365 日の証明書が生成されます。必要に応じて、期限を変更してください。
