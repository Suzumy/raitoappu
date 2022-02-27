# Simultaneous
##### セキュリティコンテストの為のCTF問題集
##### Reversing


初めにfileコマンドで実行環境を調べる
> file Simultaneous
実行結果から32bitのELFファイルということが分かるため、その環境を用意して解析を始める  

環境の準備が完了したら、まず実行してみる。
```
# ./simultaneous
Usage: ./equation_revenge x1 x2 x3 x4 x5 x6 x7 x8 x9 x10 x11 x12
```

どうやら引数を12個入れて実行する必要があるっぽい
指示に従ってもう一度実行する

```
# ./simultaneous 1 2 3 4 5 6 7 8 9 10 11 12
FLAG{
}
```
なんか文字化けして出てきているが、FLAGと出てきたので、実行方法をあってそう。
正しい引数を与えてやれば適切な変換が行われてフラグが導き出せる問題っぽい。  

次にltraceの動作を見てみる
```
# ltrace ./simultaneous 1 2 3 4 5 6 7 8 9 10 11 12
__libc_start_main(0x8048595, 13, 0xff98b094, 0x80486a0 <unfinished ...>
calloc(4,12) = 0x8443008
atoi(0xff98b971, 12, 0xf7d98dc8,0xf7f4b1b0) = 1
atoi(0xff98b973, 12, 0xf7d98dc8, 0xf7f4b1b0) = 2
atoi(0xff98b975, 12, 0xf7d98dc8, 0xf7f4b1b0) = 3
atoi(0xff98b977, 12, 0xf7d98dc8, 0xf7f4b1b0) = 4
atoi(0xff98b979, 12, 0xf7d98dc8, 0xf7f4b1b0) = 5
atoi(0xff98b97b, 12, 0xf7d98dc8, 0xf7f4b1b0) = 6
atoi(0xff98b97d, 12, 0xf7d98dc8, 0xf7f4b1b0) = 7
atoi(0xff98b97f, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                      = 8
atoi(0xff98b981, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                      = 9
atoi(0xff98b983, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                      = 10
atoi(0xff98b986, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                      = 11
atoi(0xff98b989, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                      = 12
printf("FLAG{")                                                                                                                   = 5
putchar(1, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                            = 1
putchar(2, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                            = 2
putchar(3, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                            = 3
putchar(4, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                            = 4
putchar(5, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                            = 5
putchar(6, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                            = 6
putchar(7, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                            = 7
putchar(8, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                            = 8
putchar(9, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                            = 9
putchar(10, 12, 0xf7d98dc8, 0xf7f4b1b0FLAG{
)                                                                                           = 10
putchar(11, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                           = 11
putchar(12, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                           = 12
putchar(125, 12, 0xf7d98dc8, 0xf7f4b1b0)                                                                                          = 125
exit(1} <no return ...>
+++ exited (status 1) +++
```
callocが呼ばれた後、atoiが呼ばれ、atoiは12回呼ばれている。
プログラム実行時に与える引数も12だったため、このatoiは引数に関する処理を行っていると思われる。  

次はgdb-pedaによる解析を行う。  
> gdb simultaneous
> start 1 2 3 4 5 6 7 8 9 10 11 12

gdbに解析の際は、関数の呼び出し前後や条件分岐などに着目して解析を行う。  
まずcalloc関数の実行後まで進める。  

> ni

calloc関数の実行後、eaxレジスタには関数の返り値が入るため、現在のeaxにはcalloc関数の返り値が入る。
calloc関数の戻り値はcallocによって確保されたアドレスの先頭アドレスのため、eaxにはこの先頭アドレスが入っていることになる。  
main+40のmov命令ではebp-0x24にeaxの値を入れているため、ebp-0x24へアクセスしようとする命令があれば、それは確保したアドレスの先頭アドレスを参照しようとしているということがわかる。  

次にatoi関数の実行後までniコマンドで進める

