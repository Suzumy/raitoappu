# reverse_cipher
## reversing
## picoCTF2021
***
問題として実行ファイルと、おそらくその結果と思われるテキストファイルが渡される。  
ghidraでデコンパイルした結果、以下のことが分かった。  
* flag.txtからデータを読み、rev_thisに結果を書き込んでいる
* 8文字目まではflag.txtの中身がそのまま出力されている
* 9文字目以降は、配列のindex番号 & 1の演算を行い、結果が0なら0x05を足す処理、結果が1なら2を引く処理を行っている。

このことから以下のスクリプトを作ってフラグを復元した。  
```
chipflag = "picoCTF{w1{1wq87g_9654g}"
ansflag = ""
for i in range(0, 8):
    ansflag += chipflag[i]
    
for i in range(8, len(chipflag)-1):
    if (i & 1) == 0:
        hoge = chr(ord(chipflag[i]) - 0x05)
    else:
        hoge = chr(ord(chipflag[i]) + 0x02)
    
    ansflag += hoge
ansflag += "}"
print(ansflag)
```
これを実行するとフラグがでるため、それをsubmitする。