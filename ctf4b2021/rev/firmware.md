# firmware
## Beginners_CTF_2021
## reversing
***

今回の問題はeasyだからか問題文に誘導がかなり含まれているため、それにそって考える。
問題文
```
問題文
ctf4b networks社のページからファームウェアをダウンロードしてきました。

このファイルの中からパスワードを探してください。

題材とする脆弱性
Ghidraを使ったarmhfバイナリ解析

実現するためのテーマ
IoT機器の疑似ファームウェア解析

想定する参加者が解答までに至る思考経路
armhfのパスワードチェックバイナリと雑多なファイルをまとめてzipしてある疑似ファームウェアが与えられる
binwalkなどでELFを取り出す (取り出しミスを防ぐために含まれているファイルのmd5チェックサムが初めの方にまとめて記載されていいる)
Ghidraで解析する。socketで受け取ったパスワード文字列を1文字ずつ0x53とXORしているので復元するコードを書く
```

というわけでまずは言われた通り与えられたファイルからbinwalkでELFを取り出してみる。
binwalkでファイルの中身を取り出すときには-eをつける。

```
$ binwalk -e firmware.bin

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
127           0x7F            Base64 standard index table
2343          0x927           Copyright string: "Copyright 2011-2021 The Bootstrap Authors"
2388          0x954           Copyright string: "Copyright 2011-2021 Twitter, Inc."
2452          0x994           Unix path: /github.com/twbs/bootstrap/blob/main/LICENSE)
83503         0x1462F         PNG image, 594 x 100, 8-bit grayscale, non-interlaced
83544         0x14658         Zlib compressed data, best compression
90593         0x161E1         ELF, 32-bit LSB shared object, ARM, version 1 (SYSV)
100906        0x18A2A         Unix path: /usr/lib/gcc/arm-linux-gnueabihf/9/../../../arm-linux-gnueabihf/Scrt1.o
103485        0x1943D         JPEG image data, JFIF standard 1.01
117167        0x1C9AF         PEM certificate
117786        0x1CC1A         HTML document header
118641        0x1CF71         HTML document footer
```
lsをすると_firmware.bin.extractedというファイルが新しく生成されていることがわかる。この中に取り出した
データが入っているため、この中にcdで移動する。

> cd _firmware.bin.extracted

中に入ってlsをすると、複数のファイルがあるため、この中からelfファイルを探す。
今回は拡張子がelfのものがあったため、これをまずは調べる。

```
~/_firmware.bin.extracted$ ls
14658  14658.zlib  161E1.elf  1C9AF.crt  1CC1A

~/_firmware.bin.extracted$ file 161E1.elf
161E1.elf: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, BuildID[sha1]=d4fde2a811fccb987ffb2e075b170db18f797b8a, for GNU/Linux 3.2.0, not stripped
```

fileコマンドの結果、32bitのELFファイルであることが分かったため、おそらく問題文に書かれていたELFファイルを取り出す工程は完了したと思われる。
次にこのファイルをghidraで静的解析にかけてみる。

main関数の逆コンパイル結果を見ると、いろいろ書かれているが、注目すべきポイントは問題文に書かれているため、問題文の通り0x53とXORしている行を見つける。
すると、
> if ((uint)(abStack4116[local_11e0] ^ 0x53) != auStack4520[local_11e0]) {

という行が見つかった。0x53とXORしているため、ここがポイントっぽい。
プログラムの意味を考えると、auStack4520配列を一つずつ比較しているっぽいため、auStack4520配列の中身が重要になりそう。
じゃあauStack4520はどこから来ているかというと
> memcpy(auStack4520,&DAT_00010ea4,0xf4);

となっているため、&DAT_00010ea4からコピーされてきた内容であることがわかる。
では&DAT_00010ea4の中身はどうなっているか調べるため、ghidra上で&DAT_00010ea4の上でダブルクリックする。
すると左側に以下の情報が出てきた。長いため0の部分は省略する。
```
                             DAT_00010ea4                                    XREF[1]:     main:00010b9e (*)   
        00010ea4 30              ??         30h    0
        00010ea8 27              ??         27h    '
        00010eac 35              ??         35h    5
        00010eb0 67              ??         67h    g
        00010eb4 31              ??         31h    1
        00010eb8 28              ??         28h    (
        00010ebc 3a              ??         3Ah    :
        00010ec0 63              ??         63h    c
        00010ec4 27              ??         27h    '
        00010ecc 37              ??         37h    7
        00010ed0 36              ??         36h    6
        00010ed4 25              ??         25h    %
        00010ed8 62              ??         62h    b
        00010edc 30              ??         30h    0
        00010ee0 36              ??         36h    6
        00010ee8 35              ??         35h    5
        00010eec 3a              ??         3Ah    :
        00010ef0 21              ??         21h    !
        00010ef4 3e              ??         3Eh    >
        00010ef8 24              ??         24h    $
        00010efc 67              ??         67h    g
        00010f00 21              ??         21h    !
        00010f04 36              ??         36h    6
        00010f0c 32              ??         32h    2
        00010f10 3d              ??         3Dh    =
        00010f14 32              ??         32h    2
        00010f18 62              ??         62h    b
        00010f1c 2a              ??         2Ah    *
        00010f20 20              ??         20h     
        00010f24 3a              ??         3Ah    :
        00010f28 60              ??         60h    `
        00010f30 21              ??         21h    !
        00010f34 36              ??         36h    6
        00010f38 25              ??         25h    %
        00010f3c 60              ??         60h    `
        00010f40 32              ??         32h    2
        00010f44 62              ??         62h    b
        00010f48 20              ??         20h     
        00010f50 32              ??         32h    2
        00010f58 3f              ??         3Fh    ?
        00010f5c 63              ??         63h    c
        00010f60 27              ??         27h    '
        00010f68 3c              ??         3Ch    <
        00010f6c 35              ??         35h    5
        00010f74 66              ??         66h    f
        00010f78 36              ??         36h    6
        00010f7c 30              ??         30h    0
        00010f80 21              ??         21h    !
        00010f84 36              ??         36h    6
        00010f88 64              ??         64h    d
        00010f8c 20              ??         20h     
        00010f90 2e              ??         2Eh    .
        00010f94 59              ??         59h    Y
```

入力値を0x53とXORした結果がこれと合っていればいいというif文だっため、逆にこれをXORしてやれば
元の文が分かることになる。

復号するプログラムを書いていく。
```
list = [0x30,0x27,0x35,0x67,0x31,0x28,0x3a,0x63,0x27,0x0c,0x37,0x36,0x25,0x62,0x30,0x36,0x0c,0x35,
0x3a,0x21,0x3e,0x24,0x67,0x21,0x36,0x0c,0x32,0x3d,0x32,0x62,0x2a,0x20,0x3a,0x60,0x0c,0x21,0x36,0x25,
0x60,0x32,0x62,0x20,0x0c,0x32,0x0c,0x3f,0x63,0x27,0x0c,0x3c,0x35,0x0c,0x66,0x36,0x30,0x21,0x36,0x64,
0x20,0x2e,0x59]

for i in list:
    hoge = i ^ 0x53
    print(chr(hoge), end="")
```

実行するとフラグが出てくる。