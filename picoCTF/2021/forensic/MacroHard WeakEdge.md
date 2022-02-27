# MacroHard WeakEdge
## picoCTF2021
## Forensics
***
問題としてpptmファイルが渡される。pptmはpptxと違いマクロが使える。  
が、マクロを見てみてもno flagと書かれている。どうやらマクロは関係ないらしい。
調べたら、powerpointはzipファイルなどで構成されているらしい。
そのため、とりあえずunzipにかけてみる。  
> unzip unzip Forensics\ is\ fun.pptm
  
すると以下のような出力が出てくる。
> ~前略  
inflating: ppt/slideLayouts/_rels/slideLayout9.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout10.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout11.xml.rels  
  inflating: ppt/theme/theme1.xml  
 extracting: docProps/thumbnail.jpeg  
  inflating: ppt/vbaProject.bin  
  inflating: ppt/presProps.xml  
  inflating: ppt/viewProps.xml  
  inflating: ppt/tableStyles.xml  
  inflating: docProps/core.xml  
  inflating: docProps/app.xml  
  inflating: ppt/slideMasters/hidden
    
最後にhiddenというファイルがある。名前が気になるためファイルを開いてみてみると文字列が書かれてあった。  
base64変換がかけられた文字列っぽいため、空白を取り除き、base64デコードにかけるとフラグが出てくるため、これをsubmitする。