その後、nearpc 10を入力する。nearpcは現在のeip(次に実行する命令が入っているレジスタ)を中心に逆アセンブル結果を表示する命令。その結果が以下
```
gdb-peda$ nearpc 10
   0x8048613 <main+126>:        mov    eax,DWORD PTR [eax]
   0x8048615 <main+128>:        sub    esp,0xc
   0x8048618 <main+131>:        push   eax
   0x8048619 <main+132>:        call   0x80483d0 <atoi@plt>
=> 0x804861e <main+137>:        add    esp,0x10
   0x8048621 <main+140>:        mov    DWORD PTR [esi],eax
   0x8048623 <main+142>:        add    DWORD PTR [ebp-0x2c],0x1
   0x8048627 <main+146>:        mov    eax,DWORD PTR [ebp-0x2c]
   0x804862a <main+149>:        cmp    eax,DWORD PTR [ebp-0x20]
   0x804862d <main+152>:        jl     0x80485f1 <main+92>
```

これを見ると、main+149でeaxとebp-0x20を比較、main+152のjlで条件が真ならmain+92にジャンプするという流れでループが出来上がっている。
この時eaxの値はmain+142で0x1がaddつまり加算されている。この時main+149のcmp命令ではeaxとどんな値を比較しているのか調べてみる。  
> x/wd $ebp-0x20

12と出たことから、eaxと12を比較していることになる。

ループが終了するとmain+154へ行く
その時のアセンブリがこちら
```
0x8048627 <main+146>:        mov    eax,DWORD PTR [ebp-0x2c]
   0x804862a <main+149>:        cmp    eax,DWORD PTR [ebp-0x20]
   0x804862d <main+152>:        jl     0x80485f1 <main+92>
=> 0x804862f <main+154>:        sub    esp,0xc
   0x8048632 <main+157>:        push   DWORD PTR [ebp-0x24]
   0x8048635 <main+160>:        call   0x804854f <check>
   0x804863a <main+165>:        add    esp,0x10
   0x804863d <main+168>:        mov    DWORD PTR [ebp-0x1c],eax
```
main+160でcheck関数が呼ばれている。この関数の引数は直前にpushされているebp-0x24のため、calloc関数で
確保した領域の先頭アドレスを渡していることがわかる。  
また、関数の戻り値となるeaxはebp-0x1cに格納されている。ebp-0x1cは実はexitへの引数になっているため、
exitを0にするためには、checkの戻り値を0にする必要がある。  

次にcheck関数の中を見る。
> disas check
```
gdb-peda$ disas check
Dump of assembler code for function check:
   0x0804854f <+0>:     push   ebp
   0x08048550 <+1>:     mov    ebp,esp
   0x08048552 <+3>:     sub    esp,0x10
   0x08048555 <+6>:     mov    DWORD PTR [ebp-0x4],0x0
   0x0804855c <+13>:    jmp    0x8048588 <check+57>
   0x0804855e <+15>:    mov    eax,DWORD PTR [ebp-0x4]
   0x08048561 <+18>:    push   eax
   0x08048562 <+19>:    push   DWORD PTR [ebp+0x8]
   0x08048565 <+22>:    call   0x80484fb <sum>
   0x0804856a <+27>:    add    esp,0x8
   0x0804856d <+30>:    mov    edx,eax
   0x0804856f <+32>:    mov    eax,DWORD PTR [ebp-0x4]
   0x08048572 <+35>:    mov    eax,DWORD PTR [eax*4+0x8048980]
   0x08048579 <+42>:    cmp    edx,eax
   0x0804857b <+44>:    je     0x8048584 <check+53>
   0x0804857d <+46>:    mov    eax,0x1
   0x08048582 <+51>:    jmp    0x8048593 <check+68>
   0x08048584 <+53>:    add    DWORD PTR [ebp-0x4],0x1
   0x08048588 <+57>:    cmp    DWORD PTR [ebp-0x4],0xb
   0x0804858c <+61>:    jle    0x804855e <check+15>
   0x0804858e <+63>:    mov    eax,0x0
   0x08048593 <+68>:    leave
   0x08048594 <+69>:    ret
End of assembler dump.
```
となっている。  

