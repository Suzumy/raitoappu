# Milkslap
## picoCTF2021
## Forensics
***
問題文の絵文字がリンクになっている。リンクを踏むと牛乳をかけられた男の絵が出てくる。  
リンクを踏む問題ではあるが、問題区分がForensicsなので、どこかから画像を引っ張ってくる必要がある。  
デベロッパーツールから、Networkタブを見ると、画像データが読み込まれていることがわかるため、これをダウンロードする。  
次にこの画像を調べていく。今回はzstegというツールを使って調べる。  
どうやらpngやbmpの隠されたデータを見つけるためのツールらしい。  
まずダウンロード  
>  gem install zsteg
  
使えるようになったら実際に画像に使ってみる。  
> zsteg -s first concat_v.png
  
以下出力
> imagedata           .. file: FoxPro FPT, blocks size 65280, next free block index 30592456  
b1,b,lsb,xy         .. text: "picoCT{imag3_m4n1pul4t10n_sl4p5}\n"  
b1,bgr,lsb,xy       ..
後略
  
二行目辺りにフラグが表示されているため、これをsubmitする。