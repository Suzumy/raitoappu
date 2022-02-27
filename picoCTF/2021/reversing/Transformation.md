# Transformation
## reversing
## picoCTF2021
***

まずファイルの種類を調べる
```
$ file enc
enc: UTF-8 Unicode text, with no line terminators
```
どうやらUTF-8の文字が書かれているっぽい  
catで中身を読んでみる  

```
$ cat enc
灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸強㕤㐸㤸扽
```

とりあえずこれがフラグ出ないことはわかる。  
わからないため、一文字目を16進数で表示してみる。  

```
str = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸強㕤㐸㤸扽"
print(hex(ord(str[0])))
```

結果は0x7069だった。0x70はasciiでpを示すためもう答えっぽいが、一応プログラムを見る。  
```
''.join([chr((ord(flag[i]) << 8) + ord(flag[i + 1])) for i in range(0, len(flag), 2)])
```
8bit左シフトして、それと次の文字を足す。これを1文字飛ばして行う的な処理っぽい。  

8bit左シフトして、それに次の文字を足すと、フォーマット的には0x最初の文字次の文字ってかんじになるはず。  
ここでさっき登場した数字0x7069を見てみる。これを右に8bitシフトすると0x70、足された0x69はasciiでiを示す。  
ここから最初の文字はpiを示すことが分かる。  
この流れで与えらた文字列を解読していく  

```
str = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸強㕤㐸㤸扽"
for i in str:
    print(chr(ord(i) >> 8), end="")
    print(chr(ord(i) & 0x00ff), end="")
```

この結果がフラグになるため、それをsubmitする。