# Mac OS に docker をインストールして、Ruby on Rails の開発環境を準備するまで。

Docker Toolbox を初めにインストールします。
https://store.docker.com/editions/community/docker-ce-desktop-mac

Docker + ansible で Centos6  + nginx + mySQL + Ruby on Rails の開発環境を準備します。
共有フォルダは、Mac OS ~/docker_rails として CentOSは /home/docker とします。
SSHのポートは 2022 とします。
HTTPのポートは 8080 とします。

## カレントフォルダに github からクローンして docker_rails を作成する

```
cd
git clone https://github.com/bluemooninc/docker_rails
```

## クローンしたフォルダに移動して、docker image をビルドする。

```
cd docker_rails
docker pull centos:6.9
docker build --rm -t local/centos6 .
```

## フォルダ共有・ポートを指定してdocker image を実行する

```
docker run -d -p 2022:22 -p 8080:80 -v ~/docker_rails:/home/docker --privileged local/centos6 /sbin/init
```

## イメージ実行されているか確認してSSHで中に入る

```
docker ps
ssh -p 2022 root@127.0.0.1
```

## 共有設定した docker フォルダに移動して、ansible-playbook を実行する。

```
cd /home/docker
sh playbook.sh
```

## 一旦exit して再度sshで入って確認

```
[root@localhost docker]# rails -v
Rails 5.1.4
```

## Helloworld する

Helloworld アプリを作成してポート3000での起動を確認します。

```
cd /home/docker
mkdir sample
cd sample
rails new helloworld
cd helloworld
vi Gemfile
## 以下コメントアウトを外す
# gem 'therubyracer', platforms: :ruby
bundle install
rails s
=> Booting Puma
=> Rails 5.1.4 application starting in development
=> Run `rails server -h` for more startup options
Puma starting in single mode...
* Version 3.11.2 (ruby 2.4.3-p205), codename: Love Song
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

## Unicorn を入れる

```
vi Gemfile
# 以下を最下行に追加
gem 'unicorn'
# gemをインストール
bundle install
#  unicorn の config をコピーする
cp /home/docker/config/unicorn.rb ./config
# Rakefile をコピーする
cp /home/docker/config/Rakefile .
```

## unicorn の実行とワーカーの確認


```
rake unicorn:start
bundle exec unicorn_rails -D -c /home/docker/sample/helloworld/config/unicorn.rb -E development
[root@localhost helloworld]# ps ax | grep ruby
27286 pts/1    S+     0:00 grep ruby
[root@localhost helloworld]# ps ax | grep rails
27267 ?        Sl     0:02 unicorn_rails master -D -c /home/docker/sample/helloworld/config/unicorn.rb -E development
27276 ?        Sl     0:00 unicorn_rails worker[0] -D -c /home/docker/sample/helloworld/config/unicorn.rb -E development
27279 ?        Sl     0:00 unicorn_rails worker[1] -D -c /home/docker/sample/helloworld/config/unicorn.rb -E development
27282 ?        Sl     0:00 unicorn_rails worker[2] -D -c /home/docker/sample/helloworld/config/unicorn.rb -E development
27288 pts/1    S+     0:00 grep rails
```

再起動

```
rake unicorn:restart
```

終了

```
rake unicorn:stop
```

## nginx に設定を追加

```
## socktとproxyを設定追加したファイルを nginx へ
cp /home/docker/config/default.conf /etc/nginx/conf.d
/etc/init.d/nginx restart
```

3000ポートからsockに変更
コメントアウトを変更する。

vi config/unicorn.rb
```
# 同一マシンでNginxとプロキシ組むならsocketが良い
listen "/tmp/unicorn.sock"
# listen 3000

# pid file path Capistranoとか使う時は要設定
pid "/tmp/unicorn.pid"
```

rake unicorn:start