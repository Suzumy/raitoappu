# sneeki snek
## Killer Queen CTF.
## rev
***

初めにテキストファイルが与えられるため、まずはこれを見てみる。
```
  4           0 LOAD_CONST               1 ('')
              2 STORE_FAST               0 (f)

  5           4 LOAD_CONST               2 ('rwhxi}eomr\\^`Y')
              6 STORE_FAST               1 (a)

  6           8 LOAD_CONST               3 ('f]XdThbQd^TYL&\x13g')
             10 STORE_FAST               2 (z)

  7          12 LOAD_FAST                1 (a)
             14 LOAD_FAST                2 (z)
             16 BINARY_ADD
             18 STORE_FAST               1 (a)

  8          20 LOAD_GLOBAL              0 (enumerate)
             22 LOAD_FAST                1 (a)
             24 CALL_FUNCTION            1
             26 GET_ITER
        >>   28 FOR_ITER                48 (to 78)
             30 UNPACK_SEQUENCE          2
             32 STORE_FAST               3 (i)
             34 STORE_FAST               4 (b)

  9          36 LOAD_GLOBAL              1 (ord)       
             38 LOAD_FAST                4 (b)
             40 CALL_FUNCTION            1
             42 STORE_FAST               5 (c)

 10          44 LOAD_FAST                5 (c)
             46 LOAD_CONST               4 (7)
             48 BINARY_SUBTRACT
             50 STORE_FAST               5 (c)

 11          52 LOAD_FAST                5 (c)
             54 LOAD_FAST                3 (i)
             56 BINARY_ADD
             58 STORE_FAST               5 (c)

 12          60 LOAD_GLOBAL              2 (chr)       
             62 LOAD_FAST                5 (c)
             64 CALL_FUNCTION            1
             66 STORE_FAST               5 (c)

 13          68 LOAD_FAST                0 (f)
             70 LOAD_FAST                5 (c)
             72 INPLACE_ADD
             74 STORE_FAST               0 (f)
             76 JUMP_ABSOLUTE           28

 14     >>   78 LOAD_GLOBAL              3 (print)     //print(f)
             80 LOAD_FAST                0 (f)
             82 CALL_FUNCTION            1
             84 POP_TOP
             86 LOAD_CONST               0 (None)
             88 RETURN_VALUE
```
よくわからなかったため調べてみると、どうやらpythonのdisというものを使って生成されたものらしく、pythonのコードをディスアセンブリしたものであることが分かった。
つまりこの問題はこのコードから元のコードを復元できたらフラグが出てくるタイプの問題だと検討が付く。
後はコードを見ながら復元していく。まずは手元で簡単なコードを書いてそれをdisしてみる。
```
>>> def hoge():
	a = 1+2
	print(a)
```
以下dis結果
```
import dis
>>> dis.dis(hoge)
  3           0 LOAD_CONST               3 (3)
              3 STORE_FAST               0 (a)

  4           6 LOAD_FAST                0 (a)
              9 PRINT_ITEM          
             10 PRINT_NEWLINE       
             11 LOAD_CONST               0 (None)
             14 RETURN_VALUE
```
普通に難しいが、LOAD_CONSTが3でSTORE_FASTがaになっているため、この2行から
> a = 3

が出てきそうだと仮定する。とりあえずこの前提でコードを復元していく。
すると、最初の2行
```
0 LOAD_CONST               1 ('')
2 STORE_FAST               0 (f)
```
が
> f = ''

であると想像できる。この調子で戻していくと
```
f = ''
a = 'rwhxi}eomr\\^`Y'
z = 'f]XdThbQd^TYL&\x13g'
```
というコードが出来上がった。
次に
```
12 LOAD_FAST                1 (a)
14 LOAD_FAST                2 (z)
16 BINARY_ADD
18 STORE_FAST               1 (a)
```

はどうするかを考える。16のBINARY_ADDに注目する。これを調べると、どうやらスタックにpushされた二つの値を足し算するっぽい。12,14でLOAD_FASTがあるため、ここでpushしてると思われる。
ということでここではおそらく
> a = a + z

という処理をしていると考えられる。

こんな感じで復元していくと、最終的に以下のコードになった。

```
f = ''
a = 'rwhxi}eomr\\^`Y'
z = 'f]XdThbQd^TYL&\x13g'
a = a + z
for i, b in enumerate(a):
    #print(i, b)
    c = ord(b)
    c = c-7
    c = c+i
    c = chr(c)
    print(i, c)
    f = f+c
print(f)
```
これを実行するとフラグが出てくるので、これをsubmitする。