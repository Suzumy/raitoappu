# Trivial Flag Transfer Protocol
## picoCTF2021
## Forensic
***
始めにpacapファイルが渡されるため、これを見てみる。  
見てみると、TFTPの通信が目立つパケットファイルになっている。  
しばらく触っていると、TFTPの通信データについてエクスポートできる情報があったため、まずはこれをエクスポートした。  
すると3つのdmpファイルと1つのテキストファイル、2つの不明ファイルが発見出来た。ファイル名を見てみると、instructions.txt(説明)と書かれてあるファイルがあったので、まずはこれを開くと、
> GSGCQBRFAGRAPELCGBHEGENSSVPFBJRZHFGQVFTHVFRBHESYNTGENAFSRE.SVTHERBHGNJNLGBUVQRGURSYNTNAQVJVYYPURPXONPXSBEGURCYNA
  
と書かれていた。これをROT13で変換すると

> TFTP DOESNT ENCRYPTO UR TRAFFIC SO WE MUST DISGUISEOUR FLAG TRANSFER.
FIGURE OUT AWAY TO HIDE THE FLAG AND I WILL CHECK BACK FOR THE PLAN
  
どうやらplanをチェックしろと書かれているらしい。  
言われた通りplanというファイルをテキストファイルで開いてみると、

> VHFRQGURCEBTENZNAQUVQVGJVGU-QHRQVYVTRAPR.PURPXBHGGURCUBGBF
  
さっきと同じくROT13で変換をかけると

> IUSEDTHEPROGRAMANDHIDITWITH-DUEDILIGENCE.CHECKOUTTHEPHOTOS
  
翻訳するとプログラムを使って何かを隠した。写真をチェックしろと書いてる。

プログラムに当てはまりそうなものはこのファイルの中には.dedファイルしかないため、debファイルを解凍してみる。

どうやらプログラムはsteghideだったらしい。
>steghide extract -p 'DUEDILIGENCE' -sf picture3.bmp
  
とするとflag.txtが出てくるため、これを読み取ってsubmitする。