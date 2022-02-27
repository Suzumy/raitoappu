# GET aHEAD
## picoCTF2019
## Web
***

URLを開くと赤と青を選択するボタンがある。  
特にこれを触ることなく、問題文のGET aHEADから、ページにヘッドでリクエストすればいいとわかる。  
> curl --head http://mercury.picoctf.net:45028/

結果が
```
HTTP/1.1 200 OK
flag: picoCTF{r3j3ct_th3_du4l1ty_775f2530}
Content-type: text/html; charset=UTF-8
```
フラグをsubmitして終了