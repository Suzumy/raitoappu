# passcode
## setodaNoteCTF2021
## rev
***

問題ファイルをダウンロードすると、passcode_~~.zipといったファイルが渡されるため、これをまずは解凍する。  
  
中にはpasscodeというファイルが入っている。この時点ではファイルの種類が不明なため、一応fileコマンドでファイルの種類を確かめる

> file passcode

すると以下のように返ってくる。  
```
passcode: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=8be572b7a0563868ee29af143b2df0c7d6b1636d, for GNU/Linux 3.2.0, stripped
```

とりあえず64bitの実行ファイルっぽいので、linux環境で実行してみる。

```
./passcode

Enter the passcode: hoge
Invalid passcode. Too short.
```

どうやらhogeでは短いらしい。  
多分入力されたパスワードを判定して、正しければフラグを返すプログラムだろうとあたりをつけて、ghidraで解析してみる。  

とりあえずFunctionsを開いていみるが、mainと書かれたところが存在しないため、ざっとみてメインっぽい処理が書かれているところを探していく。  
すると、FUN_00101185というところにそれっぽいプログラムが書かれてあった。  

これを読むと、
> iVar1 = strcmp((char *)&local_108,"20150109");

といった比較があるため、パスワードとして入力する値は20150109だとわかる。  

もう一度プログラムを実行して確かめてみる。
```
Enter the passcode: 20150109
The passcode has been verified.

Flag is : flag{^^^^^^}
```

ということでフラグがみつかったため、これをsubmitする.
(フラグは伏せる)
