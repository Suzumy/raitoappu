﻿# c0rrupt
## picoCTF2021
## forensics
***

始めに謎のファイルが渡される。ファイルの種類がわからないため、Bzに渡し、バイナリの最後の文字列を検索すると、
どうやらPNGファイルであることが分かった。

pngファイルがpngファイルであるためにはいくつか決まったルールがある。  
> ・先頭の8bitは必ず89 50 4e 47 0d 0a 1a 0aになる  
・IHDRのchunk typeが常に49 48 44 52になる  
・IDATのchunk typeが常に49 44 41 52になる  

これらのポイントを修正していくと、画像ファイルが修復されるため、これを開くとフラグが書かれてある。