# shop
## reversing
## picoCTF2021
***

問題に接続先が示されるため、とりあえず接続してみる
> nc mercury.picoctf.net 37799.

すると買い物をする画面になる
```
Welcome to the market!
=====================
You have 40 coins
        Item            Price   Count
(0) Quiet Quiches       10      12
(1) Average Apple       15      8
(2) Fruitful Flag       100     1
(3) Sell an Item
(4) Exit
Choose an option:
```

数字を選択して売り買いができるっぽい。
2にflagと書かれているため、多分これを購入するとフラグが見れるけど、現時点ではポイントが足りないため、ポイントを増やす必要がある。
いろんなところで負の数を打ってバグがないか確かめる
```
Choose an option:
0
How many do you want to buy?
-100
You have 1040 coins
        Item            Price   Count
(0) Quiet Quiches       10      112
(1) Average Apple       15      8
(2) Fruitful Flag       100     1
(3) Sell an Item
(4) Exit
Choose an option:
1
How many do you want to buy?
-100
You have 2540 coins
        Item            Price   Count
(0) Quiet Quiches       10      112
(1) Average Apple       15      108
(2) Fruitful Flag       100     1
(3) Sell an Item
(4) Exit
Choose an option:
```
何度かやってみると、どうやら購入個数を入力する際、負の数を入れると所持コインが増えるっぽい。
この時点で100ポイントを超えているため、flagを購入する。

```
You have 2540 coins
        Item            Price   Count
(0) Quiet Quiches       10      112
(1) Average Apple       15      108
(2) Fruitful Flag       100     1
(3) Sell an Item
(4) Exit
Choose an option:
2
How many do you want to buy?
1
Flag is:  [112 105 99 111 67 84 70 123 98 52 100 95 98 114 111 103 114 97 109 109 101 114 95 53 57 49 97 56 57 53 97 125]
```

flagが10進数で出てきたため、これをascii文字列に直す  

```
flag = [112, 105, 99, 111, 67, 84, 70, 123, 98, 52, 100, 95, 98, 114, 111, 103, 114, 97, 109, 109, 101, 114, 95, 53, 57, 49, 97, 56, 57, 53, 97, 125]

for num in flag:
    print(chr(num), end="")
```

これを実行すると、フラグが出るため、これをsubmitする。