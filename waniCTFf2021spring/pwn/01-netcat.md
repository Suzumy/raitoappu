# netcat
## waniCTF2021spring
## pwn

***

<h2> dockerでサーバを建てる</h2>
始めにdockerでサーバを建てる。githubに問題に関係するファイル及びDockerfile, docker-compose.ymlはあるためこれをダウンロードして使う。  

始めにdockerイメージを作成する。
> docker build -t netcat .

これを実行するとイメージが作成される。次にイメージを使って実際にサーバを建てる。
> docker compose up

これでサーバを建てることに成功した。

<h2> サーバに接続する</h2>
次に建てたサーバに接続する。今回はローカル環境でやっているため、ローカルのサーバに接続する。  

> nc localhost 9001

9001はポート番号で、docker-compose.ymlに定義されていた値を使用した。

<h2> writeup</h2>
実際に解いていく。  
サーバに接続が完了すると、打ち込んだコマンドの結果が返ってくることがわかる。このためまずはファイルの中身を調べるコマンドを打つ.

> ls

するとflag.txtというのがあるため、

> cat flag.txt

をする。するとフラグが得られるため、これをsubmitする。