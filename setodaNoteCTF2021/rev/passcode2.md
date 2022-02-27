# passcode2
## setodaNoteCTF2021
## rev
***

先ほどと同じく解凍して、とりあえず実行してみる。  

```
./passcode2
Enter the passcode: hoge
Invalid passcode. Too short.
```

passcodeの時と同じくパスワードを入力して、それが正しければフラグが出てくる問題っぽい  

同様にghidraを使ってプログラムを見ていく。  

passcodeの時と同じくghidraを使ってもmainというfunctionがないため、それっぽいとこを探していく。  
すると、FUN_00101175というところにそれっぽいプログラムがあった。これを見ていくと、まず入力文字列の長さを判定するプログラムが書かれている。軽く見ると  
```
if (sVar3 < 0xb) {
    printf("Invalid passcode. Too short.");
}
```
となっているため、少なくとも11文字以上必要であることがわかる。  

こんな感じで見ていくと、入力は11文字である必要があることがわかる。  

また、ソースコードを見てみると、変数宣言の辺りで
```
local_124[0] = 0x18;
local_124[1] = 0x1f;
local_124[2] = 4;
local_124[3] = 0x79;
local_120 = 0x4f;
local_11f = 0x5a;
local_11e = 4;
local_11d = 0x18;
local_11c = 0x1a;
local_11b = 0x1b;
local_11a = 0x1e;
```

と書かれている。数えてみるとちょうど11文字なのでフラグに無関係ということはなさそう。  

また、プログラム中に
```
while ((sVar3 = strlen((char *)local_124), local_10 < sVar3 && (*(byte *)((long)&local_118 + local_10) == (local_124[local_10] ^ 0x2a))))
```
と書かれている。最後に0x2aとなっているため、多分この辺で変換処理をしていることが分かる。  

この二つの情報から、変数の中身を0x2aでXORするとフラグになるのではと考え、以下のプログラムを作成した。  

```

"""
list = (0x18, 0x1f, 4, 0x79, 0x4f, 0x5a, 4, 0x18, 0x1a, 0x1b, 0x1e)

print(len(list))

for i in list:
    num = i ^ 0x2a
    print(chr(num), end="")
```
これを実行すると、
>25.Sep.2014

が出てくるため、これを入力値に与えてみる。  
```
./passcode2
Enter the passcode: 25.Sep.2014
The passcode has been verified.

Flag is : flag{------}
```
ということでフラグは伏せるが出てきたため、これをsubmitする。
