## 概要

Nginxのアップデート手順のまとめです

## 手順
### Nginxのバージョン確認

以下のコマンドでバージョンを確認

```bash
nginx -v
```

### Nginxの設定ファイルのバックアップ

以下のコマンドで設定ファイルのバックアップを作成

```bash
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.<先ほど確認したバージョン>.backup
```

### 最新のNginxをインストール

まず`/etc/apt/sources.list`を以下のコマンドを使い、エディターで開く

```bash
sudo vi /etc/apt/sources.list
```

一番下に以下の二行を追加し、保存

```bash
deb https://nginx.org/packages/ubuntu/ focal nginx
deb-src https://nginx.org/packages/ubuntu/ focal nginx
```

その後、以下のコマンドを実行してNginxのアップデートを行う

```bash
sudo apt update
sudo apt install nginx
```

この時GPGキー周りでエラーになった場合は以下のコマンドを実行してから再度上記のアップデート手順を行うこと

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys <エラーで表示されているキー>
sudo apt update
sudo apt install nginx
```