## 概要

PostgreSQLのアップデート対応の手順書

## 手順
### 現在のPostgreSQLのバージョンを確認

以下のコマンドで現在のPostgreSQLのバージョンを確認

```bash
psql --version
```

### PostgreSQLのバックアップを取得

以下のコマンドでPostgreSQLのバックアップを取得

```bash
pg_dump --username=mastodon --no-owner mastodon > pgbuckup.`date +%Y%m%d_%H%M%S`.pgbump
```

### 最新のPostgreSQLをインストール

まずは最新のPostgreSQLをインストールするためにリポジトリの追加などを行います。

```bash
# ファイル リポジトリ構成を作成します。
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# リポジトリ署名キーをインポートします。
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key 追加 -

# パッケージリストを更新します。
sudo apt update

# PostgreSQL の最新バージョンをインストールします。
# 特定のバージョンが必要な場合は、「postgresql」の代わりに「postgresql-12」などを使用します。
sudo apt install postgresql -y
```

あとは複数のPostgreSQLがインストールされているかどうかを以下のコマンドで確認します。

```bash
dpkg --get-selections | grep postgres
```

### PostgreSQLとMastodonのサービスを停止

アップデートにあたってPostgreSQLとMastodonを一旦止めます。

```bash
sudo systemctl stop mastodon-web
sudo systemctl stop mastodon-streaming
sudo systemctl stop mastodon-sidekiq
sudo systemctl stop mastodon-sidekiq-sub
sudo systemctl stop postgresql
```

### 新しいPostgreSQLのバージョンのデフォルトクラスター名を変更

新しいPostgreSQLのバージョンのクラスター名を変更しておきます。
古いバージョンのクラスターからアップデートする際に名前が重複するのを避けるためです。

```bash
sudo pg_renamecluster <新しいPostgreSQLのバージョンを指定> main main_pristine
```

### 古いPostgreSQLのバージョンをアップデート

以下のコマンドで古いPostgreSQLのバージョンからアップデートを行います。

```bash
sudo pg_upgradecluster <古いPostgreSQLのバージョンを指定> main
```

### PostgreSQLとMastodonのサービスを起動

最後にアップデートにあたって止めていたPostgreSQLとMastodonを起動します。

```bash
sudo systemctl start postgresql
sudo systemctl start mastodon-web
sudo systemctl start mastodon-streaming
sudo systemctl start mastodon-sidekiq
sudo systemctl start mastodon-sidekiq-sub
```

正常にPostgreSQLのバージョンが移行されているか以下のコマンドなどで確認します。

```bash
# PostgreSQLのバージョンを確認
psql --version

# PostgreSQLのクラスター情報から確認
pg_lsclusters
```

その後、実際にMastodonで
- フォロー・フォロワー関係などが問題ないか
- 投稿ができるかどうか
- 各種タイムラインが正しく表示されるか
などを確認しましょう

## 参考記事

[Linux downloads (Ubuntu) ](https://www.postgresql.org/download/linux/ubuntu/)

[Upgrading PostgreSQL Version on Ubuntu Server](https://gorails.com/guides/upgrading-postgresql-version-on-ubuntu-server)
