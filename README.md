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
