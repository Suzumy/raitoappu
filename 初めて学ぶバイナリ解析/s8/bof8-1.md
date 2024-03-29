# bof8-1
## 始めて学ぶバイナリ解析
## pwn
***

main関数では本来vuln関数しか呼ばれないが、リターンアドレスを書き換えることで呼び出されていないpwn関数を呼び出す演習。  
具体的にはvuln関数内のscanfにてBOFを起こし、リターンアドレスを書き換える。  

まず、何文字でBOFが起こるか調べる。手順は以下  
1 pattc で適当な文字列を生成する  
2 生成した文字列を実際に入力し、BOFを起こす  
3 BOFを起こすことに成功したら、何文字目からオーバーしたかを調べるため、pattoを使う。  
まずはgdbを起動する。  
> gdb ./a.out

起動したらまずpattcで文字列の生成を行う。この時文字数は適当に決めるが、プログラムから配列の大きさは48となっているため、余裕をもって60ぐらいにする。
> pattc 60

すると文字列が生成されるため、これをコピーする。  
次に入力のタイミングまでプログラムを進める。普通にプログラムを動かせば入力のタイミングで止まるため、一旦普通にプログラムを動かす。 
> run

すると「Starting program...」みたいな文字列が出てきて、入力待ちになる。ここで先ほどコピーした文字列を張り付ける。するとSegmetation faultと実行画面に表示され、EIPには自分が入力した文字列の一部分が表示されていることが分かる。ここに表示されている文字列は、4文字以上の時、他とかぶることが無いようになってるため、最初から何文字目でオーバーしたのかが確実にわかるようになっている。  
何文字目でオーバーしたのか調べるため、pattoを使う。EIPの欄を見ると、「EIP: 0x400.... ('A)AA')」みたいな感じになっているはずなので、これをコピーして使う。
> patto EIPの文字列をコピーしたもの

すると、「found at offset: 32」とかって出てくる。つまり32文字目でオーバーしたということになる。また、EIPは次に実行する命令が入ったアドレスのため、EIPの値をpwn関数のアドレスに書き換えると、pwn関数が呼び出せることになる。つまり、今回は32文字の適当な文字列+pwn関数のアドレスを入力することで、関数のリターンアドレスを書き換え、pwn関数を呼び出すことができるということになる。  
pwn関数のアドレスはpコマンドで調べる  
> p pwn

すると、$1 ..... 0x5655556dみたいな数字が出てくる。これがpwn関数のアドレスとなる。  
先ほど、32文字の適当な文字列+pwn関数のアドレスでBOFを起こせるということが分かったため、これを作っていく、32文字の文字列は適当に作ればいいので、pythonなどでprint('a' * 32)とかして出た出力をコピーしてやればいい。それの後に、pwnのアドレスをいれることになるが、ここで一つ注意がある。今回アドレスは0x5655555dと出ているが、入力する際は、これをリトルエンディアンで入力する必要があるため、数字としては5d 55 55 56みたいになる。また、16進数であることを示す\xを追加して、\x6d\x55\x55\x56となる。これを先ほど作成した32文字の文字列と合体させると、
> aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\x6d\x55\x55\x56

となる。これを入力に渡すので、echoコマンドを使い、以下のようなコマンドを入力してやる。
> echo -e 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa\x6d\x55\x55\x56' | ./a.out

これを実行すると、画面に「hacked!」と表示され、pwn関数を呼び出すことに成功したことがわかる。これでも解法としてはOkだが、もっと簡単に答えを作る為にpwntoolsというものがあるため、これを利用した解法も作る。  

以下にpwntoolsを使ったプログラムを示す
```
from pwn import *　                     #pwntoolsのインポート
p = process("./a.out")      #a.outの実行プロセスにアタッチする
p.sendline("A"*32+"\x6d\x55\x55\x56")　#入力にBOFを起こす文字列を入力させる
print(p.recvline()) #受け取った文字列を表示させる。
```

これを実行すると、同じくhacked!が表示され、pwn関数が呼ばれたことが分かった。