+6でebp-0x4に0x0が入る。つまりebp-0x4が初期化されている。その後jmp命令でcheck+57に飛ぶ
+57ではebp-0x4と0xb(11)を比較、次にjleでebp-0x4 が 11以下ならばcheck+15にジャンプする。  
また、+53ではaddでebp-0x4に1が足されているため、つまりebp-0x4はカウンタとして使われているということが分かる。  

+15からは、ebp-0x4とebp-0x8を引数にsum関数を呼ぼうとしている。sum関数実行後はcheck+30でedxに戻り値が格納され、
eaxはebp-0x4に上書きされる。その後、eax*4+0x8048980で値が更新される。
この時0x8048980に格納されている値を見てみる。

> x/12wd 0x8048980
```
gdb-peda$ x/12wd 0x8048980
0x8048980 <b>:  4885    10656   14249   7244
0x8048990 <b+16>:       17152   -22439  -41829  -7222
0x80489a0 <b+32>:       -12530  18797   13201   22623
```

ここまでで、12回のループでsum関数の戻り値がすべて参照される値と等しくなる必要があると考えられる。  

続いてsum関数の解析を行う
> disas sum
```
gdb-peda$ disas sum
Dump of assembler code for function sum:
   0x080484fb <+0>:     push   ebp
   0x080484fc <+1>:     mov    ebp,esp
   0x080484fe <+3>:     sub    esp,0x10
   0x08048501 <+6>:     mov    DWORD PTR [ebp-0x8],0x0
   0x08048508 <+13>:    mov    DWORD PTR [ebp-0x4],0x0
   0x0804850f <+20>:    jmp    0x8048544 <sum+73>
   0x08048511 <+22>:    mov    eax,DWORD PTR [ebp-0x4]
   0x08048514 <+25>:    lea    edx,[eax*4+0x0]
   0x0804851b <+32>:    mov    eax,DWORD PTR [ebp+0x8]
   0x0804851e <+35>:    add    eax,edx
   0x08048520 <+37>:    mov    ecx,DWORD PTR [eax]
   0x08048522 <+39>:    mov    edx,DWORD PTR [ebp+0xc]
   0x08048525 <+42>:    mov    eax,edx
   0x08048527 <+44>:    add    eax,eax
   0x08048529 <+46>:    add    eax,edx
   0x0804852b <+48>:    shl    eax,0x2
   0x0804852e <+51>:    mov    edx,DWORD PTR [ebp-0x4]
   0x08048531 <+54>:    add    eax,edx
   0x08048533 <+56>:    mov    eax,DWORD PTR [eax*4+0x8048740]
   0x0804853a <+63>:    imul   eax,ecx
   0x0804853d <+66>:    add    DWORD PTR [ebp-0x8],eax
   0x08048540 <+69>:    add    DWORD PTR [ebp-0x4],0x1
   0x08048544 <+73>:    cmp    DWORD PTR [ebp-0x4],0xb
   0x08048548 <+77>:    jle    0x8048511 <sum+22>
   0x0804854a <+79>:    mov    eax,DWORD PTR [ebp-0x8]
   0x0804854d <+82>:    leave
   0x0804854e <+83>:    ret
End of assembler dump.
```

まずsum+6と+13でebp-0x8とebp-0x4に0x0を入れる。そのあと+20のjmpでsum+73までジャンプし、
cmp命令でebp-0x4と11を比較する。その後条件に寄ってsum+22にジャンプするというループが構成されていることが分かる。

次にループ内の+22の処理から見る。
+22ではeaxにebp-0x4の値を格納し、続いてedxに4倍した値を入れる。
sum+32からはebp-0x8の値をeaxに格納し、続いてeax=edx+eaxを行う。

