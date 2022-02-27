# WebNet1
## picoCTF2021
## Forensics 
***

問題としてpcapファイルと復号キーが渡される。
pcapファイルは暗号化通信をキャプチャしているため、復号キーを使って読める状態にする。

pcapファイルを開き、編集 -> 設定 -> Protocols -> SSL -> RSA Key list -> edit
IPaddress:128.237.140.23
Port:443
Protocols:http
key file:ダウンロードしたファイルを選択

これでOKを押すと復号された状態でpcapファイルが表示される

この状態でTLSv1.2と書かれたパケットを選択し、追跡 -> SSLストリームを選択すると、画像を含むデータがやり取りされているのが分かる。

ここを見ているとフラグっぽいものがあるため、これをsubmitしてもいいが、これは画像データの中身のため、実際に画像データを抽出して確認してみる。

ファイル -> オブジェクトをエクスポート -> http　でhttp通信でやり取りされるデータを保存できる。
これで画像を見られるため、見てみると、いたって普通の画像になっている。
forensicではexsisデータにフラグがあることがあるため、これを見てみると、アーティストの欄にフラグがあることが分かるため、これをsubmitする。