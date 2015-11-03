# イメージ作成の元となるイメージを指定
FROM nginx

# ファイルを追加する
# 追加されるファイルは、ビルド時のカレントディレクトリからの相対パス
# (絶対パスは指定できないので注意!)
# ローカルのファイル以外にも、URLを指定することでインターネット上のファイルも追加可能
ADD index.html /usr/share/nginx/html/

# コマンドを実行する
RUN date > /usr/share/nginx/html/date.txt

