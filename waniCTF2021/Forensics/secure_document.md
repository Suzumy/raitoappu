# secure_document
## Wanictf 2021 spring
## Forensics
***
問題として、パスワード保護されたzipファイルとパスワードのヒントになるテキストファイルが渡される。  
どうやらAutoHotkeyというものが使われているらしい。  
具体的には以下のように解読していく。  
まず入力されたワードが the password for today is wani となっている。  
theを入力すると、
```
::the::
Send, +wani
return
```
からWaniが入力される。
次にpasswordを入力すると、
```
::password::
Send, +c+t+f
return
```
からCTFが入力される。
この時点で文字列がWaniCTFとなる。
次にforの部分が、
```
::for::
Send, {home}{right 3}{del}1{end}{left 2}{del}7
return
```
となっており、  
> {home}{right 3}{del}1

から左から3番目の文字を消し、1を出力し、  
> {end}{left 2}{del}7

から右から2番目が消えて7が出力される。  
この時点でWan1C7Fが出力されている。  
次にtodayを入力すると、  
```
::today::
FormatTime , x,, yyyyMMdd
SendInput, {home}{right 4}_%x%
return
```
から_yyyyMMddの形で左から4番目に日付が入力される。  
日付はzipファイルのファイル名から20210428だと推測できる。  
この時点でWan1_20210501C7Fが入力される。  
次にisを入力すると、
```
::is::
Send, _{end}{!}{!}{left}
return
```
から、先ほどの続き(つまり日付の次)に_を出力した後、一番最後に!!を出力する。最後にカーソル位置を一つ左にずらす。
最後にnaniを入力すると、
```
:*?:ni::
Send, ^a^c{Esc}{home}password{:} {end}
```
から、naを入力したあと、パスワードのコピーが行われる。
この時点で文字列がWan1_20210428_C7F!na!となっているため、これをzipの解凍パスワードとして入力する。  
すると画像ファイルが出てくるため、これを開いて出てきたフラグをsubmitする。