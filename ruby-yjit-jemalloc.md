## 概要

Ruby 3.2へとバージョンを上げる際の手順です。
YJITとJemallocを併用します。

## 手順

まず、`mastodon`ユーザーへ移行します。

```bash
sudo su - mastodon
```

その後`live`ディレクトリまで移動します。

```bash
cd live
```

最後に、以下のコマンドでrbenv経由でjemallocとYJITを有効にしてRuby 3.2.1をインストールします。

```bash
RUBY_CONFIGURE_OPTS="--with-jemalloc --enable-yjit" rbenv install 3.2.1
```

インストール完了後、`bundle install`などで依存関係の再インストールを行います。

```bash
gem install
bundle install
yarn install --lock-file
RAILS_ENV=production bundle exec rails assets:precompile
```

インストール完了後、`root`へ戻り各種サービス(webとsidekiq)に以下のコードを追加します。

```
Environment="RUBY_OPT='--yjit'"
Environment="LD_PRELOAD=<jemmalocの共有ライブラリまでのパス>"
```

あとは以下のコマンドで設定を読み込みなおせばOK

```bash
sudo systemctl daemon-reload
sudo systemctl restart mastodon-web
sudo systemctl restart mastodon-streaming
sudo systemctl restart mastodon-sidekiq
sudo systemctl restart mastodon-sidekiq-sub
```
