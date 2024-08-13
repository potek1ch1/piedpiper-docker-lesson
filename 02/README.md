# 自作イメージを作ろう
　前回は，すでにあるNGINXイメージをもとに起動したコンテナをアレンジしました．しかし今回は，Linuxのイメージをもとに，自分だけのNGINXイメージを作ります．前回使用したNGINXイメージをはじめ，基本的に提供されているあらゆるイメージはLinuxイメージを使用して作られています．このセクションを通して，提供されているイメージがどのように作られているかを学習します．  
前回のセクションで学習したコマンドの解説は省略します．
## Linuxイメージの取得，コンテナの起動
　今回はubuntuのイメージを取得します．
```
$ docker pull ubuntu:20.04
```
コンテナを起動します
```
$ docker run -it -p 8000:80 -d --name self-made-nginx ubuntu:20.04
```
ここで新たに`-d`オプションが出てきたので解説します．これはプロセスをバックグラウンドで起動します．`-d`をつけなければターミナルにアクセスログなどが流れ，操作できなかったと思います．`-d`をつけることで，ログが流れず，ターミナルの操作が引き続きできます．また，ubuntuイメージは，起動時にフォアグラウンドで動くものがないので`-it`オプションを付けます

## NGINXのインストール
　ubuntuコンテに入りnginxをインストールします
```
$ docker exec -it self-made-nginx /bin/sh

# nginxのインストール
$ apt update
$ apt install -y nginx

# リージョンを選択するプロンプトに従い
# Asia → Tokyo を選択

# nginxのインストール成功確認
$ nginx -v
```

設定フォルダと，htmlファイル群の確認
```
$ cd /etc/nginx/
$ cd /usr/share/nginx/html
```

nginxを起動しましょう
```
$ service nginx start
```
<http://localhost:8000>にアクセスし，デフォルトページが表示されていれば完了です．

Nginxコンテナの最低限の操作はこれで以上です．  
しかし，せっかくですのでなにかページを追加してみましょう．  
　配布しているhtmlフォルダには，前回と同様のファイルに加え，`hoge.html`があらかじめ入っています．このフォルダに自作のhtmlファイルを入れてみると自作コンテナ感がでていいのではないでしょうか．　　
　それではファイルをコンテナに移して行きます．
```
#コンテナから出る
$ exit

# ファイルをコピー
$ docker container cp 02/html/ self-made-nginx:/usr/share/nginx/
```
コンテナに再度入り，該当のディレクトリのファイルが更新されているかを確認します．
```
$ ls /usr/share/nginx/html
```
しかしまだ，<http://localhost:8000>にアクセスしても正しくページが更新されていません．  
設定ファイルも変更する必要があります．ここでは詳しく扱いませんが，`/etc/nginx/sites-enabled/default`の設定を変更する必要がありそうですので正しく変更します．一度コンテナから出て次のコマンドを実行してください．
```
$ docker container cp 02/sites-enabled/default self-made-nginx:/etc/nginx/sites-enabled/default
```
そしてコンテナに入り直します．設定ファイルを変更した際はnginxを再起動する必要がります．次のコマンドで再起動を行います．
```
# service nginx restart
```
<http://localhost:8000>にアクセスしたときに，変更が反映されていれば成功です．コンテナの状態をイメージにしておきましょう．
```
$ docker stop self-made-nginx
$ docker commit self-made-nginx self-made-nginx
```
このイメージからコンテナを起動したあとに，nginxを別途起動する必要があります．
　今回のセクションでは，ubuntuのイメージを用いて，nginxイメージを作成しました．公式が提供しているnginxイメージを含めあらゆるイメージがLinuxディストリビューションのイメージを用いて作成されています．なので，このセクションで行った内容がDockerイメージを作成する基礎知識となります．次回は，イメージの状態をファイルで定義する方法を学習します．お疲れ様でした．