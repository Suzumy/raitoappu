# Matryoshka doll
## 大会：picoCTF2021
### 分野：Forensics
* * *
問題として、dolls.jpgが渡される。  
binwalkにかけてみると、どうやらzipファイルが隠されているようなので、これを取り出していく作業を行う。

青い空を見上げればいつもそこに白い猫を起動し、dolls.jpgを読み込み、ファイル、データ抽出を行うと、binwalkと同じくzipが隠れていることが分かる。これをリスト選択項目を保存で保存し、zipファイルの解凍を行うと、またpngファイルが出てくる。  
これを繰り返し行うと、フラグが出てくるため、これをsubmitする。