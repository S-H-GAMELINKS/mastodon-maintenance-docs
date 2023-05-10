## 概要

Redisのアップデート対応の手順書

## 手順
### 現在のRedisのバージョンを確認

以下のコマンドで現在のRedisのバージョンを確認

```bash
redis-server --version
```

### Redisのバックアップを取得

以下のコマンドでRedisのバックアップを取得

```bash
cp /var/lib/redis/dump.rdb /var/lib/redis/dump-<先ほど確認したバージョン>.rdb
```

### 最新のRedisをインストール

まずはMastodonの各種サービスとRedisを止める

```bash
sudo systemctl stop mastodon-web
sudo systemctl stop mastodon-streaming
sudo systemctl stop mastodon-sidekiq
sudo systemctl stop mastodon-sidekiq-sub
sudo systemctl stop redis
```

次にリポジトリをaptインデックスに追加する

```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

```

次に、パッケージのアップデートを行う

```bash
sudo apt update
```

最後に`Redisをインストールし、Redisを起動

```bash
sudo apt install redis
sudo systemctl start redis
```

あとはRedisのバージョンを確認してアップデートでいていればOK

```bash
redis-server --version
```

### タイムラインのキャッシュクリアと再構築

mastodonユーザーでログインし、タイムラインのキャッシュを削除

```bash
sudo su - mastodon
cd live
RAILS_ENV=production bundle exec ./bin/tootctl feeds clear
```

その後、タイムラインを再構築。

```bash
RAILS_ENV=production bundle exec ./bin/tootctl feeds build --all --concurrency 15
```

最後に、root権限を持つユーザーに戻り、各種Mastodonのサービスを起動すればOK

```bash
exit
sudo systemctl start mastodon-web
sudo systemctl start mastodon-streaming
sudo systemctl start mastodon-sidekiq
sudo systemctl start mastodon-sidekiq-sub
```