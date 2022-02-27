# SelfReference
## セキュリティコンテストの為のCTF問題集
## Reversing
***

問題として実行ファイルが与えられるため、まずはファイルの種類を調べる。  
> file SelfReference

結果
```
SelfReference: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=2eab3d3c43485546624750fefbf0ac345054105b, stripped
```

ここから、
・ELFファイルである  
・i386である  
・実行ファイル内のシンボル情報が削除されている  
ことがわかる。  

とりあえず32bitで実行できそうなので、32bitマシンを準備して実行してみる
> ./SelfReference

```
Usage :
  ./SelfReference -encrypt <str>

--------------------------------

FLAG{
[-] Decrypt function is implemented.
FLAG (encrypted)(hex)    : 7d 56 18 43 15 67 0f 0a 1c 28 3b 76 05 30 00 50 54 0c 59 09 1f 7d 0d 3a 02 7a 08 7e 01 40 57 60 11 3e 05 2d 05 0f 00 00 06 55 30
}
```

ここから、
・フラグ形式はFLAG{....}
・フラグが暗号化されている
・-encrypt str　の形で文字列を暗号化することができる

ぐらいのことが分かる
試しに-encryptを使ってみる
```
# ./SelfReference -encrypt hogehoge
plain            : hogehoge
encrypted(hex)   : 1e 00 33 31 58 1a 54 1f
```
同じ単語を繰り返しても同じ結果にならないため、同じ文字が同じ数字に対応してるわけではなさそう  

文字列が暗号化されて表示されていることが分かるため、暗号化しているプログラムを見つけ出し、そこから復号化のコードを
書いている方向で進める  

まず暗号化関数がどこにあるかを調べる。  
> ltrace ./SelfReference -encrypt abcd
```
__libc_start_main(0x8048759, 3, 0xff9758e4, 0x8048c80 <unfinished ...>
calloc(128, 1)                                                            = 0x8c0d008
calloc(256, 1)                                                            = 0x8c0d090
calloc(256, 1)                                                            = 0x8c0d198
calloc(1024, 1)                                                           = 0x8c0d2a0
strcmp("-encrypt", "-encrypt")                                            = 0
strncpy(0x8c0d008, "abcd", 128)                                           = 0x8c0d008
fopen("./SelfReference", "rb")                                            = 0x8c0d6a8
fseek(0x8c0d6a8, 0xfffffc00, 2, 0xf7f7c000)                               = 0
fgets("XV5xxMLwKP8KaayCSG04vQVv0kMSA3ZT"..., 1024, 0x8c0d6a8)             = 0x8c0d2a0
strlen("abcd")                                                            = 4
srand(4, 1024, 0x8c0d6a8, 0xf7f7c000)                                     = 0
rand(0xff975838, 0, 4, 0x8c0d2a0)                                         = 0x754e7ddd
rand(0xff975838, 1, 4, 477)                                               = 0x11265233
rand(0xff975838, 2, 4, 563)                                               = 0x18799942
rand(0xff975838, 3, 4, 322)                                               = 0x214a541e
printf("plain \t\t : %s\n", "abcd"plain                  : abcd
)                                       = 16
printf("encrypted(hex) \t : ")                                            = 19
strlen("abcd")                                                            = 4
printf("%02x ", 0x3)                                                      = 3
strlen("abcd")                                                            = 4
printf("%02x ", 0x13)                                                     = 3
strlen("abcd")                                                            = 4
printf("%02x ", 0x28)                                                     = 3
strlen("abcd")                                                            = 4
printf("%02x ", 0x3e)                                                     = 3
strlen("abcd")                                                            = 4
putchar(10, 62, 0x8c0d2a0, 0encrypted(hex)       : 03 13 28 3e
)                                             = 10
free(0x8c0d008)                                                           = <void>
free(0x8c0d090)                                                           = <void>
free(0x8c0d198)                                                           = <void>
free(0x8c0d2a0)                                                           = <void>
+++ exited (status 0) +++
```
初めにstrcmpでencryptがあるか調べて、あったら次の引数の値を暗号化する流れっぽい