+39からはebp+0xcの値をedxに代入。ebp-0xcはcheckの第二引数で0-11に変化する値になっている。
これをeaxにも格納し、+44ではaddで同じeaxを足す。つまり2倍する処理を行っている。  
そのあとeaxとedxをaddする。この時同じ値を3回足したことになるため、三倍しているのと同義になる。
さらにshl命令で2bitシフトしているため4倍していることになり、結果eaxにはebp-0xcに格納された値を12倍した値が格納されていることになる。
sum+51でebp-0x4をedxに移動後、eaxに加算する。最初のループではeaxは0だが、check関数が二回目のループの場合は12が加算されることになる。
次のmov命令でeax*4+0x8048740が示す値をeaxに代入する。sumのループも考えると12*12で計144個の値が格納されることになる。  

実際に0x8048740の値を見てみる
> x/144wd 0x8048740
```
gdb-peda$ x/144wd 0x8048740
0x8048740 <cofficients>:        -72     -74     91      59
0x8048750 <cofficients+16>:     53      -95     -32     -39
0x8048760 <cofficients+32>:     93      76      -31     22
0x8048770 <cofficients+48>:     78      -84     -96     69
0x8048780 <cofficients+64>:     -21     -72     89      -26
0x8048790 <cofficients+80>:     21      65      3       49
0x80487a0 <cofficients+96>:     -46     11      -39     54
0x80487b0 <cofficients+112>:    57      -14     59      -10
0x80487c0 <cofficients+128>:    77      -34     0       99
0x80487d0 <cofficients+144>:    27      4       52      23
0x80487e0 <cofficients+160>:    -1      43      -41     13
0x80487f0 <cofficients+176>:    9       -70     -16     91
0x8048800 <cofficients+192>:    60      -92     84      58
0x8048810 <cofficients+208>:    -8      -6      91      8
0x8048820 <cofficients+224>:    -30     -11     -5      -96
0x8048830 <cofficients+240>:    -91     -16     -96     51
0x8048840 <cofficients+256>:    -56     -85     -52     46
0x8048850 <cofficients+272>:    -78     87      96      -83
0x8048860 <cofficients+288>:    -32     -80     -80     54
0x8048870 <cofficients+304>:    -28     -85     -38     -75
0x8048880 <cofficients+320>:    5       32      -80     -72
0x8048890 <cofficients+336>:    5       -18     6       74
0x80488a0 <cofficients+352>:    -9      -64     30      -44
0x80488b0 <cofficients+368>:    -26     -6      -22     13
0x80488c0 <cofficients+384>:    30      61      -100    63
0x80488d0 <cofficients+400>:    -19     -92     68      -38
0x80488e0 <cofficients+416>:    -11     -96     44      -50
0x80488f0 <cofficients+432>:    59      4       99      -62
0x8048900 <cofficients+448>:    -34     -89     -52     87
0x8048910 <cofficients+464>:    22      38      86      15
0x8048920 <cofficients+480>:    -75     -92     -21     62
0x8048930 <cofficients+496>:    -77     -31     -10     90
0x8048940 <cofficients+512>:    83      89      66      -17
0x8048950 <cofficients+528>:    -5      23      -29     16
0x8048960 <cofficients+544>:    25      50      95      65
0x8048970 <cofficients+560>:    -57     35      4       22
```

変数としてcofficientsが与えられたことが分かった。
またsum+63のimulではeaxとコマンドライン引数の値が格納されたecxを乗算する
そしてeaxをebp-0x8に加算する。次の命令はループのため12回繰り返されることになる。
つまり、cofficientsの値とコマンドライン引数の乗算結果の合計がebp-0x8に格納され、sum+77のjleで
ジャンプしない場合はsum+79のmovでebp-0x8が格納され、戻り値となる。

以上から、sumはcofficientsの値とコマンドライン引数の値の乗算結果の合計を返す関数であることがわかる。
check関数の結果も踏まえると、12回のループすべてでcofficientsの12個の値とコマンドライン引数の乗算結果の合計が
bの値と等しくなる必要があることがわかる。

これを未知数が12の連立方程式として計算するとフラグが出る。