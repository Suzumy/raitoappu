# Simple conversion
## WaniCTF2021 spring
## crypto
***
問題として、pythonのプログラムとoutputというファイルが渡される。
おそらくpythonのプログラムによってフラグが暗号化され、それがoutputの中に書かれているというものだと思う。  
プログラムは以下のようになっている。  
```
from const import flag  
def bytes_to_integer(x: bytes) -> int:  
    x = int.from_bytes(x, byteorder="big")
    return x  
print(bytes_to_integer(flag))
```
int.from_bytesで変換しているっぽい。  
from_bytesはバイト列を整数に変換するもので、例えば  
```
b =  b'\x00\x80'
int.from_bytes(b, 'big')
```
とすると、128が出力される。という風に使う。  
今回、outputの中を見てみると、
> 709088550902439876921359662969011490817828244100611994507393920171782905026859712405088781429996152122943882490614543229  

と書かれている。これがおそらく出力された整数だと考えされるため、これをバイト列に直すために、to_bytesを使う。
to_bytesは  
> 数列.to_bytes(バイト数, 'big or little')
  
みたいな感じで使う。今回バイト数はわからないため、適当に100とか入れて見ると、以下のようなコードになる。
```
flag = 
709088550902439876921359662969011490817828244100611994507393920171782905026859712405088781429996152122943882490614543229
flag = flag.to_bytes(100, 'big')
print(flag)
```
すると出力の最後の方にフラグが出てくるため、これをsubmitする。