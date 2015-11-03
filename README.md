docker-tutorial
===============

このチュートリアルでは、以下のようなことをやります。

* Dockerインストール
* コンテナの起動
* イメージの作成
* Docker Composeを使った複数コンテナの管理

以下の説明はWindows環境で試した場合のものになっています。
他のOSで試す場合はちょっと違うところがあります。

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
ここで`IP 192.168.99.100`とIPアドレスが表示されるので、覚えておいてください。

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

試しに、Webサーバであるnginxのコンテナを起動してみます。

```
$ docker run -dit -p 8080:80 nginx
```

このようにコンソールに入力すると、何かモリモリとダウンロードするようなログが流れます。
完了すると、先ほど表示されたIPアドレスに、ポート番号8080でアクセスできるはずですので
ブラウザでアクセスしてみます。

```
$ start http://192.168.99.100:8080/
```

`Welcome to nginx!`という表示がされたらOKです。
これでnginxコンテナが起動できました。

コンテナの起動について、もう少し見てみましょう。

まず、以下のように`docker ps`コマンドを入力します。

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                           NAMES
4de574d1d648        nginx               "nginx -g 'daemon off"   6 seconds ago       Up 7 seconds        443/tcp, 0.0.0.0:8080->80/tcp   boring_galileo
```

`docker ps`コマンドでは、作成されたコンテナが一覧表示されます。
IDやイメージ名、ステータス、利用しているポート、名前などが表示されます。

次に、以下のように`docker images`コマンドを入力します。

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
nginx               latest              914c82c5a678        5 days ago          132.8 MB
```

`docker images`では、現在このDocker Engineにインストールされているイメージの一覧が表示されます。
先ほど`docker run`の際に何かモリモリとダウンロードされていましたが、
それはこのnginxイメージがダウンロードされていたというわけです。

ここでダウンロードしたngixnイメージは、[Docker Hub](https://hub.docker.com/)という公式レジストリサービスからダウンロードされています。
Docker Hubではnginx以外にも様々なイメージが作成され公開されています。


もう一度、今回実行した`docker run`コマンドを見てみましょう。

```
$ docker run -dit -p 8080:80 nginx
```

`docker run`は複数のコマンドライン引数をうけとります。
詳しくは、[docker.comのドキュメント](https://docs.docker.com/reference/commandline/run/)を参照してください。

今回指定しているオプションは、それぞれ以下のような意味があります。

```
-d (detach)			: Run container in background and print container ID
-i (interactive)	: Keep STDIN open even if not attached
-t (tty)			: Allocate a pseudo-TTY
-p (publish)		: Publish a container's port(s) to the host
```

-d, -i, -tは、あまりよくわかっていないですが、とりあえず付けておいて問題ないと思います。
-pは何らかのサーバをホストマシン外部に公開するときに設定するオプションで、
コロンの前が外部から接続するときのポート番号、コロンの後がコンテナに接続するときのポート番号を指定します。

最後に、今回作成したコンテナを削除します。

```
$ docker stop boring_galileo
boring_galileo
$ docker rm boring_galileo
boring_galileo
```

`docker stop`や`docker rm`コマンドでコンテナを停止・削除することができます。
コマンドラインオプションで指定するコンテナ名は、コンテナIDでもコンテナ名でもよいです。

# 2. イメージの作成

前回はDocker Hubからイメージをダウンロードしてそのまま利用しましたが、
今回はイメージを自分で作成(ビルド)してみます。
とはいっても、イメージを1から自分で作るのは大変なので、
既存のイメージからカスタマイズしたい部分だけ変更するという手法を使います。

イメージを作成する方法はいくつかあるのですが、
今回は最も正当法のような、Dockerfileを使う方法にします。

今回作成するカスタムイメージは、[オフィシャルのnginxイメージ](https://hub.docker.com/_/nginx/)をカスタマイズして
デフォルトのトップページを別のものにしてみます。

2_myimage ディレクトリ以下に、今回作成するイメージの元となるファイル一式を用意しました。
それぞれ順に説明していきます。

Dockerfileはイメージ作成手順を示すもので、
イメージをビルドする際に実行される命令を羅列したものです。
詳しくは、[ドキュメント](https://docs.docker.com/reference/builder/)を参照してください。

今回のDockerfileには、index.htmlをイメージに追加する命令ADDと、
任意のコマンドを実行する命令RUNが記述されています。

それでは、イメージをビルドしてみます。
イメージのビルドには、`docker build`コマンドを利用します。

```
$ docker build -t myimage 2_myimage/
Sending build context to Docker daemon 3.072 kB
Step 0 : FROM nginx
 ---> 914c82c5a678
Step 1 : ADD index.html /usr/share/nginx/html/
 ---> 586e412e51aa
Removing intermediate container 91355d550459
Step 2 : RUN date > /usr/share/nginx/html/date.txt
 ---> Running in 6fd8df84e254
 ---> 3eaa4e8b041e
Removing intermediate container 6fd8df84e254
Successfully built 3eaa4e8b041e
SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.
```

ここでは、2_myimageディレクトリ以下にあるDockerfileを用いてイメージmyimageをビルドすることを指示しています。
ビルドの各Stepでは、Dockerfileに書かれたそれぞれの命令が実行されていることがわかるかと思います。

イメージがビルドできたので、起動してみましょう。

```
$ docker run -dit -p 8080:80 myimage
2a8167fcc122a14c6d659bb3aaf68165a1e619a842398e186263806c1043d766

$ start http://192.168.99.100:8080/
```

「こんにちはDocker!」と書かれたindexページが表示されたかと思います。
これでカスタムイメージができました。

次に進む前に、以下のコマンドを実行して作成済みコンテナをすべて削除します。
このコマンドは、コンテナをリストアップする`docker ps`コマンドと、コンテナを削除する`docker rm`コマンドを組み合わせたものです。

```
$ docker rm -f `docker ps -qa`
2a8167fcc122
```

次に、再度`docker build`を実行してみてください。
実行ログを前回のものと比較してみましょう。

```
$ docker build -t myimage 2_myimage/
Sending build context to Docker daemon 3.072 kB
Step 0 : FROM nginx
 ---> 914c82c5a678
Step 1 : ADD index.html /usr/share/nginx/html/
 ---> Using cache
 ---> 72ee4187bc64
Step 2 : RUN date > /usr/share/nginx/html/date.txt
 ---> Using cache
 ---> f6733f985890
Successfully built f6733f985890
SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.
```

「Using cache」と出力されています。
これは、`docker build`では各手順の結果が変わらない(と思われる)場合、前回ビルド時のキャッシュが使われるという機能が働いたことを意味しています。
このキャッシュの仕組みがあるため、部分的にDockerfileを変更した場合のビルドは比較的短時間で済むケースが多く、
トライアンドエラーが非常にやりやすいです。
便利ですねー。

