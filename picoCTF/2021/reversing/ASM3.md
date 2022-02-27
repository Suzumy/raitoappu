# ASM3
## reversing
## picoCTF2021
***
問題として以下のプログラムが渡される。  
```
asm3:
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
	<+3>:	xor    eax,eax	
	<+5>:	mov    ah,BYTE PTR [ebp+0xb] 
	<+8>:	shl    ax,0x10 
	<+12>:	sub    al,BYTE PTR [ebp+0xd] 
	<+15>:	add    ah,BYTE PTR [ebp+0xe] 
	<+18>:	xor    ax,WORD PTR [ebp+0x12] 
	<+22>:	nop
	<+23>:	pop    ebp
	<+24>:	ret    
```
まず、関数が呼ばれた時点でのスタックの状態を考える。  
引数として(0xdff83990,0xeeff29ae,0xfa706498)が渡されるため、スタックの状態は以下のようになる  
<table border = 2>
<tr><th>ebp</th></tr>
<tr><th>mainへのリターンアドレス</th></tr>
<tr><th>dff83990</th></tr>
<tr><th>eeff29ae</th></tr>
<tr><th>fa706498</th></tr>
</table>

この状態からプログラムを見ていく。先頭2行はebpとespを操作してasm3が使うスタックを準備している。  
> <+3>:	xor    eax,eax  

xorは引数1と2を排他的論理和で計算し、eaxに入れる。今回引数1と2は同じ値のためeaxの値は0となる。  
><+5>:	mov    ah,BYTE PTR [ebp+0xb]  

ahは1byte(2桁)のデータを入れることができる。ebp+0xbはどこを指すか考えると、0xb = 11のため、ebp+0x8とebp+0xcの間、つまりdff83990を指すことになる。しかしahは1byteのデータしか入らないため、この中から2桁しか入らないことになる。ではどの2桁かというと、この1行はメモリ上にリトルエンディアンで入っているため、実際にはこのように入っている。  
+8 ->  90  
+9 ->  39  
+10 -> f8  
+11 -> df  
bは11を指すため、ahにはdfが入る。
> <+8>:	shl    ax,0x10  

shlは左にビットシフトする命令である。今回はaxを0x10ビットシフトする。axは先頭8bitをah、下位8bitをalで構成されるため、axは現在df00が入っていることになる。これを左に0x10bitシフトするため、結果axには0x0000となる
> <+12>:	sub    al,BYTE PTR [ebp+0xd]  

subは引き算の命令。今回はal = al-BYTE PTR [ebp+0xd]という計算を行う。axが0000のため、alは現在00、ebp+0xdは0x29を指すため、00-29 = FFFF...D7、FFF...は無視してD7となる。
> <+15>:	add    ah,BYTE PTR [ebp+0xe] 

上と同じ要領で00 + ff = ffとなる。

> <+18>:	xor    ax,WORD PTR [ebp+0x12]  

この時点でahがff、alがd7のため、axはffd7になっている。  
次にebp+0x12はどこを指すかだが、0x12は18を指すため、18byteを参照する。すると0xfa70という値が出てくる。  
ここから、ffd7 xor fa70をすればいいことがわかる。  
ffd7 xor fa70 = 05a7  

> <+22>:	nop
ちなみにnopは何もしないという意味の命令のため、答えは変わらないため、先ほど出た答えをsubmitする。  

まとめるとこんな感じ  
```
asm3:
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
	<+3>:	xor    eax,eax	//eax = 0
	<+5>:	mov    ah,BYTE PTR [ebp+0xb] // ah = df
	<+8>:	shl    ax,0x10 //ax = df00 -> 0000
	<+12>:	sub    al,BYTE PTR [ebp+0xd] //00 - 29 = D7
	<+15>:	add    ah,BYTE PTR [ebp+0xe] //00 + ff = ff
	<+18>:	xor    ax,WORD PTR [ebp+0x12] //ffd7 xor fa70 = 05a7
	<+22>:	nop
	<+23>:	pop    ebp
	<+24>:	ret    

```