このencryptというキーワードを使って、暗号化関数を絞り込む。文字列を使った検索にはrabin2を使用する  

> rabin2 -z ./SelfReference
```
[Strings]
nth paddr      vaddr      len size section type  string
-------------------------------------------------------
0   0x00000d03 0x08048d03 8   9    .rodata ascii -encrypt
1   0x00000d0c 0x08048d0c 14  15   .rodata ascii plain \t\t : %s\n
2   0x00000d1b 0x08048d1b 19  20   .rodata ascii encrypted(hex) \t :
3   0x00000d2f 0x08048d2f 5   6    .rodata ascii %02x
4   0x00000d35 0x08048d35 8   9    .rodata ascii -decrypt
5   0x00000d40 0x08048d40 31  32   .rodata ascii [-] This option is implemented.
6   0x00000d60 0x08048d60 15  16   .rodata ascii Invalid option.
7   0x00000d70 0x08048d70 8   9    .rodata ascii Usage :
8   0x00000d7c 0x08048d7c 32  33   .rodata ascii   ./SelfReference -encrypt <str>
9   0x00000da0 0x08048da0 35  36   .rodata ascii \n\n--------------------------------\n
10  0x00000dc4 0x08048dc4 5   6    .rodata ascii FLAG{
11  0x00000dca 0x08048dca 26  27   .rodata ascii FLAG (encrypted)(hex) \t :
12  0x00000de8 0x08048de8 36  37   .rodata ascii [-] Decrypt function is implemented.
```

encryptが出ているのは一行目の部分のため、0x08048d03のアドレスに-encryptという文字列が含まれることが
分かった。  
暗号化関数ではこの文字列を参照すると思われるので、この文字列を参照しているアドレスを調べる。  

まずradare2を起動する
> radare2 ./SelfReference

次にファイルの読み込みを行う
> aa
> aac

終わったら、axtコマンドでアドレスを参照しているアドレスを調べる。  
> axt 0x08048d03
```
main 0x804880a [DATA] push str._encrypt
```
0x804880aで文字列が参照されていることが分かった。

次に、このアドレスに移動する。
> s 0x804880a

