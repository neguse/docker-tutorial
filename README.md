docker-tutorial
===============

このチュートリアルでは、以下のようなことをやります。

* Dockerインストール
* コンテナの起動
* イメージの作成
* Docker Composeを使った複数コンテナの管理

以下の説明はWindows環境で試した場合のものになっていますが、
他のOSでもさほど違いはないはずです。

# 0. 前準備

[Docker Toolbox](https://www.docker.com/docker-toolbox)をダウンロードしてインストールします。
Docker ToolboxはDockerをPCで試すために必要な各種ツールがセットになったものです。

インストールが完了すると、`Docker Quickstart Terminal`というショートカットが作成されるので、
これを実行します。

すると、コンソールウィンドウが開かれ、以下のようなログが流れるはずです。

```
Machine default already exists in VirtualBox.
Starting machine default...
Starting VM...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
Setting environment variables for machine default...


                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/

docker is configured to use the default machine with IP 192.168.99.100
For help getting started, check out the docs at https://docs.docker.com
NOTE: When using interactive commands, prepend winpty. Examples: 'winpty docker run -it ...', 'winpty docker exec -it ...'.
```

このコンソールウィンドウにコマンドを入力していろいろやっていくことになります。


また、docker-tutorialのリポジトリを`git clone`しておいてください。
このコンソールは`msys`のコンソールとなっていて、Windowsとパス表記が異なるのですが、
以下のように読み替えてcloneしたリポジトリのディレクトリにcdします。

```
Windows上のパス例: C:\work\docker-tutorial
msys上のパス例   : /c/work/docker-tutorial
この場合、以下のコマンドを入力することでcdできる
  cd /c/work/docker-tutorial
```

# 1. コンテナの起動

