# Disk, disk, sleuth!
## picoCTF2021
## Forensics
***
始めにあるファイルが渡される。ファイルの種類がわからないため、まずはfileコマンドでファイル種別を調べる。

> file dds1-alpine.flag.img.gz
  
出力がこんな感じ
> dds1-alpine.flag.img.gz: gzip compressed data, was "dds1-alpine.flag.img", last modified: Tue Mar 16 00:19:38 2021, from Unix
  
どうやらgzipファイルみたいだからとりあえず解凍してみる。  
> gzip -d dds1-alpine.flag.img.gz
  
するとdds1-alpine.flag.imgというファイルが出てきたため、今度はこれをfileコマンドにかけてみる。  
> file dds1-alpine.flag.img
  
どうやらディスクイメージファイルっぽい。stringsコマンドでフラグがでないか確かめる。
> strings dds1-alpine.flag.img | grep pico
  
これでフラグが出たため、submitする。