そして逆アセンブル
> pd
```
[0x0804880a]> pd
|           0x0804880a      68038d0408     push str._encrypt           ; 0x8048d03 ; "-encrypt"
|           0x0804880f      50             push eax
|           0x08048810      e8cbfcffff     call sym.imp.strcmp         ; int strcmp(const char *s1, const char *s2)
|           0x08048815      83c410         add esp, 0x10
|           0x08048818      85c0           test eax, eax
|       ,=< 0x0804881a      0f8507010000   jne 0x8048927
|       |   0x08048820      8b45a4         mov eax, dword [var_5ch]
|       |   0x08048823      83c008         add eax, 8
|       |   0x08048826      8b00           mov eax, dword [eax]
|       |   0x08048828      83ec04         sub esp, 4
|       |   0x0804882b      6880000000     push 0x80                   ; 128
|       |   0x08048830      50             push eax
|       |   0x08048831      ff75b4         push dword [var_4ch]
|       |   0x08048834      e877fdffff     call sym.imp.strncpy        ; char *strncpy(char *dest, const char *src, size_t  n)
|       |   0x08048839      83c410         add esp, 0x10
|       |   0x0804883c      8b45a4         mov eax, dword [var_5ch]
|       |   0x0804883f      8b00           mov eax, dword [eax]
|       |   0x08048841      83ec08         sub esp, 8
|       |   0x08048844      50             push eax
|       |   0x08048845      ff75c0         push dword [var_40h]
|       |   0x08048848      e89efeffff     call fcn.080486eb
|       |   0x0804884d      83c410         add esp, 0x10
|       |   0x08048850      8945c4         mov dword [var_3ch], eax
|       |   0x08048853      837dc400       cmp dword [var_3ch], 0
|      ,==< 0x08048857      7542           jne 0x804889b
|      ||   0x08048859      83ec0c         sub esp, 0xc
|      ||   0x0804885c      ff75b4         push dword [var_4ch]
|      ||   0x0804885f      e89cfcffff     call sym.imp.free           ; void free(void *ptr)
|      ||   0x08048864      83c410         add esp, 0x10
|      ||   0x08048867      83ec0c         sub esp, 0xc
|      ||   0x0804886a      ff75b8         push dword [var_48h]
|      ||   0x0804886d      e88efcffff     call sym.imp.free           ; void free(void *ptr)
|      ||   0x08048872      83c410         add esp, 0x10
|      ||   0x08048875      83ec0c         sub esp, 0xc
|      ||   0x08048878      ff75bc         push dword [var_44h]
|      ||   0x0804887b      e880fcffff     call sym.imp.free           ; void free(void *ptr)
|      ||   0x08048880      83c410         add esp, 0x10
|      ||   0x08048883      83ec0c         sub esp, 0xc
|      ||   0x08048886      ff75c0         push dword [var_40h]
|      ||   0x08048889      e872fcffff     call sym.imp.free           ; void free(void *ptr)
|      ||   0x0804888e      83c410         add esp, 0x10
|      ||   0x08048891      83ec0c         sub esp, 0xc
|      ||   0x08048894      6a01           push 1                      ; 1
|      ||   0x08048896      e8b5fcffff     call sym.imp.exit           ; void exit(int status)
|      `--> 0x0804889b      83ec04         sub esp, 4
|       |   0x0804889e      ff75c0         push dword [var_40h]
|       |   0x080488a1      ff75b4         push dword [var_4ch]
|       |   0x080488a4      ff75b8         push dword [var_48h]
|       |   0x080488a7      e835030000     call fcn.08048be1
|       |   0x080488ac      83c410         add esp, 0x10
|       |   0x080488af      83ec08         sub esp, 8
|       |   0x080488b2      ff75b4         push dword [var_4ch]
|       |   0x080488b5      680c8d0408     push str.plain__t_t_:__s_n  ; 0x8048d0c ; "plain \t\t : %s\n"
|       |   0x080488ba      e831fcffff     call sym.imp.printf         ; int printf(const char *format)
|       |   0x080488bf      83c410         add esp, 0x10
|       |   0x080488c2      83ec0c         sub esp, 0xc
|       |   0x080488c5      681b8d0408     push str.encrypted_hex___t_:_ ; 0x8048d1b ; "encrypted(hex) \t : "
|       |   0x080488ca      e821fcffff     call sym.imp.printf         ; int printf(const char *format)
|       |   0x080488cf      83c410         add esp, 0x10
|       |   0x080488d2      c745b0000000.  mov dword [var_50h], 0
|      ,==< 0x080488d9      eb23           jmp 0x80488fe
|      ||   0x080488db      8b55b0         mov edx, dword [var_50h]
|      ||   0x080488de      8b45b8         mov eax, dword [var_48h]
|      ||   0x080488e1      01d0           add eax, edx
```

0x08048810で比較が行われていることが分かった。
また、この流れで呼ばれる関数の中で標準関数を除くと
・fcn.080486eb
・fcn.08048be1
の二つがあることが分かった。暗号化関数は自作関数であると思われるため、この二つのどちらかが暗号化関数であると仮定する。  

まずfcn.080486ebを細かく見ていく  
引数を見ると、直前にpushされているものが二つある。  
・第二引数 - eaxレジスタに格納されている値 -> 問題ファイルを実行した際の第0引数の値(ファイル自身を示す)
・第一引数 - local_40hが示すアドレス -> callocで確保した1024バイトの領域のアドレス

続いて逆アセンブルで動作を見ていく  
> s fcn.08048be1
> pdf

```
; CALL XREFS from main @ 0x8048848, 0x80489f7
/ 110: fcn.080486eb (int32_t arg_8h, int32_t arg_ch);
|           ; var int32_t var_14h @ ebp-0x14
|           ; var int32_t var_10h @ ebp-0x10
|           ; var int32_t var_ch @ ebp-0xc
|           ; arg int32_t arg_8h @ ebp+0x8
|           ; arg int32_t arg_ch @ ebp+0xc
|           0x080486eb      55             push ebp
|           0x080486ec      89e5           mov ebp, esp
|           0x080486ee      83ec18         sub esp, 0x18
|           0x080486f1      c745ec010000.  mov dword [var_14h], 1
|           0x080486f8      83ec08         sub esp, 8
|           0x080486fb      68008d0408     push 0x8048d00              ; "rb"
|           0x08048700      ff750c         push dword [arg_ch]
|           0x08048703      e888feffff     call sym.imp.fopen          ; file*fopen(const char *filename, const char *mode)
|           0x08048708      83c410         add esp, 0x10
|           0x0804870b      8945f0         mov dword [var_10h], eax
|           0x0804870e      837df000       cmp dword [var_10h], 0
|       ,=< 0x08048712      7507           jne 0x804871b
|       |   0x08048714      c745ec000000.  mov dword [var_14h], 0
|       `-> 0x0804871b      83ec04         sub esp, 4
|           0x0804871e      6a02           push 2                      ; 2
|           0x08048720      6800fcffff     push 0xfffffc00             ; 4294966272
|           0x08048725      ff75f0         push dword [var_10h]
|           0x08048728      e803feffff     call sym.imp.fseek          ; int fseek(FILE *stream, long offset, int whence)
|           0x0804872d      83c410         add esp, 0x10
|           0x08048730      85c0           test eax, eax
|       ,=< 0x08048732      7407           je 0x804873b
|       |   0x08048734      c745ec000000.  mov dword [var_14h], 0
|       `-> 0x0804873b      83ec04         sub esp, 4
|           0x0804873e      ff75f0         push dword [var_10h]
|           0x08048741      6800040000     push 0x400                  ; 1024
|           0x08048746      ff7508         push dword [arg_8h]
|           0x08048749      e8c2fdffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
|           0x0804874e      83c410         add esp, 0x10
|           0x08048751      8945f4         mov dword [var_ch], eax
|           0x08048754      8b45ec         mov eax, dword [var_14h]
|           0x08048757      c9             leave
\           0x08048758      c3             ret
```
callに注目して、関数の流れを見ていくと、
・fopenで問題ファイルをバイナリで開く
・fseekでファイルポインタを後ろから0xfffffc00バイトの位置に移動する
・fgetsで1024バイト取得

みたいな流れで、暗号化っぽいことは特にしてなさそうに感じる。  

次にfcn.08048be1を細かく見ていく
引数を見ると、3つpushされている
・local_40hに格納されているアドレス -> fcn.08048be1で格納されたアドレス
・local_4hが示すアドレス -> プログラム実行時に指定した第二引数のアドレス
・local_48hが示すアドレス

続いて逆アセンブル
> s fcn.08048be1
> pdf
```
; CALL XREF from main @ 0x80488a7
/ 130: fcn.08048be1 (int32_t arg_8h, int32_t arg_ch, int32_t arg_10h);
|           ; var int32_t var_14h @ ebp-0x14
|           ; var int32_t var_10h @ ebp-0x10
|           ; var int32_t var_ch @ ebp-0xc
|           ; var int32_t var_4h @ ebp-0x4
|           ; arg int32_t arg_8h @ ebp+0x8
|           ; arg int32_t arg_ch @ ebp+0xc
|           ; arg int32_t arg_10h @ ebp+0x10
|           0x08048be1      55             push ebp
|           0x08048be2      89e5           mov ebp, esp
|           0x08048be4      53             push ebx
|           0x08048be5      83ec14         sub esp, 0x14
|           0x08048be8      83ec0c         sub esp, 0xc
|           0x08048beb      ff750c         push dword [arg_ch]
|           0x08048bee      e87df9ffff     call sym.imp.strlen         ; size_t strlen(const char *s)
|           0x08048bf3      83c410         add esp, 0x10
|           0x08048bf6      8945f0         mov dword [var_10h], eax
|           0x08048bf9      8b45f0         mov eax, dword [var_10h]
|           0x08048bfc      83ec0c         sub esp, 0xc
|           0x08048bff      50             push eax
|           0x08048c00      e85bf9ffff     call sym.imp.srand          ; void srand(int seed)
|           0x08048c05      83c410         add esp, 0x10
|           0x08048c08      c745ec000000.  mov dword [var_14h], 0
|       ,=< 0x08048c0f      eb44           jmp 0x8048c55
|      .--> 0x08048c11      e8aaf9ffff     call sym.imp.rand           ; int rand(void)
|      :|   0x08048c16      89c2           mov edx, eax
|      :|   0x08048c18      89d0           mov eax, edx
|      :|   0x08048c1a      c1f81f         sar eax, 0x1f
|      :|   0x08048c1d      c1e816         shr eax, 0x16
|      :|   0x08048c20      01c2           add edx, eax
|      :|   0x08048c22      81e2ff030000   and edx, 0x3ff              ; 1023
|      :|   0x08048c28      29c2           sub edx, eax
|      :|   0x08048c2a      89d0           mov eax, edx
|      :|   0x08048c2c      8945f4         mov dword [var_ch], eax
|      :|   0x08048c2f      8b55ec         mov edx, dword [var_14h]
|      :|   0x08048c32      8b4508         mov eax, dword [arg_8h]
|      :|   0x08048c35      01d0           add eax, edx
|      :|   0x08048c37      8b4dec         mov ecx, dword [var_14h]
|      :|   0x08048c3a      8b550c         mov edx, dword [arg_ch]
|      :|   0x08048c3d      01ca           add edx, ecx
|      :|   0x08048c3f      0fb61a         movzx ebx, byte [edx]
|      :|   0x08048c42      8b4df4         mov ecx, dword [var_ch]
|      :|   0x08048c45      8b5510         mov edx, dword [arg_10h]
|      :|   0x08048c48      01ca           add edx, ecx
|      :|   0x08048c4a      0fb612         movzx edx, byte [edx]
|      :|   0x08048c4d      31da           xor edx, ebx
|      :|   0x08048c4f      8810           mov byte [eax], dl
|      :|   0x08048c51      8345ec01       add dword [var_14h], 1
|      :|   ; CODE XREF from fcn.08048be1 @ 0x8048c0f
|      :`-> 0x08048c55      8b45ec         mov eax, dword [var_14h]
|      :    0x08048c58      3b45f0         cmp eax, dword [var_10h]
|      `==< 0x08048c5b      7cb4           jl 0x8048c11
|           0x08048c5d      90             nop
|           0x08048c5e      8b5dfc         mov ebx, dword [var_4h]
|           0x08048c61      c9             leave
\           0x08048c62      c3             ret
```

callに注目してみていくと、
・strlenで第二引数の文字列の長さを比較
・この返り値をvar_10hに記録
・srandでvar_10hの値を元にseed値を変更
・0x08048c55にジャンプ
・var_14hとvar_10hを比較して、var_14hが小さければ0x8048c11にジャンプ
・randでランダム数値を生成
・生成した値をeaxとedxに入れる
・eaxの値を0x1f算術右シフトする
・先ほど右シフトした値を0x16論理右シフトする
・上でシフトした値とedxを足してedxに代入する
・edxと0x3ffをand演算してedxに代入
・edxからeaxを引いてedxに代入

暗号化の手法が分かったため、これを復号するプログラムを書いてFLAGを生成する

